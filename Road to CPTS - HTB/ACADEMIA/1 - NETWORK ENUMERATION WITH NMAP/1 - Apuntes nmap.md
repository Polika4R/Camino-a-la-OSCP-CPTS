## HANDSHAKE
El **handshake de tres vías** (three-way handshake) es el proceso mediante el cual se establece una conexión TCP entre un cliente y un servidor. 

Se compone de tres pasos:
1. SYN (SYNCRONIZE)
	1. El cliente envío un paquete TCP con flag SYN activado.
	2. Indica que quiere una conexión y indica un número de secuencia inicial.
2. SYN - ACK
	1. El servidor responde con un paquete que tiene los flags SYN y ACK activados.
	2. Acepta la conexión (ACK) y también envía su propio número de secuencia inicial (SYN).
3. ACK
	1. El cliente envía un paquete con el flag ACK activado.
	2. Confirma que recibió correctamente el SYN-ACK del servidor.
4. Si el puerto está cerrado el host responde con RST-ACK


## -sT vs -sS
- **-sS (SYN scan):**
	- Se ejecuta por defeto si somos root.
	- Requiere root porque necesita crear paquetes TCP manualmente (construir paquetes SYN a bajo nivel) y enviarlos sin usar el sistema operativo para completar la conexión. Esto solo es posible con raw sockets, que están restringidos a usuarios con privilegios (root) por razones de seguridad.)
	- Rápido y sigiloso
	- No completa la conexión (solo **SYN → SYN-ACK → RST).**

- **-sT (TCP connect scan):**
	- NO Requiere root.
	- Lento y ruidoso
	- Hace el handshake completo (SYN → SYN-ACK → ACK).

## --disable-arp-ping
En una red local (misma subred), cuando haces un escaneo, nmap intenta averiguar qué hosts están activos enviando peticiones ARP (Address Resolution Protocol) para ver si alguien responde. Esto es muy fiable en redes locales, ya que todos los dispositivos deben responder a ARP si están activos.

**¿Qué hace --disable-arp-ping?**
Fuerza a Nmap a no enviar peticiones ARP, incluso si está escaneando una IP de su misma red local. En cambio, usa los métodos que tú le especifiques (por ejemplo, SYN scan, TCP connect, etc.).
Por defecto, si Nmap detecta que la IP objetivo está en la misma subred (por ejemplo, 10.129.2.28 si tú estás en 10.129.x.x), puede usar **ARP en vez de un ping normal (ICMP)**.

Sin aplicar `--disable-arp-ping` vería:
```
>sudo nmap 10.129.2.28 -p 21 --packet-trace -n 

SENT (0.0010s) ARP who-has 10.129.2.28 tell 10.129.2.1
RCVD (0.0025s) ARP reply 10.129.2.28 is-at de:ad:be:ef:00:01
SENT (0.0030s) TCP 10.129.2.1:56789 > 10.129.2.28:21 S ...
RCVD (0.0045s) TCP 10.129.2.28:21 > 10.129.2.1:56789 RA ...
```

Nmap primero intenta **descubrir si el host está activo** usando ARP (si estás en la misma LAN).
Vería en wireshark:
```
Who has 192.168.1.10? Tell 192.168.1.X  ← (paquete ARP request)
192.168.1.10 is at aa:bb:cc:dd:ee:ff   ← (respuesta ARP)

```
Esto ocurre antes de que empiecen los SYN.

**Con --disable-arp-ping**:
Nmap no envía paquetes ARP, y pasa directamente a enviar los paquetes del escaneo SYN.
Vería en wireshark:
```
192.168.1.X → 192.168.1.10: TCP SYN to port 21
192.168.1.10 → 192.168.1.X: TCP RST (si el puerto está cerrado)
```

#### Ventajas de desactivar ARP en pruebas
- **Tráfico más limpio**: Solo ves los paquetes TCP relevantes para el escaneo.
- **Ideal para analizar escaneos SYN** con herramientas como **Wireshark**.
- **Útil en laboratorios** donde ya sabes que el host está activo.

|Flag usado|¿Hace ARP Ping?|¿Hace ICMP Ping?|¿Escanea?|
|---|---|---|---|
|Sin `-Pn`, sin `--disable-arp-ping`|✅ Sí (si está en la misma red)|✅ Sí|✅ Sí|
|Con `-Pn`|❌ No|❌ No|✅ Sí|
|Con `--disable-arp-ping`|❌ No|✅ (si no hay `-Pn`)|✅ Sí|

## Modo  escaneo -sS
En esta prueba, recordamos que estamos haciendo un escaneo -sS, porque lo estamos haciendo como root y no estamos indicando lo contrario (-sT). Pues, con --disable-arp-ping veremos de una manera más clara las etapas de SYN -   SYN/ACK - RST.     Recordamos que hace RST porque el modo de escaneo -sS (SYN SCAN) **NO** acaba con un (SYN → SYN-ACK → ACK), sino con un (SYN → SYN-ACK → RST (no responde))

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:39 CEST
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.11.1.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

En la línea SENT, podemos ver que (10.10.14.2) enviamos un paquete TCP con el indicador SYN (S) a nuestro destino (10.129.2.28). En la siguiente línea RCVD, vemos que el destino responde con un paquete TCP que contiene los indicadores RST y ACK (RA).
El recibir una respuesta de RA (RST y ACK) significa que el puerto está cerrado.

Si me hubiese respondido con SYN/ACK, yo hubiese enviado un RST (en la modalidad -sS).
## Modo  escaneo -sT (Connect Scan)

El escaneo de conexión TCP de Nmap (-sT) utiliza el protocolo TCP de three-way handshake para  determinar si un puerto específico en un host de destino está abierto o cerrado. 

El escaneo envía un paquete SYN al puerto de destino y espera una respuesta. Se considera abierto si el puerto de destino responde con un paquete SYN-ACK y cerrado si responde con un paquete RST.

El escaneo de conexión (también conocido como escaneo de conexión TCP completo) es muy preciso porque completa el protocolo de enlace TCP de tres vías, lo que permite determinar el estado exacto de un puerto (abierto, cerrado o filtrado). 

Ojo! Tanto el escaneo -sS como el -sT tienen envío inicial por mi parte de (SYN) y el servidor responde con SYN/ACK en caso de que esté abierto, o con ACK/RST en caso de que esté cerrado. La respuesta del Host es la misma en ambos casos. Con el escaneo -sT, el handshake se establece de manera completa, pues, esto evita falsos positivos.

Sin embargo un escaneo -sT no es el más sigiloso. De hecho, el escaneo de conexión es una de las técnicas menos sigilosas, ya que establece una conexión completa, lo que genera registros en la mayoría de los sistemas y es fácilmente detectado por las soluciones IDS/IPS modernas. Es útil si la precisión es prioritaria.

El escaneo `-sT` (Connect scan) utiliza la llamada estándar del sistema operativo llamada `connect()`, que establece una conexión TCP completa con el puerto de destino. 
Esto significa que envía un paquete SYN, espera el SYN-ACK de respuesta y luego envía el ACK final, completando el handshake TCP como lo haría cualquier aplicación normal. 
Por eso, el escaneo `-sT` se comporta como una conexión legítima y suele ser aceptado por firewalls personales que bloquean paquetes sospechosos.

En cambio, el escaneo `-sS` (SYN scan) no utiliza `connect()`. En lugar de eso, construye manualmente paquetes TCP con la bandera SYN y los envía usando raw sockets, sin completar la conexión. Si recibe un SYN-ACK, responde con un RST para cerrar rápidamente la conexión antes de que se establezca. Esto hace que el escaneo `-sS` sea más sigiloso, pero también puede ser bloqueado por firewalls que detectan y descartan estos intentos a nivel de paquete.

------
En este ejemplo, forzamos un escaneo -sT  (connect scan()) con el siguiente comando.
Parar atención en el "CONN", que indica que se inicia una conexión completa.
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:26 CET
CONN (0.0385s) TCP localhost > 10.129.2.28:443 => Operation now in progress
CONN (0.0396s) TCP localhost > 10.129.2.28:443 => Connected
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```

----------------
## Puertos filtrados
Cuando un puerto se muestra como filtrado, puede deberse a varias razones:

Los firewalls tienen reglas definidas para gestionar conexiones específicas. Los paquetes pueden descartarse o rechazarse. Cuando se descarta un paquete, Nmap no recibe respuesta de nuestro objetivo y, por defecto, la tasa de reintentos (--max-retries) se establece en 10. Esto significa que Nmap reenviará la solicitud al puerto de destino para determinar si el paquete anterior se gestionó incorrectamente.

Veamos un ejemplo en el que el firewall descarta los paquetes TCP que enviamos para el escaneo de puertos. 
Escaneamos el puerto TCP 139, que ya se mostraba como filtrado. Para poder rastrear cómo se gestionan los paquetes enviados, desactivamos de nuevo las solicitudes de eco ICMP (-Pn), la resolución DNS (-n) y el escaneo de ping ARP (--disable-arp-ping).

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```
En el último escaneo, observamos que Nmap envió dos paquetes TCP con el indicador SYN. Por la duración del escaneo (2,06 s), podemos apreciar que tardó mucho más que los anteriores (~0,05 s). 


La situación es diferente si el firewall rechaza los paquetes. Para ello, analizamos el puerto TCP 
445, que se gestiona según dicha regla del firewall.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

Como respuesta, recibimos una respuesta ICMP de tipo 3 y código de error 3, que indica que el puerto deseado es inaccesible. Sin embargo, si sabemos que el host está activo, podemos asumir con seguridad que el firewall de este puerto está rechazando los paquetes, y tendremos que analizarlo con más detalle más adelante.

-----------
## Escaneo de puertos UDP
Algunos administradores de sistemas a veces olvidan filtrar los puertos UDP además de los TCP. 
Dado que UDP es un protocolo sin estado y no requiere un protocolo three-way handshake como el protocoloTCP, no recibimos ninguna confirmación. 
En consecuencia, el tiempo de espera es mucho mayor, lo que hace que el escaneo UDP (-sU) sea mucho más lento que el escaneo TCP (-sS).

- Puertos **cerrados** UDP suelen responder rápido con ICMP y el escaneo es rápido diciendo "puerto inalcanzable".
- Puertos **abiertos** UDP a menudo no responden.
- Puertos **filtrados** UDP no responden y hacen que el escaneo tarde mucho más, porque Nmap tiene que esperar varios timeouts y reintentos para concluirlo.

Veamos un ejemplo de cómo se ve un escaneo UDP (-sU) y qué resultados nos proporciona.
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -F -sU

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:01 CEST
Nmap scan report for 10.129.2.28
Host is up (0.059s latency).
Not shown: 95 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 98.07 seconds
```

Otra desventaja es que a menudo no recibimos respuesta porque Nmap envía datagramas vacíos a los puertos UDP escaneados y no recibimos respuesta. Por lo tanto, no podemos determinar si el paquete UDP ha llegado. Si el puerto UDP está abierto, solo recibimos respuesta si la aplicación está configurada para ello.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:15 CEST
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ttl=57 id=9122 iplen=78
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ttl=64 id=13222 iplen=257
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.0031s latency).

PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```


Si recibimos una respuesta ICMP con el código de error 3 (puerto inalcanzable), sabemos que el puerto está efectivamente cerrado.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:25 CEST
SENT (0.0445s) UDP 10.10.14.2:63825 > 10.129.2.28:100 ttl=57 id=29925 iplen=28
RCVD (0.1498s) ICMP [10.129.2.28 > 10.10.14.2 Port unreachable (type=3/code=3) ] IP [ttl=64 id=11903 iplen=56 ]
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.11s latency).

PORT    STATE  SERVICE REASON
100/udp closed unknown port-unreach ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in  0.15 seconds
```


Para todas las demás respuestas ICMP**, los puertos escaneados se marcan como (abiertos|filtrados).
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:32 CEST
SENT (0.0380s) UDP 10.10.14.2:52341 > 10.129.2.28:138 ttl=50 id=65159 iplen=28
SENT (1.0392s) UDP 10.10.14.2:52342 > 10.129.2.28:138 ttl=40 id=24444 iplen=28
Nmap scan report for 10.129.2.28
Host is up, received user-set.

PORT    STATE         SERVICE     REASON
138/udp open|filtered netbios-dgm no-response
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```


# Comandos útiles

| Comando                                                                  | Uso                                                                                                                                                                                                                                                                                  | Ejemplo / Uso                                                                                                                                  |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| sudo nmap <ip_red>/24 -sn <br>-oA tnet                                   | Detecta hosts activos. Realiza ping scan sin escanear puertos (-sn). Guarda la salida en tres formatos: .nmap (salida normal); .gnmap (formato grepeable) .xml (formato XML para herramientas)                                                                                       |                                                                                                                                                |
| sudo nmap -sn <br>-oA tnet -iL <lista.lst>                               | Partiendo de un listado de IP contenido en hosts.lst, escanea los hosts activos presentes en dicha lista.                                                                                                                                                                            | sudo nmap -sn <br>-oA tnet -iL hosts.lst                                                                                                       |
| sudo nmap -sn -oA tnet \<ip1> \<ip2> \<ip3>                              | Escaneo de los **hosts activos** de una pequeña parte de la red.                                                                                                                                                                                                                     | sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20                                                                                     |
| sudo nmap -sn -oA tnet 1.2.3.4-20                                        | Escaneo de los **hosts activos** de un rango de IP dado                                                                                                                                                                                                                              | sudo nmap -sn -oA tnet 10.129.2.18-20                                                                                                          |
| sudo nmap 1.2.3.4 -sn -oA host                                           | Escaneo de **1 host activo único**                                                                                                                                                                                                                                                   | sudo nmap 10.129.2.18 -sn -oA host                                                                                                             |
| sudo nmap 1.2.3.4 <br>-sn -oA host -PE <br>--packet-trace<br>            | Escaneo de **1 host activo** forzando el ping con ICMP Echo Request (-PE) y mostrando el detalle de todos los paquetes enviados y recibidos durante el escaneo (--packet-trace). Útil para diagnóstico y análisis profundo del tráfico de red. Guarda resultados en varios formatos. | También usa ARP ping por defecto, salvo que no pueda usarlo (LAN).                                                                             |
| sudo nmap 1.2.3.4 -sn -oA host -PE --packet-trace --disable-arp-ping<br> | Realiza un ping scan para detectar si el host está activo usando **solo ICMP Echo Request** (ping normal), desactivando el ping ARP que Nmap usa por defecto en redes locales. Muestra en detalle los paquetes enviados y recibidos para analizar la comunicación de red.            | Útil para analizar la respuesta a ICMP puro, ideal para depurar filtrados o comportamiento de firewall y dispositivos que bloqueen ARP o ICMP. |
## Descubrimiento de puertos

| Comando                                                                  | Uso                                                                                                                                                                                                                                                                                                                   | Ejemplo / Uso                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| sudo nmap 1.2.3.4 --top-ports=10                                         | Escanea los 10 puertos más comunes del host 1.2.3.4                                                                                                                                                                                                                                                                   |                                                                                                                                                                                                                                                                                                            |
| sudo nmap 1.2.3.4 -p 22,25,80,139,445                                    | Escanea específicamente los puertos 22, 25, 80, 139 y 445 en el host 1.2.3.4                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                            |
| sudo nmap 1.2.3.4 -p-                                                    | Escanea todos los puertos TCP (1-65535) en el host 1.2.3.4                                                                                                                                                                                                                                                            |                                                                                                                                                                                                                                                                                                            |
| sudo nmap 1.2.3.4 -p 22-445                                              | Escanea el rango de puertos TCP desde el 22 hasta el 445 en el host 1.2.3.4                                                                                                                                                                                                                                           |                                                                                                                                                                                                                                                                                                            |
| sudo nmap 1.2.3.4 -p <num_port> --packet-trace -Pn -n --disable-arp-ping | Escanea el puerto "num_port" del host 1.2.3.4 mostrando el detalle de los paquetes enviados y recibidos (--packet-trace), sin hacer ping previo (-Pn), sin resolver nombres DNS (-n) y sin usar ping ARP (--disable-arp-ping).  <br>El escaneo envía un paquete TCP con flag **SYN (S)** para iniciar conexión.  <br> | La respuesta con flags **RST+ACK (RA)** indica que el puerto está cerrado: el host reconoce el intento pero rechaza la conexión.  <br>Este método es útil para analizar el estado real del puerto y la conectividad evitando interferencias de pings o resoluciones DNS que puedan alterar los resultados. |
> **sudo nmap 1.2.3.4 -p <num_port> --packet-trace -Pn -n --disable-arp-ping**

El comando `sudo nmap 1.2.3.4 -p <num_port> --packet-trace -Pn -n --disable-arp-ping` sirve para escanear un puerto específico (`<num_port>`) del host con IP `1.2.3.4`, mostrando además el detalle de todos los paquetes enviados y recibidos durante el escaneo gracias a la opción `--packet-trace`. Esto permite entender con precisión cómo se comunica Nmap con el host objetivo a nivel de red.

Las opciones `-Pn` y `--disable-arp-ping` hacen que Nmap no realice un ping previo ni un ping ARP para detectar si el host está activo, asumiendo que está disponible. Esto es especialmente útil en redes donde los pings están bloqueados o cuando el objetivo no está en la red local. Por otro lado, `-n` desactiva la resolución DNS, evitando retrasos o posibles errores por problemas en el nombre de dominio, ya que se trabaja solo con direcciones IP.

Durante el escaneo, Nmap envía un paquete TCP con la bandera SYN (S) al puerto objetivo para iniciar la conexión, como se hace en el primer paso del protocolo TCP. Si el puerto está abierto, el host responderá con un paquete que tiene las banderas SYN y ACK, indicando que acepta la conexión. Pero si el puerto está cerrado, el host responderá con un paquete que tiene las banderas RST y ACK (RA), que significa que reconoce la conexión pero la rechaza.

Este método es muy útil para obtener información precisa sobre el estado del puerto, evitando interferencias que puedan causar los pings o las resoluciones DNS, y aportando un nivel de detalle muy alto al mostrar el intercambio de paquetes. Así se puede diagnosticar mejor la conectividad y la configuración del host escaneado.


### Detección de versiones
Cuando uso `nmap` con `-sV` para detectar versiones de servicios, **Nmap intenta identificar el software y la versión que está corriendo en cada puerto**. Para hacerlo, se basa **principalmente en leer los banners** que los servicios envían automáticamente al conectarse.
Un _banner_ es un mensaje que un servicio (como un servidor web, SMTP, etc.) **envía automáticamente cuando alguien se conecta** a su puerto. Suele contener información como:
- El tipo de servicio (SMTP, HTTP, etc.)
- La versión del software (por ejemplo, _Postfix smtpd_)
- A veces, también el sistema operativo (como _Ubuntu_)

```shell
sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```


- Nmap **escanea todos los puertos** (`-p-`)
- Intenta **detectar versiones** (`-sV`)
- Desactiva el ping y otras resoluciones para evitar ser bloqueado (`-Pn`, `-n`, `--disable-arp-ping`)
- **Muestra todo el tráfico que envía y recibe** (`--packet-trace`)

Como salida, se obtiene:
![[Pasted image 20250730162250.png]]

Si paramos atención a los resultados, vemos como el puerto 25 tiene un servicio reconocido como "smtp", pero mirando con detalle todo el contenido, vemos lo siguiente:

![[Pasted image 20250730162325.png]]

Ahí podemos ver:
- El servicio: **ESMTP Postfix**
- El sistema operativo: **Ubuntu**
- El nombre del host: **inlane**

El resumen rápido de nmap no incluía ni "Ubuntu" ni "inlane".

nmap **NO lo muestra todo porque tiene que interpretar y "entender" los banners** para incluir información en su resumen. Si el banner contiene algo que **Nmap no reconoce como importante**, lo ignora en el resumen final. Es una limitación de su sistema de detección automática.

Como alternativa, **manualmente podemos ejecutar**:
- `sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28`

18:28:07.128564 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S], seq ..., length 0
18:28:07.255151 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.], seq ..., length 0
18:28:07.319306 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.], ..., length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)



-------------
**QUESTIONS**:
**1. Based on the last result, find out which operating system it belongs to. Submit the name of the operating system as result.**

**Last result:**
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:12 CEST
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

Vemos que el TTL de respuesta es de 128, pues, indica que estamos ante un windows.
Respuesta: *windows*

**2.  Find all TCP ports on your target. Submit the total number of found TCP ports as the answer.**
La máquina víctima a atacar tiene IP "10.129.2.49"
Ejecutamos un:
```
nmap -sCV --open 10.129.2.49
```

Obteniendo como respuesta:
```
Scanning 10.129.2.49 [1000 ports]
Discovered open port 445/tcp on 10.129.2.49
Discovered open port 110/tcp on 10.129.2.49
Discovered open port 139/tcp on 10.129.2.49
Discovered open port 22/tcp on 10.129.2.49
Discovered open port 80/tcp on 10.129.2.49
Discovered open port 143/tcp on 10.129.2.49
Discovered open port 31337/tcp on 10.129.2.49
Completed SYN Stealth Scan at 06:37, 0.09s elapsed (1000 total ports)
```
Respuesta: *7*

**3. Enumerate the hostname of your target and submit it as the answer. (case-sensitive)**
