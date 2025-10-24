Netsh es una herramienta de línea de comandos de Windows que facilita la configuración de red de un sistema Windows específico.

Permite:
- Ver la configuración del firewall
- Añadir servidores proxy
- Crear reglas de reenvío de puertos


Tomemos como ejemplo el siguiente escenario, donde nuestro host comprometido es la estación de trabajo de un administrador de TI con Windows 10 (10.129.15.150, 172.16.5.25). 


![[88.webp]]

Podemos usar netsh.exe para reenviar todos los datos recibidos en un puerto específico (por ejemplo, el 8080) a un host remoto en un puerto remoto. Esto se puede realizar con el siguiente comando:


Desde la máquina pivot:
```cmd-session
C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```


Verificamos que el proxy esté funcionando correctamente: 
```cmd-session
C:\Windows\system32> netsh.exe interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.15.150   8080        172.16.5.25     3389
```

Tras configurar el portproxy en nuestro host pivote basado en Windows, intentaremos conectarnos al puerto 8080 de este host desde nuestro host de ataque mediante xfreerdp. Una vez enviada la solicitud desde nuestro host de ataque, el host Windows enrutará nuestro tráfico según la configuración del proxy configurada por netsh.exe.

---
**Target: 10.129.50.3
RDP to  with user "htb-student" and password "HTB_@cademy_stdnt!"**
**1. Using the concepts covered in this section, take control of the DC (172.16.5.19) using xfreerdp by pivoting through the Windows 10 target host. Submit the approved contact's name found inside the "VendorContacts.txt" file located in the "Approved Vendors" folder on Victor's desktop (victor's credentials: victor:pass@123) . (Format: 1 space, not case-sensitive)**

Me conecto a la máquina pivot con:
```
xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.50.3
```

Ejecuto:
```
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.50.3 connectport=3389 connectaddress=172.16.5.19
```

```
netsh.exe interface portproxy show v4tov4
```

```
C:\Windows\system32>netsh.exe interface portproxy show v4tov4                                                                                                                                     Address         Port        Address         Port
------------------------------------------------                 10.10.15.95     8080        172.16.5.19     3389    
```

Desde la máquina atacante ejecuto:
```
xfreerdp /v:10.129.50.3:8080 /u:victor /p:pass@123
```

Y dentro del archivo "Vendor Contacts" encuentro la flag: Jim Flipflop