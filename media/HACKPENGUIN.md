# HACKPENGUIN — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **HACKPENGUIN** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Haremos un escaneo profundo de los puertos abiertos de la maquina vulnerable.

## 2. Enumeración

Ahora al ver el puerto 80, haremos una exploración con gobuster de directorios
que no llegamos a ver.

Nos encontramos con este, donde nos descargaremos la imagen para analizarla.

## 3. Explotación

Utilizamos streghide, pero vemos que tiene una contraseña.

Lo pasamos por esta herramienta para hacer una fuerza bruta y ver si
encontramos la contraseña.

## 4. Post-explotación

Ahora vemos que nos da una base de datos.

Con la herramienta de keepass2john vamos a intentar de igual manera hacer
fuerza bruta y ver si encontramos la contraseña.

## 5. Escalada de privilegios

Al encontrarla podemos meternos y ver las contraseñas que tenga guardadas.

Ingresamos en keepass

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
