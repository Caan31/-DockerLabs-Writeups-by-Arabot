# CONSOLELOG — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **CONSOLELOG** de DockerLabs.

---

## 1. Reconocimiento

Lo primero que haremos será desplegar la máquina.

Lo primero que haremos será un escaneo sencillo con nmap y el parámetro -Pn
por si el servidor tiene bloqueado las conexiones ping.

## 2. Enumeración

Ahora que sabemos los puertos que están abiertos, vamos a hacer un escaneo
más profundo, con la versión que cuenta y todo cada puerto.

Vamos a ver con que nos encontramos en el servidor http, vemos que no tenemos
nada interesante.

## 3. Explotación

Utilizaremos la herramienta gobuster para buscar algún fichero mas dentro del
servidor http.

Tenemos un directorio que se llama backend, dentro de este contamos con varios
ficheros uno de ellos es server.js, vamos a mirarlo y podemos encontrar una
contraseña que da acceso si el token es correcto, así que haremos un ataque de
fuerza bruta a esta contraseña.

## 4. Post-explotación

Con la herramienta hydra y un diccionario de usuarios comunes, después
podremos ver que encontramos el usuario lovely

Ya que encontramos el usuario vamos a conectarnos mediante ssh que esta por el
puerto 5000

## 5. Escalada de privilegios

Una vez seamos el usuario lovely vamos a ver a que tenemos privilegios para
escalar hasta root, con el comando sudo -l

Ahora buscaremos en la herramienta Gtfobins buscaremos como podemos
escalar mediante el binario nano

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
