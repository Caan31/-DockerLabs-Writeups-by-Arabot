# BORAZUWARACHCTF — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **BORAZUWARACHCTF** de DockerLabs.

---

## 1. Reconocimiento

Desplegaremos la maquina

## 2. Enumeración

Haremos un ping para comprobar que tenemos conexión a la maquina víctima.

## 3. Explotación

Haremos un escaneo con nmap, -Pn desactiva la detección de hosts activos
(ping), tratando todos los objetivos como si estuvieran en línea, útil cuando los
pings están bloqueados por firewalls.

## 4. Post-explotación

Ahora que ya sabemos que puertos están abiertos, haremos un escaneo más
profundo con la versión de cada servicio con -p y -sCV

## 5. Escalada de privilegios

Vemos que tenemos el puerto 80 abierto, así que miraremos que contiene la
pagina del servidor.
Vemos que tiene una imagen, inspeccionando el código no nos encontramos nada
así que descargaremos la imagen para ver que podemos encontrar.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
