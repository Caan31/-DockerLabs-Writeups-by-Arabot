# INTERNSHIP — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **INTERNSHIP** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar el laboratorio.

Haremos un escaneo profundo de los puertos del laboratorio, y luego miraremos
el archivo.

Vamos a explorar el servidor http que tiene el laboratorio.

Vamos a ver que la pagina dirige a //gatekeeperhr.com

## 2. Enumeración

Vamos a editar nuestro fichero de /etc/hosts para apuntar a esta dirección cuando
escribamos en nuestro navegador.

Ahora podemos ver que los botones de la pagina web son interactivos, vamos a
intentar hacer un SQL inyection y vemos que el resultado es positivo.

Al ingresar, podemos ver una cantidad de nombres, vamos a crear un fichero txt
donde estén todos estos nombres.

Ahora con la herramienta dirb que es similar a gobuster, para hacer un ataque de
fuerza bruta de directorios y archivos web.

## 3. Explotación

Vemos que contamos con un directorio llamado /spam, así que lo miraremos.
Al inspeccionar la página, ya que está en negro podremos ver que hay un
comentario.

Después de una búsqueda sobre este texto, vemos que nos da una contraseña.

Haremos un ataque de fuerza bruta con la lista de usuarios que hicimos y con la
contraseña que hemos conseguido.

Vemos que corresponde al usuario pedro, así que vamos a conectarnos por ssh.

## 4. Post-explotación

Hemos encontrado una flag dentro del usuario.
Con el comando ps aux para ver procesos que esta corriendo en el sistema.
Podemos ver el usuario valentina hace un proceso cada 30 segundos que
podemos editar.

Vamos a preparar una reverse Shell.

Ahora desde nuestra maquina vamos a ponernos en escucha y así poder tener
acceso.

Ahora haciendo igual una búsqueda, encontramos una flag y una imagen.

## 5. Escalada de privilegios

Vamos a copiarnos la imagen en nuestro sistema para poder mirarla.

Vemos que nos encontramos con esto, con la herramienta steghide para extraer
información dentro de una imagen.

Vemos que tenemos un fichero llamado secret.txt, si lo miramos posiblemente
sea una contraseña.

Hacemos el intento con el usuario de valentina y vemos que tenemos éxito en la
conexión.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
