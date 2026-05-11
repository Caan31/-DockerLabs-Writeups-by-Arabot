# MEMESPLOIT — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **MEMESPLOIT** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Ahora haremos un escaneo profundo de los puertos abiertos de la máquina.

Al ver que tiene un servicio http, vamos a visitar la pagina que tiene.

## 2. Enumeración

Vemos que seleccionando nos encontramos con palabras que estaban ocultas.

Ahora vamos a hacer un escaneo a smb

Lo que encontramos es que tenemos un directorio compartido llamado
share_memehydra

## 3. Explotación

Y encontramos dos usuarios.

Ahora si nos logeamos con el usuario memehydra, podemos ver que la contraseña
es una de las frases que estaba oculta en la pagina web.
Tiene un fichero zip, así que nos lo llevaremos a nuestra maquina y veremos que
tiene dentro.

Vemos que tiene contraseña, pero la contraseña también es una de las frases que
estaban ocultas en la página web.
Encontramos el usuario y la contraseña.

## 4. Post-explotación

Ahora nos conectamos por ssh a este usuario.

Con sudo -l para ver como podemos escalar privilegios nos encontramos un
script, lo vamos a investigar.

Buscamos por el nombre a ver donde ejecuta este script

## 5. Escalada de privilegios

Vemos que tiene varios ejecutables, el principal es actionban.sh

Miramos y vemos que todos tienen restricciones porque son de root, pero el grupo
del directorio es securuty y si vemos nuestro usuario, también podemos encontrar
que somos del mismo grupo.

Con estos permisos podemos hacer lo siguiente.
Copiaremos los datos en otro formato, luego eliminaremos el script original.
Comprobamos que se eliminó correctamente, luego le cambiamos el nombre al
que hicimos la copia con el nombre original y así podemos tener permisos para
escribir.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
