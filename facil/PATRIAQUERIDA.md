# PATRIAQUERIDA — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PATRIAQUERIDA** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos de esta maquina.

## 2. Enumeración

Ahora al ver que tenemos los puertos ssh y http abiertos, vamos a utilizar gobuster
para listar directorios.

Nos encontramos este index.php y nos indica que tenemos que mirar el archivo
oculto de un directorio.

## 3. Explotación

Lo miramos y vemos que es un path tranversal y podemos ver ficheros.

Ahora listamos el /etc/passwd y vemos los usuarios con los que cuenta.

## 4. Post-explotación

Ahora con la contraseña que encontramos antes, vamos a intentar conectarnos
con uno de los dos usuarios.

Vemos que tiene una nota y esta la contraseña de Mario.

## 5. Escalada de privilegios

Ahora como Mario haciendo varias pruebas la única forma de escalar a root es por
un binario de Python

Con ayuda de gtfobins vemos que comando tenemos que ejecutar.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
