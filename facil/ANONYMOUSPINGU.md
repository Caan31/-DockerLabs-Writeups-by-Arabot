# ANONYMOUSPINGU — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **ANONYMOUSPINGU** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos de la máquina.

Vemos que cuenta con un servidor web, así que exploraremos a ver si
encontramos algo interesante.

## 2. Enumeración

Con dirb vamos a ver que encontramos algunos directorios escondidos.

Como vimos contamos con el servicio ftp y en el propio escaneo nos dice que el
usuario Anonymous está habilitada, así que miraremos que nos podemos
encontrar.

Vemos que contamos con todos los permisos en el directorio upload que esta
vacío por el momento.

## 3. Explotación

Desde nuestro host vamos a crear este fichero de prueba en php para luego poder
ejecutar una reverse Shell.

Lo subimos al servidor fpt

Y ahora vamos a ejecutar la reverse Shell

## 4. Post-explotación

Vemos que estamos dentro y vamos a ver como podemos escalar privilegios con
gtfobins.

Una vez dentro del usuario vamos a seguir encontrando otro usuario con el cual
escalaremos también.

Ahora vemos que contamos con el binario de chown, que buscando un poco
podemos editar un fichero que elijamos, en este caso será el /etc/passwd para
poder quitar la contraseña del usuario root.

## 5. Escalada de privilegios

Como vemos el usuario root cuenta con contraseña (x), lo que intentaremos será
quitarle la contraseña a este usuario.

Ejecutamos los comandos que nos da gtfobins con el directorio que nosotros
queramos en este caso /etc.

Y ahora con sed -i vamos a reemplazar texto en el archivo
s → significa “substituir”.
root:x: → es el texto que busca.
root:: → es el texto que pone en su lugar.
g → significa que haga el reemplazo en todas las coincidencias de cada línea

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
