# UPLOAD — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **UPLOAD** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina

Haremos un escaneo simple con nmap y el parámetro -Pn por si el servidor no
permite conexiones ping

## 2. Enumeración

Ahora que sabemos que esta el puerto 80 podemos hacer un escaneo mas
profundo para saber la versión con el parámetro -sCV

Vamos a explorar con lo que contamos en el servidor apache

## 3. Explotación

Vemos que podemos subir ficheros

Ahora vamos a hacer una búsqueda dentro de los directorios del servidor con la
herramienta gobuster.

## 4. Post-explotación

Vemos que tiene un directorio que se llama uploads y que almacena todos los
archivos que subimos

Vamos a subir un archivo .php malicioso para poder ejecutar comandos como
cmd desde la url

## 5. Escalada de privilegios

Vemos que funciona al subirlo y ejecutarlo desde la url ?cmd=id

Lo que haremos ahora es una reverse Shell, así que nos pondremos en escucha en
nuestra maquina por el puerto 443

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
