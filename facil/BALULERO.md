# BALULERO — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **BALULERO** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Hacemos un escaneo profundo para ver los puertos por donde podemos hacer el
ataque.

## 2. Enumeración

Vemos que cuenta con un servidor http, así que veremos que contiene dentro de
esta pagina.

Vemos que cuenta con un script así que vamos a mirarlo.

## 3. Explotación

Tenemos una pista que dentro de un fichero hay un usuario con su contraseña

Ahora nos conectaremos por ssh

## 4. Post-explotación

Vamos a hacer la escalada de privilegios y vemos que el usuario chocolate cuenta
con permisos sudo de php

Con ayuda de gtfobins vamos a mirar como podemos escalar privilegios

## 5. Escalada de privilegios

Ahora como no vemos nada comprometedor con este usuario, vamos a utilizar la
herramienta pspy64, nos la descargaremos en la maquina vulnerable.

Habilitaremos la ejecución del programa y vamos a ejecutarlo

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
