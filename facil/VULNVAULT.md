# VULNVAULT — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **VULNVAULT** de DockerLabs.

---

## 1. Reconocimiento

Desplegamos el laboratorio

Haremos un escaneo simple para identificar los puertos abiertos del laboratorio.

Ahora que sabemos los puertos que tenemos abiertos, vamos a buscar la versión
con la que cuentan.

Vamos a ver que la pagina cuenta con generar un reporte, así que vamos a hacer
una prueba de un fichero php, vemos que nos genera un reporte.txt y no ejecuta
nada.

## 2. Enumeración

Vamos a ejecutar un ls -la para ver si lista algo

Vemos que podemos ejecutar como una consola.

Vamos a ver el passwd del laboratorio.

Vemos que tenemos el usuario samara, así que veremos que nos encontramos en
su directorio.

## 3. Explotación

Tenemos dos txt, vemos que uno pertenece a root y otro a tamara, los vamos a
listar y ver que nos encontramos.

Al ver que no nos encontramos nada, vamos a la carpeta .ssh a ver si nos
encontramos algo.

Vemos que tenemos el id_rsa, vamos a copiárnoslo en nuestro equipo para poder
ingresar con ssh a ese usuario

Ahora en nuestro equipo crearemos un fichero llamado id_rsa y pegaremos la
contraseña.

## 4. Post-explotación

Le daremos permisos de lectura y escritura para poder conectarnos

Con el fichero creado ahora podremos conectarnos con el usuario samara

Ejecutamos sudo -l para ver como podemos escalar privilegios, pero vemos que
no podemos ejecutarlo.

Ahora buscaremos binarios a ver si encontramos algo, pero nada.

## 5. Escalada de privilegios

Vamos a instalarnos la herramienta pspy64 que nos permitirá Ver procesos
ejecutados por otros usuarios, en este caso buscamos root.

Lo instalaremos en nuestra maquina local y luego abriremos un servidor http para
poder conectarnos desde el laboratorio.

Damos permisos de ejecución a la herramienta y la ejecutamos

Vemos que se ejecuta mucho un fichero por el usuario root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
