# BYPASSME — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **BYPASSME** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar nuestra maquina vulnerable.

Hacemos un escaneo profundo de los puertos que están abiertos.

## 2. Enumeración

Podemos ver que tenemos un servidor web en la maquina, vamos a mirarlo y nos
encontramos con un panel de logeo. Vamos a hacer un intento de SQL Inyection,
Admin
admin ‘ OR ‘1’=’1’ -- -

Vemos que accedemos como administrador, ahora si inspeccionamos esta pagina
nos encontramos con un comentario.

## 3. Explotación

Asi que al hacer una búsqueda con dirb encontramos una pagina /logs y podemos
complementar con lo que nos pone de logs.txt y encontraremos vamos usuarios,
uno de ellos Albert y la contraseña.

Vamos a conectarnos por ssh y estamos dentro de este usuario.

## 4. Post-explotación

Vamos a intentar ver si podemos hacer la escalada de privilegios por sudo o SUID,
pero vemos que no va a ser posible.

Ejecutaremos ps aux para ver los procesos que se ejecutan y vemos que hay uno
que lo ejecuta el usuario conx así que vamos a mirarlo.

## 5. Escalada de privilegios

Vemos que contamos con los permisos para poder ejecutar el siguiente comando
que esto nos permite conectarnos por un socket.

Ahora vemos que somos el usuario conx, así que haremos una reverse Shell

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
