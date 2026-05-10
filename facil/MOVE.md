# MOVE — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **MOVE** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina

Hacemos un escaneo rápido con -Pn por si no permite las conexiones ping el
servidor

Ahora que sabemos los puertos abiertos, vamos a buscar la versión de cada uno
con -sCV

## 2. Enumeración

Como tenemos un servidor apache vamos a hacer un escaneo de directorios con
gobuster

Vemos como tenemos un fichero, así que lo vamos a ver.

Ahora vamos a ver que contiene el puerto 3000 y vemos que es grafana y cuenta
con la versión 8.3.0

## 3. Explotación

Vamos a buscar a ver si existe algún exploit que podamos aprovechar

Vamos a localizar ese exploit y lo copiaremos para ejecutarlo en nuestra maquina

Para ver como funciona vamos a ejecutarlo con -h y vemos que simplemente
tenemos que colocar el host a donde va dirigido el exploit.

## 4. Post-explotación

Una vez ejecutado nos permite leer ficheros, vamos primero a leer el passwd para
ver con que usuarios contamos, vemos que tenemos un usuario llamado freddy

Si recordamos en maintenance.txt nos decía que la contraseña se encontraba en
/tmp/pass.txt así que veremos que tiene este fichero.

Ya que parece una contraseña intentaremos directamente acceder con eso al
usuario freddy que encontramos

## 5. Escalada de privilegios

Vemos que nos registramos sin problema, así que ahora queda la escalada de
privilegios, ejecutamos sudo -l y vemos que tenemos permiso de ejecutar como
administrador un archivo .py

Lo que haremos será eliminar este archivo y crear uno con el mismo con el código
que nosotros queramos

Vamos a ejecutar el siguiente script que esto nos permite abrir una nueva Shell
desde bash

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
