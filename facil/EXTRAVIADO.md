# Extraviado — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** Hack_Viper  
**Fecha de creación:** 12/01/2025  
**Técnicas:** Nmap · Base64 en web · SSH · Ficheros ocultos · Base64 encadenada · Acertijo → contraseña root

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Base64 al final de la página](#2-enumeración-web--base64-al-final-de-la-página)
3. [Acceso SSH como daniela](#3-acceso-ssh-como-daniela)
4. [Movimiento lateral — fichero .secreto con contraseña de diego](#4-movimiento-lateral--fichero-secreto-con-contraseña-de-diego)
5. [Escalada a root — acertijo en fichero oculto .-](#5-escalada-a-root--acertijo-en-fichero-oculto--)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

```bash
sudo bash auto_deploy.sh extraviado.tar
```

> IP asignada: `172.17.0.2`.

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN PuertosExtraviado
cat PuertosExtraviado
```

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

---

## 2. Enumeración web — Base64 al final de la página

Visitamos `http://172.17.0.2` — página de inicio Apache por defecto pero con contenido propio. Al llegar al final de la página encontramos dos cadenas codificadas:

```
ZGFuaWVsYQ==  :  Zm9jYXJvamE=
```

Las decodificamos en nuestra máquina:

```bash
echo "ZGFuaWVsYQ==" | base64 -d
daniela

echo "Zm9jYXJvamE=" | base64 -d
focaroja
```

> 💡 Esto es un usuario (**daniela**) y una contraseña (**focaroja**) en Base64 — un encoding, no cifrado. Trivialmente reversible.

---

## 3. Acceso SSH como daniela

```bash
ssh daniela@172.17.0.2
```

```
daniela@172.17.0.2's password: focaroja
daniela@dockerlabs:~$
```

Exploramos el directorio:

```bash
daniela@dockerlabs:~$ ls
Desktop
daniela@dockerlabs:~$ cd Desktop/
daniela@dockerlabs:~/Desktop$ cat nota
Daniela no recuerdo donde guarde la password de root, si la encuentras me dices.
```

Hay tres usuarios en el sistema: daniela, diego, ubuntu.

---

## 4. Movimiento lateral — fichero .secreto con contraseña de diego

Buscamos ficheros ocultos con `ls -la`:

```bash
daniela@dockerlabs:~$ ls -la
drwxrwxr-x 2 daniela daniela 4096 Jan  9 2025 .secreto
```

```bash
daniela@dockerlabs:~$ cd .secreto/
daniela@dockerlabs:~/.secreto$ ls
passdiego
daniela@dockerlabs:~/.secreto$ cat passdiego
YmFsbGVuYW5lZ3JhM0po
```

Decodificamos el Base64:

```bash
echo "YmFsbGVuYW5lZ3JhM0po" | base64 -d
ballenanegra3Jh
```

> 💡 Doble capa: primero Base64 en la web (usuario/contraseña), luego otro Base64 en fichero oculto. El principio es el mismo — Base64 no es seguridad.

Accedemos como diego:

```bash
daniela@dockerlabs:~/.secreto$ su diego
Password: ballenanegra3Jh
diego@dockerlabs:/home/daniela/.secreto$
```

---

## 5. Escalada a root — acertijo en fichero oculto .-

Exploramos el directorio de diego:

```bash
diego@dockerlabs:~$ ls -la
-rw-rw-r-- 1 diego diego  15 Jan  9 2025 pass
drwxrwxr-x 2 diego diego 4096 Jan 11 2025 .passroot
drwxrwxr-x 3 diego diego 4096 Jan  9 2025 .local
```

El fichero `pass` dice:

```bash
diego@dockerlabs:~$ cat pass
donde estara?
```

Exploramos `.passroot` — hay un fichero `.pass`:

```bash
diego@dockerlabs:~/.passroot$ cat .pass
YWNhdGFtcG9jb2VzdGE=
```

```bash
echo "YWNhdGFtcG9jb2VzdGE=" | base64 -d
acatampocoesta
```

Pero esta contraseña no funciona para root. Seguimos buscando. En `.local/share/` encontramos un fichero con nombre raro `.-`:

```bash
diego@dockerlabs:~/.local/share$ cat .-
password de root

En un mundo de hielo, me muevo sin prisa,
con un pelaje que brilla, como la brisa.
No soy un rey, pero en cuentos soy fiel,
de un color inusual, como el cielo y el mar tambien.
Soy amigo de los ni~nos, en historias de ensue~no.
Quien soy, que en el frio encuentro mi due~no?
```

> 💡 El acertijo describe un **oso azul** — animal de pelaje azul, vive en el frío, es de los niños, en cuentos. La respuesta es **osoazul**.

```bash
diego@dockerlabs:~/.local/share$ su root
Password: osoazul
root@dockerlabs:/home/diego/.local/share# whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Dónde | Impacto |
|----------------|-------|---------|
| **Credenciales en Base64 en página web** | `http://172.17.0.2` (footer) | Usuario y contraseña SSH sin protección real |
| **Contraseña de diego en fichero oculto .secreto** | `~/.secreto/passdiego` | Movimiento lateral |
| **Contraseña de root como acertijo en fichero oculto .-** | `~/.local/share/.-` | Escalada a root |

**Para defenderse:**
- Base64 no es cifrado. Cualquier cadena Base64 es inmediatamente decodificable. Nunca usarlo para "proteger" credenciales.
- Los ficheros ocultos (con punto) son visibles con `ls -la` — no son una medida de seguridad.
- Las contraseñas basadas en acertijos o referencias culturales son adivinables con contexto.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
