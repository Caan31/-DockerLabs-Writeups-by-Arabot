# ALLIEN — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **ALLIEN** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar el laboratorio

Haremos un escaneo simple con nmap y el parámetro -Pn por si el laboratorio no
permite las conexiones ping,

Ahora que sabemos los puertos que están abiertos, vamos a buscar con que
versión cuentan.

Vamos a ver con que nos encontramos en el servidor web.

Haremos una búsqueda de directorios web con gobuster a ver si nos encontramos
con algo interesante.

## 2. Enumeración

Al no encontrar nada vulnerable vamos a centrarnos en el servicio samba.
Vamos a utilizar la herramienta enum4linux que es para enumerar información a
través del protocolo SMB

Podemos ver que encontramos un usuario así que ahora con un ataque de fuerza
bruta con la herramienta crackmapexec que nos servirá para detectar el usuario
con contraseñas válidas

Al encontrar el usuario y la contraseña, ahora vamos a investigar un poco con
smbmap para ver a que tenemos permisos

Al hacer una búsqueda podemos encontrar unas credenciales dentro de la
carpeta backup24.

Nos copiaremos estas credenciales a nuestra maquina para explorarlas.

## 3. Explotación

Podemos encontrar las credenciales de administrador que anteriormente vimos
que contamos con ese usuario.

Accedemos y vamos a investigar a que tenemos permisos.

Podemos ver que contamos con permisos de leer y escribir en una carpeta.

Si nos fijamos, esta es la carpeta donde se encuentran todos los ficheros del
servidor web.

Vamos a meter un archivo php malicioso para poder ejecutarlo desde un
navegador, el punto es poder hacer una reverse Shell.

## 4. Post-explotación

Una vez dentro vamos a escribir el archivo php que hicimos

Ahora si nos vamos a un navegador podemos ver que podemos ejecutar
comandos como una consola

Vamos a ponernos en escucha por el puerto 443

Desde https://www.revshells.com/ vamos a hacernos una reverse Shell y así
podernos conectar con una interfaz.

Ahora vemos que contamos con la terminal.

## 5. Escalada de privilegios

Para que sea más cómodo navegar y ejecutar mejor atajos haremos un
tratamiento TTY.

Ahora podemos ver que somos www-data.

Haremos un sudo -l para ver si podemos de alguna manera hacer una escalada de
privilegios.

Desde https://gtfobins.github.io/ buscaremos si contamos con algún comando
para poder escalar privilegios.

Lo ejecutamos y podemos ver que ahora somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
