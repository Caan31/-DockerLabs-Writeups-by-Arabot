# CHOCOLATELOVERS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **CHOCOLATELOVERS** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Vamos a hacer un escaneo profundo de esta m

Vamos a ver que nos encontramos en el servidor web que tenemos.

## 2. Enumeración

Si inspeccionamos la pagina vemos que hay comentarios donde nos indica un
directorio

Vemos que es un blog de nibbleblog

Explorando un poco tenemos un login, probamos con admin admin y vemos que
tenemos acceso.

## 3. Explotación

Tenemos un dashboard, he buscado que versión es para buscar alguna
vulnerabilidad.

Ahora buscando información he encontrado en incibe que para esta versión hay
una vulnerabilidad con un plugin.

Descargamos este plugin y vemos que podemos subir archivos.

## 4. Post-explotación

Desde nuestro host creamos este fichero para luego subirlo y así poder hacer una
reverse Shell.

Probamos que el archivo que metimos puede ejecutarse.

Una vez conectados, vamos a hacer la escalada de privilegios, cuenta el usuario
chocolate con permisos en el binario php.

## 5. Escalada de privilegios

Con ayuda de gtfobins vamos a ver como escalar correctamente.

Una vez dentro hicimos pruebas con sudo -l y buscando algún SUID vulnerable
pero no encontramos nada interesante, así que utilizaremos la herramienta
pspy64.

Una vez la tengamos en la maquina vulnerable, lo vamos a ejecutar y vemos que
tenemos un fichero en /opt.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
