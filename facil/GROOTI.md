# GROOTI — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **GROOTI** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Vamos a hacer un escaneo profundo de la máquina y vamos a mirar que puertos
están abiertos.

Vamos a ver que la máquina cuenta con un servidor web y ahora vamos a explorar
un poco.

## 2. Enumeración

Encontramos después de buscar un rato una contraseña que nos dice encontrar
donde ponerla, así que seguiremos buscando.

Al utilizar dirb encontramos esta pagina oculta en el servidor con varios usuarios y
un archivo que nos deja descargarlo.

Al final de este archivo encontramos el siguiente comando.

## 3. Explotación

Nos conectamos con la contraseña que encontramos antes y vemos que
funciona, así que vamos a mirar las base de datos que tienen.

Al explorar un poco en la base de datos vemos unas rutas, así que vemos la de
secret.

Encontramos esta interfaz donde nos permite escribir un numero del 1 al 100, por
cada numero nos descarga un fichero, al mirar un poco contábamos en la pagina
principal que el usuario es grooti16, así que vamos a probar con el numero 16.

## 4. Post-explotación

Vemos que nos descarga un zip que contiene passwords.

Vemos que tiene contraseña pero es la misma de antes password1

Ahora crearemos un fichero con los usuarios que encontramos en la pagina web.

## 5. Escalada de privilegios

Al hacer un ataque con hydra, vemos que nos enseña la contraseña.

Nos conectamos por ssh y vamos a ver como podemos escalar privilegios.

Intentamos hacer una escalada por sudo y por SUID pero vemos que no podemos.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
