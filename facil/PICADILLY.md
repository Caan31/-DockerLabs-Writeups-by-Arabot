# PICADILLY — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PICADILLY** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Escanearemos los puertos de esta maquina a ver como vulnerar esta maquina.

## 2. Enumeración

Vemos que tiene un servicio http y https, así que vamos a hacer un escaneo de
ambos con gobuster.

Exploramos el primer directorio oculto y vemos un usuario y con una contraseña
para descifrar.

## 3. Explotación

Explorando el servicio https, vemos que podemos subir ficheros, así que
subiremos un fichero php por donde haremos una reverse Shell.

Vemos que todos los ficheros que subimos los almacena en un directorio que
encontramos con gobuster.

## 4. Post-explotación

Comprobamos que funciona el script que subimos

Hacemos la reverse Shell y vemos que estamos conectados, una vez conectados
vemos que un usuario es mateo y la pista la tenemos en el servicio http.

## 5. Escalada de privilegios

Investigando un poco sabemos que se trata de un cifrado cesar, así que
buscaremos de que se trata.

La contraseña correcta era easycrazy, nos logeamos como mateo y vemos como
podemos escalar privilegios.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
