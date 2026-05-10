# PARADISE — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PARADISE** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

## 2. Enumeración

Hacemos un escaneo profundo de los puertos de la maquina vulnerable

## 3. Explotación

Visitamos el servidor web con el que cuenta e investigando un poco vamos
encontrarnos varios directorios, en uno de ellos en la de galería de imágenes vi en
el código de la pagina un cifrado.

## 4. Post-explotación

Vamos a descifrarlo y al ser todo junto puede ser la contraseña de un usuario o un
directorio.

## 5. Escalada de privilegios

Al intentar varias cosas, llegamos a ver que es un directorio y cuenta un txt

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
