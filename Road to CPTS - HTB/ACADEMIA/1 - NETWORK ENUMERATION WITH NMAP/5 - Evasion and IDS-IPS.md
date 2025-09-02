**Nmap** es una herramienta de escaneo de redes muy potente que, además de hacer un simple escaneo de puertos, **puede intentar evitar que los sistemas de seguridad detecten o bloqueen su actividad**. Los firewalls y los sistemas IDS/IPS (Sistemas de Detección/Prevención de Intrusiones) están diseñados para identificar y bloquear escaneos sospechosos o ataques.

Para evadir estos sistemas, Nmap utiliza técnicas como:

- **Fragmentación de paquetes:** En lugar de enviar paquetes completos, Nmap los divide en fragmentos muy pequeños. Algunos firewalls o IDS pueden tener dificultades para reensamblar estos fragmentos correctamente y detectar que es un escaneo. Esto puede permitir que el escaneo pase sin ser detectado o bloqueado.
- **Uso de señuelos (decoys):** Nmap puede enviar paquetes desde varias direcciones IP falsas (decoys) junto con la IP real que realiza el escaneo. Esto confunde al IDS/IPS, que no puede saber cuál es la fuente real, dificultando así la identificación del atacante.

----

Los paquetes pueden descartarse o rechazarse. Los paquetes descartados se ignoran y el host no devuelve ninguna respuesta.

Esto es diferente para los paquetes rechazados que se devuelven con un indicador RST. Estos paquetes contienen diferentes tipos de códigos de error ICMP o no contienen nada. Estos errores pueden ser:
- Red inaccesible
- Red prohibida
- Host inaccesible
- Host prohibido
- Puerto inaccesible
- Proto inaccesible

El método de escaneo TCP ACK **(-sA)** de Nmap es **mucho más difícil de filtrar para cortafuegos y sistemas IDS/IPS** que los escaneos SYN (-sS) o Connect (sT) habituales, ya que solo envía un paquete TCP con el indicador ACK. Cuando un puerto está cerrado o abierto, el host debe responder con un indicador RST. A diferencia de las conexiones salientes, los cortafuegos suelen bloquear todos los intentos de conexión (con el indicador SYN) desde redes externas. Sin embargo, el cortafuegos suele pasar por alto los paquetes con el indicador ACK, ya que no puede determinar si la conexión se estableció inicialmente desde la red externa o la interna.
**El objetivo es identificar puertos filtrados/no filtrados, no su estado real (abierto o cerrado)**

Si observamos estos escaneos, veremos cómo difieren los resultados:

### **Con SYN-Scan (-sS):**
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:56 CEST
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:22 S ttl=53 id=22412 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:25 S ttl=50 id=62291 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:21 S ttl=58 id=38696 iplen=44  seq=4092255222 win=1024 <mss 1460>
RCVD (0.0329s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=40884 iplen=72 ]
RCVD (0.0341s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
RCVD (1.0386s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
SENT (1.1366s) TCP 10.10.14.2:57348 > 10.129.2.28:25 S ttl=44 id=6796 iplen=44  seq=4092320759 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.0053s latency).

PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp open     ssh
25/tcp filtered smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

**Enviado:**  
Paquetes TCP con el flag **SYN** (iniciar conexión TCP).

**Recibido:**

- Puerto 22 → Responde con **SYN-ACK** → Nmap lo marca como **open**.
- Puerto 21 → Responde con **ICMP Port Unreachable** → Nmap lo marca como **filtered**.
- Puerto 25 → No hay respuesta → Nmap lo marca como **filtered**.

**Resultado:**
```
PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp open     ssh
25/tcp filtered smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

```


### **Con ACK-Scan (-sA):**
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:57 CEST
SENT (0.0422s) TCP 10.10.14.2:49343 > 10.129.2.28:21 A ttl=49 id=12381 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:22 A ttl=41 id=5146 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:25 A ttl=49 id=5800 iplen=40  seq=0 win=1024
RCVD (0.1252s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=55628 iplen=68 ]
RCVD (0.1268s) TCP 10.129.2.28:22 > 10.10.14.2:49343 R ttl=64 id=0 iplen=40  seq=1660784500 win=0
SENT (1.3837s) TCP 10.10.14.2:49344 > 10.129.2.28:25 A ttl=59 id=21915 iplen=40  seq=0 win=1024
Nmap scan report for 10.129.2.28
Host is up (0.083s latency).

PORT   STATE      SERVICE
21/tcp filtered   ftp
22/tcp unfiltered ssh
25/tcp filtered   smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

**Enviado:**  
Paquetes TCP con el flag **ACK** (como si formaran parte de una conexión existente).

**Recibido:**

- Puerto 22 → Responde con **RST** → Nmap lo marca como **unfiltered**.
- Puerto 21 → Responde con **ICMP Port Unreachable** → Nmap lo marca como **filtered**.
- Puerto 25 → No responde → Nmap lo marca como **filtered**.

**Resultado:**
```
PORT   STATE      SERVICE
21/tcp filtered   ftp
22/tcp unfiltered ssh
25/tcp filtered   smtp

```


###  ¿Qué significa esto?

#### Sobre el puerto 22:
- Con `-sS` → responde con SYN-ACK → **está abierto**.
- Con `-sA` → responde con RST → **no hay firewall filtrando ACKs hacia ese puerto**.
 Esto confirma que **el puerto 22 está abierto** y no está **filtrado por un cortafuegos**.

#### Sobre los puertos 21 y 25:
- Con ambos escaneos (`-sS` y `-sA`), el estado es **filtered**.
- Con ACK-scan, el hecho de que **no se reciba un RST** indica que **un firewall está filtrando esos paquetes**.
Esto significa que probablemente **hay un firewall bloqueando el tráfico TCP entrante hacia los puertos 21 y 25**.

---

###  Conclusión general

|Puerto|Escaneo SYN (`-sS`)|Escaneo ACK (`-sA`)|Interpretación|
|---|---|---|---|
|21|filtered|filtered|Filtrado por firewall, sin respuesta clara|
|22|open|unfiltered|Está abierto y **no filtrado** por firewall|
|25|filtered|filtered|Probablemente **filtrado por firewall**|

## Utilidad del -sA frente a -sS:
Caso de un escaneo `-sS` y **todo te sale “filtered”**. No sabes si hay servicios activos o no.

Entonces haces un `-sA`:
- Si **algunos puertos responden con RST** → esos puertos **no están filtrados**.
- Si **otros no responden** → están **filtrados**.

**Esto te permite saber:**
- Qué puertos **están siendo filtrados activamente**.
   - Qué puertos podrían usarse para **crear conexiones encubiertas** (por ejemplo, con túneles TCP).

# Detecciones por IDS/IPS
Detectar sistemas IDS/IPS durante una prueba de penetración es complejo, ya que estos supervisan el tráfico de forma pasiva. Los IDS solo alertan ante posibles amenazas, mientras que los IPS actúan automáticamente para prevenir ataques. Se recomienda usar varios VPS con IP distintas, ya que si una IP se bloquea tras un escaneo agresivo, es probable que haya un sistema de detección. Esto obliga a adoptar técnicas más discretas y ocultar las interacciones con la red. La identificación de estas medidas permite ajustar la estrategia del pentester para evitar la detección y continuar la evaluación de forma más sigilosa.

# Señuelos

Cuando se realiza una prueba de penetración, los administradores pueden bloquear el acceso a redes o servicios en función del origen del tráfico. 
En muchos casos, subredes completas de ciertas regiones quedan bloqueadas por defecto, o bien sistemas IPS reaccionan automáticamente bloqueando la IP del atacante. 
Para evitarlo, **Nmap ofrece la opción de escaneo con señuelos (_decoys_) mediante el parámetro `-D`.**

**Este método consiste en insertar varias direcciones IP falsas (señuelos) en la cabecera IP de los paquetes enviados. Así se disfraza la IP real del atacante entre múltiples direcciones. Por ejemplo, con `-D RND:5` se generan 5 IPs aleatorias, entre las cuales se incluye la IP real en una posición aleatoria.**

> ⚠️ Es importante que las IPs señuelo estén activas o parezcan válidas, ya que si el sistema objetivo detecta múltiples paquetes SYN no válidos, podría activar mecanismos de defensa como el bloqueo por SYN-flood.

Un ejemplo práctico sería:
```
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```
`
Este comando genera 5 IPs aleatorias como señuelos (-D RND:5).

En el ejemplo, se observa que solo la IP real recibe respuesta del servidor, lo que permite inferir que los paquetes señuelo fueron ignorados por el objetivo.

En casos donde los paquetes con IPs aleatorias son descartados por routers o ISPs, es preferible usar direcciones IP de otros VPS controlados por el atacante, mejorando la legitimidad del tráfico.

Ejemplo de uso:
```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 16:14 CEST
SENT (0.0378s) TCP 102.52.161.59:59289 > 10.129.2.28:80 S ttl=42 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0378s) TCP 10.10.14.2:59289 > 10.129.2.28:80 S ttl=59 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 210.120.38.29:59289 > 10.129.2.28:80 S ttl=37 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 191.6.64.171:59289 > 10.129.2.28:80 S ttl=38 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 184.178.194.209:59289 > 10.129.2.28:80 S ttl=39 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 43.21.121.33:59289 > 10.129.2.28:80 S ttl=55 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
RCVD (0.1370s) TCP 10.129.2.28:80 > 10.10.14.2:59289 SA ttl=64 id=0 iplen=44  seq=4056111701 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.099s latency).

PORT   STATE SERVICE
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

# Prueba de Reglas de Firewall con Nmap

Durante una evaluación de seguridad, es fundamental identificar qué puertos están filtrados por un firewall. Una técnica sencilla consiste en realizar un escaneo específico a un puerto determinado para comprobar su estado. En este ejemplo, se prueba el puerto 445 (usado por SMB) con detección de sistema operativo activada:

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p445 -O

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:23 CEST
Nmap scan report for 10.129.2.28
Host is up (0.032s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.14 seconds
```

- El host está activo y responde.
- El puerto 445 aparece como **filtered**, lo que indica que un firewall está bloqueando los paquetes (probablemente sin enviar respuesta).
- La detección del sistema operativo no es concluyente, ya que hay demasiadas coincidencias de huellas digitales.
- El salto de red es de solo 1 **hop**, lo que sugiere que el objetivo está en la misma red o muy cerca del host atacante.

# Escaneo usando IP de Origen Diferente en Nmap

Para evadir reglas de firewall o sistemas IDS/IPS que bloquean ciertas IPs o subredes, **es posible modificar la dirección IP origen desde la que se envían los paquetes. Nmap permite esta técnica con la opción `-S`, especificando una IP distinta a la real.
```
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
```

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:16 CEST
Nmap scan report for 10.129.2.28
Host is up (0.010s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Linux 2.6.32 - 2.6.35 (94%), Linux 2.6.32 - 3.5 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.11 seconds
```


# SYN-Scan desde el puerto DNS (53) con Nmap

Para evadir firewalls y sistemas IDS/IPS, se puede realizar un escaneo SYN usando el puerto DNS (TCP/53) como puerto de origen, ya que este puerto suele estar permitido y menos filtrado.

El comando usado es:
```
sudo nmap <IP objetivo> -p <puerto> -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

Para comprobar si un puerto filtrado (bloqueado por firewall o IDS/IPS) acepta conexiones desde el puerto DNS (53), se puede usar Netcat con la opción de origen en el puerto 53.

Ejemplo de comando:
```
ncat -nv --source-port 53 <IP objetivo> <puerto>
```


# Proxy DNS

Por defecto, **Nmap realiza una resolución inversa de DNS (reverse DNS) a menos que se indique lo contrario**, con el fin de obtener información más relevante sobre el objetivo. 
Estas consultas DNS suelen permitirse en la mayoría de los casos, ya que el servidor web objetivo normalmente debe ser localizado y visitado. **Las consultas DNS se realizan sobre el puerto UDP 53.** Tradicionalmente, el puerto TCP 53 solo se utilizaba para las llamadas "transferencias de zona" entre servidores DNS o para transferencias de datos mayores a 512 bytes. Sin embargo, esto está cambiando cada vez más debido a las implementaciones de IPv6 y DNSSEC, lo que provoca que muchas consultas DNS también se realicen por el puerto TCP 53.

No obstante, Nmap aún nos permite especificar manualmente qué servidores DNS utilizar con la opción `--dns-server <ns>,<ns>`. Este método puede ser muy útil si estamos en una **zona desmilitarizada (DMZ)**, ya que los servidores DNS de la empresa suelen ser más confiables que los externos. Por ejemplo, podríamos usar esos DNS internos para interactuar con los hosts de la red interna.

Otra técnica es usar el puerto TCP 53 como **puerto de origen** en nuestros escaneos, mediante la opción `--source-port`. Si el administrador ha configurado el firewall para confiar en ese puerto y los IDS/IPS no lo filtran adecuadamente, es posible que nuestros paquetes TCP pasen sin ser detectados ni bloqueados.

```
nmap -sSCV --open 10.129.2.48 --source-port 53
```

# Escaneo de sistema operativo
Para escanear el sistema operativo con **Nmap**, se usa la opción `-O`, que activa la detección de sistema operativo (OS detection).

```
sudo nmap -O <IP_objetivo>
```

