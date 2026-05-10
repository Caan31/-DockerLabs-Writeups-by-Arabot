# TPROOT — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **TPROOT** de DockerLabs.

---

## 1. Reconocimiento

MAQUINA TPROOT

Vamos a desplegar la maquina Tproot

Haremos un ping para ver que tenemos conexión con el servidor

## 2. Enumeración

Hacemos un escaneo rápido con ( nmap ) y ( -Pn ) para evitar determinar si el host
este activo antes de escanearlo

Ahora sabiendo los puertos que están abiertos vamos a buscar específicamente
esos dos puertos con ( -p ) y con ( -sCV ) veremos la versión más específica de
cada servicio.

Como primer intento intentaremos conectarnos por ftp con el usuario Anonymous
que de forma predeterminada no cuenta con contraseña.

## 3. Explotación

Al ver que no es posible vamos a mirar que pagina web nos encontramos.

Podríamos hacer una búsqueda con gobuster para ver si encontramos otros
directorios colgados de esta página, pero veremos que no encontramos nada.

Ahora buscaremos un sploit con la versión de ftp, podemos ver que encontramos
dos puertas abiertas, uno con un programa de Python y otro con Metasploit.

## 4. Post-explotación

Abriremos Metasploit

Lo buscaremos y con (use) nos permitirá usar este exploit

Para mirar la información que nos pide para poder ejecutarlo, escribiremos (info).

## 5. Escalada de privilegios

Podremos ver que una de las cosas requeridas es el RHOSTS que es la ip de
nuestro objetivo.

Lo podremos proporcionar con set RHOSTS y la ip del servidor. Y para ejecutar el
exploit tendremos que poner (run)

Una vez ejecutado podemos ver que somos root

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
