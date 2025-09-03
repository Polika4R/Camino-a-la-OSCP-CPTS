# Protocolo SMB y Samba (Resumen para PYME)

SMB (Server Message Block) es un protocolo cliente-servidor que permite:
- Compartir archivos y carpetas.
- Acceder a impresoras, routers u otros recursos de red.
- Comunicación entre procesos de diferentes sistemas.

 Principalmente utilizado en sistemas **Windows**.  
 También disponible en **Linux/Unix** gracias a **Samba** (software libre).

## Funcionamiento general
- Usa **TCP/IP** como protocolo de transporte.
- Establece conexión mediante protocolo de enlace de 3 vías (3-way handshake).
- Utiliza **puerto TCP 445** para SMB directo (sin NetBIOS).
- Permite controlar permisos con **Listas de Control de Acceso (ACLs)**.
  - ACLs independientes de los permisos locales del sistema de archivos.

### Puertos utilizados

| Protocolo  | Puertos TCP   |
| ---------- | ------------- |
| NetBIOS    | 137, 138, 139 |
| SMB / CIFS | 445 (directo) |

## Versiones del protocolo SMB

| Versión      | Compatibilidad                          | Características principales                                      |
|--------------|------------------------------------------|------------------------------------------------------------------|
| **CIFS**     | Windows NT 4.0                           | Comunicación mediante NetBIOS                                    |
| **SMB 1.0**  | Windows 2000                             | Conexión directa por TCP                                         |
| **SMB 2.0**  | Windows Vista, Server 2008               | Mejoras de rendimiento, firma de mensajes, caché                 |
| **SMB 2.1**  | Windows 7, Server 2008 R2                | Mecanismos de bloqueo                                            |
| **SMB 3.0**  | Windows 8, Server 2012                   | Multicanal, cifrado, acceso a almacenamiento remoto              |
| **SMB 3.0.2**| Windows 8.1, Server 2012 R2              | —                                                                |
| **SMB 3.1.1**| Windows 10, Server 2016                  | Cifrado AES-128, verificación de integridad                      |

## Integración con Active Directory

- **Samba 3**: puede unirse a dominios de Active Directory.
- **Samba 4**: actúa como **Controlador de Dominio (DC)**.
- Daemons involucrados:
  - `smbd`: gestiona archivos e impresoras.
  - `nmbd`: implementa funciones de NetBIOS.
## Nombres de host y grupos de trabajo

- Cada host forma parte de un **grupo de trabajo** SMB.
- **NetBIOS** permite registrar nombres de host mediante:
  - **NBNS** (NetBIOS Name Server).
  - **WINS** (Windows Internet Name Service).

## Configuración predeterminada

Samba ofrece una amplia gama de opciones de configuración. 
Se definen mediante un archivo de texto ubicado en:
```shell-session
Polika4RM@htb[/htb]$ cat /etc/samba/smb.conf
```

Obseramos como predeterminado (obviando las líneas comentadas con #):
```shell-session
Polika4RM@htb[/htb]$ cat /etc/samba/smb.conf | grep -v "#\|\;" 

[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

Vemos la configuración global y dos recursos compartidos destinados a impresoras. La configuración global corresponde a la configuración del servidor SMB disponible, que se utiliza para todos los recursos compartidos. Sin embargo, en los recursos compartidos individuales, la configuración global puede sobrescribirse, lo que puede ser incluso incorrecto. 

#### Parámetros comunes en la configuración de Samba (smb.conf)
A continuación se muestra una tabla con los parámetros más utilizados en la configuración de Samba, útil para compartir directorios en red:

| **Configuración**                | **Descripción**                                                                 |
|----------------------------------|---------------------------------------------------------------------------------|
| `[sharename]`                    | El nombre del recurso compartido de red (sección del bloque).                  |
| `workgroup = WORKGROUP/DOMAIN`  | Grupo de trabajo o dominio que verá el cliente.                               |
| `path = /path/here/`            | Ruta local al directorio que se compartirá.                                    |
| `server string = STRING`        | Descripción del servidor al conectarse.                                        |
| `unix password sync = yes`      | Sincroniza las contraseñas UNIX con las de Samba.                              |
| `usershare allow guests = yes`  | Permite a usuarios no autenticados acceder a recursos compartidos.             |
| `map to guest = bad user`       | Trata a usuarios inválidos como invitados (guest).                             |
| `browseable = yes`              | El recurso será visible en el explorador de red.                               |
| `guest ok = yes`                | Permite acceder sin necesidad de autenticación.                                |
| `read only = yes`               | Permite solo lectura sobre los archivos del recurso compartido.                |
| `create mask = 0700`            | Permisos por defecto para archivos nuevos (solo el propietario puede acceder). |
Estos parámetros se configuran normalmente en el archivo `/etc/samba/smb.conf`


#### Configuraciones peligrosas en Samba (`smb.conf`)

Algunas configuraciones de Samba, aunque útiles para entornos internos o pruebas, pueden exponer recursos compartidos a accesos no deseados si no se gestionan adecuadamente. A continuación se describen configuraciones comunes que pueden representar  **riesgos de seguridad**: 

| **Configuración**        | **Descripción**                                                                 |
|--------------------------|---------------------------------------------------------------------------------|
| `browseable = yes`       | Permite que el recurso compartido aparezca en la lista de recursos disponibles. Esto facilita la navegación para empleados, pero también para atacantes. |
| `read only = no`         | Permite modificar archivos. Si no está bien restringido, se puede usar para cargar malware. |
| `writable = yes`         | Equivalente a `read only = no`. Permite escritura total en el recurso.          |
| `guest ok = yes`         | Permite acceso sin contraseña. Muy peligroso si se aplica a datos sensibles.   |
| `enable privileges = yes`| Otorga privilegios avanzados a ciertos usuarios mediante SIDs. Mal configurado, puede escalar privilegios. |
| `create mask = 0777`     | Archivos nuevos tienen permisos de lectura, escritura y ejecución para todos.  |
| `directory mask = 0777`  | Directorios nuevos también son accesibles y modificables por cualquiera.        |
| `logon script = script.sh` | Ejecuta un script al iniciar sesión. Si se manipula, puede usarse para persistencia o ejecución remota. |
| `magic script = script.sh` | Ejecuta un script especial cuando se detecta su creación. Otro vector de ejecución automática. |
| `magic output = script.out` | Guarda la salida del `magic script`. Útil para pentesting, pero también puede filtrar información. |
# Enumeración SMB con *smbclient*

Con el comando: 
```
Polika4RM@htb[/htb]$ smbclient -N -L //10.129.14.128
```
Puedo **ver que recursos compartidos existen.** Por defecto, print$ y IPC$ siempre están.:
```shell-session
Polika4RM@htb[/htb]$ smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

Para acceder a un recurso, ejecutaré:
```shell-session
smbclient //10.129.14.128/<nombre_rescurso>
```

Una vez dentro del recurso, puedo listar los recursos con:
```
smb: \> ls
```

Y descargarlos el recurso, en nuestro directorio desde donde hemos iniciado samba, con el comando:
```
smb: \> get <nombre_recurso>
```

# Estado de samba

Teniendo acceso en el servidor donde tenemos el servicio samba corriendo, podemos ver información de este mediante el comando:

```
root@samba:~# smbstatus
```

```shell-session
root@samba:~# smbstatus

Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -           
```


# RPCclient

Con RPCCLIENT puedo realizar varias solicitudes para obtener información:

```shell-session
rpcclient -U "" 1.2.3.4
```

| **Comando**                        | **Descripción**                                                                               |
| ---------------------------------- | --------------------------------------------------------------------------------------------- |
| `srvinfo`                          | Muestra información general del servidor.                                                     |
| `enumdomains`                      | Enumera todos los dominios visibles en la red.                                                |
| `querydominfo`                     | Muestra detalles del dominio, usuarios y controladores.                                       |
| `netshareenumall`                  | Lista todos los recursos compartidos disponibles (shares).                                    |
| `netsharegetinfo <nombre_recurso>` | Muestra información detallada sobre un recurso compartido específico.                         |
| `enumdomusers`                     | Enumera todos los usuarios del dominio.                                                       |
| `queryuser <RID>`                  | Proporciona información detallada sobre un usuario (requiere RID). Ver comando `enumdomusers` |
# smbMAP y CrackMapExec

La información que ya hemos obtenido `rpcclient`también puede obtenerse con otras herramientas. Por ejemplo, las herramientas **SMBMap** y **CrackMapExec** también son ampliamente utilizadas y útiles para la enumeración de servicios SMB.

**smbmap**:
Enumera recursos disponibles:
```shell-session
smbmap -H 1.2.3.4
```

![[Pasted image 20250801141018.png]]
O con crackmapexec podemos también listar estos recuros:

```shell-session
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

```shell-session
Polika4RM@htb[/htb]$ crackmapexec smb 10.129.14.128 --shares -u '' -p ''

SMB         10.129.14.128   445    DEVSMB           [*] Windows 6.1 Build 0 (name:DEVSMB) (domain:) (signing:False) (SMBv1:False)
SMB         10.129.14.128   445    DEVSMB           [+] \: 
SMB         10.129.14.128   445    DEVSMB           [+] Enumerated shares
SMB         10.129.14.128   445    DEVSMB           Share           Permissions     Remark
SMB         10.129.14.128   445    DEVSMB           -----           -----------     ------
SMB         10.129.14.128   445    DEVSMB           print$                          Printer Drivers
SMB         10.129.14.128   445    DEVSMB           home                            INFREIGHT Samba
SMB         10.129.14.128   445    DEVSMB           dev                             DEVenv
SMB         10.129.14.128   445    DEVSMB           notes           READ,WRITE      CheckIT
SMB         10.129.14.128   445    DEVSMB           IPC$                            IPC Service (DEVSM)
```


# enum4linux-ng - Uso básico

"enum4linux-ng" es una herramienta para **enumerar información en servidores SMB/Windows**. Permite recopilar datos valiosos durante auditorías o pentesting.

El parámetro `-A` ejecuta un **escaneo agresivo** que realiza varias consultas para obtener la máxima información posible del servidor SMB.


Instalación previa con:
```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/cddmp/enum4linux-ng.git
Polika4RM@htb[/htb]$ cd enum4linux-ng
Polika4RM@htb[/htb]$ pip3 install -r requirements.txt
```

Se ejecuta con:

```shell-session
Polika4RM@htb[/htb]$ ./enum4linux-ng.py 10.129.14.128 -A
```


**QUESTIONS**
**Target: 10.129.244.49**
**1. What version of the SMB server is running on the target system? Submit the entire banner as the answer.**
Ejecuto escaneo básico de versión de servicois, con:
```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 4.6.2
445/tcp  open  netbios-ssn Samba smbd 4.6.2
2049/tcp open  nfs_acl     3 (RPC #100227)
```

El servicio SMB (corriendo por el puerto 445), utiliza una versión *Samba smbd 4.6.2*

**2. What is the name of the accessible share on the target?**
Ejecuto una numeración de recursos compartidos, y observo:
```
smbclient -N -L //10.129.244.49

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	sambashare      Disk      InFreight SMB v3.1
	IPC$            IPC       IPC Service (InlaneFreight SMB server (Samba, Ubuntu))

```

Que el recurso compartido de denomina *sambashare*

** 3. Connect to the discovered share and find the flag.txt file. Submit the contents as the answer.**
Me conecto al recurso compartido con:
```
smbclient //10.129.244.49/sambashare
Password for [WORKGROUP\htb-ac-1876550]:
Try "help" to get a list of possible commands.
smb: \> 
```

Como contraseña no introduzco nada (doy enter sin ningun texto). Dentro del servicio accedo a la carpeta "contents" con un *cd contents*, me descargo el archivo flag.txt con *get flag.txt* y haciendo un cat desde mi máquina Kali, observo que la flag es: *HTB{o873nz4xdo873n4zo873zn4fksuhldsf}*

**4. Find out which domain the server belongs to.**
Accedo al cliente rcp con el comando:
```
rpcclient -U "" 10.129.244.49
```
Ingreso una contraseña nula (le doy al enter sin escribir contraseña).
Y ejecuto:
```
rpcclient $> querydominfo
Domain:		DEVOPS
Server:		DEVSMB
Comment:	InlaneFreight SMB server (Samba, Ubuntu)
Total Users:	0
Total Groups:	0
Total Aliases:	0
Sequence No:	1756882846
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
```

Siendo al respuesta al ejercicio el dominio: *DEVOPS*

**5. Find additional information about the specific share we found previously and submit the customized version of that specific share as the answer.**

Dentro del servicio RCP establecido en el ejercicio anterior, ejecuto:
```
rpcclient $> netsharegetinfo sambashare
netname: sambashare
	remark:	InFreight SMB v3.1
	path:	C:\home\sambauser\
	password:	
	type:	0x0
	perms:	0
	max_uses:	-1
	num_uses:	1
revision: 1
type: 0x8004: SEC_DESC_DACL_PRESENT SEC_DESC_SELF_RELATIVE 
DACL
	ACL	Num ACEs:	1	revision:	2
	---
	ACE
		type: ACCESS ALLOWED (0) flags: 0x00 
		Specific bits: 0x1ff
		Permissions: 0x1f01ff: SYNCHRONIZE_ACCESS WRITE_OWNER_ACCESS WRITE_DAC_ACCESS READ_CONTROL_ACCESS DELETE_ACCESS 
		SID: S-1-1-0
```

Siendo al respuesta del ejercicio *InFreight SMB v3.1*

**5.  What is the full system path of that specific share? (format: "/directory/names")**
En el mismo output anterior, veo que el path del recurso sambashare es */home/sambauser*
