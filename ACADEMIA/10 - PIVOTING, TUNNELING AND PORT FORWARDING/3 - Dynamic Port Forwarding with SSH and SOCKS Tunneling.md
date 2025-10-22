El Port Forwarding es una técnica que nos permite redireccionar una petición de comunicación de un puerto a otro.
Normalmente se utilizan puertos TCP.

Sin embargo, se pueden utilizar diferentes protocolos de capa de aplicación, como SSH o incluso SOCKS (capa no relacionada con la aplicación), para encapsular el tráfico reenviado. Esto puede ser eficaz para eludir los firewalls y utilizar los servicios existentes en el host comprometido para migrar a otras redes.
Es decor, en vez de enviar directamente una conexión por un puerto “normal” (p. ej. abrir un puerto TCP para un servicio), se encapsula ese tráfico dentro de otra conexión que la red ya permite — por ejemplo una sesión SSH o un proxy SOCKS — y así el tráfico original viaja dentro de ese «túnel».  
Eso hace que el tráfico parezca tráfico legítimo al firewall y, por tanto, pueda pasar.

**SOCKS**: es un tipo de proxy genérico. No está hecho para una aplicación concreta (por eso a veces se describe como “no relacionado con la aplicación”): acepta conexiones y retransmite prácticamente cualquier tipo de tráfico (HTTP, FTP, juegos, etc.) a través del proxy.

---

## SSH Local Port Forwarding


Veamos un ejemplo:
- El atacante abre (localmente) el puerto **1234** en su máquina y configura el túnel SSH con destino al servidor víctima.
    - Conceptualmente pide: `localhost:1234` → túnel SSH → `victim:localhost:3306`.
- La conexión se establece sobre **SSH (puerto 22)** entre el Attack Host y el Victim Server. Todo el tráfico del túnel viaja dentro de esa sesión SSH.
    - Aquí **no** se reescribe el puerto 22: SSH simplemente transporta datos entre las dos máquinas.
- Cuando el atacante conecta a **localhost:1234** en su máquina, esos bytes viajan cifrados por la sesión SSH (puerto 22) hasta la máquina víctima.
- En la víctima, SSH entrega ese tráfico al servicio destino, que es **localhost:3306** (MySQL). MySQL lo recibe como si viniera localmente en su puerto 3306.
El puerto 1234 del host atacante queda conectado "tunneled" con el puerto 3306 del Victim Server. 


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

El comando -L indica al cliente SSH que solicite al servidor SSH que reenvíe todos los datos que enviamos a través del puerto 1234 a localhost:3306 en el servidor Ubuntu. De esta forma, podremos acceder al servicio MySQL localmente en el puerto 1234. Podemos usar Netstat o Nmap para consultar nuestro host local en el puerto 1234 y verificar si el servicio MySQL se reenvió.

#### Confirmando el "Port Forward" con Netstat

El siguiente comando (a ejecutar desde el atacante) me muestra:
- Una escucha en el localhost (127.0.0.1 // ::1) tanto para IPV4 como para IPV6
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
Devolviéndonos como efectivamente se ha añadido un puerto "1234" abierto con el servicio mysql corriendo detrás:
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

## Configuración para pivotar
Ahora, si escribo "ifconfig" en el host de Ubuntu, veré que este servidor tiene varias tarjetas de red (NIC):
- Una conectada a nuestro host de ataque (ens192); Porque la red 10.129.xxx.xxx
- Una que se comunica con otros hosts dentro de una red diferente (ens224);
- La interfaz de bucle invertido (lo).

## Pivoting mediante ifconfig



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

No puedo acceder directamente a la red `172.16.5.0/23` desde mi máquina atacante porque no existe una ruta directa: mi atacante está en otra subred y, por tanto, no puede hablar con esos hosts como si estuviera dentro de la red. Sin embargo, sí puedo establecer una conexión con el servidor Ubuntu, y ese servidor **sí** tiene acceso a la red objetivo. 

- IP Atacante 10.10.15.5
- IP UBUNTU  10.129.15.50 // 172.16.5.129
Que dos máquinas tengan IPs en subredes distintas **no impide** que se comuniquen: basta que haya **enrutamiento** (o NAT/VPN/overlay) entre esas subredes. Por eso desde `10.10.15.5` puedes hacer SSH a `10.129.15.50` aunque aparentemente estén en redes diferentes.

Posibles razones por las que puedes conectar
1. **Hay un router** entre ambas subredes que sabe cómo enrutar tráfico `10.10.x.x ⇄ 10.129.x.x`.
2. **NAT**: alguna máquina o appliance está traduciendo direcciones y te permite alcanzar la final.
3. **VPN / overlay de la plataforma (ej. HTB, laboratorio)**: la plataforma conecta internamente distintas subredes para los participantes.
4. **Rutas estáticas / gateways** configurados en los switches/hosts que habilitan la comunicación.

Aprovechando esa conexión se puede usar al servidor como un punto intermedio: en lugar de intentar conectar cada host de la red remota desde tu equipo, haces que el servidor Ubuntu actúe como un “relevo” que manda y recibe tráfico en tu nombre.

Para ello se utiliza un proxy SOCKS montado sobre el túnel SSH. El proxy SOCKS es un servicio que acepta conexiones desde tus programas y les reenvía el tráfico hacia direcciones externas, actuando como representante de tu máquina dentro de la red remota. Al encaminar ese proxy por la conexión SSH hacia el servidor Ubuntu, todo el tráfico que envíen tus herramientas locales pasa cifrado por la sesión SSH, llega al servidor y desde ahí se dirige a los hosts de la red `172.16.5.0/23`. Desde la perspectiva de las aplicaciones en tu equipo, parece que se están comunicando directamente con máquinas internas, cuando en realidad las peticiones viajan primero hasta el servidor y este las entrega a la red interna.

Esta técnica es especialmente útil cuando no sabes qué servicios hay dentro de la red remota y necesitas explorar o probar distintos destinos: en vez de crear túneles puntuales para cada puerto conocido, un proxy SOCKS permite enviar tráfico arbitrario (HTTP, SSH, escáneres, etc.) a través del host intermedio. Hay dos versiones comunes del protocolo SOCKS: SOCKS4, más básico y solo TCP, y SOCKS5, que añade opciones como autenticación y soporte para UDP. En todos los casos, lo imprescindible es que puedas establecer la conexión SSH al servidor intermedio; si esa conexión no existe, no hay forma de usarlo como puente.

![[22.webp]]En la imagen superior, el host de ataque inicia el cliente SSH y solicita al servidor SSH que le permita enviar datos TCP a través del socket SSH. El servidor SSH responde con una confirmación y el cliente SSH comienza a escuchar en localhost:9050. Los datos que envíe se difundirán a toda la red (172.16.5.0/23) a través de SSH. Podemos usar el siguiente comando para realizar este reenvío dinámico de puertos:

## Habilitando el "Dynamic Port Forwarding" con SSH
```shell-session
Polika4RM@htb[/htb]$ ssh -D 9050 ubuntu@10.129.202.64
```

El argumento -D solicita al servidor SSH que habilite el reenvío dinámico de puertos. Una vez habilitado, necesitaremos una herramienta que pueda enrutar los paquetes de cualquier herramienta a través del puerto 9050. Podemos hacerlo usando la herramienta proxychains, que puede redirigir conexiones TCP a través de servidores proxy TOR, SOCKS y HTTP/HTTPS, y también permite encadenar varios servidores proxy. Con proxychains, también podemos ocultar la dirección IP del host solicitante, ya que el host receptor solo verá la IP del host pivote. Proxychains se usa a menudo para forzar que el tráfico TCP de una aplicación pase por proxies alojados como SOCKS4/SOCKS5, TOR o HTTP/HTTPS.