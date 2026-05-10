# REFLECTION — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **REFLECTION** de DockerLabs.

---

## 1. Reconocimiento

Desplegaremos el laboratorio.

Haremos un escaneo con nmap -Pn por si la maquina no permite las conexiones
ping.

## 2. Enumeración

Ahora al ver los puertos con los que contamos, haremos un escaneo avanzado
para ver la versión con la que cuenta cada servicio.

Vamos a explorar el servidor http del laboratorio, vemos que cuenta con una
interfaz bastante grafica.

## 3. Explotación

Explorando un poco y al realizar pruebas en cada laboratorio donde explica
diferentes maneras de XSS, encontramos el usuario y la contraseña de ssh.

Ahora nos conectaremos a este usuario.

## 4. Post-explotación

Vemos si tenemos algún permiso de sudo y vemos que no contamos con nada.

Haremos una búsqueda mas profunda a ver que permisos SUID cuenta la
maquina que se pueda vulnerar.

## 5. Escalada de privilegios

Vamos a probar con el binario env, así que buscaremos desde GTFobins como
podríamos vulnerar.

Ahora lo ejecutaremos en nuestra maquina y vemos que contamos con acceso al
usuario root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
