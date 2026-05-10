# DOCKERLABS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **DOCKERLABS** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Haremos un escaneo profundo de los puertos de la maquina vulnerable.

Vemos que solo tenemos el puerto de http abierto, así que mientras exploramos la
pagina principal, ejecutamos gobuster para encontrar directorios dentro del
servidor.

## 2. Enumeración

Nos encontramos con una pagina donde podemos subir archivos.

Vamos a crear un script con php para poder hacer una reverse Shell.

Al intentarlo subir, vemos que no nos permite por no ser un fichero .zip

## 3. Explotación

Le cambiaremos .php por .phar que es el equivalente a .zip de php.

Ahora al subirlo vemos que se subió correctamente.

Aquí encontramos el script que subimos.

## 4. Post-explotación

Hacemos una pequeña prueba que si funciona correctamente para hacer la
reverse Shell.

Ejecutamos una reverse Shell

Vemos que nos conectamos correctamente, vamos a ver que podemos ejecutar
cut y grep como sudo.

## 5. Escalada de privilegios

Despues de buscar un rato, en opt, tenemos un fichero donde nos indica la ruta de
la clave de root.

Con ayuda de gtfobins vemos como podemos ejecutar para escalar privilegios.

Ejecutamos los comandos y vemos que somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
