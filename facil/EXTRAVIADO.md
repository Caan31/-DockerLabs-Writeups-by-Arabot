# EXTRAVIADO — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **EXTRAVIADO** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina

Haremos un escaneo profundo de la máquina.

## 2. Enumeración

En el fichero que hemos generado podemos ver los puertos abiertos de la
máquina.

Vamos a ver que contiene el servidor web y podemos ver que al final de la página
encontramos una codificación en base64.

## 3. Explotación

Utilizaremos nuestra propia maquina para poder ver que es lo que contiene ese
código.

Podemos ver un supuesto usuario y posiblemente la contraseña, así que
intentaremos acceder por ssh.

## 4. Post-explotación

Ahora explorando un poco, podemos ver una nota que nos da indicios que la
contraseña de root tiene que estar por aquí escondida.

Al buscar un poco mas en ficheros ocultos, podemos ver un fichero .secreto
donde encontramos la contraseña del usuario diego en base64

## 5. Escalada de privilegios

Al ver cuantos usuarios podemos ver que un usuario es diego.

`
Accederemos al perfil de diego y explorando un poco podremos encontrar un
fichero pass, vamos a ver que contiene.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
