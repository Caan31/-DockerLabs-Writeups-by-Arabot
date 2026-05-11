# INJ3CT0RSS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **INJ3CT0RSS** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Ahora haremos un escaneo profundo para ver los puertos abiertos del servidor.

Vemos que tenemos el servicio de http, así que vamos a mirar que contiene.

Explorando un poco encontramos un panel de login.

## 2. Enumeración

Aplicando sql injection podemos ver que ingresamos como administrador.

No encontramos nada interesante así que vamos a utilizar sqlmap para ver si
encontramos algo dentro de la base de datos.

Vemos que tenemos bases de datos, así seguiremos explorando que
encontramos.

Al ver un supuesto directorio, lo exploramos y vemos un fichero .zip

## 3. Explotación

Lo intentamos extraer y vemos que cuenta con una contraseña.

Utilizaremos zip2john para generar un hash y luego así poder utilizar john y así
encontrar la contraseña.

Encontramos las credenciales de un usuario.

Ahora nos conectamos como este usuario y vemos

## 4. Post-explotación

Ahora vemos que tenemos el permiso del binario busybox en un directorio.

Con ayuda de gtfobin veremos cómo podemos escalar privilegios al otro usuario y
ejecutaremos los comandos.

Ahora vemos que encontramos en su directorio sus credenciales.

Nos conectamos por remoto al usuario por ssh.

## 5. Escalada de privilegios

Vemos a que tenemos permisos y vemos que al binario cat.

Lo que haremos será ver la clave privada del usuario root.

Nos lo creamos dentro de nuestro host y vamos a dar permisos con chmod.

Nos conectamos por ssh con la clave que hemos hecho en nuestro host y vemos
que somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
