# TRUST — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🔵 Muy Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **TRUST** de DockerLabs.

---

## 1. Reconocimiento

MAQUINA TRUST

Vamos a desplegar la maquina

## 2. Enumeración

Haremos un ping para comprobar que tenemos conexión a la maquina

Con el comando nmap para buscar los puertos, primero haremos un escaneo
general con -Pn que es para omitir la detección de host.
Una vez visto los puertos podremos abiertos buscar la versión que tienen
ejecutando nmap -p(especificando los puertos que encontramos) -sCV para poder
encontrar la versión de los servicios y -Pn para omitir la detección de host

## 3. Explotación

Podemos ver que tenemos un servidor apache, así que visitaremos la pagina web,
al ver que no cuenta con nada podríamos intentar hacer un ataque con gubuster
para encontrar algún directorio o archivo de la raíz.

Tendremos que ejecutarlo como administrador, dir -w para especificar la ruta del
archivo de alguna seclist desde donde buscaremos los directorios, -u para
especificar la ruta del servidor, -x para especificar extensiones de archivo que
deben probarse al realizar el ataque.

## 4. Post-explotación

Vemos que encontramos /secret.php así que lo probaremos.

Al colocarlo en la URL podremos mirar que Mario es un usuario y ahora sabiendo
que contamos con el puerto ssh abierto podríamos intentar hacer un ataque a
este usuario.

## 5. Escalada de privilegios

Utilizaremos hydra para hacer el ataque, -l para especificar el usuario y -P para el
archivo desde donde tendremos las contraseñas.
Especificaremos que es el servicio ssh.

Podremos observar que ha encontrado la contraseña que es chocolate.
Ahora ingresaremos por ssh a ese usuario.
Al ingresar las credenciales podremos ver que estamos dentro.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
