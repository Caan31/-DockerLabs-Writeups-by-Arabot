# NODECLIMB — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **NODECLIMB** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la máquina.

Vamos a asegurarnos que tenemos conexión a la maquina con:
ping -c 4 172.17.0.2

## 2. Enumeración

Ahora vamos a hacer un escaneo sencillo con nmap y el parámetro -Pn por si el
servidor no permite las conexiones ping

Al saber los puertos que están abiertos, ahora podremos hacer un escaneo más
profundo con -p21,22 y buscando la versión de los servicios con -Scv
Podemos ver ejecutando esto que el usuario Anonymous está habilitado por ftp y
contamos con un archivo .zip.

## 3. Explotación

Nos registramos y como habíamos visto contamos con un fichero .zip, lo
pasaremos a nuestro host para ver que podemos hacer con él.

Lo intentamos extraer con unzip y vemos que nos pide una contraseña.

## 4. Post-explotación

Con el comando zip2john que se utiliza para convertir archivos ZIP protegidos por
contraseña en hashes que pueden ser procesados por herramientas de cracking
de contraseñas lo guardaremos como hash.

Ahora que tenemos el hash vamos a utilizar la herramienta john y el rockyou para
descifrar la contraseña.

## 5. Escalada de privilegios

Al ya contar con la contraseña podremos ingresar y ver que contamos con un
fichero .txt, miraremos que tiene con cat y vemos que es el usuario y la
contraseña.

Nos conectaremos por ssh con la contraseña encontrada.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
