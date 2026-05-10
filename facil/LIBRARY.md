# LIBRARY — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **LIBRARY** de DockerLabs.

---

## 1. Reconocimiento

Lo primero que haremos será desplegar nuestra máquina.

Haremos un escaneo de puertos abiertos con nmap y el parámetro -Pn por si el
servidor no permite conexiones ping.

## 2. Enumeración

Ahora sabiendo los puertos que tenemos abiertos podemos hacer un escaneo
mas profundo para saber las versiones.

Vamos a mirar que nos encontramos en el servidor http.

## 3. Explotación

Con la herramienta gobuster vamos a utilizar para descubrir recursos ocultos en
un servidor web o en otros servicios de la máquina.

¿Podemos ver que contiene un index.php con un texto que podría ser una
contraseña?

## 4. Post-explotación

Vamos a utilizar la herramienta hydra para encontrar algún usuario que contenga
esta contraseña.

Ahora podemos registrarnos con el usuario que hemos encontrado por ssh.

## 5. Escalada de privilegios

Una vez nos hemos logeado haremos un sudo -l para ver si contamos con
privilegios para hacer un escalado.

Vemos que tenemos privilegios para ejecutar un script de Python como sudo, así
que la opción que tome es eliminar ese script y crear otro con permisos de
ejecución y así poder ejecutarlo.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
