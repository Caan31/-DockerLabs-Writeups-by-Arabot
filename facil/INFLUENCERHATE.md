# INFLUENCERHATE — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **INFLUENCERHATE** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Haremos un escaneo profundo de los puertos de la maquina.

Vemos que el servicio http, nos pide que inicie sesión.

## 2. Enumeración

Haremos un ataque de fuerza bruta con hydra con los siguientes parametros.
-C /ruta/al/archivo → usa un archivo de combinaciones. Cada línea debe estar
en el formato usuario:contraseña. Hydra leerá esas líneas y probará cada par tal
cual.
-s 80 → puerto a usar (aquí el puerto 80, típico de HTTP). Es redundante en este
caso porque http-get por defecto usa 80, pero fuerza el puerto.
http-get → módulo/servicio que indica a hydra que pruebe autenticación HTTP vía
GET. Esto normalmente se usa contra recursos que requieren HTTP Basic (o
similar) auth. No es el módulo correcto para sitios con formularios HTML (para
formularios se usa http-form-post o http-get-form con parámetros).

Ahora que tenemos las credenciales, podemos poner la contraseña y ejecutar
burp suite para tener más información.

Con gobuster ahora que tenemos la Authorization lo que haremos será listar
directorios con los siguientes parametros.
-H "Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4="
Añade una cabecera HTTP personalizada en todas las peticiones. es la cabecera
de Basic Auth que sacamos con Burp.

## 3. Explotación

Ahora que tenemos un login.php lo vamos a explorar.

Despues de varias pruebas, volvemos a utilizar burp para obtener más
información y poder hacer algo con eso.

Utilizando la herramienta wfuzz vamos a encontrar la contraseña de admin.
wfuzz
La herramienta (web fuzzer / brute-forcer para aplicaciones web).
-c
Coloriza la salida (más legible en terminal).
-z file,/usr/share/wordlists/rockyou.txt
-z define el payload (fuente de datos). file,PATH indica que se leerá la wordlist
desde ese archivo: en este caso rockyou.txt (lista de contraseñas). Por cada línea
de esa lista wfuzz hará una prueba.
-t 50
Threads concurrentes: 50 peticiones en paralelo. Aumenta velocidad, pero
también carga y ruido en el objetivo.
--hh=2848
Oculta (hide) las respuestas cuyo número de caracteres en el cuerpo sea 2848. Es
una forma común de filtrar la página de "login fallido" (si esa página tiene siempre
ese tamaño) para que wfuzz muestre sólo respuestas con tamaños distintos —
posibles indicios de login correcto u otra respuesta útil. En wfuzz hay filtros
similares para códigos (--hc), líneas (--hl) y palabras (--hw).
-H "Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4="
Añade esa cabecera HTTP en todas las peticiones. Decodificando la Base64
aHR0cGFkbWluOmZodHRwYWRtaW4= obtienes httpadmin:fhttpadmin
(usuario:contraseña). Con esto pruebas el formulario mientras el servidor también
recibe la cabecera Basic Auth — útil si la página requiere autenticación HTTP
además del formulario.
-H "Content-Type: application/x-www-form-urlencoded"
Indica que el cuerpo POST es un formulario web estándar (clave=valor), que es lo
que login.php esperará normalmente.
-d "username=admin&password=FUZZ"
Cuerpo de la petición POST. FUZZ es el marcador que wfuzz sustituye por cada
entrada de la wordlist (cada contraseña de rockyou.txt). Estás probando el usuario
fijo admin con cada contraseña.

## 4. Post-explotación

http://172.17.0.2/login.php
URL objetivo (la acción del formulario).

Encontramos un usuario

Ahora utilizaremos hydra para encontrar la contraseña por ssh.
Intentamos hacer una escalada de privilegios, pero no encontramos ninguna
forma, así que utilizaremos una herramienta para hacer fuerza bruta al usuario
root.

## 5. Escalada de privilegios

Despues al intentar con wget, curl, no tenemos éxito para compartir esta
herramienta y como sabemos el usuario y la contraseña de ssh, lo compartiremos
por scp.

Ahora lo ejecutamos y nos da la contraseña

Somos root.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
