# FINDYOURSTYLE — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **FINDYOURSTYLE** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Vamos a hacer un escaneo profundo en la maquina para ver los puertos que tiene
abiertos.

Vemos que solo cuenta con un servidor web, así que vamos a explorar la página,
aunque en el escaneo ya nos dice que es un Drupal 8

## 2. Enumeración

De igual manera vamos a ver la versión de drupal para luego con metaesploit
buscar una vulnerabilidad.

Haremos la búsqueda y vemos que hay varias versiones con vulnerabilidades.

Ejecutando el exploit vemos que nos pide el host, así que lo escribiremos y luego
ejecutaremos.

## 3. Explotación

Una vez dentro haremos una reverse Shell para que sea más cómodo movernos.

Lo primero que haremos será ver los usuarios, intentamos varias formas de
escalar privilegios pero no encontramos ninguna forma, así que esta vez
utilizaremos linpeas.

Desde nuestro host vamos a buscar donde lo tenemos instalado y lo
compartiremos por un servidor con Python.

## 4. Post-explotación

Una vez descargado en la maquina víctima, vamos a darle permisos de ejecución y
lo ejecutaremos.

Después de que termine buscando hemos encontrado la contraseña que
seguramente sea del usuario ballenita.

Nos logeamos con ese usuario y vemos que estamos dentro.

## 5. Escalada de privilegios

Ahora haremos la escalada de privilegios de sudo.

Vemos que tenemos permisos con ls y grep así que listaremos a ver si
encontramos algo en el directorio de root. Vemos que si así que ahora lo
miraremos.

Con ayuda de gtfobins vamos a ver como podemos listar esto.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
