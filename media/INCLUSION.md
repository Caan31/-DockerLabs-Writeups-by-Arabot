# INCLUSION — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **INCLUSION** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Haremos un escaneo profundo de los puertos abiertos de la maquina vulnerable.

## 2. Enumeración

Vemos que tiene el puerto http abierto pero no encontramos nada interesante.

Haremos una búsqueda con gobuster y veremos que tenemos un directorio
llamado shop. Dentro de este también haremos una búsqueda.

## 3. Explotación

Vemos que tenemos una pista En el error, este error es de PHP que nos indica un
supuesto fichero llamado archivo.

Buscaremos haciendo fuzzing sobre este para listar el /etc/passwd

## 4. Post-explotación

Ya que tenemos dos usuarios, haremos un ataque de fuerza bruta a uno de ellos
con hydra y obtendremos la contraseña

Ahora nos conectamos por ssh a este usuario.

## 5. Escalada de privilegios

Al buscar y no encontrar nada para poder utilizar, vamos a utilizar la herramienta
su forcé para hacer fuerza bruta a los usuarios del laboratorio.

Nos lo compartimos desde nuestro host por ssh.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
