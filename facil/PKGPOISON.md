# PKGPOISON — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PKGPOISON** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

## 2. Enumeración

Haremos un escaneo profundo de los puertos abiertos de la máquina.

## 3. Explotación

Haremos una búsqueda con gobuster de directorios del servidor.

## 4. Post-explotación

Investigando un poco encontramos una nota donde nos indica un usuario.

## 5. Escalada de privilegios

Haremos un ataque de fuerza bruta con hydra y veremos que nos entrega la
contraseña.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
