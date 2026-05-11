# MAPACHE2 — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **MAPACHE2** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos de la máquina.

## 2. Enumeración

Vemos que tenemos el servicio http, así que haremos un gobuster para listar
directorios escondidos.

Encontramos un login.php.

## 3. Explotación

Tambien encontramos una pista en el puerto 3306

Despues de hacer un ataque con el rockyou y no tener resultados, haremos un
ataque con un diccionario generado con cewl, que recolecta datos y genera un
diccionario.

## 4. Post-explotación

Ahora hacemos el ataque de fuerza bruta y vemos que encontramos la
contraseña.

Ahora cuando nos logeamos, encontramos la siguiente pista.
El usuario será Kinder y contraseña medusa.

## 5. Escalada de privilegios

Accedemos por ssh a este usuario.

Ahora hacemos un sudo -l para ver si contamos con con binarios por donde
podamos escalar privilegios.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
