# AMOR — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  

---

## Resumen

Resolución de la máquina **AMOR** de DockerLabs.

---

## 1. Reconocimiento

Vamos a desplegar la maquina vulnerable.

Hacemos un escaneo profundo de los puertos abiertos en la maquina vulnerable.

## 2. Enumeración

Vemos que tiene un servidor http, exploramos la pagina y nos encontramos con
dos posibles usuarios, Juan y Carlota

Ahora haciendo un ataque de fuerza bruta con hydra nos encontramos con la
contraseña de carlota.

## 3. Explotación

Nos conectamos por ssh

Explorando un poco este usuario encontramos una imagen, la vamos a pasar a
nuestro host y así inspeccionarla.

## 4. Post-explotación

Utilizaremos scp para pasarnos la imagen y luego con steghide para ver lo que
contiene, tiene un txt con una posible contraseña.

Utilizamos cyberchef para averiguar la contraseña.

## 5. Escalada de privilegios

Vemos que contamos con los usuarios oscar y Ubuntu, así que probamos esta
contraseña con alguno de los dos

Ahora como oscar ejecutamos sudo -l a ver si podemos escalar privilegios como
sudo.

## 6. Lección aprendida

Esta máquina enseña a encadenar técnicas de reconocimiento, enumeración y explotación para comprometer un sistema Linux y escalar hasta root.

> 💡 **Consejo para principiantes:** Si te atascas, vuelve al paso de enumeración — casi siempre hay algo que no viste la primera vez.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs*  
*¿Te ha ayudado? Dale una ⭐ al [repositorio](https://github.com/Caan31/Maquinas_DockerLabs)*
