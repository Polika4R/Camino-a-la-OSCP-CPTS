Microsoft SQL (MSSQL) es un sistema de gestión de bases de datos relacionales de Microsoft, basado en SQL. A diferencia de MySQL, MSSQL es de código cerrado y fue diseñado originalmente para Windows, aunque ahora también tiene versiones para Linux y macOS. Es especialmente popular en entornos .NET por su integración nativa con esta plataforma.

Para administrar MSSQL, se utiliza SQL Server Management Studio (SSMS), una aplicación cliente que puede instalarse tanto en el servidor como en cualquier otro equipo desde donde se gestione la base de datos. Esto implica que sistemas con SSMS y credenciales guardadas pueden representar un riesgo si son vulnerables, pues permitirían acceso directo a la base de datos.

Se pueden usar muchos otros clientes para acceder a una base de datos que se ejecuta en MSSQL. Entre ellos se incluyen:
- mssql-cli	
- PowerShell de SQL Server
- HeidiSQL	SQL Pro	
- mssqlclient.py de Impacket


Entre los clientes MSSQL mencionados (mssql-cli, SQL Server PowerShell, HeidiSQL, SQLPro e Impacket's mssqlclient.py), los pentesters suelen preferir **mssqlclient.py** de Impacket porque este proyecto viene preinstalado en muchas distribuciones de pentesting, como KALI.

Para saber si está instalado y dónde se encuentra, se puede usar el comando `locate mssqlclient`:

```shell-session
Polika4RM@htb[/htb]$ locate mssqlclient

/usr/bin/impacket-mssqlclient
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

-----

MSSQL cuenta con bases de datos de sistema predeterminadas que nos ayudan a comprender la estructura de todas las bases de datos que pueden estar alojadas en un servidor de destino. 
A continuación, se presentan las bases de datos predeterminadas y una breve descripción de cada una:

|Base de datos|Descripción|
|---|---|
|**master**|Registra toda la información del sistema para la instancia del servidor SQL.|
|**model**|Base plantilla que sirve como estructura para cada nueva base de datos creada. Los cambios en esta base afectan a las nuevas bases creadas.|
|**msdb**|Usada por el SQL Server Agent para programar trabajos y alertas.|
|**tempdb**|Almacena objetos temporales durante la ejecución de consultas y procesos.|
|**resource**|Base de datos de solo lectura que contiene objetos del sistema incluidos con SQL Server.|

----
# Configuración por defecto
Cuando un administrador instala y configura MSSQL para que sea accesible en red, el servicio de SQL normalmente se ejecuta bajo la cuenta **NT SERVICE\MSSQLSERVER**.

La conexión desde el cliente suele hacerse mediante **Autenticación de Windows**. Además, por defecto, **no se aplica cifrado obligatorio** en las conexiones, lo que puede implicar riesgos de seguridad si no se protege adecuadamente la comunicación.

Cuando MSSQL está configurado para usar **Windows Authentication**, el sistema operativo Windows se encarga de procesar la solicitud de inicio de sesión, validando las credenciales contra la base local SAM o contra un controlador de dominio que hospeda Active Directory.

- **Ventajas:** Usar Active Directory facilita la auditoría de actividad y el control de acceso en entornos Windows.
    
- **Riesgos:** Si una cuenta es comprometida, puede permitir **escalada de privilegios** y movimientos laterales dentro del dominio.
    

Para comprender mejor la configuración por defecto y posibles errores del administrador, es recomendable hacer pruebas en un entorno controlado, como una máquina virtual, desde la instalación hasta la configuración.

---

### Configuraciones peligrosas

Adoptar la perspectiva de un administrador IT durante una auditoría o pentesting ayuda a identificar configuraciones inseguras. Los administradores suelen estar bajo presión y con múltiples proyectos, lo que puede generar errores que comprometan servicios críticos, incluyendo MSSQL.

Aunque las configuraciones pueden variar mucho según la organización, algunos aspectos comunes que conviene revisar son:

- Clientes MSSQL que **no usan cifrado** para conectarse al servidor.
    
- Uso de **certificados autofirmados** cuando se aplica cifrado, los cuales pueden ser susceptibles a suplantación.
    
- Uso de **named pipes**, que pueden representar un vector de ataque.
    
- Credenciales débiles o por defecto del usuario **sa** (administrador de MSSQL), ya que a menudo este usuario no se desactiva.

----

# FOOTPRINTING

## SCRIPTS DE NMAP
**NMAP cuenta con scripts de MSSQL predeterminados que pueden usarse para acceder al puerto TCP 1433 predeterminado, en el que escucha MSSQL.**


El siguiente script de NMAP con script proporciona información útil. Podemos ver que el nombre de host, el nombre de la instancia de la base de datos, la versión del software de MSSQL y si las canalizaciones con nombre están habilitadas. Nos resultará útil añadir estos descubrimientos a nuestras notas.


```shell-session
Polika4RM@htb[/htb]$ sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

Salida:
```shell-session
Polika4RM@htb[/htb]$ sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248

Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-08 09:40 EST
Nmap scan report for 10.129.201.248
Host is up (0.15s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: SQL-01
|   NetBIOS_Domain_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Domain_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763

Host script results:
| ms-sql-dac: 
|_  Instance: MSSQLSERVER; DAC port: 1434 (connection failed)
| ms-sql-info: 
|   Windows server name: SQL-01
|   10.129.201.248\MSSQLSERVER: 
|     Instance name: MSSQLSERVER
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|     Named pipe: \\10.129.201.248\pipe\sql\query
|_    Clustered: false

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.52 seconds
```
## METASPLOIT

Usando metasploit, podemos utilizar el siguiente módulo.
```
msfconsole
use scanner/mssql_ping
	set rhosts 1.2.3.4
	run
```

## MSSQLCLIENT.py

**Si partimos de credenciales válidas**, podemos conectarnos remotamente al servidor MSSQL y comenzar a interactuar con las bases de datos usando: T-SQL (Transact-SQL). 
Autenticándonos correctamente, podremos ejecutar comandos directamente en el motor de bases de datos.

Desde la máquina víctima o cualquier máquina atacante, se puede usar Impacket y su cliente `mssqlclient.py` para conectarse, por ejemplo:

```
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

Tras introducir la contraseña, la conexión se establece con cifrado TLS si es necesario, y se configura el contexto (base de datos, idioma, tamaño de paquete, etc.).

Una vez conectados, es buena práctica obtener un panorama general del sistema listando las bases de datos disponibles con esta consulta:

```
select name from sys.databases`
```

El resultado suele mostrar las bases de datos por defecto como:
- master
- tempdb
- model
- msdb


Captura general:
```shell-session
Polika4RM@htb[/htb]$ python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-01): Line 1: Changed database context to 'master'.
[*] INFO(SQL-01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands

SQL> select name from sys.databases

name                                                           -----------------------------------------------------------------------------------
master                                                         
tempdb                                                         
model                                                          
msdb                                                           Transactions    
```



------------

**\[+1 CUBE ] Enumerate the target using the concepts taught in this section. List the hostname of MSSQL server.

Ejecutamos el comando:
```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.137.255
```

Y observamos una línea donde aparece:
```
_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.129.137.255:1433: 
|     Target_Name: ILF-SQL-01
|     NetBIOS_Domain_Name: ILF-SQL-01
|     NetBIOS_Computer_Name: ILF-SQL-01
|     DNS_Domain_Name: ILF-SQL-01
|     DNS_Computer_Name: ILF-SQL-01
|_    Product_Version: 10.0.17763

```

Siendo la respuesta: ***ILF-SQL-01***


**\[+1 CUBE ] Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server.

Localizamos el cliente.py necesario para conectarnos a dicha base de dataos:
```
locate mssqlclient.py
/opt/pipx/venvs/netexec/bin/mssqlclient.py
/root/.local/bin/mssqlclient.py
/root/.local/share/pipx/venvs/impacket/bin/mssqlclient.py
/usr/local/bin/mssqlclient.py
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

Ejecutamos desde dicha ruta:

```
python3 mssqlclient.py backdoor@10.129.137.255 -windows-auth
```

Ingresamos la contraseña *Password1*

Y ejecutamos desde dentro de la base de datos un:
```
SQL (ILF-SQL-01\backdoor  dbo@master)> select name from sys.databases
```


