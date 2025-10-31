## Enumeración de la política de contraseñas - desde Linux - con credenciales

Podemos obtener la política de contraseñas del dominio de varias maneras, dependiendo de cómo esté configurado el dominio y de si disponemos o no de credenciales válidas. 
Con credenciales válidas, la política de contraseñas también se puede obtener de forma remota mediante herramientas como CrackMapExec o rpcclient:

```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Dumping password info for domain: INLANEFREIGHT
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Minimum password length: 8
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Password history length: 24
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Maximum password age: Not Set
SMB         172.16.5.5      445    ACADEMY-EA-DC01  
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Password Complexity Flags: 000001
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Refuse Password Change: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password Store Cleartext: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password Lockout Admins: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password No Clear Change: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password No Anon Change: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password Complex: 1
SMB         172.16.5.5      445    ACADEMY-EA-DC01  
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Minimum password age: 1 day 4 minutes 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Reset Account Lockout Counter: 30 minutes 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Locked Account Duration: 30 minutes 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Account Lockout Threshold: 5
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Forced Log off Time: Not Set
```

|**Parámetro Clave**|**Valor Obtenido**|**Importancia para el Ataque**|
|---|---|---|
|**Account Lockout Threshold**|**5**|Este es el límite máximo de intentos fallidos de inicio de sesión antes de que una cuenta se bloquee. **Debes probar 4 contraseñas o menos por cuenta** en cada ciclo de ataque.|
|**Reset Account Lockout Counter**|**30 minutes**|Si una cuenta se bloquea o tiene intentos fallidos, el contador de bloqueo se restablece después de **30 minutos**.|
|**Locked Account Duration**|**30 minutes**|Una vez bloqueada, la cuenta permanecerá así durante 30 minutos.|
|**Minimum password length**|**8**|Cualquier contraseña que intentes debe tener al menos 8 caracteres.|
|**Domain Password Complex**|**1**|La complejidad está habilitada, lo que significa que las contraseñas deben cumplir con requisitos como mayúsculas, minúsculas, números o símbolos.|
### Enumeración de la política de contraseñas - desde Linux - Sesiones SMB NULL con rpcclient
Sin credenciales, es posible obtener la política de contraseñas mediante una sesión nula SMB o un enlace LDAP anónimo. 
- La primera opción es mediante una sesión nula SMB: 
	  Estas sesiones permiten que un atacante no autenticado recupere información del dominio, como un listado completo de usuarios, grupos, equipos, atributos de cuentas de usuario y la política de contraseñas del dominio. 
	  
Podemos usar rpcclient para comprobar si un controlador de dominio tiene acceso mediante sesiones nulas SMB. Con el comando "querydomaininfo" podemos obtener información del dominio:

```shell-session
Polika4RM@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
Total Groups:	0
Total Aliases:	37
Sequence No:	1
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
```

Seguidamente, podemos ejecutar: "getdompwinfo" para listas las políticas de contraseñas: 
```
rpcclient $> getdompwinfo
min_password_length: 8
password_properties: 0x00000001
	DOMAIN_PASSWORD_COMPLEX
```

### Enumeración de la política de contraseñas - desde Linux - Sesiones SMB NULL con enum4linux & enum4linux-ng
Probemos esto con enum4linux:
```shell-session
Polika4RM@htb[/htb]$ enum4linux -P 172.16.5.5
... ...
 ================================================== 
|    Password Policy Information for 172.16.5.5    |
 ================================================== 

[+] Attaching to 172.16.5.5 using a NULL share
[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:172.16.5.5)

[+] Trying protocol 445/SMB...
[+] Found domain(s):

	[+] INLANEFREIGHT
	[+] Builtin

[+] Password Info for Domain: INLANEFREIGHT

	[+] Minimum password length: 8
	[+] Password history length: 24
	[+] Maximum password age: Not Set
	[+] Password Complexity Flags: 000001

		[+] Domain Refuse Password Change: 0
		[+] Domain Password Store Cleartext: 0
		[+] Domain Password Lockout Admins: 0
		[+] Domain Password No Clear Change: 0
		[+] Domain Password No Anon Change: 0
		[+] Domain Password Complex: 1

	[+] Minimum password age: 1 day 4 minutes 
	[+] Reset Account Lockout Counter: 30 minutes 
	[+] Locked Account Duration: 30 minutes 
	[+] Account Lockout Threshold: 5
	[+] Forced Log off Time: Not Set

[+] Retieved partial password policy with rpcclient:

Password Complexity: Enabled
Minimum Password Length: 8

enum4linux complete on Tue Feb 22 17:39:29 2022
```

La herramienta enum4linux-ng es una reescritura de enum4linux en Python, pero incluye funciones adicionales como la capacidad de exportar datos como archivos YAML o JSON, que posteriormente se pueden procesar o usar con otras herramientas. También admite la salida de datos con colores, entre otras características.
Utilizaremos el parámetro -oA para guardar la salida en el un arhico llamado "ilfreight":

```shell-session
Polika4RM@htb[/htb]$ enum4linux-ng -P 172.16.5.5 -oA ilfreight

ENUM4LINUX - next generation

<SNIP>

 =======================================
|    RPC Session Check on 172.16.5.5    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user session
[-] Could not establish random user session: STATUS_LOGON_FAILURE

 =================================================
|    Domain Information via RPC for 172.16.5.5    |
 =================================================
[+] Domain: INLANEFREIGHT
[+] SID: S-1-5-21-3842939050-3880317879-2865463114
[+] Host is part of a domain (not a workgroup)
 =========================================================
|    Domain Information via SMB session for 172.16.5.5    |
========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: ACADEMY-EA-DC01
NetBIOS domain name: INLANEFREIGHT
DNS domain: INLANEFREIGHT.LOCAL
FQDN: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

 =======================================
|    Policies via RPC for 172.16.5.5    |
 =======================================
[*] Trying port 445/tcp
[+] Found policy:
domain_password_information:
  pw_history_length: 24
  min_pw_length: 8
  min_pw_age: 1 day 4 minutes
  max_pw_age: not set
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: true
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: 5
domain_logoff_information:
  force_logoff_time: not set

Completed after 5.41 seconds
```

Seguidamente, podremos numerar dicho contenido con:
```
Polika4RM@htb[/htb]$ cat ilfreight.json 

{
    "target": {
        "host": "172.16.5.5",
        "workgroup": ""
    },
    "credentials": {
        "user": "",
        "password": "",
        "random_user": "yxditqpc"
    },
    "services": {
        "SMB": {
            "port": 445,
            "accessible": true
        },
        "SMB over NetBIOS": {
            "port": 139,
            "accessible": true
        }
    },
    "smb_dialects": {
        "SMB 1.0": false,
        "SMB 2.02": true,
        "SMB 2.1": true,
        "SMB 3.0": true,
        "SMB1 only": false,
        "Preferred dialect": "SMB 3.0",
        "SMB signing required": true
    },
    "sessions_possible": true,
    "null_session_possible": true,

<SNIP>
```

## Enumeración de sesiones SMB nulas - desde Windows
Es menos común realizar este tipo de ataque de sesión nula desde Windows, pero podríamos usar el comando net use \\\host\ipc$ "" /u:"" para establecer una sesión nula desde una máquina Windows y confirmar si podemos realizar más ataques de este tipo.

```cmd-session
C:\htb> net use \\DC01\ipc$ "" /u:""
The command completed successfully.
```

También podemos utilizar un usuario y contraseña para ingresar en el servicio SMB: 
```cmd-session
C:\htb> net use \\DC01\ipc$ "" /u:guest
System error 1331 has occurred.

This user can't sign in because this account is currently disabled
```

o con contraseña:
```cmd-session
C:\htb> net use \\DC01\ipc$ "password" /u:guest
System error 1326 has occurred.

The user name or password is incorrect.
```


## Enumeración de la política de contraseñas - desde Linux - Enlace anónimo LDAP

Un "LDAP Anonymous Bind" (o **enlace anónimo LDAP**) es una forma de **conectarse** a un servicio de directorio LDAP **sin proporcionar un nombre de usuario ni una contraseña**.
- **LDAP** es un protocolo que permite a las aplicaciones acceder y gestionar información en un **directorio** (como una gran base de datos de usuarios, grupos, ordenadores, etc.).
- Un **Bind** es el acto de establecer una conexión y autenticarse con el servicio LDAP.
- Un Anonymous Bind es cuando esa conexión se establece sin autenticación.

Los accesos anónimos a LDAP permiten a atacantes no autenticados obtener información del dominio, como un listado completo de usuarios, grupos, equipos, atributos de cuentas de usuario y la política de contraseñas del dominio. 
Esta es una configuración heredada y, desde Windows Server 2003, solo los usuarios autenticados pueden iniciar solicitudes LDAP. Aún vemos esta configuración ocasionalmente, ya que un administrador puede haber configurado una aplicación para permitir accesos anónimos y haber otorgado más permisos de los previstos, lo que permite a usuarios no autenticados acceder a todos los objetos de Active Directory.

Con un acceso anónimo a LDAP, podemos usar herramientas de enumeración específicas de LDAP, como windapsearch.py, ldapsearch, ad-ldapdomaindump.py, etc., para obtener la política de contraseñas. Con ldapsearch, el proceso puede ser algo engorroso, pero es posible. Un ejemplo de comando para obtener la política de contraseñas es el siguiente:

##### Utilizando ldapsearch
```shell-session
Polika4RM@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

forceLogoff: -9223372036854775808
lockoutDuration: -18000000000
lockOutObservationWindow: -18000000000
lockoutThreshold: 5
maxPwdAge: -9223372036854775808
minPwdAge: -864000000000
minPwdLength: 8
modifiedCountAtLastProm: 0
nextRid: 1002
pwdProperties: 1
pwdHistoryLength: 24
```

|**Parte del Comando**|**Significado**|**Función Específica**|
|---|---|---|
|`ldapsearch`|La herramienta.|Es el programa de línea de comandos para realizar consultas a un servidor LDAP.|
|`-h 172.16.5.5`|Host.|Especifica la **dirección IP** del servidor al que se quiere conectar (en este caso, `172.16.5.5`).|
|`-x`|Simple Authentication.|Indica a `ldapsearch` que use la autenticación **simple**. **Esto permite el "Anonymous Bind" (enlace anónimo)**, es decir, intenta conectarse sin proporcionar un nombre de usuario ni una contraseña.|
|`-b "DC=INLANEFREIGHT,DC=LOCAL"`|Base DN (Distinguished Name).|Define el punto de partida (la **raíz**) de la búsqueda en el directorio, que es el nombre de dominio.|
|`-s sub`|Search Scope.|Le dice a la herramienta que realice una búsqueda en **subárbol** (`sub`), lo que significa que buscará en toda la estructura del directorio debajo de la Base DN.|
|`"*"`|Filter.|Especifica que la consulta debe devolver **todos los objetos** (`*`).|
|`|`|Pipe.|
|`grep -m 1 -B 10 pwdHistoryLength`|Filtro.|Busca la línea que contiene el atributo **`pwdHistoryLength`** (longitud del historial de contraseñas) y muestra **esa línea** junto con las **10 líneas anteriores** (`-B 10`).|
## Enumeración de la política de contraseñas - desde Windows

Si podemos autenticarnos en el dominio desde un host Windows, podemos usar binarios integrados de Windows, como net.exe, para obtener la política de contraseñas. 
También podemos usar diversas herramientas como PowerView, CrackMapExec (adaptado a Windows), SharpMapExec, SharpView, etc.

El uso de comandos integrados resulta útil si accedemos a un sistema Windows y no podemos transferirle herramientas, o si el cliente nos ubica en un sistema Windows, pero no tenemos forma de transferirle herramientas. Un ejemplo que utiliza el binario integrado net.exe es:

#### Enumeración de la política de contraseñas - desde Windows - NETCAT

```cmd-session
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```

#### Enumeración de la política de contraseñas - desde Windows - POWERSHELL
PowerView nos proporciona la misma salida que nuestro comando "net accounts", aunque en un formato diferente, e indicó que la complejidad de contraseñas está habilitada (PasswordComplexity=1).
```powershell-session
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy

Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=1; MaximumPasswordAge=-1; MinimumPasswordLength=8; PasswordComplexity=1;
                 PasswordHistorySize=24; LockoutBadCount=5; ResetLockoutCount=30; LockoutDuration=30;
                 RequireLogonToChangePassword=0; ForceLogoffWhenHourExpire=0; ClearTextPassword=0;
                 LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
Version        : @{signature="$CHICAGO$"; Revision=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Path           : \\INLANEFREIGHT.LOCAL\sysvol\INLANEFREIGHT.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHI
                 NE\Microsoft\Windows NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy
```

---
**1. What is the default Minimum password length when a new domain is created? (One number)**
Respuesta teórica: 7

**2. What is the minPwdLength set to in the INLANEFREIGHT.LOCAL domain? (One number)**
Respuesta teórica: 8

