El Port Forwarding es una técnica que nos permite redireccionar una petición de comunicación de un puerto a otro.
Normalmente se utilizan puertos TCP.

Sin embargo, se pueden utilizar diferentes protocolos de capa de aplicación, como SSH o incluso SOCKS (capa no relacionada con la aplicación), para encapsular el tráfico reenviado. 
En vez de enviar directamente una conexión por un puerto “normal” (p. ej. abrir un puerto TCP para un servicio), se encapsula ese tráfico dentro de otra conexión que la red ya permite — por ejemplo una sesión SSH o un proxy SOCKS — y así el tráfico original viaja dentro de ese «túnel».  
Eso hace que el tráfico parezca tráfico legítimo al firewall y, por tanto, pueda pasar.

**SOCKS**: es un tipo de proxy genérico. No está hecho para una aplicación concreta (por eso a veces se describe como “no relacionado con la aplicación”): acepta conexiones y retransmite prácticamente cualquier tipo de tráfico (HTTP, FTP, juegos, etc.) a través del proxy.

---

## SSH Local Port Forwarding

Veamos un ejemplo:
- El atacante abre (localmente) el puerto **1234** en su máquina y configura el túnel SSH con destino al servidor víctima.
    - Conceptualmente pide: `localhost:1234` → túnel SSH → `victima:localhost:3306`.
- La conexión se establece sobre **SSH (puerto 22)** entre el Attack Host y el Victim Server. Todo el tráfico del túnel viaja dentro de esa sesión SSH.
    - Aquí **no** se reescribe el puerto 22: SSH simplemente transporta datos entre las dos máquinas.
- Cuando el atacante conecta a **localhost:1234** en su máquina, esos bytes viajan cifrados por la sesión SSH (puerto 22) hasta la máquina víctima.
- En la víctima, SSH entrega ese tráfico al servicio destino, que es **localhost:3306** (MySQL). MySQL lo recibe como si viniera localmente en su puerto 3306.
El puerto 1234 del host atacante queda "tunneled" con el puerto 3306 del Victim Server. 

- **Número de “cambios de puerto” lógicos:** **1** (1234 → 3306).
- **Puertos implicados en toda la conexión:** **3** (1234, 22, 3306), porque SSH usa el puerto 22 para transportar el túnel.

![[11.webp]]

Nota: Cada diagrama de red presentado en este módulo está diseñado para ilustrar los conceptos tratados en la sección correspondiente. El direccionamiento IP que se muestra en los diagramas no siempre coincide exactamente con los entornos de laboratorio que se mostrarán a continuación. Asegúrese de concentrarse en comprender el concepto y descubrirá que los diagramas le resultarán muy útiles. Después de leer esta sección, consulte la imagen anterior para reforzar los conceptos.

#### Escaneando el "Pivot Target"

Al hacerle un nmap al Pivot Target, observamos que SOLO el puerto SSH (22) está abierto:
```shell-session
Polika4RM@htb[/htb]$ nmap -sT -p22,3306 10.129.202.64

Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-24 12:12 EST
Nmap scan report for 10.129.202.64
Host is up (0.12s latency).

PORT     STATE  SERVICE
22/tcp   open   ssh
3306/tcp closed mysql

Nmap done: 1 IP address (1 host up) scanned in 0.68 seconds
```


Para acceder al servicio MySQL, podemos: 
- Acceder al servidor por SSH desde Ubuntu: 
	  Eso significa que desde la máquina atacante no puedo abrir una conexión directa a victim:3306 porque MySQL no está expuesto hacia fuera ni por el firewall ni por la configuración. Limitación: sólo desde la propia máquina Ubuntu puedo acceder a localhost:3306. Mi máquina atacante NO puede directamente abrir `victim:3306` porque MySQL escucha solo en `localhost` del servidor.

- o puedo redirigirlo a nuestro host local en el puerto 1234 y acceder localmente:
	  Una ventaja de acceder localmente es que, si queremos ejecutar un exploit remoto en el servicio MySQL, no podremos hacerlo sin redirección de puertos. Esto se debe a que MySQL está alojado localmente en el servidor Ubuntu, en el puerto 3306. Por lo tanto, usaremos el siguiente comando para redirigir nuestro puerto local (1234) por SSH al servidor Ubuntu:

#### Executing the Local Port Forward

El comando -L indica al cliente SSH que solicite al servidor SSH que reenvíe todos los datos que enviamos a través del puerto 1234 a localhost:3306 en el servidor Ubuntu. De esta forma, podremos acceder al servicio MySQL localmente en el puerto 1234. Podemos usar Netstat o Nmap para consultar nuestro host local en el puerto 1234 y verificar si el servicio MySQL se reenvió.

```shell-session
Polika4RM@htb[/htb]$ ssh -L 1234:localhost:3306 ubuntu@10.129.202.64

ubuntu@10.129.202.64's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 24 Feb 2022 05:23:20 PM UTC

  System load:             0.0
  Usage of /:              28.4% of 13.72GB
  Memory usage:            34%
  Swap usage:              0%
  Processes:               175
  Users logged in:         1
  IPv4 address for ens192: 10.129.202.64
  IPv6 address for ens192: dead:beef::250:56ff:feb9:52eb
  IPv4 address for ens224: 172.16.5.129

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

66 updates can be applied immediately.
45 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable
```

#### Confirmando el "Port Forward" con Netstat

El siguiente comando (a ejecutar desde el atacante) me muestra:
- Una escucha en el localhost (127.0.0.1 // ::1) tanto para IPV4 como para IPV6 por el puerto 1234:
```shell-session
Polika4RM@htb[/htb]$ netstat -antp | grep 1234

(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:1234          0.0.0.0:*               LISTEN      4034/ssh            
tcp6       0      0 ::1:1234                :::*                    LISTEN      4034/ssh   
```

### Confirmación del Port Forward con Nmap
Seguidamente, ejecutaremos desde el atacante: 
```shell-session
Polika4RM@htb[/htb]$ nmap -v -sV -p1234 localhost
```

Devolviéndonos como efectivamente se ha añadido un puerto "1234" abierto por parte de la víctima con el servicio mysql corriendo detrás:
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-24 12:18 EST
NSE: Loaded 45 scripts for scanning.
Initiating Ping Scan at 12:18
Scanning localhost (127.0.0.1) [2 ports]
Completed Ping Scan at 12:18, 0.01s elapsed (1 total hosts)
Initiating Connect Scan at 12:18
Scanning localhost (127.0.0.1) [1 port]
Discovered open port 1234/tcp on 127.0.0.1
Completed Connect Scan at 12:18, 0.01s elapsed (1 total ports)
Initiating Service scan at 12:18
Scanning 1 service on localhost (127.0.0.1)
Completed Service scan at 12:18, 0.12s elapsed (1 service on 1 host)
NSE: Script scanning 127.0.0.1.
Initiating NSE at 12:18
Completed NSE at 12:18, 0.01s elapsed
Initiating NSE at 12:18
Completed NSE at 12:18, 0.00s elapsed
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0080s latency).
Other addresses for localhost (not scanned): ::1

PORT     STATE SERVICE VERSION
1234/tcp open  mysql   MySQL 8.0.28-0ubuntu0.20.04.3

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.18 seconds
```

Si quisiésemos redireccionar varios puertos, podríamos hacerlo repitiendo la sintaxis "local port:server:port":
```shell-session
Polika4RM@htb[/htb]$ ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```


---
## Configuración para pivotar
Ahora, si escribo "ifconfig" en el host de Ubuntu, veré que este servidor tiene varias tarjetas de red (NIC):
- Una conectada a nuestro host de ataque (ens192); Porque la red 10.129.xxx.xxx
- Una que se comunica con otros hosts dentro de una red diferente (ens224);
- La interfaz de bucle invertido (lo).

## Pivoting mediante "ifconfig"


```shell-session
ubuntu@WEB01:~$ ifconfig 

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.202.64  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 dead:beef::250:56ff:feb9:52eb  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:52eb  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:52:eb  txqueuelen 1000  (Ethernet)
        RX packets 35571  bytes 177919049 (177.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10452  bytes 1474767 (1.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens224: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.5.129  netmask 255.255.254.0  broadcast 172.16.5.255
        inet6 fe80::250:56ff:feb9:a9aa  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:a9:aa  txqueuelen 1000  (Ethernet)
        RX packets 8251  bytes 1125190 (1.1 MB)
        RX errors 0  dropped 40  overruns 0  frame 0
        TX packets 1538  bytes 123584 (123.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 270  bytes 22432 (22.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 270  bytes 22432 (22.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```




A diferencia de nuestro escenario anterior (sobre el cual sabíamos a qué puerto acceder), en el **escenario actual** desconozco que servicios se encuentran al otro lado de la red.

Lo que se puede hace es escanear rangos más pequeños de IP en la red (172.16.5.1-200) o la subred completa (172.16.5.0/23).
Desde el atacante no se puede realizar este escaneo directo debido a que no tiene rutas a la red 172.16.5.0/23.
Para logar esto, se deberá hacer un reenvío dinámico de puertos y pivotar nuestros paquetes de red a través del servidor víctima UBUNTU. 

Podemos hacerlo iniciando un receptor de SOCKS en nuestro local host (atacante) y luego configurando SSH para reenviar ese tráfico por SSH a la red (172.16.5.0/23) tras conectarnos al HOST víctima.

Un Proxy SOCKS local es un servidor que corre escuchando típicamente en (127.0.0.1) y en el puerto 1080 (por ejemplo). Cuando una aplicación se conecta a ese puerto local, el proxy toma el tráfico y lo reenvía hacia el destina a través del servidor/proveedor que hace de salida (tunel SSH en nuestro caso). 

Este concepto se denomina tunelización SSH sobre proxy SOCKS.
SOCKS = Socket Secure:
	Protocolo que facilita la comunicación con servidores con restricciones de firewall.
A diferencia de la mayoría de los casos donde se inicia una conexión para acceder a un servicio, en el caso de SOCKS, el tráfico inicial lo genera un **cliente** SOCKS, que se conecta al servidor SOCKS controlado por el usuario que desea acceder a un servicio en el lado del cliente. 
Una vez establecida la conexión, el tráfico de red se puede enrutar a través del servidor SOCKS en nombre del cliente conectado .


Utilizar PROXY SOCKS para pivotar es útil debido a que:
- Elude restricciones de firewall.
- Pueden pivotar creando una ruta a un servidor externo desde redes NAT:
	  Cuando una máquina está detrás de NAT normalmente no puede aceptar conexiones entrantes desde Internet, pero sí puede abrir conexiones salientes. Si esa máquina abre una conexión saliente (por ejemplo, una sesión SSH) hacia un servidor externo, esa sesión puede usarse como túnel para que el servidor externo (o tú) envíe/reciba tráfico a través de la máquina detrás del NAT. Es usar la conexión saliente para permitir comunicación que de otro modo estaría bloqueada.



Tomemos como ejemplo la imagen a continuación, donde tenemos una red NAT de 172.16.5.0/23, a la que no podemos acceder directamente:

![[22.webp]]

Tenemos dos máquina involucradas: 
- **Atacante:**
	  IP: 10.10.15.5. 
	  Quien ejecuta el cliente SSH, levanta el SOCKS local en localhost:9050 y ejecuta proxychains.
- **Víctima:**
	  IP: 10.129.15.50 (pública // enlace) + 172.16.5.129 (interfaz interna).
	  En esta máquina corre SSH por 0.0.0.0:22 y actúa como punto de salida hacia la red interna 172.16.5.0/23.


Proceso:
1. Desde el atacante (10.10.15.5) lanzo un cliente SSH solicitando un "Dynamic Por Forward (SOCKS)" que se alojará en mi localhost:9050. 
2. El cliente SSH (atacante) crea y escucha por 127.0.0.1:9050. Este socket local es mi proxy SOCKS.
3. Todas las conexiones que mi herramienta "proxychains" envía a 127.0.0.1:9050, se encapsulan en la sesión SSH hacia la víctima (10.129.15.50) mediante el puerto 22 (debido a que es un servicio SSH).
4. La víctima recibe ese tráfico y establece las conexiones hacia los HOSTS/SERVICIOS dentro de la red interna.

Es necesario utilizar PROXYCHAINS porque muchas herramientas (como nmap) no soportan proxies SOCKS de forma nativa, y PROXYCHAINS lo que hace es forzar a dichas herramientas a enviar su tráfico a través de un PROXY SOCKS definido (127.0.01:9050) sin que la aplicación tenga que saber nada del proxy.
## Habilitación del reenvío dinámico de puertos con SSH
El siguiente comando se ejecuta desde el atacante. 

El argumento -D solicita al servidor SSH que habilite el reenvío dinámico de puertos:
```shell-session
Polika4RM@htb[/htb]$ ssh -D 9050 ubuntu@10.129.202.64
```
Una vez habilitado, necesitaré una herramienta que pueda enrutar los paquetes de cualquier herramienta (como nmap) a través del puerto 9050.
Podemos utilizar pues la herramienta "proxychains", que puede redirigir conexiones TCP a través de servidores proxy SOCKS. 


#### Checking /etc/proxychains.conf
Para informar a proxychains que debemos usar el puerto 9050, debemos modificar el archivo de configuración de proxychains, ubicado en /etc/proxychains.conf. Podemos agregar "socks4 127.0.0.1 9050" en la última línea si aún no está allí.

```shell-session
Polika4RM@htb[/htb]$ tail -4 /etc/proxychains.conf

# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

#### Using Nmap with Proxychains
Ahora que tenemos el proxychain correctamente configurado para que escuche por tal puerto, y tenemos establecida la conexión SSH, deberíamos poder indicar a las herramientas internas que la paquetería saliente la envíe al proxy SOCKS.
Cuando inicia "nmap" con proxychains, con el siguiente comando podré enrutar todos los paquetes al puerto local 9050, que es donde está escuchando nuestro cliente SSH el cual renviará todos los paquetes por SSH a la red 172.16.5.0/23.

```shell-session
Polika4RM@htb[/htb]$ proxychains nmap -v -sn 172.16.5.1-200

ProxyChains-3.1 (http://proxychains.sf.net)

Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-24 12:30 EST
Initiating Ping Scan at 12:30
Scanning 10 hosts [2 ports/host]
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.2:80-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.5:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.6:80-<--timeout
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0

<SNIP>
```

Hay dos limitaciones importantes a tener en cuenta: 
1. Al usar proxychains sólo es fiable realizar escaneos de TCP connect (-sT), porque proxychains no manipula ni entiende paquetes parciales o raw; los escaneos que envían fragmentos (por ejemplo SYN scan -sS) devolverán resultados erróneos. 
2. Las comprobaciones de "host alive" basadas en ICMP pueden fallar —especialmente contra sistemas Windows, donde el firewall suele bloquear los pings— por lo que en entornos así conviene usar la opción -Pn para suprimir los pings y forzar el escaneo directo.

---

### nmap y Proxychains

El siguiente escaneo de nmap muestra varios puertos abiertos, entre ellos el TCP3389 correspondiente a un servicio RDP:

```shell-session
Polika4RM@htb[/htb]$ proxychains nmap -v -Pn -sT 172.16.5.19

ProxyChains-3.1 (http://proxychains.sf.net)
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-24 12:33 EST
Initiating Parallel DNS resolution of 1 host. at 12:33
Completed Parallel DNS resolution of 1 host. at 12:33, 0.15s elapsed
Initiating Connect Scan at 12:33
Scanning 172.16.5.19 [1000 ports]
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:1720-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:445-<><>-OK
Discovered open port 445/tcp on 172.16.5.19
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:8080-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:23-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:135-<><>-OK
Discovered open port 135/tcp on 172.16.5.19
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:110-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:21-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:554-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:995-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Discovered open port 3389/tcp on 172.16.5.19
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:443-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:8888-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:139-<><>-OK
Discovered open port 139/tcp on 172.16.5.19
```

### Metasploit y proxychains
Podemos utilizar de la misma forma "msfconsole" con proxychains para utilizar módulos auxiliares de metasploit que busquen vulnerabilidades del RDP.
```shell-session
Polika4RM@htb[/htb]$ proxychains msfconsole

ProxyChains-3.1 (http://proxychains.sf.net)

       =[ metasploit v6.1.27-dev                          ]
+ -- --=[ 2196 exploits - 1162 auxiliary - 400 post       ]
+ -- --=[ 596 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: Adapter names can be used for IP params 
set LHOST eth0

msf6 > 
```

Vamos a usar el módulo auxiliar "rdp_scanner" para checkear si el host de la red interna está escuchando en el puerto 3389:
```shell-session
msf6 > search rdp_scanner

Matching Modules
================

   #  Name                               Disclosure Date  Rank    Check  Description
   -  ----                               ---------------  ----    -----  -----------
   0  auxiliary/scanner/rdp/rdp_scanner                   normal  No     Identify endpoints speaking the Remote Desktop Protocol (RDP)


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/rdp/rdp_scanner

msf6 > use 0
msf6 auxiliary(scanner/rdp/rdp_scanner) > set rhosts 172.16.5.19
rhosts => 172.16.5.19
msf6 auxiliary(scanner/rdp/rdp_scanner) > run
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK

[*] 172.16.5.19:3389      - Detected RDP on 172.16.5.19:3389      (name:DC01) (domain:DC01) (domain_fqdn:DC01) (server_fqdn:DC01) (os_version:10.0.17763) (Requires NLA: No)
[*] 172.16.5.19:3389      - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### xfreerdp y Proxychains
Finalmente, podremos ejecutar "xfreerdp" con el comando:
```shell-session
Polika4RM@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123

ProxyChains-3.1 (http://proxychains.sf.net)
[13:02:42:481] [4829:4830] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[13:02:42:482] [4829:4830] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[13:02:42:482] [4829:4830] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[13:02:42:482] [4829:4830] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
```

---

**Target(s): 10.129.232.53**
**SSH to  with user "ubuntu" and password "HTB_@cademy_stdnt!"**
**1. You have successfully captured credentials to an external facing Web Server. Connect to the target and list the network interfaces. How many network interfaces does the target web server have? (Including the loopback interface)**


En este primer ejercicio simplemente me conecto por "ssh" a la máquina víctima con las credenciales dadas y ejecuto un "ipconfig" o "ifconfig" en función de si sea Linux o Windows. El target se llama "ubuntu", pues será "ifconfig":

```
ubuntu@WEB01:~$ ifconfig
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.232.53  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 dead:beef::250:56ff:fe94:a12d  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:fe94:a12d  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:94:a1:2d  txqueuelen 1000  (Ethernet)
        RX packets 384  bytes 40807 (40.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 136  bytes 16514 (16.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens224: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.5.129  netmask 255.255.254.0  broadcast 172.16.5.255
        inet6 fe80::250:56ff:fe94:311d  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:94:31:1d  txqueuelen 1000  (Ethernet)
        RX packets 70  bytes 6138 (6.1 KB)
        RX errors 0  dropped 12  overruns 0  frame 0
        TX packets 32  bytes 2364 (2.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 222  bytes 17675 (17.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 222  bytes 17675 (17.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
En total exiten 3 interfaces.

**2. Apply the concepts taught in this section to pivot to the internal network and use RDP (credentials: victor:pass@123) to take control of the Windows target on 172.16.5.19. Submit the contents of Flag.txt located on the Desktop.

Ejecuto un escaneo inicial de los puertos, mostrándome que solo existen disponibles los:
```
nmap -sV 10.129.232.53
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-10-22 02:40 CDT
Nmap scan report for 10.129.232.53
Host is up (0.0040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

MAQUINA HOST (10.10.14.149)    <-------------->    MÁQUINA UBUNTU (10.129.232.53 + 172.16.5.19)

El servicio proxychains de mi máquina atacante está configurado para escuchar por el puerto 9050. Comprobado con:
```
tail -4 /etc/proxychains.conf 
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

Voy a levantar un proxy SOCKS local para redireccionar todos mis puertos a un túnel SSH concreto:
```
ssh -f -N -D 9050 ubuntu@10.129.232.53
```

Seguido a esto, puedo asegurarme que estoy escuchando por el 9050 con el comando:
```
netstat -antp | grep 9050
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:9050          0.0.0.0:*               LISTEN      18626/ssh           
tcp6       0      0 ::1:9050                :::*                    LISTEN      18626/ssh 
```

Ejecuto desde mi máquina atacante un escaneo de servicios hacia el target que está fuera de mi red (pero que a través de proxychains podré alcanzar).
IMPORTANTE:
- Tengo que añadir -sT:
	  para crear paquetes que no utilicen connect()
- Tengo que añadir -Pn:
	  para omitir el ping de vuelta de Host, de lo contrario, no establecerá conexion.

Ejecuto:
```
proxychains nmap -sVT 172.16.5.19 -Pn
```

Y me devuelve: 
```
PORT     STATE    SERVICE   VERSION
3385/tcp filtered qnxnetman
```

Ejecuto finalmente:
```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

Y me devuelve un RDP de windows, donde en el escritorio está la flag: N1c3Piv0t

