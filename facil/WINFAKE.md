# WINFAKE — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **WINFAKE** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Ahora hacemos un escaneo profundo de los puertos abiertos de la maquina.

## 2. Enumeración

Vemos que cuenta con el servicio http, así que, inspeccionando un poco, vemos
que tiene oculto en el body un nombre, (pipe) que lo mas seguro es que sea un
nombre de usuario.

Hacemos un ataque con hydra a este usuario y vemos que somos pipe

## 3. Explotación

Al conectarnos nos damos cuenta de que es como una Shell de Windows,
comandos, etc.

Al momento de intentar entrar a la carpeta de root, vemos que nos expulsa.
Pero nos indica la ubicación del archivo de donde se esta ejecutando todo.

## 4. Post-explotación

Vamos a mirarlo y es un script completo que hace la simulación de una Shell de
Windows, donde nos prohíbe ejecutar comandos de Linux.

Nos permite ejecutar su root, así que tendremos que buscar como encontramos la
contraseña directamente de root.

## 5. Escalada de privilegios

Volviendo a inspeccionar la pagina web, vemos que tiene un acróstico inicial.

Al hacer esto de la pagina web principal, obtenemos esta frase.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
