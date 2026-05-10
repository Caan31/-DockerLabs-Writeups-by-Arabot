# WHOAMI — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **WHOAMI** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Vamos a hacer un escaneo profundo de los puertos de la máquina.

Al ver que tenemos un servidor web, vamos a utilizar dirb para listar directorios
ocultos de la pagina web.

## 2. Enumeración

Vemos que tiene una pagina con wordpress

Y hay un directorio donde podemos ver un archivo para descargar, así que lo
miraremos.

Vamos a ver que encontramos en el archivo un usuario y contraseña que lo
colocaremos en el login de wordpress.

## 3. Explotación

Una vez dentro, vamos a ver los plugins que tiene para buscar alguna
vulnerabilidad.

Buscando encontramos este repositorio de github

Lo exploramos un poco para poder ver como funciona esta herramienta.

## 4. Post-explotación

Vemos que nos integra una Shell, la exploramos y vemos lo que hace.

Tenemos una Shell interactiva, para poder trabajar correctamente haremos una
reverse Shell.

Vemos que tenemos un problema y que hace como puente esta ip, así que la
reverse Shell probaremos con nuestra ip y si no con la que hace de puente.

## 5. Escalada de privilegios

La ejecutamos y vemos que temeos acceso, vamos a ver como hacer la escalada
de privilegios.

Ahora vamos a ver que con el usuario rafa podemos escalar y nos ayudaremos de
gtfobins.

Ahora con el usuario ruben también haremos lo mismo.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
