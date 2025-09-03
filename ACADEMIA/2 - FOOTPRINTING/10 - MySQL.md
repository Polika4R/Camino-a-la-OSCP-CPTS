MySQL es un sistema de gestión de bases de datos relacional (RDBMS) de código abierto desarrollado por Oracle. Utiliza SQL para gestionar datos estructurados en tablas de filas y columnas, funcionando bajo un modelo cliente-servidor con un servidor que almacena y distribuye la información y uno o varios clientes que la consultan. Está optimizado para alto rendimiento y uso eficiente del espacio, y suele guardar bases de datos en archivos `.sql`.

Una **base de datos relacional** organiza la información en tablas que pueden relacionarse entre sí mediante **claves primarias** (identifican registros) y **claves foráneas** (conectan tablas). Esto evita duplicaciones, mantiene la integridad de los datos y permite consultas complejas con SQL, como vincular una tabla de clientes con otra de pedidos para saber quién compró qué.

## Clientes MySQL
**Los clientes MySQL pueden recuperar y editar los datos mediante consultas estructuradas al motor de base de datos.**  La inserción, eliminación, modificación y recuperación de datos se realiza mediante el lenguaje de bases de datos SQL.

Uno de los mejores ejemplos del uso de bases de datos es el **CMS WordPress**. WordPress almacena todas las publicaciones, nombres de usuario y contraseñas creadas en su propia base de datos, a la que solo se puede acceder desde el host local.


## MySQL databases
Suele combinarse con un sistema operativo Linux, PHP y un servidor web Apache, y también se conoce en esta combinación como LAMP (Linux, Apache, MySQL, PHP), o al usar Nginx, como LEMP.


# Default Configuration

La configuración predeterminada de MySQL se resume a:

```shell-session
Polika4RM@htb[/htb]$ sudo apt install mysql-server -y
Polika4RM@htb[/htb]$ cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'

[client]
port		= 3306
socket		= /var/run/mysqld/mysqld.sock

[mysqld_safe]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
skip-host-cache
skip-name-resolve
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql
explicit_defaults_for_timestamp

symbolic-links=0

!includedir /etc/mysql/conf.d/
```

## Riesgos y amenazas
MySQL puede presentar riesgos si está mal configurado. Opciones clave para la seguridad incluyen:

- **usuario**, **contraseña**, **admin_address**: definen credenciales y dirección de escucha; si el archivo de configuración tiene permisos inseguros, pueden leerse en texto plano y permitir acceso total a la base de datos y datos sensibles.
- **debug** y **sql_warnings**: muestran información detallada en errores, útil para administradores pero peligrosa si es visible públicamente, pues podría facilitar ataques, incluso ejecución de comandos del sistema mediante inyección SQL.
- **secure_file_priv**: limita importaciones/exportaciones de datos para reducir riesgos.
    
Una mala gestión de estas configuraciones puede exponer información crítica y abrir la puerta a ataques.


# Footprinting the Service

Normalmente, el servidor MySQL se ejecuta en el puerto TCP 3306, y podemos escanearlo con Nmap para obtener información más detallada.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-21 00:53 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE     VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-brute: 
|   Accounts: 
|     root:<empty> - Valid credentials
|_  Statistics: Performed 45010 guesses in 5 seconds, average tps: 9002.0
|_mysql-databases: ERROR: Script execution failed (use -d to debug)
|_mysql-dump-hashes: ERROR: Script execution failed (use -d to debug)
| mysql-empty-password: 
|_  root account has empty password
| mysql-enum: 
|   Valid usernames: 
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     web:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 seconds, average tps: 10.0
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Thread ID: 13
|   Capabilities flags: 65535
|   Some Capabilities: SupportsLoadDataLocal, SupportsTransactions, Speaks41ProtocolOld, LongPassword, DontAllowDatabaseTableColumn, Support41Auth, IgnoreSigpipes, SwitchToSSLAfterHandshake, FoundRows, InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, LongColumnFlag, SupportsCompression, ODBCClient, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: YTSgMfqvx\x0F\x7F\x16\&\x1EAeK>0
|_  Auth Plugin Name: caching_sha2_password
|_mysql-users: ERROR: Script execution failed (use -d to debug)
|_mysql-variables: ERROR: Script execution failed (use -d to debug)
|_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.21 seconds
```

Debemos ser cuidadosos con los resultados y **confirmar manualmente la información obtenida**, ya que parte de ella podría resultar un **falso positivo**. El análisis anterior es un excelente ejemplo de ello, ya que sabemos con certeza que el servidor MySQL de destino no usa una contraseña vacía para el usuario root, sino una contraseña fija.


# Interactuando con un servidor MySQL

- Conectarse a servidor MySQL con usuario especificado \<root>, sin proporcionar contraseña.
```shell-session
Polika4RM@htb[/htb]$ mysql -u root -h 10.129.14.132
```

- Conectarse a servidor MySQL con usuario y contraseña especifidada.
```shell-session
Polika4RM@htb[/htb]$ mysql -u root -pP4SSw0rd -h 10.129.14.128
```

Dentro de la base de datos, podemos hacer un:
>show databases;
>select version();
>use mysql;
>show tables;


Si analizamos las bases de datos existentes, veremos que ya existen varias. Las bases de datos más importantes para el servidor MySQL son el esquema de sistema (sys) y el esquema de información (information_schema). El esquema de sistema contiene tablas, información y metadatos necesarios para la gestión. Puede encontrar más información sobre esta base de datos en el manual de referencia de MySQL.


Resumen de comandos MySQL:

- **`mysql -u <user> -p<password> -h <IP>`** → Conecta al servidor MySQL (sin espacio tras `-p`).
- **`show databases;`** → Muestra todas las bases de datos.
- **`use <database>;`** → Selecciona una base de datos.
- **`show tables;`** → Lista las tablas de la base seleccionada.
- **`show columns from <table>;`** → Muestra las columnas de una tabla.
- **`select * from <table>;`** → Muestra todo el contenido de la tabla.
- **`select * from <table> where <column> = "<string>";`** → Busca un valor específico en una columna.

------

**\[+1 CUBE ] Enumerate the MySQL server and determine the version in use. (Format: MySQL X.X.XX)

Ejecutamos:
```
sudo nmap -sSCV --open --min-rate 2000 -Pn -n -v 10.129.42.195
```
Y entre otros datos, encontramos:

```
3306/tcp open  mysql    MySQL 8.0.27-0ubuntu0.20.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.27-0ubuntu0.20.04.1
|   Thread ID: 9
|   Capabilities flags: 65535

```

Respuesta: ***MySQL 8.0.27***

**\[+1 CUBE ] During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"?

En primer lugar, accedemos a la base de datos con las credenciales facilitadas con:
```
mysql -u robin -probin -h 10.129.42.195
```

Hacemos un *show databases* y observamos:
```
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| customers          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

Seleccionamos la columna *"customers"*:
```
MySQL [(none)]> use customers
```

Miramos que tablas tenemos disponibles, con *show tables*:
```
MySQL [customers]> show tables;
+---------------------+
| Tables_in_customers |
+---------------------+
| myTable             |
+---------------------+
```

Y fianalmente con un:
```
MySQL [customers]> select * from myTable;
```

Observamos que la respuesta es: ***ultrices\@google.htb***


