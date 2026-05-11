# CHATME — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **CHATME** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable

Ahora haremos un escaneo profundo de los puertos que tiene abierto.

Explorando la pagina, vemos una referencia a un dominio, así que lo colocaremos
en nuestra carpeta de /etc/hosts

## 2. Enumeración

Haremos un escaneo de esta pagina con gobuster.

Nos salta un error que tendremos que corregir.
Una vez escaneado vemos varios ficheros.

Si ingresamos a la página, podemos ver que tenemos la simulación de un chat de
WhatsApp

## 3. Explotación

Buscando por internet encontramos que whatshapp tuvo una vulnerabilidad
inyectando un fichero.

Encontramos un script que nos entrega una reverse Shell del equipo.

Preparamos la maquina para realizar la reverse Shell.

## 4. Post-explotación

Seguiremos los pasos del script y lo subiremos al chat.

Si esperamos mas o menos 1 minuto, veremos que nos conecta a una Shell.

Vemos que tenemos permisos al servicio de procmail, así que nos pondremos a
investigar cómo podemos vulnerar estos privilegios.

## 5. Escalada de privilegios

Vemos que no encontramos la carpeta, así que la podremos crear.

Definimos una variable con una ruta y esta es la forma en la que se
configura el archivo .procmailrc, lo que hemos cambiado es touch para que
cree un directorio de la variable que habíamos indicado anteriormente.

Haremos que ejecute touch /tmp/pwned como root, creando este
directorio con propietario root, el texto test se pierde y comprobamos que
se ejecutó correctamente.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
