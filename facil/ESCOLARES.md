# ESCOLARES — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **ESCOLARES** de DockerLabs.

---

## 1. Reconocimiento

Desplegamos la maquina vulnerable.

Vamos a hacer un escaneo profundo en la maquina,

Vamos a explorar la pagina web que tiene el servidor.

## 2. Enumeración

Vemos que hay un comentario con un directorio.

Dentro de este vemos una pista que es el admin de wordpress

Al intentar acceder en wordpress vemos el dns a donde apunta esta ip, así que lo
configuraremos en nuestro host en /etc/hosts

## 3. Explotación

Después de hacer varias pruebas, la opción que use fue utilizar la herramienta
cupp para crear claves con coincidencia del usuario que encontramos y así hacer
el ataque de fuerza bruta.

Una vez generado el fichero .txt vamos a hacer el ataque con wpscan

Al ingresar con las credenciales que encontramos vemos que podemos manejar
los archivos de este.

## 4. Post-explotación

Dentro de un tema vemos que podemos escribir y modificar archivos, así que
creamos un archivo php para luego hacer una reverse Shell.

Probamos que funciona el código php que ingresamos.

Una vez dentro vemos que tenemos un fichero secret.txt y al parecer es la
contraseña de luisillo, un usuario.

## 5. Escalada de privilegios

Ahora ingresaremos como luisillo

Una vez dentro ejecutaremos sudo -l para ver si contamos con permisos de sudo
en algún binario y vemos que tenemos en awk

Con ayuda de gtfobins vamos a ejecutar el comando y vemos que somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
