# WINTERFELL — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **WINTERFELL** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de la maquina para ver sus puertos abiertos.

Vemos que cuenta con un servidor web, smb y ssh, así que primero miraremos la
pagina web que aloja.

Como no encontramos nada interesante, utilizaremos gobuster para hacer un
escaneo de directorios escondidos.

## 2. Enumeración

Nos encontramos con /dragón, asi que lo miraremos.

Vemos que tiene varios nombres de películas, están juntos así que supongo que
serán contraseñas

Con los nombres que nos aparece en la web principal hacemos dos ficheros para
hacer un ataque de fuerza bruta.

Vamos a ver que directorios cuenta el servidor smb

## 3. Explotación

Y ahora con crackmapexec haremos un ataque de fuerza bruta para encontrar un
usuario y contraseña

Nos registramos como jon y su contraseña

Listamos y vemos que tenemos un fichero, lo pasaremos a nuestra maquina host y
lo miraremos.

Tenemos un cifrado

## 4. Post-explotación

Utilizaremos cyberchef para descifrarlo.

Ahora nos conectamos como jon por ssh y haremos la escalada de privilegios.

Vemos que tenemos permisos para ejecutar como el usuario aria un script de
Python.

Lo vamos a eliminar y crear uno nuevo con el mismo nombre.

## 5. Escalada de privilegios

Lo hacemos ejecutable y luego lo ejecutamos y vemos que somos el usuario aria.

Ahora hacemos igual un sudo -l y encontramos con el usuario daenerys que puede
listar y ver.

Tenemos la contraseña de esta usuaria.

Una vez dentro vemos que tiene permisos para ejecutar como sudo un fichero
oculto llamado .shell.sh

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
