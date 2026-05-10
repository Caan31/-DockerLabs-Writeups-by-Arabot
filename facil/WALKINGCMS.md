# WALKINGCMS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **WALKINGCMS** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar el laboratorio

Ahora haremos un escaneo profundo para ver los puertos abiertos del laboratorio.

## 2. Enumeración

Exploramos el servidor web con el que cuentan y vemos que no hay nada.

Haremos un escaneo rápido con dirb y veremos que cuenta con una pagina de
Wordpress

## 3. Explotación

Asi que usaremos la herramienta wpscan que es para escanear vulnerabilidades
en wordpress, enumeraremos para ver si encuentra usuarios, plugins y temas
vulnerables.

Vemos que nos encontró un usuario llamado Mario.

## 4. Post-explotación

Ahora con esa información vamos a hacer otro escaneo para ver si encontramos la
contraseña.

Al encontrarla, ingresaremos al modo administrador.

## 5. Escalada de privilegios

Editaremos el fichero index.php para poder escribir el código que queramos,
haremos una reverse Shell.

Ahora guardamos y vemos que podemos ejecutar comandos como en una
consola.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
