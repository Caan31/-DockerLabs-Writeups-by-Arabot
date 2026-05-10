# ELEVATOR — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **ELEVATOR** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Vamos a hacer un escaneo profundo de los puertos de la maquina vulnerable

## 2. Enumeración

Ahora explorando un poco el servicio http que tiene, no encontramos nada
comprometedor.

Haremos un escaneo de directorios con gobuser, al hacer el primer escaneo
notamos algo raro de la carpeta /themes, así que haremos un escaneo

## 3. Explotación

Encontramos un fichero archivo.html

Cuando lo visitamos vemos que tiene la posibilidad de subir ficheros .jpg

## 4. Post-explotación

Creamos un fichero para hacer una reverse Shell

Lo intentamos subir y vemos que no es posible, que solo nos deja .jpg

## 5. Escalada de privilegios

Vamos a cambiarle el nombre a este fichero

Lo subimos y vemos que ahora si lo sube correctamente.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
