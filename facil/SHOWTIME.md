# SHOWTIME — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **SHOWTIME** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina

Haremos un escaneo profundo en esta maquina y vemos que cuenta con un
servidor http.

Vamos a explorar a ver que tiene esta pagina

Explorando un poco vemos que cuenta con un login, donde intentamos hacer sql
inyection y no encontramos resultados.

En este caso utilizaremos la herramienta burp suite, como es la primera vez que la
utilizo, estos son los pasos para configurarla.
Dentro en el apartado de proxy, vamos a mirar que esta corriendo como localhost
en el puerto correspondiente.

## 2. Enumeración

Ahora tendremos que activar la opcion de interceptar.

Para que funcione correctamente instalamos una extensión, se llama Foxy Proxy y
añadiremos un proxie donde contecte este localhost y el puerto

Ahora antes de ingresar cualquier usuario, tenemos que activar la extensión.

Vemos que intercepta la petición correctamente

Vamos a guardar esto para ahora utilizar sqlmap

## 3. Explotación

Importante: apagar la intercepción para continuar con todo.

Vamos a usar sqlmap para hacer un escaneo de las bases de datos que existen
-r: lee el fichero que contiene la petición http.
--batch: todas las preguntas que hace, las pone como predeterminado

Vemos que tiene una base de datos users

Vemos que tiene una tabla de usuarios

Ahora –dump para extraer los datos de una tabla, una vez identificados.

## 4. Post-explotación

Vemos que tenemos 3 usuarios y contraseñas, así que probamos y vemos que con
el usuario joe tenemos acceso a un panel que ejecuta código python

Haremos una reverse Shell para poder tener conexión.

Ejecutamos el código Python para poder tener conexión.

Una vez dentro en directorio tmp vamos a revisar que contamos con un txt

Al parecer tenemos un listado de trucos, nos lo guardaremos para mas adelante.

## 5. Escalada de privilegios

Vemos los usuarios con los que cuenta la maquina

Vamos a guardarnos las posibles contraseñas y convertir todos los caracteres en
minúsculas.

Haremos un ataque con hydra con los usuarios y posibles contraseñas, para eso
volcamos todo dentro de unos txt

Nos conectamos con el usuario joe

Vemos que el usuario Luciano cuenta con permisos sudo en el binario posh

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
