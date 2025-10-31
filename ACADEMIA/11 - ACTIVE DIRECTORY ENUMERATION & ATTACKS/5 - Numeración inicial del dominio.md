**Tareas:**
- Enumerar la red interna, identificar Hosts, servicios críticos y posibles vias de Foothold.
- Formas activas y pasivas de identificar usuarios, hosts y vulnerabilidades serán de gran utilidad para el futuro.
- Documentarlo todo. Es extremadamente importante. 

Vamos a empezar partiendo de una prueba de penetración "ciega". No disponemos de credenciales ni usuarios ni ningún otro tipo de información.

| **Categoría de Datos**                                            | **Objetivo Principal en el Pentest**                                                                                                                                                                       |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AD Users (Usuarios de AD)**                                     | **Enumerar cuentas válidas** para utilizarlas como objetivos en ataques de **Password Spraying**.                                                                                                          |
| **AD Joined Computers (Equipos Unidos a AD)**                     | **Identificar máquinas críticas** en la red. Esto incluye activos de alto valor como **Controladores de Dominio**, servidores de archivos, servidores SQL, servidores web y servidores de correo Exchange. |
| **Key Services (Servicios Clave)**                                | **Localizar y analizar los servicios fundamentales** que sustentan el dominio: **Kerberos** (autenticación), **NetBIOS**, **LDAP** (directorio) y **DNS** (resolución de nombres).                         |
| **Vulnerable Hosts and Services (Hosts y Servicios Vulnerables)** | **Encontrar "victorias rápidas"** (_quick wins_), es decir, cualquier _host_ o servicio que pueda ser explotado fácilmente para **obtener un punto de apoyo (foothold)** inicial en la red.                |

--

## Metodología y orden de trabajo

**Enumerar un entorno de AD puede ser abrumador si se aborda sin una metodología.**

Existe una gran cantidad de datos almacenados en AD, y su cribado puede llevar mucho tiempo si no se analiza en etapas progresivas, y es probable que pasemos por alto algunos. 
Necesitamos establecer un plan de acción y abordarlo paso a paso. Cada persona trabaja de forma ligeramente diferente, así que a medida que adquirimos más experiencia, comenzaremos a desarrollar nuestra propia metodología repetible que mejor se adapte a nuestras necesidades:

Comenzaremos con la identificación pasiva de cualquier host en la red, seguida de una validación activa de los resultados para obtener más información sobre cada host (qué servicios se están ejecutando, nombres, posibles vulnerabilidades, etc.). Una vez que sepamos qué hosts existen, podemos proceder a sondearlos en busca de cualquier dato interesante que podamos obtener de ellos. Tras completar estas tareas, deberíamos reflexionar sobre la información disponible. En este punto, con suerte, tendremos un conjunto de credenciales o una cuenta de usuario para establecernos en un host unido al dominio o la capacidad de iniciar la enumeración de credenciales desde nuestro host de ataque Linux.

Analicemos algunas herramientas y técnicas que nos ayudarán con esta enumeración.

### Identifying Hosts

Podemos utilizar Wireshark y TCPDump para escuchar que está pasando en la red y ver que Hosts y tipo de tráfico de red existe y podemos capturar (escenario "black-box").

Escuchamos paquetería ARP "requests/reply" , mDNS y demás:
- Las ARP requests and replies (solicitudes y respuestas) son cruciales porque revelan qué direcciones IP están realmente activas y respondiendo en mi segmento de red local.
- mDNS (Multicast DNS): Es la forma en que los dispositivos (como impresoras o dispositivos Apple) anuncian sus servicios y resuelven nombres sin un servidor DNS central. Simplemente están "gritando" su nombre y servicios a todos en la red local.

Iniciamos el wireshark con:
```
sudo -E wireshark
```

Y observamos que, si filtramos por:
- paquetería ARP:
	  Encontramos peticiones del tipo "Who has 10.129.201.78? Tell 10.129.46.183". Esto nos dá información tanto de quien solicita la IP (TELL ...) como a quien se lo pregunta (Who has ...).	 
![[ea-wireshark.webp]]
- La paquetería MDNS nos notifica que existe el Host "ACADEMY-EA-WEB01".
![[ea-wireshark-mdns.webp]]


En este caso hemos utilizado WireShark, pero podríamos utilizar en caso de no tener GUI tcpdump, net-creds, NetMiner, ....

### Herramienta: Responder
Responder es una herramienta diseñada para escuchar, analizar y envenenar solicitudes y respuestas LLMNR, NBT-NS y MDNS:

LLMNR, NBT-NS y mDNS son protocolos usados para resolver nombres dentro de una red local cuando no hay un servidor DNS disponible.
- LLMNR, propio de Microsoft, funciona por multicast en el puerto UDP 5355 y permite que un equipo pregunte “¿quién tiene este nombre?”; otro responde con la IP correspondiente.
- NBT-NS es una versión más antigua, basada en NetBIOS y el puerto UDP 137, que hace algo similar mediante broadcast, muy usada por sistemas Windows antiguos para compartir archivos. 
- mDNS, en cambio, es el equivalente moderno usado por servicios como Bonjour o Avahi: trabaja sobre UDP 5353 y la dirección multicast 224.0.0.251 para resolver nombres que terminan en .local y descubrir impresoras o dispositivos.


Responder tiene muchas más funciones, pero por ahora solo la utilizamos en modo Analizar. Esto escuchará pasivamente la red y no enviará paquetes envenenados. Analizaremos esta herramienta con más detalle en secciones posteriores.

Inicializamos responder con el comando:
```
sudo responder -I ens224 -A 
```
![[responder-example 1.gif]]


Al iniciar Responder con el modo de análisis (- A) pasivo habilitado, veremos el flujo de solicitudes en nuestra sesión. 
Encontramos también algunos HOSTS no encontrados previamente. Los anotaremos para crear una buena lista de IP y sus nombres de host DNS.

## FPING
Vamos a hacer algunas comprobaciones activas. Empezaremos por la herramienta **fping**.
Esta utiliza solicitudes del tipo ICMP para contactar e interactuar con un HOST. 
Tenemos la posibilidad de enviar paquestes ICMP a varios HOSTS.

Ejecutaremos:
```bash
Polika4RM@htb[/htb]$ fping -asgq 172.16.5.0/23
```

El parámetro:
- -a: muestra targets alive.
- -s: imprime estadísticas al final del escaneo.
- -g: sirve para enumerar una subred y no poner cada dirección manualmente
- -q: oculta el mensaje "172.16.5.5 is alive // is unreachable"

Obtenemos una salida del tipo:
```shell-session
Polika4RM@htb[/htb]$ fping -asgq 172.16.5.0/23

172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.200
172.16.5.225
172.16.5.238
172.16.5.240

     510 targets
       9 alive
     501 unreachable
       0 unknown addresses

    2004 timeouts (waiting for response)
    2013 ICMP Echos sent
       9 ICMP Echo Replies received
    2004 other ICMP received

 0.029 ms (min round trip time)
 0.396 ms (avg round trip time)
 0.799 ms (max round trip time)
       15.366 sec (elapsed real time)
```

## Escaneo con nmap
Ahora que tenemos una lista de hosts activos en nuestra red, podemos enumerarlos con más detalle con **nmap**.

Buscamos determinar qué servicios ejecuta cada host, identificar hosts críticos como controladores de dominio y servidores web, e identificar hosts potencialmente vulnerables para investigarlos posteriormente. 

Dado que nos centramos en AD, tras un análisis general, sería prudente centrarnos en los protocolos estándar que suelen acompañar a los servicios de AD, como DNS, SMB, LDAP y Kerberos, por nombrar algunos. A continuación, se muestra un ejemplo rápido de un análisis sencillo con Nmap.

Ejecutaremos: 
```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

La opción **`-iL`** le dice a **Nmap** que **lea los objetivos** (las direcciones IP o los nombres de host que quieres escanear) **desde un archivo** en lugar de especificarlos directamente en la línea de comandos.

Como salida, obtenemos: 
```shell-session
Nmap scan report for inlanefreight.local (172.16.5.5)
Host is up (0.069s latency).
Not shown: 987 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-04-04 15:12:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
|_ssl-date: 2022-04-04T15:12:53+00:00; -1s from scanner time.
| ssl-cert: Subject:
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
<SNIP>  
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-DC01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2022-04-04T15:12:45+00:00
<SNIP>
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: ACADEMY-EA-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## IMPORTANTÍSIMO!! IDENTIFICACIÓN DEL DOMAIN CONTROLLER:
Nuestros análisis nos han proporcionado el estándar de nombres utilizado por NetBIOS y DNS. 
Observamos que algunos hosts tienen RDP abierto y nos han indicado la dirección del controlador de dominio: INLANEFREIGHT.LOCAL (ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL). Los resultados a continuación muestran algunos resultados interesantes relacionados con un host posiblemente obsoleto (no incluido en nuestro laboratorio actual).

Al realizar un:

```shell-session
Polika4RM@htb[/htb]$ nmap -A 172.16.5.100

Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 13:42 EDT
Nmap scan report for 172.16.5.100
Host is up (0.071s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  https?
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7600 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2008 R2 10.50.1600.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-04-08T17:38:25
|_Not valid after:  2052-04-08T17:38:25
|_ssl-date: 2022-04-08T17:43:53+00:00; 0s from scanner time.
| ms-sql-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-CTX1
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-CTX1.INLANEFREIGHT.LOCAL
|_  Product_Version: 6.1.7600
Host script results:
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| ms-sql-info: 
|   172.16.5.100:1433: 
|     Version: 
|       name: Microsoft SQL Server 2008 R2 RTM
|       number: 10.50.1600.00
|       Product: Microsoft SQL Server 2008 R2
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_nbstat: NetBIOS name: ACADEMY-EA-CTX1, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:c7:1c (VMware)
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7600 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::-
|   Computer name: ACADEMY-EA-CTX1
|   NetBIOS computer name: ACADEMY-EA-CTX1\x00
|   Domain name: INLANEFREIGHT.LOCAL
|   Forest name: INLANEFREIGHT.LOCAL
|   FQDN: ACADEMY-EA-CTX1.INLANEFREIGHT.LOCAL
|_  System time: 2022-04-08T10:43:48-07:00

<SNIP>
```

En el resultado anterior, podemos ver que tenemos un host potencial con un sistema operativo obsoleto (Windows 7, 8 o Server 2008, según el resultado). Esto nos interesa, ya que significa que hay sistemas operativos heredados ejecutándose en este entorno de AD. También significa que existe la posibilidad de que exploits antiguos como EternalBlue, MS08-067 y otros funcionen y nos proporcionen un shell a nivel de SISTEMA. 

Los resultados de estos análisis nos darán una pista sobre dónde empezar a buscar posibles vías de enumeración de dominios, no solo el análisis de hosts. Necesitamos encontrar una cuenta de usuario de dominio. Analizando nuestros resultados, encontramos varios servidores que alojan servicios de dominio (DC01, MX01, WS01, etc.). Ahora que sabemos qué existe y qué servicios se están ejecutando, podemos sondear esos servidores e intentar enumerar usuarios. Asegúrese de usar el indicador -oA como práctica recomendada al realizar análisis con Nmap. Esto garantizará que tengamos los resultados de nuestros análisis en varios formatos para fines de registro y formatos que puedan manipularse e incorporarse a otras herramientas.

Probablemente volveremos a estos resultados más adelante para una enumeración más detallada, así que no los olvide:
###### Misión: Necesitamos encontrar una cuenta de usuario de dominio o acceso de nivel SISTEMA
en un host unido al dominio. 

Vamos a profundizar en la búsqueda de cuenta de usuario:

## Identificando usuarios
Necesitaremos encontrar la manera de encontrar una foothold en el dominio mediante la obtención de credenciales de texto sin cifrar o un hash de contraseña NTLM para un usuario, un shell de SISTEMA en un host unido al dominio o un shell en el contexto de una cuenta de usuario del dominio. 

Obtener un usuario válido con credenciales es fundamental en las primeras etapas de una prueba de penetración interna. Este acceso (incluso en el nivel más bajo) abre muchas oportunidades para realizar enumeraciones e incluso ataques. 

Veamos una forma de comenzar a recopilar una lista de usuarios válidos en un dominio para usarla más adelante en nuestra evaluación:

### Kerbrute - Internal AD Username Enumeration

Kerbrute puede ser una opción para enumerar usuarios.

Usaremos Kerbrute junto con las listas de usuarios jsmith.txt o jsmith2.txt de Insidetrust:
>https://github.com/insidetrust/statistically-likely-usernames/blob/master/jjsmith.txt

Podemos dirigir Kerbrute al controlador de dominio que encontramos anteriormente y proporcionarle una lista de palabras. La herramienta es rápida y nos proporcionará resultados que nos permitirán saber si las cuentas encontradas son válidas o no, lo cual es un excelente punto de partida para lanzar ataques como el robo de contraseñas.

Para empezar a usar Kerbrute, podemos descargar binarios precompilados para realizar pruebas desde Linux, Windows y Mac, o podemos compilarlo nosotros mismos. Esta suele ser la mejor práctica para cualquier herramienta que implementemos en un entorno cliente. Para compilar los binarios que utilizaremos en el sistema que elijamos, primero clonamos el repositorio:


Nos clonamos el repositorio y lo compilamos con "make":
```bash
sudo git clone https://github.com/ropnop/kerbrute.git


```
Con el comando "make help" veremos todas las opciones de compilaciones posibles: 
```shell-session
Polika4RM@htb[/htb]$ make help

help:            Show this help.
windows:  Make Windows x86 and x64 Binaries
linux:  Make Linux x86 and x64 Binaries
mac:  Make Darwin (Mac) x86 and x64 Binaries
clean:  Delete any binaries
all:  Make Windows, Linux and Mac x86/x64 Binaries
```

Para compilar todas las arquitecturas existentes contra las que podremos atacar (linux, windows, darwin, amd), usaremos sudo make all

```bash
sudo make all         
```

Nos creará un directorio llamado "dist" que contendrá
```bash
Polika4RM@htb[/htb]$ ls dist/

kerbrute_darwin_amd64  kerbrute_linux_386  kerbrute_linux_amd64  kerbrute_windows_386.exe  kerbrute_windows_amd64.exe
```

Testeamos que funcione: 
```shell-session
Polika4RM@htb[/htb]$ ./kerbrute_linux_amd64 
```

Y la añadimos al Path para tenerlo a mano siempre: 
```shell-session
Polika4RM@htb[/htb]$ sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

## Enumerando usuarios con Kerbrute
Ejecutaremos:
```shell-session
Polika4RM@htb[/htb]$ kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

Y nos valida un conjunto de 56 usuarios:
```shell-session
Polika4RM@htb[/htb]$ kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users

2021/11/17 23:01:46 >  Using KDC(s):
2021/11/17 23:01:46 >   172.16.5.5:88
2021/11/17 23:01:46 >  [+] VALID USERNAME:       jjones@INLANEFREIGHT.LOCAL
2021/11/17 23:01:46 >  [+] VALID USERNAME:       sbrown@INLANEFREIGHT.LOCAL
2021/11/17 23:01:46 >  [+] VALID USERNAME:       tjohnson@INLANEFREIGHT.LOCAL
2021/11/17 23:01:50 >  [+] VALID USERNAME:       evalentin@INLANEFREIGHT.LOCAL

 <SNIP>
 
2021/11/17 23:01:51 >  [+] VALID USERNAME:       sgage@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       jshay@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       jhermann@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       whouse@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       emercer@INLANEFREIGHT.LOCAL
2021/11/17 23:01:52 >  [+] VALID USERNAME:       wshepherd@INLANEFREIGHT.LOCAL
2021/11/17 23:01:56 >  Done! Tested 48705 usernames (56 valid) in 9.940 seconds
```


## Identificando posibles vulnerabilidades

**Qué es la cuenta SYSTEM**
- `NT AUTHORITY\SYSTEM` es la cuenta integrada con **máximos privilegios** en Windows; muchos servicios (incluidos de terceros) se ejecutan con ella.
- En un equipo unido a dominio, una cuenta SYSTEM puede **suplantar la cuenta del equipo** y **enumerar Active Directory**, lo que equivale prácticamente a tener una cuenta de dominio.

**Formas comunes de obtener SYSTEM**
- Explotación remota de vulnerabilidades (ej. MS08-067, EternalBlue, BlueKeep).
- Abuso de servicios que corren como SYSTEM o abusos de `SeImpersonate` (herramientas tipo _Juicy Potato_ — eficacia dependiente de la versión de Windows).
- Escalada de privilegios local por vulnerabilidades (ej. fallos del Programador de tareas).
- Conseguir admin local en un host de dominio y usar herramientas como `psexec` para abrir shell como SYSTEM.

**Qué puede hacer un atacante con SYSTEM**
- **Enumerar el dominio** (herramientas integradas o ofensivas: PowerView, BloodHound).
- Realizar ataques como **Kerberoasting / ASREPRoasting**.
- Capturar/rellevar hashes (p. ej. Net-NTLMv2) y hacer ataques de retransmisión SMB (herramientas como Inveigh).
- **Suplantación de tokens** para secuestrar cuentas de dominio con privilegios.
- Atacar **listas de control de acceso (ACLs)** y otras configuraciones.

**Precauciones durante pruebas**
- Elegir herramientas según el **alcance**: pruebas ruidosas (no evasivas) vs. pruebas evasivas/evaluación adversarial (red team).
- En evaluaciones evasivas, **priorizar sigilo**: muchas herramientas (p. ej. escaneos masivos con Nmap) activan SOC/blue team.
- **Acordar por escrito** los objetivos y límites con el cliente antes de comenzar.

**Siguiente fase (breve)**
- Buscar cuentas de usuario de dominio mediante técnicas como **envenenamiento LLMNR/NBT-NS** y **password spraying** — efectivas para establecer punto de apoyo, pero requieren cuidado y conocimiento de las herramientas.


---
**Target: 10.129.208.9**
**SSH to 10.129.208.9 with user "htb-student" and password "HTB_@cademy_stdnt!"**
**1. From your scans, what is the "commonName" of host 172.16.5.5 ?**

Nos conectamos por RDP a la máquina "htb-student" con:
```
xfreerdp /u:htb-student /v:10.129.208.9 /p:HTB_@cademy_stdnt!
```

Ejecutando un script básico en nmap:

```
sudo nmap -v -A 172.16.5.5 
```

Me devuelve un campo que indica:
```
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL, DNS:INLANEFREIGHT.LOCAL, DNS:INLANEFREIGHT
| Issuer: commonName=INLANEFREIGHT-CA
```

Siendo la respuesta:  "ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL"

**2. What host is running "Microsoft SQL Server 2019 15.00.2000.00"? (IP address, not Resolved name)**
Ejecuto un:

```
fping -asgq 172.16.5.0/23
172.16.5.5
172.16.5.130
172.16.5.225

     510 targets
       3 alive
     507 unreachable
       0 unknown addresses

    2028 timeouts (waiting for response)
    2031 ICMP Echos sent
       3 ICMP Echo Replies received
    2028 other ICMP received

 0.070 ms (min round trip time)
 0.501 ms (avg round trip time)
 0.747 ms (max round trip time)
       14.704 sec (elapsed real time)
```

La IP 172.16.5.225 soy yo, pues la descarto.

Ejecuto:
```
nmap -sV 172.16.5.130 172.16.5.5
```

Y observo:

``` 
nmap -sV 172.16.5.130 172.16.5.5                                           
Starting Nmap 7.92 ( https://nmap.org ) at 2025-10-28 12:19 EDT            
Nmap scan report for 172.16.5.130                                          <------------------------------
Host is up (0.061s latency).
Not shown: 992 closed tcp ports (conn-refused)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
808/tcp   open  ccproxy-http?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000         <------------------------------
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
16001/tcp open  mc-nmf        .NET Message Framing
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for inlanefreight.local (172.16.5.5)
Host is up (0.061s latency).
Not shown: 988 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-28 16:19:26Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: Host: ACADEMY-EA-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 2 IP addresses (2 hosts up) scanned in 96.45 seconds
```

Siendo la respuesta al ejercicio: 172.16.5.130