# OBSESSION — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **OBSESSION** de DockerLabs.

---

## 1. Reconocimiento

Primero vamos a desplegar la maquina víctima.

Haremos un ping para comprobar que tenemos comunicación con la maquina
víctima.

## 2. Enumeración

Haremos un escaneo con nmap y con -Pn que desactiva la detección de hosts
activos (ping), tratando todos los objetivos como si estuvieran en línea, útil cuando
los pings están bloqueados por firewalls.

Ahora que sabemos los puertos abiertos, vamos a hacer un escaneo más
profundo y miraremos con que versión están los servicios.

## 3. Explotación

Por intentar vamos a meternos con el usuario Anonymous al servicio ftp y vemos
que podemos ingresar.

Vamos a mirar que contiene el servicio ftp y podemos ver que cuenta con dos
archivos .txt, los importaremos a nuestra maquina host y miraremos que
contienen.

## 4. Post-explotación

Por los mensajes podemos deducir que el usuario russoski existe.

Ahora inspeccionando la pagina web por si encontramos algo más, encontramos
este comentario.

## 5. Escalada de privilegios

Haremos un ataque con hydra a este usuario para encontrar la contraseña.

Al encontrar la contraseña podemos entrar por ssh.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
