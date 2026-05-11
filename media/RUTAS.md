# RUTAS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **RUTAS** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Ahora haremos un escaneo profundo de la maquina para ver sus puertos abiertos.

Vemos que tenemos dos documentos que podemos ver con el usuario
Anonymous de ftp

Con zip2john vamos a sacar un hash de este fichero para poder encontrar la
contraseña posteriormente con john.

Ahora vemos el contenido que tenia, nos da una pista que consigamos la imagen y
un enlace.

Mirando el repositorio, donde tiene almacenadas las imágenes, simplemente
pegamos la dirección y tenemos la imagen que buscábamos.

## 2. Enumeración

Ahora con steghide, veremos unos datos que se extrajeron, donde tenemos un
usuario y contraseña.

Con gobuster, haremos una búsqueda de paginas ocultas, una de ellas es un
index.php.

Esta es la pagina.

Si miramos el código, lo único que no llega a encajar es este posible dominio, al
que pondremos en nuestra carpeta de /etc/hosts

Nos pide una contraseña, es la que habíamos descubierto anteriormente.

Con burpduite, vamos a obtener el siguiente código para poder de nuevo realizar
una búsqueda con gobuster.

## 3. Explotación

Al no ver ningún resultado, lo que haremos sera utilizar wfuzz para ver si
encontramos algo.

Lo miramos en nuestro navegador y vemos que con LFI no es posible, así que
vamos a pasar a ejecutar un RFI.

Vamos a meter este código en la pagina web.

Al abrir un servidor http con Python, escribimos en la url, lo siguiente y veremos
todos los ficheros que tenemos en nuestro host.

Hacemos una prueba para ver si funciona el script que metimos.

Ahora haremos una reverse Shell.

## 4. Post-explotación

Ahora que estamos dentro, ejecutamos sudo -l y vemos que tenemos permisos
con el usuario Norberto.

Lo ejecutamos y vemos que ejecuta un fichero llamado head.

Asi que crearemos un fichero en /tmp por la pista que nos dieron anteriormente y
escribiremos el siguiente código, le daremos permiso de ejecución y veremos que
volviendo a ejecutar el script, ahora somos Norberto.

Miramos el directorio y vemos que hay una carpeta oculta con las credenciales.

Con ayuda lo desciframos y tenemos la contraseña de este usuario.

Al conectarnos con ssh vemos que nos da una bash siendo el usuario maría.

## 5. Escalada de privilegios

Ahora explorando su carpeta, también encontramos su contraseña.

Al no encontrar nada interesante, vamos a psarnos linpeas y ver que nos
encontramos.

Vemos que tenemos el siguiente fichero.

Lo miramos y vemos el script.

Vemos que tenemos permisos de escritura así que vamos a editarlo.

Añadiremos chmod u+s para cuando escribamos bash -p, seamos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
