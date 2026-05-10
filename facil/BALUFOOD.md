# BALUFOOD — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **BALUFOOD** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la máquina.

Ahora vamos a hacer un escaneo profundo de los puertos abiertos.

## 2. Enumeración

Vemos que cuenta con un puerto 5000, hacemos un escaneo especifico a ese
puerto para ver que contiene.

Vemos que es un servidor http, así que vamos a explorar un poco a ver que nos
encontramos.

## 3. Explotación

Vemos que hay un botón que pone admin.

Hay un panel para iniciar sesión, como tenemos login admin por predeterminado
vamos a intentar con la contraseña admin por probar.

## 4. Post-explotación

Ahora dentro nos encontramos con un panel, si inspeccionamos la pagina
encontraremos un comentario donde nos pone un usuario y su contraseña que
intentaremos conectar por ssh.

Ahora vamos a ver un poco dentro de este usuario a ver que encontramos.

## 5. Escalada de privilegios

Encontramos un script que vamos a ver que contiene, vemos una contraseña, así
que vamos a ver que usuarios tiene este servidor y ver si pertenece alguien.

Nos conectamos y vemos que pertenece a ese usuario, nuevamente vamos a
explorar un poco para ver si encontramos algo.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
