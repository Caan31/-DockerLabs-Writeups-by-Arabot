# FORBIDDENHACK — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **FORBIDDENHACK** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Hacemos un escaneo profundo de los puertos de la maquina y vemos que cuenta
con un servicio http.

Al verlo podemos ver que nos indica una dirección bypass403.pw así que lo
editaremos en /etc/hosts

Ahora vemos que no tenemos permisos, así que utilizaremos burpsite

## 2. Enumeración

Ahora ponemos como referencia la pagina y como vimos en la pagina principal
tenemos un index.php. vemos que hemos accedido al contenido de la máquina.

Utilizaremos la herramienta ffuz para fuzear esta pagina y ver si encontramos
algun parámetro que este leyendo el archivo.

Encontramos el parámetro pages.

Ahora utilizaremos una herramienta LFI, para poder añadir y ejecutar comandos
como una consola.

## 3. Explotación

Al darnos todas estas líneas de comando, vamos a escribirlo en el siguiente orden
para comprobar que funciona correctamente.

Vemos que nos da el usuario

Ahora preparamos para hacer una reverse Shell.

Este será el comando para poder realizar correctamente la reverse Shell.

## 4. Post-explotación

Una vez dentro investigando un poco encontramos la contraseña del otro usuario.

Al ver que esta cifrado, vemos su contenido y tenemos la contraseña

Ahora vemos que tenemos permisos de sudo para ejecutar un programa.

Utilizaremos strings para poder visualizarlo correctamente.

## 5. Escalada de privilegios

Vemos que da un error al faltar un argumento de una carpeta después de -r

Buscamos si tenemos algún fichero con el nombre de furb.

Lo listamos de la forma en la que nos indican que funciona el script

Al ver la pista, vemos si esta en el directorio de root y así es, tenemos la
contraseña y ahora somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
