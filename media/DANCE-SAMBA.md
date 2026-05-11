# DANCE SAMBA — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **DANCE SAMBA** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos de esta máquina.

Vemos que cuenta con varios servicios abiertos, lo primero que haremos será ver
el .txt que nos indica el escaneo.
Vemos una posible contraseña de un usuario.

Ahora vamos a hacer una enumeración del servicio smb

## 2. Enumeración

Nos encontramos con varias pistas, los servicios compartidos y un usuario,
macarena

Ahora hacemos una enumeración para ver que permisos tiene esta usuaria en la
carpeta. Vemos que puede leer y escribir.

Listamos y vemos que tiene un .txt así que lo pasaremos a nuestro host y lo
miraremos

Es la flag del usuario.

## 3. Explotación

Ya que tenemos permisos para escribir, vamos añadirle una clave rsa para poder
conectarnos por ssh. Así que crearemos primero el directorio con mkdir.

Ahora desde nuestro host vamos a generar una clave ssh

Una vez lo tengamos, vamos a dirigirnos en donde nos la genero y la copiaremos
en el directorio donde estamos trabajando, para mas comodidad.

Vamos a leer la clave publica y la vamos a copiar con el nombre authorized_heys

## 4. Post-explotación

Ahora vuelta al servicio smb, vamos a subir los dos archivos que hemos hecho

Al listar, se debería de ver así.

Ahora tendríamos que poder ingresar con la clave privada que tenemos.

Explorando un poco encontramos un cifrado.

## 5. Escalada de privilegios

Si lo vemos nos da una contraseña.

Vemos que es la contraseña de macarena, así que ejecutamos sudo -l para ver si
permisos de sudo en algun binario, vemos que en file.

Explorando un poco más encontramos un txt donde solo tiene permisos sudo.

Con ayuda de gtfobins, vemos como podemos aprovechar este binario.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
