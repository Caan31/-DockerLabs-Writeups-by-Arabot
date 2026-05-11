# SITES — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **SITES** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos del servidor.

## 2. Enumeración

Vemos que tiene el servicio http, así que exploraremos la página.
Tenemos que quedarnos con sites-available y sitio.conf

Al no encontrar nada más en la pagina, utilizaremos gobuster para buscar
directorios.

## 3. Explotación

Encontraremos un fichero vulnerable.php

Haremos un fuzzeo con wfuzz, así que primero lo haremos básico para mirar y
luego aplicar filtros.

## 4. Post-explotación

Una vez lo tenemos, aplicamos los filtros.

Encontramos la pagina y vemos que encontramos un usuario llamado chocolate.

## 5. Escalada de privilegios

Exploramos la ruta que antes nos indicaba la página índex.

Ahora vemos que nos indica el directorio donde encontramos la contraseña del
usuario.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
