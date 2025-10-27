
La tunelización ICMP encapsula el tráfico en paquetes ICMP que contienen "echo" solicitudes y respuestas.
Cuando un host dentro de una red con cortafuegos puede hacer ping a un servidor externo, puede encapsular su tráfico dentro de la solicitud de "echo" de ping y enviarlo a dicho servidor. 
El servidor externo puede validar este tráfico y enviar una respuesta adecuada, lo cual resulta extremadamente útil para la exfiltración de datos y la creación de túneles pivote a un servidor externo.

Usaremos la herramienta *ptunnel-ng* para crear un túnel entre nuestro servidor Ubuntu y el host atacante. 
Una vez creado el túnel, podremos redirigir nuestro tráfico a través del cliente ptunnel-ng. Podemos iniciar el servidor ptunnel-ng en el host pivote de destino. 

## Configurando "ptunnel-ng"
Comencemos configurando ptunnel-ng.
En nuestra máquina atacante, lo clonamos de github con:

```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/utoni/ptunnel-ng.git
```

Una vez que el repositorio ptunnel-ng se clona en nuestra máquina atacante, podemos ejecutar el script autogen.sh ubicado en la raíz del directorio ptunnel-ng:

```shell-session
Polika4RM@htb[/htb]$ sudo ./autogen.sh 
```

Tras ejecutar "autogen.sh", ptunnel-ng se puede usar tanto en el lado del cliente como del servidor. Ahora debemos transferir el repositorio desde nuestro host de ataque al host objetivo. Como en secciones anteriores, podemos usar SCP para transferir los archivos. 
Si queremos transferir el repositorio completo y los archivos que contiene, debemos usar la opción -r con SCP:

## Transfiriendo y ejecutando el Ptunnel-ng al/en el Pivot Host

Compartimos el archivo "ptunnel-ng" con nuestra máquina Pivot, con el siguiente comando:

```
Polika4RM@htb[/htb]$ scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```

Iniciamos el servicio con:
```shell-session
ubuntu@WEB01:~/ptunnel-ng/src$ sudo ./ptunnel-ng -r10.129.202.64 -R22

[sudo] password for ubuntu: 
./ptunnel-ng: /lib/x86_64-linux-gnu/libselinux.so.1: no version information available (required by ./ptunnel-ng)
[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Forwarding incoming ping packets over TCP.
[inf]: Ping proxy is listening in privileged mode.
[inf]: Dropping privileges now.
```

La máquina UBUNTU:
- -r10.129.202.64: 
	  Este es el parámetro para el host remoto (remote host). Le dice al servidor ptunnel-ng a dónde debe reenviar el tráfico que recibe. En este caso, le está diciendo que reenvíe el tráfico a la IP 10.129.202.64 (¡la misma máquina donde se está ejecutando!).
- -R22: Este es el parámetro para el puerto remoto (remote port). Especifica a qué puerto del "host remoto" (definido en -r) debe enviar el tráfico. El puerto 22 es el puerto estándar de SSH.

El comando le dice a la máquina WEB01 (con IP 10.129.202.64): "Inicia un servidor de túnel ICMP. Cuando recibas tráfico ICMP (ping) de un cliente ptunnel-ng, saca los datos que vienen escondidos y redirígelos a tu propia IP (10.129.202.64) al puerto 22 (el servicio SSH)."
## Conectándonos desde el atacante al "ptunnel-ng" server. 
Desde la máquina atacante ejecuto:

```shell-session
Polika4RM@htb[/htb]$ sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22

[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Relaying packets from incoming TCP streams.
```
- -p10.129.202.64: 
	  Esta es la IP del servidor ptunnel-ng al que te estás conectando (el proxy o peer). 
- -l2222:
	  Le dice a tu propia máquina de atacante que escuche (listen) en el puerto TCP local 2222. Este puerto es la entrada a tu túnel.
- -r10.129.202.64: 
	 Define el host remoto de destino final (el remote host). Es a dónde el servidor (en WEB01) debe reenviar el tráfico que recibe.- 
- -R22: Define el puerto remoto de destino final (el remote port).

Este comando: Abre el puerto TCP 2222 en mi máquina local (localhost). Cuando alguien se conecte a este puerto (la propia máquina), toma ese tráfico, escóndelo dentro de paquetes de ping (ICMP) y envíaselo al servidor ptunnel-ng que está en 10.129.202.64. Dile a ese servidor que, cuando reciba mis pings, reenvíe los datos al destino final 10.129.202.64 en el puerto 22 (SSH).
## Tunelizando una conexión SSH con ICMP Tunnel
Ejecutaré desde el atacante:
```shell-session
Polika4RM@htb[/htb]$ ssh -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 11 May 2022 03:10:15 PM UTC

  System load:             0.0
  Usage of /:              39.6% of 13.72GB
  Memory usage:            37%
  Swap usage:              0%
  Processes:               183
  Users logged in:         1
  IPv4 address for ens192: 10.129.202.64
  IPv6 address for ens192: dead:beef::250:56ff:feb9:52eb
  IPv4 address for ens224: 172.16.5.129

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

144 updates can be applied immediately.
97 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Wed May 11 14:53:22 2022 from 10.10.14.18
ubuntu@WEB01:~$ 
```

Si se configura correctamente, podremos ingresar credenciales a través del servicio SSH mediente un túnel ICMP. 

## Estableciendo el Dynamic Port Forwarding por SSH

El siguiente comando está creando un túnel dentro de otro túnel, y se puede ejecutar sin la necesidad de ejecutar el comando anterior (ssh -p2222 -lubuntu 127.0.0.1).

Su objetivo principal es usar el parámetro -D 9050 para crear un proxy SOCKS dinámico en tu máquina de atacante. El parámetro -D en el comando ssh es el encargado de establecer el Dynamic Port Forwarding (Reenvío Dinámico de Puertos).
```shell-session
Polika4RM@htb[/htb]$ ssh -D 9050 -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)
<snip>
```
Hago "ssh -D 9050 -p2222 ..." para crear un proxy SOCKS (dynamic port forward) en mi máquina local cuya salida se encamina, cifrada y autenticada por SSH, hacia la máquina pivote (WEB01) a través del túnel ICMP que cree con ptunnel-ng. Ese proxy te permite emitir cualquier conexión TCP (p. ej. navegador, nmap vía proxychains, herramientas) y que salga desde la red interna del pivote (172.16.5.x).

Este comando anterior lo dejaré en segundo plano.


#### Proxychaining a través del ICMP Tunnel
Con esto, podré ejecutar proxychains:
```shell-session
Polika4RM@htb[/htb]$ proxychains nmap -sV -sT 172.16.5.19 -p3389

ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-11 11:10 EDT
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Nmap scan report for 172.16.5.19
Host is up (0.12s latency).

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.78 seconds
```

---

**Target:10.129.255.103**
**SSH to 10.129.255.103 (ACADEMY-PIVOTING-LINUXPIV) with user "ubuntu" and password "HTB_@cademy_stdnt!"**
**1. Using the concepts taught thus far, connect to the target and establish an ICMP tunnel. Pivot to the DC (172.16.5.19, victor:pass@123) and submit the contents of C:\Users\victor\Downloads\flag.txt as the answer.**

Por desactualización de la máquina, utilizaré el método rpivot.
Siendo la respuesta: N3Tw0rkTunnelV1sion!

