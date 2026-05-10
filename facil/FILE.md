# FILE — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **FILE** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Haremos un escaneo profundo de los puertos abiertos de la maquina.

Revisamos el servidor web que tiene y vemos que no cuenta con nada, así que
haremos un escaneo con gobuster para encontrar directorios ocultos.

## 2. Enumeración

Encontramos un directorio donde podemos subir ficheros.

Creamos un fichero php para poder ejecutar comandos de cmd y así hacer la
reverse Shell.
Cambiamos la extensión del fichero porque no nos permite subir directamente un
php.

Ahora en el otro directorio que encontramos vemos que esta lo que hemos subido,
haremos los pasos para la reverse Shell.

## 3. Explotación

Una vez conectado vemos los usuarios con los que cuenta esta máquina.

Despues de probar varias cosas, al final vamos a utilizar el programa:
https://github.com/Maalfer/Sudo_BruteForce.git
Lo compartiremos desde nuestra maquina local y luego lo ejecutaremos.

Ahora que somos un usuario vemos que tiene una imagen, nos la pasaremos a
nuestra maquina local para explorarla.

## 4. Post-explotación

Para poder tenerla vamos a modificar la carpeta html para poder tener permisos
de mover ficheros y así descargárnoslo.

Con steghide vamos a ver que cuenta con una contraseña así que utilizaremos
stegcracker para encontrar esta.

Nos da un hash que utilizaremos la web crackstation para descifrarla.

## 5. Escalada de privilegios

Vemos que tenemos eso, así que probaremos con cada usuario a ver cual tiene
esa contraseña

Ahora veremos como ir escalando privilegios con gtfobins

Ahora vemos que es un fichero Python que eliminaremos y crearemos uno nuevo
con código para poder tener acceso a root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
