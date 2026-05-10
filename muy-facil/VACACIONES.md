# VACACIONES — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **VACACIONES** de DockerLabs.

---

## 1. Reconocimiento

Lo primero que haremos será desplegar el contenedor donde está la maquina
vulnerable.

Una vez desplegada, haremos un ping para comprobar que tenemos conexión,
podemos ver por el ttl que es una maquina Linux

Haremos un escaneo simple para comprobar que puertos están abiertos y -Pn
para que no detecte que nuestro host tiene conexión.

## 2. Enumeración

Una vez comprobamos que puertos están abiertos podemos hacer una búsqueda
más detallada especificando los puertos con -p y ver la versión con -sCV.

Vamos a ver que contiene el servidor web y vemos una página en blanco donde
inspeccionaremos el código de esta, podemos ver un comentario de juan para
camilo que ha dejado un correo importante.

Como ya contamos con dos posibles usuarios haremos un ataque de fuerza bruta.
Primero crearemos un archivo .txt donde estén estos dos usuarios.

## 3. Explotación

Haremos el ataque con medusa ya que podremos ver los paquetes uno a uno y si
para alguno de los dos usuarios se resuelve y así intenta con el otro.

Podemos ver que capturo la contraseña de camilo que es password1 y continua
con la búsqueda del usuario juan.

Ahora ingresaremos por ssh con el usuario camilo.

## 4. Post-explotación

Intentamos ver si tenemos algún privilegio con sudo -l y vemos que no

Explorando un poco podremos ver que contamos con 3 usuarios, camilo, juan y
pedro, podríamos intentar hacer un ataque a pedro buscando su contraseña de
igual manera con medusa o con hydra.

Recordemos que a camilo juan le envió un correo así que podríamos revisar el
directorio /var/mail que suele contener los buzones de correo de los usuarios
locales del sistema.
Como podemos ver es el mensaje de juan para camilo con su contraseña.

## 5. Escalada de privilegios

Ahora con las credenciales de juan podremos acceder por ssh.
Podemos comprobar de nuevo con sudo -l si tenemos privilegios y poder así
escalar hasta ser usuario root

Buscaremos en gtfobins si podemos con este binario escalar privilegios

Ejecutamos el comando y podemos ver que ahora somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
