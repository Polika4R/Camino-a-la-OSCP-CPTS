En el mundo actual la mayoría de las aplicaciones utilizan TLS para cifrar datos confidenciales en tránsito. 
Sin embargo, no todos los entornos son completamente seguros. Los sistemas heredados, los servicios mal configurados o las aplicaciones de prueba iniciadas sin HTTPS pueden dar lugar al uso de protocolos sin cifrar, como HTTP o SNMP. 

Estas brechas representan una valiosa oportunidad para los atacantes: la posibilidad de buscar credenciales en el tráfico de red de texto plano. 

En esta sección, exploraremos técnicas prácticas para identificar información expuesta, como nombres de usuario y contraseñas, dentro de protocolos de texto plano comunes mediante Wireshark. También analizaremos brevemente Pcredz, una herramienta que puede escanear rápidamente el tráfico de red en busca de dichos datos.

La siguiente tabla enumera varios protocolos comunes junto con sus contrapartes cifradas. Si bien ahora es más común encontrar las versiones seguras, hubo una época en que los protocolos de texto plano se usaban ampliamente:


|Unencrypted Protocol|Encrypted Counterpart|Description|
|---|---|---|
|`HTTP`|`HTTPS`|Used for transferring web pages and resources over the internet.|
|`FTP`|`FTPS/SFTP`|Used for transferring files between a client and a server.|
|`SNMP`|`SNMPv3 (with encryption)`|Used for monitoring and managing network devices like routers and switches.|
|`POP3`|`POP3S`|Retrieves emails from a mail server to a local client.|
|`IMAP`|`IMAPS`|Accesses and manages email messages directly on the mail server.|
|`SMTP`|`SMTPS`|Sends email messages from client to server or between mail servers.|
|`LDAP`|`LDAPS`|Queries and modifies directory services like user credentials and roles.|
|`RDP`|`RDP (with TLS)`|Provides remote desktop access to Windows systems.|
|`DNS (Traditional)`|`DNS over HTTPS (DoH)`|Resolves domain names into IP addresses.|
|`SMB`|`SMB over TLS (SMB 3.0)`|Shares files, printers, and other resources over a network.|
|`VNC`|`VNC with TLS/SSL`|Allows graphical remote control of another computer.|


---
## Wireshark
Wireshark es un conocido analizador de paquetes que viene preinstalado en casi todas las distribuciones de Linux para pruebas de penetración. Cuenta con un potente motor de filtrado que permite una búsqueda eficiente en el tráfico de red, tanto en tiempo real como capturado. Algunos filtros básicos pero útiles incluyen:
  
|Wireshark filter|Description|
|---|---|
|`ip.addr == 56.48.210.13`|Filters packets with a specific IP address|
|`tcp.port == 80`|Filters packets by port (HTTP in this case).|
|`http`|Filters for HTTP traffic.|
|`dns`|Filters DNS traffic, which is useful to monitor domain name resolution.|
|`tcp.flags.syn == 1 && tcp.flags.ack == 0`|Filters SYN packets (used in TCP handshakes), useful for detecting scanning or connection attempts.|
|`icmp`|Filters ICMP packets (used for Ping), which can be useful for reconnaissance or network issues.|
|`http.request.method == "POST"`|Filters for HTTP POST requests. In the case that POST requests are sent over unencrypted HTTP, it may be the case that passwords or other sensitive information is contained within.|
|`tcp.stream eq 53`|Filters for a specific TCP stream. Helps track a conversation between two hosts.|
|`eth.addr == 00:11:22:33:44:55`|Filters packets from/to a specific MAC address.|
|`ip.src == 192.168.24.3 && ip.dst == 56.48.210.3`|Filters traffic between two specific IP addresses. Helps track communication between specific hosts.|
Por ejemplo, en la imagen a continuación estamos filtrando el tráfico HTTP no cifrado:
![[Pasted image 20251011093400.png]]En Wireshark, es posible localizar paquetes que contengan bytes o cadenas específicos. Una forma de hacerlo es usar un filtro de visualización como http contiene "passw". También podemos ir a Editar > Buscar paquete e introducir manualmente la consulta de búsqueda deseada. Por ejemplo, podría buscar paquetes que contengan la cadena "passw":
![[Pasted image 20251011093503.png]]

---
# Pcredz
Pcredz es una herramienta que permite extraer credenciales del tráfico en tiempo real o de capturas de paquetes de red. En concreto, permite extraer la siguiente información:

- Números de tarjetas de crédito
- Credenciales POP
- Credenciales SMTP
- Credenciales IMAP
- Cadenas de comunidad SNMP
- Credenciales FTP
- Credenciales de encabezados HTTP NTLM/Basic, así como de formularios HTTP
- Hashes NTLMv1/v2 de diversos tipos de tráfico, como DCE-RPC, SMBv1/2, LDAP, MSSQL y HTTP
- Hashes Kerberos (AS-REQ Pre-Auth tipo 23)

Para ejecutar Pcredz, se puede clonar el repositorio e instalar todas las dependencias, o bien usar el contenedor Docker proporcionado, que se detalla en la sección de instalación del archivo README.

El siguiente comando permite ejecutar Pcredz con un archivo de captura de paquetes:
```shell-session
Polika4RM@htb[/htb]$ ./Pcredz -f demo.pcapng -t -v

Pcredz 2.0.2
Author: Laurent Gaffie
Please send bugs/comments/pcaps to: laurent.gaffie@gmail.com
This script will extract NTLM (HTTP,LDAP,SMB,MSSQL,RPC, etc), Kerberos,
FTP, HTTP Basic and credit card data from a given pcap file or from a live interface.

CC number scanning activated

Unknown format, trying TCPDump format

[1746131482.601354] protocol: udp 192.168.31.211:59022 > 192.168.31.238:161
Found SNMPv2 Community string: s3cr...SNIP...

[1746131482.601640] protocol: udp 192.168.31.211:59022 > 192.168.31.238:161
Found SNMPv2 Community string: s3cr...SNIP...

<SNIP>
```

---

**Download the attached `credential-hunting-in-network-traffic` and extract the `demo.pcapng` file, then use `Wireshark` or `PCredz` to answer the following questions.

1** The packet capture contains cleartext credit card information. What is the number that was transmitted?

En nuestra máquina Linux atacante, me descargo el contenido adjunto y lo descomprimo. 
Lo abro con WireShark y aplico el filtro http contains "card"

El paquete nº 10465, en el apartado HTML FORM URL contiene dicha información, siendo la respuesta:
5156 8829 4478 9834


**2. What is the SNMPv2 community string that was used?**

Una community string es una contraseña para autenticar el acceso a dispositivos de red, utilizada en los protocolos SNMPv1 y SNMPv2c.

Ejecutando:
```
git clone https://github.com/lgandx/PCredz
cd ~/Descargas/PCredz
python3 -m venv venv
source venv/bin/activate
pip install Cython python-libpcap
```

Podré ejecutar:
```
./Pcredz -f /home/polika4r/Descargas/demo.pcapng -t -v
```

Y me dará la salida:
```
[1760175705.089569] protocol: udp 192.168.31.211:59022 > 192.168.31.238:161
Found SNMPv2 Community string: s3cr3tSNMPC0mmun1ty
```

Siendo la respuesta:
```
s3cr3tSNMPC0mmun1ty
```


**3. What is the password of the user who logged into FTP?**
Ejecutando:
```
git clone https://github.com/lgandx/PCredz
cd ~/Descargas/PCredz
python3 -m venv venv
source venv/bin/activate
pip install Cython python-libpcap
```

Podré ejecutar:
```
./Pcredz -f /home/polika4r/Descargas/demo.pcapng -t -v
```
Y me devolverá:
```
[1760175247.382545] protocol: tcp 192.168.31.243:55707 > 192.168.31.211:21
FTP User: leah
FTP Pass: qwerty123
```
Siendo la respuesta: qwerty123


**4. What file did the user download over FTP?**
Simplemente aplicando un filtro en la cabecera del WireShark de "ftp", encuentro que el campo de información de un paquete (nº10760) es "Opening BINARY mode data connection for creds.txt (44 bytes)."

Siendo la respuesta al ejercicio: 
```
creds.txt
```