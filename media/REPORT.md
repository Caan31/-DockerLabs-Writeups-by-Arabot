# REPORT — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **REPORT** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos de la maquina vulnerable.

Vemos que si vamos a la pagina nos redirige a esta dirección así que la pondremos
en nuestro /etc/hosts

Ahora vamos a gobuster y vemos que tenemos varias paginas.

## 2. Enumeración

La vulnerabilidad que haremos será de LFI, hay más en esta máquina, ya que
explorando nos indica que se puede resolver de varias maneras.
Con wfuzz haremos un escaneo y vemos que nos lista desde el directorio file.

Lo probamos y vemos que tenemos el /etc/passwd.

Ahora utilizaremos la herramienta php_filter_chain_generator

Lo que podremos hacer ahora es ejecutar comandos como si tuviéramos una
consola, así que haremos una reverse Shell.

## 3. Explotación

Nos ponemos en escucha.

Lo que tendremos que ejecutar en el navegador será de la siguiente manera.

Ahora tenemos acceso a la maquina y vamos con la escalada de privilegios.

Ahora listamos los directorios y vemos varias cosas interesantes, primero
buscaremos config.php y vemos que tenemos la contraseña del usuario root en la
base de datos.

## 4. Post-explotación

Nos metemos en la base de datos para ver que encontramos.

Listamos todo y vemos que en las columnas nos encontramos con logs de git

En la carpeta de desarrollo vemos que se encuentra .git, así que desde aquí
buscaremos los logs

Al ver que no nos deja, vamos a expotarlo a la carpeta /tmp y volvemos a ejecutar
el comando que nos indica, ahora nos dejara ver los logs.

## 5. Escalada de privilegios

Si miramos ese por adm, vemos que tenemos una contraseña.

Nos metemos como el usuario adm

En su directorio vemos que tiene un fichero .bashrc oculto.

Lo miramos y nos encontramos con una contraseña en hexadecimal.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
