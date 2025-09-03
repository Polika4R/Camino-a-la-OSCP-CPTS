**SMTP = SIMPLE MAIL TRANSFER PROTOCOL.**
Protocolo para enviar correos electrónicos en una red IP.

Puede utilizarse  entre:
- cliente y servidor
- servidor y servidor

- **Puertos usados:**
    - Puerto estándar: **25** (sin cifrado por defecto).
    - Puerto **587** para envío con autenticación y cifrado STARTTLS.
    - Puerto **465** para conexiones cifradas SSL/TLS.

- **Autenticación y seguridad:**
    - SMTP originalmente transmite todo en texto plano, incluyendo credenciales.
    - Se usa STARTTLS o SSL/TLS para cifrar la conexión y proteger datos y autenticación.
    - Muchos servidores usan ESMTP con SMTP-Auth para autenticar usuarios y evitar spam.

- **Componentes del sistema de correo:**
    - **MUA (Mail User Agent):** cliente que crea el correo.
    - **MSA (Mail Submission Agent):** servidor que valida el correo y su origen (también llamado Relay server).
    - **MTA (Mail Transfer Agent):** software que envía, recibe, revisa y almacena correos, buscando en DNS el servidor destino.

- **Problemas comunes:**
    - **Open Relay Attack:** ataque posible si un servidor SMTP está mal configurado y permite enviar correos no autorizados a través de él.

#### Flujo del correo al llegar al destino
    
El servidor SMTP destino recibe los paquetes, los ensambla para formar el correo completo.
El **Mail Delivery Agent (MDA)** transfiere el correo a la bandeja del destinatario, que puede usarse con protocolos POP3 o IMAP.

- **Cadena típica de entrega:**
    - Cliente (MUA) → Submission Agent (MSA) → Open Relay (MTA) → Mail Delivery Agent (MDA) → Mailbox (POP3/IMAP).

- **Dos desventajas principales de SMTP:**
    1. **Confirmación de entrega poco fiable:** No existe un formato estándar para confirmaciones, por lo que generalmente solo se recibe un mensaje de error en inglés si falla la entrega.
    2. **Falta de autenticación fiable al inicio:** El remitente puede falsificarse, lo que facilita ataques como el envío masivo de spam (mail spoofing) a través de relays abiertos.

- **Medidas de seguridad comunes:**
    - Uso de técnicas para filtrar correos sospechosos o moverlos a la carpeta de spam.
    - Protocolos de verificación como **DKIM** (DomainKeys Identified Mail) y **SPF** (Sender Policy Framework).
        
- **ESMTP (Extended SMTP):**
    - Es una extensión de SMTP que incluye soporte para cifrado mediante TLS.
    - Tras el comando EHLO, se usa **STARTTLS** para iniciar una conexión cifrada SSL/TLS.
    - Con la conexión cifrada, se puede usar de forma segura la extensión **AUTH PLAIN** para autenticar usuarios.

Existe un archivo de configuración ubicado en:
```
Polika4RM@htb[/htb]$ cat /etc/postfix/main.cf
```
Que recopila las características principales de configuración.
#### Telnet - HELO/EHLO
Para interactuar con un servidor SMTP, utilizaremos la herramienta TELNET.

Para hacerlo, ejecutaremos:

```
Polika4RM@htb[/htb]$ telnet <ip_servidor> <puerto_SMTP>   
				    #telnet 10.129.14.128 25
```

Dentro de la sesión, podemos ejecutar:
```
>EHLO inlanefreight.htb
```
- **HELO** es el primer mensaje que envía un cliente SMTP para presentarse ante el servidor SMTP.
- Le dice al servidor: _"Hola, aquí está el nombre de mi máquina (host), quiero empezar a enviar correos"_.
- El servidor responde con un saludo, confirmando que reconoce ese nombre y acepta la conexión.

O podemos ejecutar:
```
>EHLO mail1
```
- El cliente se presenta y el servidor devuelve todas las capacidades y extensiones que el servidor SMTP admite y que el cliente puede usar para enviar corres de forma más eficiente y segura. 


Ejemplo de uso:

```shell-session
Polika4RM@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 


HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb


EHLO mail1

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

#### Telnet - VRFY

O podemos ejecutar:
```
VRFY <nombre_usuario>
```

- **VRFY** se usa para verificar si un usuario o buzón de correo existe en el servidor SMTP.
- Por ejemplo, al escribir `VRFY root`, el servidor debería responder si ese usuario existe o no.

```shell-session
Polika4RM@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 

VRFY root
252 2.0.0 root


VRFY cry0l1t3
252 2.0.0 cry0l1t3


VRFY testuser
252 2.0.0 testuser


VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa
252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

### Uso de web proxy para acceder a SMTP

- En algunos entornos, el acceso directo al servidor SMTP puede estar bloqueado o restringido.
- Para sortear esto, se puede usar un **web proxy** que actúe como intermediario.
- El comando para pedir al proxy que abra una conexión hacia el servidor SMTP sería:

 ```
 CONNECT 10.129.14.128:25 HTTP/1.0
 ```   

- Así, el proxy establece la conexión con el servidor SMTP y permite que el cliente comunique con él a través del proxy.

## Enviar un email
- Los comandos usados para enviar un correo desde la línea de comandos son los mismos que usan clientes de correo (Thunderbird, Gmail, Outlook, etc.).
Podemos usar:    
    - **Subject** (asunto)
    - **To** (destinatario)
    - **CC** (copia)
    - **BCC** (copia oculta)
    - El contenido del mensaje.

1. Nos **conectamos** al servidor SMTP:
```
	telnet <ip_servidor> <puerto_SMTP_25>
``` 

2. **Saludo e inicio de sesión (EHLO):**
```
EHLO inlanefreight.htb
```

3. Especificamos **remitente** (MAIL FROM):
```
MAIL FROM: <cry0l1t3@inlanefreight.htb>
```
4. Especificamos **destinatario** (RCPT TO):
```
RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure
```
5. Enviamos contenido (DATA):
```
DATA
From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200

Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.
```
El punto (`.`) en línea nueva indica fin del mensaje.

6. Cerramos sesión (QUIT):
```
QUIT
```


Ejemplo completo:
```shell-session
Polika4RM@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


EHLO inlanefreight.htb

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING


MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.

250 2.0.0 Ok: queued as 6E1CF1681AB


QUIT

221 2.0.0 Bye
Connection closed by foreign host.
```


Configurar `mynetworks = 0.0.0.0/0` **es muy peligroso** porque abre el servidor SMTP a cualquier persona para que lo use de manera maliciosa. Es una práctica a evitar para proteger la seguridad y la reputación del sistema de correo.


### 1. Escaneo básico y detección de versión del servicio SMTP en el puerto 25

```
sudo nmap 10.129.14.128 -sC -sV -p25
```
- `-sC`: Ejecuta scripts de Nmap por defecto (incluye `smtp-commands` para listar comandos SMTP soportados).
    
- `-sV`: Detecta la versión del servicio.
    
- `-p25`: Escanea solo el puerto 25 (SMTP).

### 2. Escaneo específico para detectar Open Relay SMTP usando script NSE

```
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

- `--script smtp-open-relay`: Ejecuta el script NSE que realiza 16 tests para comprobar si el servidor SMTP es un open relay.
- `-v`: Modo verbose para mostrar información detallada del escaneo.

Resultado esperado para Open Relay:
- `Server is an open relay (16/16 tests)` indica que el servidor permite enviar correos sin restricciones.
- Se muestran diferentes pruebas de envío (combinaciones MAIL FROM / RCPT TO).


## Enumeración de usuarios
Con el comando:
```
nmap --script smtp-enum-users.nse  -p 25,465,587 <host>
```

O con el comando:
```
smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 10.129.3.215 -w 120 -v
```

O con el MSFCONSOLE:
```
msfconsole
use scanner/smtp/smtp_enum
set RHOST <ip_vícitma>
run
```

