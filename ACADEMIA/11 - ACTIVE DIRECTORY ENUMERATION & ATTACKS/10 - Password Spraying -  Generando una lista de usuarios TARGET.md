
### Enumeración detallada de usuarios

Existen varias formas de acceder a un listado de usuarios válidos de un dominio. 
- Aprovecharnos de una sesión SMB nula:
	  listara la totalidad de los usuarios desde el controlador del dominio. 
- Utilizando una sesión nula de SMB.
- Utilizando "Kerbrute":
	  Para validar usuarios mediante una lista de palabras de una fuente como el repositorio de GitHub de nombres de usuario estadísticamente probables, o recopilada mediante una herramienta como linkedin2username para crear una lista de usuarios potencialmente válidos.
- Utilizando un conjunto de credenciales de un sistema de ataque Linux o Windows, ya sea proporcionado por nuestro cliente u obtenido por otros medios, como el envenenamiento de respuestas LLMNR/NBT-NS mediante Responder, o incluso un ataque de fuerza bruta de contraseñas exitoso utilizando una lista de palabras más pequeña.

Independientemente del método que elijamos, es fundamental tener en cuenta la política de contraseñas del dominio. Si disponemos de una sesión nula SMB, un enlace anónimo LDAP o un conjunto de credenciales válidas, podemos enumerar la política de contraseñas. 

**Tener esta política a mano es muy útil, ya que la longitud mínima de la contraseña y si está habilitada o no la complejidad de la misma nos ayudan a formular la lista de contraseñas que probaremos en nuestros ataques de fuerza bruta. **

Deberemos anotar: 
- Las cuentas atacadas
- El controlador de dominio utilizado en el ataque
- La hora del ataque
- La fecha del ataque
- Las contraseñas intentadas

Esto nos ayudará a evitar duplicar esfuerzos. Si se produce un bloqueo de cuenta o nuestro cliente detecta intentos de inicio de sesión sospechosos, podemos proporcionarle nuestros registros para que los compare con sus sistemas de registro y se asegure de que no se haya producido ninguna actividad maliciosa en la red.

---
### Sesión nula SMB para extraer la lista de usuarios

Si no tengo una cuenta de usuario, la primera estrategia es buscar fallos de configuración en los Controladores de Dominio (DC). Puedo intentar buscar Sesiones Nulas SMB (SMB NULL sessions) o Conexiones Anónimas LDAP (LDAP anonymous binds). Si alguna de estas configuraciones está activa, me permitirá obtener una lista precisa de todos los usuarios de Active Directory, además de la política de contraseñas del dominio.

Herramientas como enum4linux, rpcclient y CrackMapExec están diseñadas para aprovechar estas vulnerabilidades. Independientemente de la herramienta que utilice siempre tendré que limpiar la salida para obtener una lista de nombres de usuario utilizable, con uno por línea. Un ejemplo es usar la herramienta enum4linux con el flag -U para simplificar el proceso.

Si logro conseguir credenciales válidas de un usuario de dominio o, mejor aún, acceso a nivel SYSTEM en un host de Windows, la enumeración se vuelve mucho más sencilla.

Cuando una aplicación o un atacante consigue ejecutar código con privilegios SYSTEM, obtiene la capacidad de "suplantar la identidad" del equipo en las comunicaciones con el dominio.
Debido a la suplantación, el flujo de autenticación funciona así:

1. El atacante ejecuta una herramienta con privilegios **`SYSTEM`** en la máquina.
2. La herramienta intenta acceder a Active Directory (por ejemplo, para consultar la lista de usuarios).
3. El Controlador de Dominio ve la solicitud y dice: "Ah, esta solicitud viene de la máquina **PC01$** (el objeto de equipo)".
4. Como **PC01$** es una cuenta legítima de dominio (y está activa), el DC permite el acceso.

#### Utilizando enum4linux

El siguiente comando listará en líneas aparte (-U) en conjunto de usuarios existentes en un servicio SMB con sesión de inicio nula.
`enum4linux` no utiliza una lista de nombres predefinida (como un diccionario). En su lugar, **consulta activamente** al servidor de destino para que este le revele la lista de cuentas.
```shell-session
Polika4RM@htb[/htb]$ enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

administrator
guest
krbtgt
lab_adm
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch
ccruz
njohnson
mholliday

<SNIP>
```

#### Utilizando rpcclient
Podemos utilizar el comando "enumdomusers" traás ejecutar prcclient: 
```shell-session
Polika4RM@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers 
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]

<SNIP>
```

#### Utilizando CrackMapExec --users
Finalmente, podemos usar CrackMapExec con la opción `--users`. 
- Esta es una herramienta útil que también muestra el recuento de intentos de inicio de sesión fallidos (badpwdcount), lo que nos permite eliminar de nuestra lista cualquier cuenta que esté cerca del umbral de bloqueo. 
- También muestra la fecha y hora del último intento de contraseña incorrecta, lo que nos permite ver qué tan cerca está una cuenta de que se restablezca su recuento de intentos de inicio de sesión fallidos. 

```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 172.16.5.5 --users

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-01-10 13:23:09.463228
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez     
```

### LDAP anónimo para obtener listas de usuarios
Podemos usar diversas herramientas para recopilar usuarios cuando encontramos un enlace LDAP anónimo. Algunos ejemplos son windapsearch y ldapsearch. Si optamos por usar ldapsearch, deberemos especificar un filtro de búsqueda LDAP válido. 

##### Ejectuando "ldapsearch" con:
```shell-session
Polika4RM@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "

guest
ACADEMY-EA-DC01$
ACADEMY-EA-MS01$
ACADEMY-EA-WEB01$
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch

<SNIP>
```


##### Ejectuando "windapsearch" con:
```shell-session
Polika4RM@htb[/htb]$ ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 2906 users: 

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local

cn: Paul Falcon
userPrincipalName: pfalcon@inlanefreight.local

cn: Fae Anthony
userPrincipalName: fanthony@inlanefreight.local

cn: Walter Dillard
userPrincipalName: wdillard@inlanefreight.local

<SNIP>
```


### KERBRUTE para obtener listas de usuarios
Podemos usar Kerbrute para enumerar cuentas válidas de AD y realizar ataques de fuerza bruta contra contraseñas.
- Esta herramienta utiliza la preautenticación Kerberos, un método mucho más rápido y potencialmente más sigiloso para realizar ataques de fuerza bruta contra contraseñas.

Probemos este método usando la lista de palabras jsmith.txt, que contiene 48 705 nombres de usuario comunes posibles en formato plano. El repositorio de GitHub statistically-likely-usernames es un excelente recurso para este tipo de ataque y contiene diversas listas de nombres de usuario que podemos usar para enumerar nombres de usuario válidos mediante Kerbrute:
>https://github.com/insidetrust/statistically-likely-usernames

Ejecutamos Kerbrute con:
```shell-session
Polika4RM@htb[/htb]$  kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:16:11 >  Using KDC(s):
2022/02/17 22:16:11 >  	172.16.5.5:88

2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jjones@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 sbrown@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 tjohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jwilson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 bdavis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 njohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 asanchez@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 dlewis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 ccruz@inlanefreight.local

<SNIP>
```

Hemos comprobado más de 48 000 nombres de usuario en poco más de 12 segundos y hemos encontrado más de 50 válidos.


## Enumeración de usuarios partiendo de credenciales válidas con CrackMapExec

```shell-session
Polika4RM@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users

[sudo] password for htb-student: 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\htb-student:Academy_student_AD! 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 1 baddpwdtime: 2022-02-23 21:43:35.059620
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez                       badpwdcount: 20 baddpwdtime: 2022-02-17 22:59:22.684613
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\pfalcon                        badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58

<SNIP>
```


---
**Target(s): 10.129.185.249**
**1. Enumerate valid usernames using Kerbrute and the wordlist located at /opt/jsmith.txt on the ATTACK01 host. How many valid usernames can we enumerate with just this wordlist from an unauthenticated standpoint?**

```
xfreerdp /u:htb-student /v:10.129.185.249 /p:HTB_@cademy_stdnt!
```

DC = Domain Controller (Controlador de Dominio).
Es el servidor que administra la autenticación y los objetos del dominio (usuarios, equipos, políticas) en Active Directory.
Servicios típicos en un DC: Kerberos (autenticación), LDAP (consultas de directorio), DNS, replicación de AD, y el repositorio de la base de datos AD (NTDS).

Vimos en secciones anteriores que el controlador de dominio tiene la IP: 172.16.5.5:
Ejectuamos:
```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 
```

Y encontramos como salida un total de 56 usuarios:
![[Pasted image 20251031152959.png]]

Respuesta: 56