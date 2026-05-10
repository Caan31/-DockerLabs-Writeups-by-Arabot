# SECRETJENKINS — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **SECRETJENKINS** de DockerLabs.

---

## 1. Reconocimiento

Desplegamos el laboratorio

Para comprobar que tenemos conexión con el laboratorio haremos un ping.

Vamos a hacer un escaneo rápido y con el parámetro -Pn por si no permite
conexiones ping el laboratorio

## 2. Enumeración

Al ver los puertos abiertos, ahora haremos un escaneo mas profundo
especificando los puertos y buscando la versión con la que cuenta cada servicio.

Vemos que tenemos un servidor Jetty por el puerto 8080, vamos a ver que nos
encontramos.

Vamos a ver que versiones en la pagina web nos encontramos por si encontramos
un exploit, vemos que tenemos una versión de Jenkins y Jetty

## 3. Explotación

Ahora buscaremos un exploit y lo vamos a intentar ejecutar

Vamos a localizar ese exploit y lo copiaremos en nuestro directorio del laboratorio.

Lo ejecutaremos para ver como se puede utilizar y vemos que nos pide poner la
URL del laboratorio atacado.

## 4. Post-explotación

Lo colocamos y nos permite elegir un directorio para descargar

Vamos a descargar el directorio passwd para ver con que usuarios nos
encontramos.

Al ver los usuarios, ahora intentaremos hacer un ataque de fuerza bruta.

## 5. Escalada de privilegios

Vemos que encontramos la contraseña así que nos conectaremos por ssh

Vamos a ver si tenemos privilegios como sudo, tenemos el binario de python3 pero
solamente con el usuario pinguinito, así que ejecutaremos como sudo y vamos a
escribir un pequeño script para poder escalar y tener una consola bash.

Ahora ya que somos el usuario volveremos a ver si tenemos privilegios para
ejecutar como sudo.
Tenemos un script al que podemos ejecutar.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
