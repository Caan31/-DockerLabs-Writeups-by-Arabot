# WALKING DEAD — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **WALKING DEAD** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Vamos a hacer un escaneo profundo de los puertos de la máquina.

## 2. Enumeración

Al ver el puerto de http abierto, utilizaremos gobuster para examinar directorios.

Exploramos los directorios y no nos encontramos nada interesante.

## 3. Explotación

Inspeccionando la pagina principal, vemos que tiene una referencia a un fichero
oculto.

Al meternos hacemos una prueba para ver si podemos realizar una reverse Shell.

## 4. Post-explotación

Al ver que se puede ejecutar, pasaremos a hacerlo.

Una vez conectado, después de hacer varias pruebas vemos que la única forma de
escalar privilegios es con el binario de Python

## 5. Escalada de privilegios

Con ayuda de gtfobins, vamos a ver como escalar.

Al escribir los comandos, vemos que ahora somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
