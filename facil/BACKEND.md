# BACKEND — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **BACKEND** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar el laboratorio.

Haremos un escaneo rápido, con el parámetro -Pn por si no permite las
conexiones ping el laboratorio y así ver los puertos abiertos.

Ahora que sabemos los puertos vamos a hacer un escaneo mas profundo,
especificando cada puerto y buscando la versión con la que cuenta.

## 2. Enumeración

Vamos a ver que nos encontramos en el servidor apache.

Al ver que tenemos un login.html vamos a intentar atacar a la base de datos del
laboratorio con sqlmap.
Con el parámetro:
-u: especificaremos la url del atacado.
--forms: Le dice a sqlmap que detecte y analice automáticamente los formularios
HTML de la página para ver si alguno es vulnerable a inyecciones SQL.
--dbs: Si encuentra una vulnerabilidad, intentará listar las bases de datos
disponibles en el servidor SQL comprometido.
--batch: Ejecuta el ataque en modo automático, sin hacer preguntas al usuario
(usa las respuestas por defecto).

Al ver que contamos con una base de datos llamada users vamos a ver que nos
encontramos en ella.
-D: Especifica una base de datos concreta.
--tables: Una vez seleccionada la base de datos, le dices a sqlmap que te muestre
todas las tablas contenidas dentro de esa base de datos.

## 3. Explotación

Ahora al saber que contamos con la tabla usuarios, haremos lo mismo para seguir
explorándola.
-T: Indicas la tabla específica dentro de esa base de datos.
--columns: Le pides a sqlmap que te muestre las columnas que tiene esa tabla

Vemos que cuentan con las siguientes columnas, vamos a seguir viendo que
contienen
-C: Columnas específicas que quieres extraer.
--dump: Le pide a sqlmap que descargue el contenido de esas columnas.

Ahora que contamos con los usuarios y contraseñas, intentaremos conectarnos
por ssh y podremos ver que nos permite conectarnos con el usuario pepe.

## 4. Post-explotación

Intentamos ejecutar sudo -l para escalar privilegios, pero vemos que no podemos
ejecutar el comando.

find / Busca en todo el sistema (desde la raíz /)
-perm -4000 Busca archivos con el bit SUID activado (permiso especial 4000)
-user root

Que pertenezcan al usuario root

## 5. Escalada de privilegios

2>/dev/null Oculta errores (como "Permiso denegado") redirigiendo la salida de
error a /dev/null

Utilizaremos el binario ls para ver si tenemos algo en el directorio de root

Y ahora al saber que contamos con un fichero .hash vamos a listarlo con grep.
Buscaremos por gtfobins la forma correcta de hacerlo.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
