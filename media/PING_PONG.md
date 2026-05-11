# PING PONG — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PING PONG** de DockerLabs.

---

## 1. Reconocimiento

Lo primero que haremos es desplegar la maquina

Haremos un ping para comprobar que tenemos conexión con la maquina.

Vamos a hacer un escaneo rápido con nmap.

Ahora sabiendo los puertos abiertos haremos un escaneo mas profundo
especificando cada puerto y buscando la versión con la que cuenta cada servicio.

## 2. Enumeración

Vamos a probar viendo que tenemos en servidor web

Haremos un escaneo con gobuster para ver si encontramos algún archivo o
directorio escondido.

Metiéndonos a los directorios que hemos encontrado podemos ver que no hay
nada interesante o que nos pueda servir.

Vamos a ver que nos encontramos por el puerto 443 y vemos que nada.

## 3. Explotación

Por el puerto 5000 contamos con un ping test, así que sabemos que podemos
ejecutar comandos como si fuera una terminal.

Al hacer pruebas podemos ver que es una terminal e incluso funciona con otros
comandos.

Lo que haremos será una reverse Shell para poder conectarnos al servidor,
utilizaremos la herramienta https://www.revshells.com/ , nos conectaremos por el
puerto 443.

Desde nuestra maquina atacante ejecutaremos el siguiente comando para poder
escuchar conexiones entrantes en un puerto específico que será el 443

## 4. Post-explotación

Al ejecutar la reverse Shell podemos ver como ahora estamos dentro del servidor
con el usuario freddy, ahora haremos un escalado de privilegios hasta llegar a root.

Vemos que contamos con permiso a dpkg pero con el usuario Bobby, así que
buscamos en gtfobins a ver como podemos escalar privilegios con este binario.

Ejecutaremos el comando especificando el usuario y la ruta a la que tenemos
permisos.

Ahora podemos ver que somo el usuario Bobby, vamos a ver que podemos
encontrarnos y vemos que ahora el usuario Gladys tiene permisos de sudo para
ejecutar el binario php.

## 5. Escalada de privilegios

Una vez ejecutado todo podemos ver que somos Gladys.

Podemos ver que el usuario chocolatito puede ejecutar el comando cut que es
para listar el contenido y filtrarlo, ya que es un usuario vamos a ver dentro de la
ruta del servidor si nos encontramos con algo.

Vamos a ejecutar los comandos para poder listar este fichero .txt como el usuario
chocolatito.

Al tener la contraseña, vamos a registrarnos con el usuario chocolatito y vamos a
ver que mas nos encontramos.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
