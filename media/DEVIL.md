# DEVIL — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟠 Media  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **DEVIL** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos abiertos de la maquina vulnerable.

## 2. Enumeración

Ahora veremos que tipo de web tiene el servidor por el servicio http.

Vamos a listar directorios con gobuster

## 3. Explotación

Vemos que tenemos un wordpress, así que miraremos a que dirección apunta y lo
pondremos en nuestro /etc/hosts

Ahora volveremos a listar y veremos mas directorios, así que iremos buscando
poco a poco.

## 4. Post-explotación

Al final encontramos en una ruta una backdoor, donde vemos que podemos subir
ficheros, intentaremos subir un script php para poder ejecutar comandos y así
hacer una reverse Shell y conectarnos a la maquina.

Lo subimos y haremos comprobaremos que funciona

## 5. Escalada de privilegios

Ahora generaremos la reverse Shell.

Una vez dentro, vemos que tenemos acceso a la carpeta de Andy y listaremos
varios ficheros que encontramos

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
