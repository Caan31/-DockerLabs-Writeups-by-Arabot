<div align="center">

# 🐋 DockerLabs — Writeups by Arabot

**Resoluciones paso a paso · En español · Para principiantes**

[![Máquinas resueltas](https://img.shields.io/badge/Máquinas%20resueltas-64-00ff88?style=flat-square&logo=docker&logoColor=white)](#)
[![Muy Fácil](https://img.shields.io/badge/Muy%20Fácil-5-0099ff?style=flat-square)](#-muy-fácil)
[![Fácil](https://img.shields.io/badge/Fácil-46-00cc66?style=flat-square)](#-fácil)
[![Media](https://img.shields.io/badge/Media-13-ff9900?style=flat-square)](#-media)
[![Autor](https://img.shields.io/badge/Autor-Arabot-ff6b35?style=flat-square&logo=github&logoColor=white)](https://github.com/Caan31)

> *"No busco llegar solo — documento cada paso para que tú también puedas."*

</div>

---

## ¿Qué encontrarás aquí?

Cada writeup sigue la misma estructura:
**Reconocimiento → Enumeración → Explotación → Escalada de privilegios → Lección aprendida**

El objetivo no es darte la flag directamente — es que entiendas **por qué** funciona cada técnica.

---

## 🗂️ Estructura del repositorio

```
Maquinas_DockerLabs/
├── README.md
├── muy-facil/          # 5 máquinas
│   ├── TRUST.md
│   ├── TPROOT.md
│   ├── VACACIONES.md
│   ├── BORAZUWARACHCTF.md
│   └── OBSESSION.md
├── facil/              # 46 máquinas
│   ├── AGUADEMAYO.md
│   ├── ALLIEN.md
│   └── ...
└── media/              # 13 máquinas
    ├── DEVIL.md
    ├── PING_PONG.md
    └── ...
```

---

## 🔵 Muy Fácil

> Punto de entrada ideal si estás empezando. Técnicas básicas bien explicadas.

| Máquina | Técnicas principales | Writeup |
|---------|---------------------|:-------:|
| TRUST | Gobuster · Hydra · SSH · vim/sudo | [📄 Ver](./muy-facil/TRUST.md) |
| TPROOT | FTP vulnerable · Metasploit | [📄 Ver](./muy-facil/TPROOT.md) |
| VACACIONES | Medusa · /var/mail · sudo | [📄 Ver](./muy-facil/VACACIONES.md) |
| BORAZUWARACHCTF | Exiftool · Hydra · SSH · sudo su | [📄 Ver](./muy-facil/BORAZUWARACHCTF.md) |
| OBSESSION | FTP anónimo · Hydra · vim/sudo | [📄 Ver](./muy-facil/OBSESSION.md) |

---

## 🟢 Fácil

> Técnicas más variadas. Buen nivel para consolidar fundamentos.

| Máquina | Técnicas principales | Writeup |
|---------|---------------------|:-------:|
| AGUADEMAYO | Dirb · Código fuente · Dcode.fr · SSH | [📄 Ver](./facil/AGUADEMAYO.md) |
| ALLIEN | Enum4linux · CrackMapExec · SMB · Reverse Shell | [📄 Ver](./facil/ALLIEN.md) |
| AMOR | Hydra · SCP · Steghide · CyberChef | [📄 Ver](./facil/AMOR.md) |
| ANONYMOUSPINGU | FTP anónimo · File Upload · chown/passwd | [📄 Ver](./facil/ANONYMOUSPINGU.md) |
| BACKEND | SQLmap · SUID · CrackStation | [📄 Ver](./facil/BACKEND.md) |
| BALUFOOD | Puerto 5000 · Código fuente · .bashrc | [📄 Ver](./facil/BALUFOOD.md) |
| BALULERO | Pspy64 · CRON · Privesc | [📄 Ver](./facil/BALULERO.md) |
| BYPASSME | SQLi · ps aux · Socket · CRON | [📄 Ver](./facil/BYPASSME.md) |
| CHOCOLATELOVERS | Nibbleblog · Plugin Upload · Pspy64 | [📄 Ver](./facil/CHOCOLATELOVERS.md) |
| CONSOLELOG | server.js · Hydra · nano/sudo | [📄 Ver](./facil/CONSOLELOG.md) |
| DOCKERLABS | File Upload (.phar) · cut/grep sudo | [📄 Ver](./facil/DOCKERLABS.md) |
| ELEVATOR | File Upload (.jpg bypass) · Escalada encadenada | [📄 Ver](./facil/ELEVATOR.md) |
| ESCOLARES | WordPress · CUPP · WPScan · awk/sudo | [📄 Ver](./facil/ESCOLARES.md) |
| EXTRAVIADO | Base64 · Ficheros ocultos · Acertijo | [📄 Ver](./facil/EXTRAVIADO.md) |
| FILE | File Upload · Steghide · Stegcracker · Python/sudo | [📄 Ver](./facil/FILE.md) |
| FINDYOURSTYLE | Drupal 8 · Metasploit · LinPEAS | [📄 Ver](./facil/FINDYOURSTYLE.md) |
| FORBIDDENHACK | 403 Bypass · LFI · PHP filter chain | [📄 Ver](./facil/FORBIDDENHACK.md) |
| GALERIA | File Upload · PATH Hijacking | [📄 Ver](./facil/GALERIA.md) |
| GROOTI | MySQL · ZIP · Hydra · CRON | [📄 Ver](./facil/GROOTI.md) |
| INFLUENCERHATE | HTTP Basic Auth · wfuzz · Burp Suite | [📄 Ver](./facil/INFLUENCERHATE.md) |
| INTERNSHIP | SQLi · ROT13 · Steghide · vim/sudo | [📄 Ver](./facil/INTERNSHIP.md) |
| JENHACK | Jenkins · Groovy Console · CyberChef | [📄 Ver](./facil/JENHACK.md) |
| LIBRARY | index.php contraseña · Python/sudo | [📄 Ver](./facil/LIBRARY.md) |
| LOS40LADRONES | Port Knocking · Hydra · bash/sudo | [📄 Ver](./facil/LOS40LADRONES.md) |
| MIRAME | Burp · SQLmap · Steghide · SUID/find | [📄 Ver](./facil/MIRAME.md) |
| MOVE | Grafana CVE · LFI · Python/sudo | [📄 Ver](./facil/MOVE.md) |
| NODECLIMB | FTP · zip2john · John · Node.js/sudo | [📄 Ver](./facil/NODECLIMB.md) |
| PARADISE | Cifrado · Hydra · sed/sudo | [📄 Ver](./facil/PARADISE.md) |
| PATRIAQUERIDA | Path Traversal · LFI · Python3/sudo | [📄 Ver](./facil/PATRIAQUERIDA.md) |
| PEQUENAS_MENTIROSAS | Hydra · Hash cracking · Python3/sudo | [📄 Ver](./facil/PEQUENAS_MENTIROSAS.md) |
| PICADILLY | Cifrado César · File Upload HTTPS | [📄 Ver](./facil/PICADILLY.md) |
| PINGUINAZO | SSTI · Reverse Shell · Java/sudo | [📄 Ver](./facil/PINGUINAZO.md) |
| PKGPOISON | Gobuster · Hydra · GTFObins | [📄 Ver](./facil/PKGPOISON.md) |
| PNTOPNTOBARRA | LFI · id_rsa · env/sudo | [📄 Ver](./facil/PNTOPNTOBARRA.md) |
| PRESSENTER | WordPress · WPScan · LinPEAS · MySQL | [📄 Ver](./facil/PRESSENTER.md) |
| REFLECTION | XSS lab · SUID/env | [📄 Ver](./facil/REFLECTION.md) |
| SECRETJENKINS | Jenkins CVE · LFI · Privesc encadenada | [📄 Ver](./facil/SECRETJENKINS.md) |
| SHOWTIME | Burp · SQLmap · Panel Python · CRON | [📄 Ver](./facil/SHOWTIME.md) |
| STELLARJWT | Cifrado · chown/passwd · Escalada encadenada | [📄 Ver](./facil/STELLARJWT.md) |
| UPLOAD | File Upload · PHP webshell · env/sudo | [📄 Ver](./facil/UPLOAD.md) |
| VULNVAULT | Command Injection · id_rsa · Pspy64 | [📄 Ver](./facil/VULNVAULT.md) |
| WALKINGCMS | WordPress · WPScan · SUID/env | [📄 Ver](./facil/WALKINGCMS.md) |
| WALKING_DEAD | Gobuster · Fichero oculto · Python/SUID | [📄 Ver](./facil/WALKING_DEAD.md) |
| WHOAMI | WordPress CVE-2021 · Escalada encadenada | [📄 Ver](./facil/WHOAMI.md) |
| WINFAKE | HTML oculto · Acróstico · su root | [📄 Ver](./facil/WINFAKE.md) |
| WINTERFELL | SMB · CrackMapExec · Escalada encadenada | [📄 Ver](./facil/WINTERFELL.md) |

---

## 🟠 Media

> Requieren encadenar varias técnicas. Recomendadas cuando ya tienes soltura con el nivel fácil.

| Máquina | Técnicas principales | Writeup |
|---------|---------------------|:-------:|
| ASUCAR | WordPress · Nuclei · openssl/sudo | [📄 Ver](./media/ASUCAR.md) |
| CHATME | WhatsApp CVE · procmail · Privesc | [📄 Ver](./media/CHATME.md) |
| DANCE-SAMBA | SMB · SSH key injection · file/sudo | [📄 Ver](./media/DANCE-SAMBA.md) |
| DEVIL | WordPress backdoor · SUID custom · Juego | [📄 Ver](./media/DEVIL.md) |
| HACKPENGUIN | Steghide · KeePass · keepass2john | [📄 Ver](./media/HACKPENGUIN.md) |
| INCLUSION | LFI · Hydra · Su_Force · php/sudo | [📄 Ver](./media/INCLUSION.md) |
| INJ3CT0RSS | SQLi · zip2john · Busybox · cat/sudo | [📄 Ver](./media/INJ3CT0RSS.md) |
| MAPACHE2 | CeWL · Hydra · apache2/init.d | [📄 Ver](./media/MAPACHE2.md) |
| MEMESPLOIT | SMB · Palabras ocultas · Grupo security | [📄 Ver](./media/MEMESPLOIT.md) |
| PING_PONG | Command Injection · Escalada 6 usuarios | [📄 Ver](./media/PING_PONG.md) |
| REPORT | LFI · PHP filter chain · Git logs | [📄 Ver](./media/REPORT.md) |
| RUTAS | FTP · RFI · PATH hijacking · LinPEAS | [📄 Ver](./media/RUTAS.md) |
| SITES | vulnerable.php · wfuzz · sed/sudo | [📄 Ver](./media/SITES.md) |

---

## 🗺️ ¿Por dónde empiezo si soy principiante?

```
1. Instala DockerLabs → https://dockerlabs.es/
2. Empieza con TRUST o VACACIONES (Muy Fácil)
3. Lee el writeup DESPUÉS de intentarlo tú solo
4. Pasa a Fácil cuando ya no te atasques en las básicas
5. Media cuando las fáciles te parezcan cómodas
```

---

<div align="center">

**¿Te ha ayudado algún writeup?** Dale una ⭐ — ayuda a que más gente lo encuentre.

[![GitHub](https://img.shields.io/badge/Más%20contenido-github.com/Caan31-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/Caan31)

*Arabot · Aprendiendo en público · Ciberseguridad en español* 🇪🇸

</div>
