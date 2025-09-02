Recomendado en primer lugar hacer un escaneo de puertos disponibles y en segundo lugar hacer un escaneo más profundo de las versiones que corren en dicho puerto.

Una vez finalizado el escaneo, veremos todos los puertos TCP con el servicio correspondiente y sus versiones que están activas en el sistema:

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:00 CEST
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).
Not shown: 65525 closed ports
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.73 seconds
```

Hay veces que los banners de los servicios no los muestra bien. Veamos este ejemplo:

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 20:10 CEST
<SNIP>
NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
Service scan match (Probe NULL matched with NULL line 3104): 10.129.2.28:25 is smtp.  Version: |Postfix smtpd|||
NSOCK INFO [0.4200s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```
Si observamos los resultados de Nmap, podemos ver el estado del puerto, el nombre del servicio y el nombre del host. 

Sin embargo, veamos esta línea:
```
NSOCK INFO \[0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 \[10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
```

Vemos que el servidor SMTP de nuestro destino nos proporcionó más información que la que mostró Nmap. 
Aquí vemos que se trata de la distribución de Linux **Ubuntu.** 

En el resumen no habla de UBUNTU:

```
PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane
```

Tras establecer la conexión TCP, el servidor suele enviar un mensaje de identificación que informa al cliente sobre el servicio activo, usando el indicador PSH en la cabecera TCP. Algunos servicios pueden retrasar o modificar esta información, o incluso eliminarla. 

Si nos conectamos manualmente con netcat y capturamos el tráfico con tcpdump, podemos ver el banner completo que Nmap no siempre muestra en su resumen.

Para ver más información, podemos utilizar:
```shell-session
nc -nv 1.2.3.4 <num_port>
```

-----
Muchos firewalls y sistemas de detección **permiten tráfico saliente** por puertos como el 53 (DNS), 80 (HTTP) y 443 (HTTPS) por parte de la máquina atacante.
```
nc -nv -p <num_port_origen> 1.2.3.4 <num_port_a_atacar>
```
o si uso ncat:
```
ncat -nv --source-port 53 <IP objetivo> <puerto>
```

- Si realizo una conexión saliente con `nc` y el puerto de origen es el 53, es más probable que el tráfico no sea bloqueado ni inspeccionado
- Esto es útil en técnicas de evasión: si un firewall revisa conexiones salientes pero ve que algo sale "desde el puerto 53", puede asumir que es tráfico DNS legítimo y dejarlo pasar.

- Si el puerto de destino es 53 y realmente hay un servidor DNS corriendo, podrías obtener respuestas tipo banner (según la implementación).

- Algunas implementaciones de servidores DNS (mal configuradas o debug) responden con más detalles, como:
    - Versión del software.
    - Nombre del servidor.
    - Configuración de zonas (si haces queries bien formadas).
