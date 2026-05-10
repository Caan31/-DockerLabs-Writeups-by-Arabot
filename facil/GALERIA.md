# GALERIA — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **GALERIA** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar el laboratorio.

Haremos un escaneo profundo a ver que servicios cuenta este servidor.

Vemos que el servidor web cuenta con una galería de fotos, examinamos la pagina
y no encontramos nada raro.

## 2. Enumeración

Vamos a utilizar dirb para hacer un escaneo de directorios escondidos, vemos que
tenemos uno, así que lo exploraremos.

Vemos que podemos subir archivos.

Subiremos un script php para poder hacer una reverse Shell desde la página.

## 3. Explotación

Vemos que si lo ejecutamos podemos ejecutar comandos como con una terminal.

Ahora nos pondremos en escucha desde nuestro host para hacer la reverse Shell.

Vemos que estamos dentro.

## 4. Post-explotación

Ejecutamos sudo -l para ver que usuario tiene permisos de sudo, vemos que para
escalar podemos utilizar el usuario gallery.

Una vez ejecutado podemos ver que somos el usuario gallery

Vemos los permisos que contamos como sudo y vemos que tenemos al parecer
como ejecutar un archivo.

## 5. Escalada de privilegios

Con la herramienta strings que sirve para extraer e imprimir todas las cadenas de
texto legibles que se encuentren en binario o de otro tipo de datos para ver que
ejecuta ese fichero.

Ahora lo ejecutamos como sudo a ver qué hace

Vemos que busca un fichero convert que no lo llega a encontrar para después
ejecutarlo, así que podemos aprovechar esto para crear un nuevo fichero y hacer
la escalada de privilegios hasta root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
