# JENHACK — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **JENHACK** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Vamos a hacer el escaneo profundo de la maquina vulnerable y vemos que cuenta
con un servidor web.

Exploramos los servidores web con los que cuenta y vemos que tiene un login.

## 2. Enumeración

Explorando un poco el código encontramos un posible usuario y contraseña del
login.

Nos registramos y vemos que tenemos un panel de control.
Explorando un poco vemos que contamos con una consola que ejecuta código.

Nos ponemos en escucha para hacer una reverse Shell

## 3. Explotación

Probando un poco vemos que con java, nos permite conectarnos.

Comprobamos que estamos dentro.

Ahora vemos que cuenta con un usuario mas que es jenkhack

## 4. Post-explotación

Ya que no tenemos permisos para ejecutar sudo -l ni encontramos ningún binario
que podamos ejecutar, vamos a buscar algún fichero con el nombre del usuario
con el que cuenta el servidor.

Vemos que tenemos un txt y tenemos un cifrado.

Utilizaremos la herramienta https://gchq.github.io/CyberChef/

## 5. Escalada de privilegios

Ahora nos logeamos y vemos que podemos ejecutar como sudo un bash.

Vemos que este bash lo que corre es un bash.sh que se encuentra en /opt, así que
vamos a ver los permisos que tenemos y así poder eliminarlo o escribir sobre el.

Eliminamos el script

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
