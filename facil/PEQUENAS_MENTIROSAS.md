# PEQUENAS MENTIROSAS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **PEQUENAS MENTIROSAS** de DockerLabs.

---

## 1. Reconocimiento

Lo primero que haremos es desplegar la maquina y vemos que contamos con la IP
172.17.02

Haremos un ping para comprobar la conexión con el servidor.

Haremos un escaneo con nmap -Pn por si el servidor no permite hacer ping,
vemos que tenemos los puertos 22 y 80.

## 2. Enumeración

Ahora haremos un escaneo más profundo ya sabiendo los puertos y buscando la
versión especifica con -sCV

Miraremos que aloja el servidor web

Podemos intuir que un usuario es a así que vamos a hacer un ataque con hydra
para buscar la contraseña.

## 3. Explotación

Ahora nos conectaremos al usuario a mediante ssh.

Tenemos varias opciones para esta máquina, la primera es listar /etc/passwd que
contiene información sobre las cuentas de usuario del sistema.

Vemos que cuenta con el usuario Spencer, podríamos hacer un ataque directo a
este usuario.

## 4. Post-explotación

La otra opción es mirar los ficheros con los que cuenta el servidor ya que en la
página web nos dio una pista que tenemos cosas en los archivos del servidor.

Listando algunos ficheros podemos ver que nos pide que hagamos un ataque al
usuario.

También hay otro fichero donde contiene un hash del usuario Spencer.

## 5. Escalada de privilegios

Podríamos descifrar el hash con alguna pagina como crackstation.net y vemos
que es la contraseña del Spencer.

La otra opción si no hubiéramos descifrado el hash es hacer el ataque de fuerza
bruta directamente, vemos que encontramos de igual manera la contraseña.

Nos registramos con las credenciales de Spencer y estamos dentro.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
