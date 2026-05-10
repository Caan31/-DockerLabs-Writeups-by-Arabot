# PRESSENTER — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PRESSENTER** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Hacemos un escaneo de los puertos que tiene abierto la maquina

Vemos que tiene un servidor http, así que miraremos la pagina a ver que nos
encontramos

Mirando el código vemos que apunta hacia un servidor, así que lo editaremos en
/etc/hosts para poder explorar.

## 2. Enumeración

Ahora podemos hacer un escaneo y vemos que cuenta con wordpress.

Haremos un escaneo primero para probar en encontrar usuarios dentro.

Una vez sepamos los usuarios, haremos un ataque de fuerza bruta para ver si nos
encontramos algo.

Nos encontramos con la contraseña de pressi, así que ingresaremos y veremos
que somos el administrador, así que podemos hacer lo que queramos dentro de
wordpress.

## 3. Explotación

Añadiremos un plugin que nos ayudara a manejar los ficheros de este servidor.

Vemos que tenemos ficheros de lectura y escritura, así que a cualquier php que
cuente con estos permisos lo editaremos para luego poder hacer una reverse
Shell.

Añadimos el siguiente código para poder ejecutar comandos de cmd.

Ahora lo guardamos y podemos ver la información donde esta este documento
ubicado.

## 4. Post-explotación

Hacemos una prueba para ver que si ejecuta comandos de cmd.

Ahora haremos una reverse Shell.

Vemos que estamos dentro del servidor.

Como no encontré nada que pueda comprometer, he utilizado linpeas para
encontrar alguna vulnerabilidad.

## 5. Escalada de privilegios

Lo compartiremos desde nuestro host a la maquina victima y lo ejecutamos.

Vemos que encontró una base de datos con la contraseña del administrador.

Vemos que entramos sin ningún problema, así que exploraremos a ver que nos
encontramos

Explorando encontramos la base de datos, tablas y un usuario y contraseña.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
