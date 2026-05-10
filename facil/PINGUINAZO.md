# PINGUINAZO — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PINGUINAZO** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Vamos a hacer un escaneo profundo de los puertos de la maquina.

## 2. Enumeración

Vemos que cuenta con un servidor web por el puerto 5000 y que podemos escribir
para ver que luego lo muestra en pantalla.

Vamos a probar a ver ejecuta {{7}} para ver si podemos hacer un ataque STTI

## 3. Explotación

Con la ayuda de la pagina
https://github.com/cybersnippets/PayloadsAllTheThings/tree/master/Server%20Si
de%20Template%20Injection

Vamos a editarlo para poder hacer un ataque con reverse Shell.

## 4. Post-explotación

Nos ponemos a escuchar desde nuestra maquina

Vemos que tenemos acceso al usuario.

## 5. Escalada de privilegios

Vemos que tenemos permisos de sudo en el binario de java, lo busque en gtfobins
y no encontré nada.

Me he ayudado de la siguiente pagina
https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/sudo-java-privilegeescalation

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
