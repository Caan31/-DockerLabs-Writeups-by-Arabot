# STELLARJWT — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **STELLARJWT** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina virtual vulnerable.

Haremos un escaneo profundo de los puertos abiertos de esta maquina

## 2. Enumeración

Vemos que tiene abierto el puerto de un servidor http, con gobuster vamos a hacer
un escaneo para buscar directorios.

Exploramos las dos paginas que hemos encontrados, en una nos da la pista para
un posible usuario o contraseña.

## 3. Explotación

Mirando el código fuente de una de las páginas encontramos un código que esta
cifrado.

Vemos que tenemos un usuario, Neptuno, con esto podemos probar varias cosas,
la solución es el apellido del alemán que descubrió Neptuno.

## 4. Post-explotación

Vemos que puede conectarse correctamente.

Explorando los directorios encontramos un fichero oculto, al verlo podemos ver
que son posibles contraseñas del usuario nasa.

## 5. Escalada de privilegios

Ahora hacemos la escalada de privilegios.

Con ayuda de gtfobins vemos que escalamos al usuario elite.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
