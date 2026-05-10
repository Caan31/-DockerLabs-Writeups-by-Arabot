# MIRAME — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **MIRAME** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar el laboratorio.

Haremos un escaneo profundo de este laboratorio y miraremos los puertos
abiertos.

## 2. Enumeración

Vemos que el servidor web cuenta con un login, vamos a interceptar las peticiones
con burp suite.

Una vez guardamos la petición, ahora utilizaremos sqlmap para ver que podemos
encontrar en la base de datos.

## 3. Explotación

Despues de probar con todos los usuarios y su contraseña y no encontrar nada
interesante, probe en buscar con el nombre directoriotravieso.

Vemos que tiene una imagen que guardaremos para ver que contiene.

## 4. Post-explotación

Vemos que cuenta con una contraseña así que buscaremos la contraseña con un
ataque de fuerza bruta utilizando stegcracker.

Vemos que nos genera un archivo .zip

## 5. Escalada de privilegios

También cuenta con una contraseña y no es la misma.

Después de utilizar zip7john y john podemos ver un usuario y contraseña.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
