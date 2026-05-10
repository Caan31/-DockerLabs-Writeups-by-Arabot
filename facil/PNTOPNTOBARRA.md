# PNTOPNTOBARRA — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PNTOPNTOBARRA** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

## 2. Enumeración

Haremos un escaneo profundo para ver que puertos tiene abierto, tiene un
servidor http y de conexión ssh.

## 3. Explotación

Exploramos el código de la página y vemos que tiene una dirección que vamos a
buscar.

## 4. Post-explotación

Vemos que es vulnerable a un ataque LFI, y vamos a explorar los ficheros de
/etc/passwd

## 5. Escalada de privilegios

Vemos que tiene el usuario nico, así que vamos a ver su id_rsa y vamos a copiar en
un fichero para luego ingresar con él.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
