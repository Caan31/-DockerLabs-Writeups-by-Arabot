# LOS40LADRONES — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **LOS40LADRONES** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Hacemos un escaneo profundo de la maquina para ver los puertos abiertos.

## 2. Enumeración

Vemos el servidor web que no cuenta con nada interesante.

Con gobuster buscaremos directorios escondidos en la maquina y nos
encontramos con un txt.

## 3. Explotación

Vemos que nos dice un supuesto usuario y lo que parecen ser puertos.

Utilizaremos la herramienta knock para tocar puertos en un orden secreto para
pedir que el servidor te abra un puerto que antes estaba oculto.

## 4. Post-explotación

Volvemos a hacer el escaneo de puertos y podemos ver que cuenta con el servicio
ssh que estaba oculto.

Como tenemos un usuario ahora utilizaremos hydra para hacer un ataque de
fuerza bruta y ver si encontramos la contraseña

## 5. Escalada de privilegios

Nos conectamos al usuario con la contraseña que ha encontrado.

Ahora ejecutamos sudo -l para ver si contamos con permisos de ejecución y
vemos que en el binario bash tenemos permisos así que simplemente es ejecutar
sudo bash -p y tendremos una consola como root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
