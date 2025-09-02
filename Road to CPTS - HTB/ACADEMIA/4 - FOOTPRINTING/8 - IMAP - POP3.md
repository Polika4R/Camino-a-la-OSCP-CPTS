
**IMAP vs POP3**
- **IMAP (Internet Message Access Protocol):**  
    Permite gestionar correos directamente en el servidor, con soporte para carpetas y sincronización entre múltiples clientes. Los emails permanecen en el servidor hasta que se eliminan. Ofrece funciones avanzadas como acceso simultáneo de varios usuarios, creación de carpetas personales, y manejo online de correos. Requiere conexión activa para gestionar correos, aunque algunos clientes permiten modo offline sincronizando luego los cambios. Usa el puerto 143 (o 993 con SSL/TLS para seguridad).  
    Ideal para quienes usan varios dispositivos y quieren mantener sus correos sincronizados y organizados.
    
- **POP3 (Post Office Protocol 3):**  
    Protocolo más simple que solo permite listar, descargar y borrar emails del servidor. No soporta carpetas ni sincronización compleja. Generalmente descarga los correos al cliente y los elimina del servidor, dificultando el acceso desde varios dispositivos.
    
- **Seguridad:**  
    IMAP transmite datos en texto plano por defecto, por lo que es común usar cifrado SSL/TLS (puertos 993 o 143 con STARTTLS) para proteger las credenciales y el contenido del correo.
    
- **Envío de correos:**  
    Para enviar emails se usa generalmente SMTP, y con IMAP se pueden sincronizar los correos enviados copiándolos en la carpeta correspondiente en el servidor.

### Configuración por defecto
- IMAP y POP3 tienen muchas opciones configurables, lo que dificulta un análisis profundo sin práctica directa.
- Se recomienda experimentar instalando en una VM los paquetes **dovecot-imapd** y **dovecot-pop3d** (por ejemplo, con `apt`) para probar configuraciones.

- La documentación oficial de Dovecot ofrece todos los detalles sobre configuraciones básicas y servicios.
### Comandos básicos para interactuar con los protocolos (desde línea de comandos)

**IMAP:**

| Comando                       | Descripción                               |
| ----------------------------- | ----------------------------------------- |
| `LOGIN username password`     | Autenticar usuario                        |
| `LIST "" *`                   | Listar todos los directorios (buzones)    |
| `CREATE "INBOX"`              | Crear un buzón                            |
| `DELETE "INBOX"`              | Borrar un buzón                           |
| `RENAME "ToRead" "Important"` | Renombrar un buzón                        |
| `LSUB "" *`                   | Listar buzones suscritos                  |
| `SELECT INBOX`                | Seleccionar buzón para acceder a mensajes |
| `UNSELECT INBOX`              | Salir del buzón seleccionado              |
| `FETCH <ID> all`              | Obtener datos de un mensaje por ID        |
| `CLOSE`                       | Eliminar mensajes marcados para borrar    |
| `LOGOUT`                      | Cerrar conexión                           |

**POP3:**

|Comando|Descripción|
|---|---|
|`USER username`|Identificar usuario|
|`PASS password`|Autenticarse con contraseña|
|`STAT`|Obtener número de correos guardados|
|`LIST`|Listar número y tamaño de correos|
|`RETR id`|Descargar correo por ID|
|`DELE id`|Eliminar correo por ID|
|`CAPA`|Mostrar capacidades del servidor|
|`RSET`|Resetear transacciones actuales|
|`QUIT`|Cerrar conexión|

### Configuraciones peligrosas y riesgos

- Configuraciones incorrectas pueden exponer datos sensibles (emails, contraseñas, etc.).
- Ejemplos de configuraciones arriesgadas en Dovecot:
    - `auth_debug`: Activa logging detallado de autenticación.
    - `auth_debug_passwords`: Muestra contraseñas en logs.
    - `auth_verbose`: Registra intentos fallidos de login y razones.
    - `auth_verbose_passwords`: Contraseñas usadas quedan registradas (a veces truncadas).
    - `auth_anonymous_username`: Permite login anónimo con un usuario específico, similar a FTP.

- Muchas empresas usan proveedores externos (Google, Microsoft), pero otras mantienen servidores propios para controlar la privacidad.
- Un mal ajuste puede permitir leer correos sensibles o interceptar credenciales.

### Footprinting del servicio IMAP y POP3

- **Puertos por defecto:**
    - **POP3:**
        - Puerto **110** (sin cifrado
        - Puerto **995** (con TLS/SSL)
    - **IMAP:**
        - Puerto **143** (sin cifrado)
        - Puerto **993** (con TLS/SSL)

- **Uso de puertos cifrados:**  
    Los puertos altos (993 para IMAP y 995 para POP3) utilizan TLS/SSL para proteger la comunicación cliente-servidor.
    
- **Escaneo con Nmap:**
    - Podemos usar Nmap para detectar estos puertos abiertos y servicios.
    - Si el servidor utiliza un certificado embebido para TLS/SSL, Nmap podrá detectar y mostrar información relacionada con el certificado durante el escaneo.

```
nmap -p 110,995,143,993 --script ssl-cert <IP-del-servidor>
```
- Esto escanea los puertos típicos de IMAP y POP3, e intenta obtener información del certificado SSL si está presente.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 22:09 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00026s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE SASL STLS TOP UIDL RESP-CODES CAPA PIPELINING
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: more have post-login STARTTLS Pre-login capabilities LITERAL+ LOGIN-REFERRALS OK LOGINDISABLEDA0001 SASL-IR ENABLE listed IDLE ID IMAP4rev1
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
993/tcp open  ssl/imap Dovecot imapd
|_imap-capabilities: more have post-login OK capabilities LITERAL+ LOGIN-REFERRALS Pre-login AUTH=PLAINA0001 SASL-IR ENABLE listed IDLE ID IMAP4rev1
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE USER SASL(PLAIN) TOP UIDL RESP-CODES CAPA PIPELINING
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.74 seconds
```

- Del resultado (por ejemplo, con Nmap) se puede obtener información valiosa como:
    - **Common Name (CN)** del certificado SSL, que indica el nombre del servidor, por ejemplo: `mail1.inlanefreight.htb`.
    - La organización propietaria del servidor de correo, en este caso, **Inlanefreight** ubicada en California.
    - Las **capacidades del servidor**, es decir, los comandos que soporta el servicio en el puerto escaneado (como LOGIN, LIST, FETCH para IMAP).
        
- **Riesgos de seguridad:**
    
    - Si un atacante logra obtener credenciales válidas (usuario y contraseña) de algún empleado, podría:
        - Acceder al servidor de correo legítimamente.
        - Leer correos privados y confidenciales.
        - Enviar correos en nombre del empleado, facilitando ataques de phishing o suplantación.
- Esto subraya la importancia de proteger las credenciales, usar cifrado (SSL/TLS), y monitorizar accesos al servidor de correo.

## Conexión a servidor IMAP con `curl`
```
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd
```

- `curl`: herramienta para transferir datos.
- `-k`: omite la verificación del certificado SSL (útil si es autofirmado).
- `imaps://`: indica conexión IMAP sobre SSL (puerto 993).
- `--user user:password`: autentica con usuario y contraseña.
- Resultado: **muestra los buzones o carpetas disponibles (ej. INBOX, Important)**

```shell-session
Polika4RM@htb[/htb]$ curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

* LIST (\HasNoChildren) "." Important
* LIST (\HasNoChildren) "." INBOX
```

Comando con modo verbose:

```
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v
```

- `-v`: muestra detalles de la conexión
    - Negociación TLS (versión y cifrado).
    - Información del certificado SSL (emisor, validez).
    - Banner del servidor IMAP (nombre y versión).
- Permite verificar la seguridad y características del servidor.

### Comando para POP3S (POP3 sobre SSL)

```
openssl s_client -connect 10.129.14.128:pop3s
```

- `openssl s_client`: herramienta para conectarse a servicios SSL/TLS.
- `-connect 10.129.14.128:pop3s`: conecta al puerto seguro POP3 (normalmente 995).
- Permite interactuar manualmente con el servidor POP3 cifrado.
- Útil para probar la conexión, ver certificado y **enviar comandos POP3.**
### Comando para IMAPS (IMAP sobre SSL)
```
openssl s_client -connect 10.129.14.128:imaps
```

- Similar al anterior, pero para el puerto seguro IMAP (normalmente 993).
- Permite probar conexión SSL/TLS y **enviar comandos IMAP manualmente.**

Los comandos que podemos utilizar los encontramos en:
https://www.atmail.com/blog/imap-101-manual-imap-sessions/

**Procesos seguido para la resolución del ejercicio:**
(seguimos esta guía):
1. Nos conectamos a "openssl s_client -connect 10.129.70.219:imaps".
   Desde este momento, ya podemos ingresar comandos.

---------------

**\[+1 CUBE ] Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer.*

Ejecuto para escanear los puertos POP3/IMAP con análisis de certificado SSL/TLS:
>nmap -p 110,995,143,993 --script ssl-cert 10.129.42.195

```
PS [10.10.15.48] /home/htb-ac-1876550 > nmap -p 110,995,143,993 --script ssl-cert 10.129.42.195     Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-05 01:10 CDT
Nmap scan report for 10.129.42.195
Host is up (0.0018s latency).

PORT    STATE SERVICE
110/tcp open  pop3
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Issuer: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
```

Y observo como efectivamente encuentro "organizationName" = *"InlaneFreight"*

**\[+1 CUBE ] What is the FQDN that the IMAP and POP3 servers are assigned to?**
En la misma captura anterior, encuentro que el dominio es: *"dev.inlanefreight.htb"*

**\[+1 CUBE ] Enumerate the IMAP service and submit the flag as the answer. (Format: HTB{...})**
Con el comando: 
```
openssl s_client -connect 10.129.42.195:imaps
```
Además de conectarme a la sessión imaps, puedo ver la flag oculta:
HTB{roncfbw7iszerd7shni7jr2343zhrj}

**\[+1 CUBE ]  What is the customized version of the POP3 server?

Puedo realizar simplemente un:
```
PS [10.10.15.48] /home/htb-ac-1876550 > nc -nv 10.129.42.195 110
```

Y ver que es la versión *InFreight POP3 v9.188*

**\[+1 CUBE ]  What is the admin email address?**
Partimos de unas credenciales dadas por el ejercicio, las cuales son robin:robin.

Me conecto al servicio IMAP (para indagar en los correos existentes) con el comando:
```
PS [10.10.15.48] /home/htb-ac-1876550 > openssl s_client -connect 10.129.42.195:imaps

```

Inicio sesión con las credenciales de *robin:robin*:
```
1 LOGIN robin robin
```

Ejecutando:
```
1 namespace
```

Me lista el conjunto de buzones sobre el cual el usuario con el que me he registrado tiene acceso.


Me devuelve un:
```
1 namespace
* NAMESPACE (("" ".")) NIL NIL
1 OK Namespace completed (0.001 + 0.000 secs).

```

Pues, tengo un buzón sin nombre "" el cual quiero listar todo su contenido "\*"

```
1 list "" "*" 
```

Devolviendo:
```
1 list "" "*"           
* LIST (\Noselect \HasChildren) "." DEV
* LIST (\Noselect \HasChildren) "." DEV.DEPARTMENT
* LIST (\HasNoChildren) "." DEV.DEPARTMENT.INT
* LIST (\HasNoChildren) "." INBOX
1 OK List completed (0.001 + 0.000 secs).

```

Voy a seleccionar "DEV.DEPARTMENT.INT"
```
1 SELECT DEV.DEPARTMENT.INT
```

Y me devuelve:
```
1 SELECT DEV.DEPARTMENT.INT
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1636414279] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
1 OK [READ-WRITE] Select completed (0.006 + 0.000 + 0.006 secs).

```

Veo que hay 1 correo al que puedo acceder debido a *"1 EXISTS"*

Voy a solicitar la totalidad del mensaje con "RFC 822", con el comando:


```
1 fetch 1 RFC822
```

```
1 fetch 1 RFC822
* 1 FETCH (RFC822 {167}
Subject: Flag
To: Robin <robin@inlanefreight.htb>
From: CTO <devadmin@inlanefreight.htb>
Date: Wed, 03 Nov 2021 16:13:27 +0200

HTB{983uzn8jmfgpd8jmof8c34n7zio}
)
1 OK Fetch completed (0.005 + 0.000 + 0.004 secs).

```

Veo como el correo es *devadmin\@inlanefreight.ht*

**\[+1 CUBE ]  Try to access the emails on the IMAP server and submit the flag as the answer. (Format: HTB{...})

En el ejercicio anterior, también veo la flag:
*HTB{983uzn8jmfgpd8jmof8c34n7zio}*

