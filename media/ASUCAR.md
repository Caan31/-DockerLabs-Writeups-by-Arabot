# ASUCAR — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **ASUCAR** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos en la maquina vulnerable.

## 2. Enumeración

Vamos a ver de que trata la web que aloja este servidor.

Vemos que al meternos con wp-admin para ver que nos encontramos, tenemos
una dirección.

## 3. Explotación

Esta dirección la vamos a meter en nuestro /etc/hosts

Ahora con nuclei vamos a listar directorios que puedan ser comprometidos de
wordpress.

## 4. Post-explotación

Encontramos la siguiente ruta.

Al explorarla vemos que tenemos un usuario, así que haremos un ataque de fuerza
bruta.

## 5. Escalada de privilegios

Ahora nos conectaremos por ssh.

Una vez dentro, vamos a ver si tenemos privilegios para algun binario con
permisos de sudo.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
