
A menudo, durante una evaluación, nos vemos limitados a una red Windows y no podemos usar SSH para pivotar. 
En estos casos, debemos usar herramientas disponibles para sistemas operativos Windows. 
SocksOverRDP es un ejemplo de una herramienta que utiliza Canales Virtuales Dinámicos (DVC) del Servicio de Escritorio Remoto de Windows. 
**DVC se encarga de tunelizar paquetes a través de la conexión RDP**. 

Algunos ejemplos de uso de esta función son la transferencia de datos del portapapeles y el uso compartido de audio. Sin embargo, también se puede usar para tunelizar paquetes arbitrarios a través de la red. 
Podemos usar SocksOverRDP para tunelizar nuestros paquetes personalizados y luego usar el proxy. Usaremos la herramienta Proxifier como servidor proxy.

Podemos comenzar descargando los binarios adecuados a nuestro host de ataque para ejecutar este ataque. Tener los binarios en nuestro host de ataque nos permitirá transferirlos a cada objetivo donde sea necesario. Necesitaremos:
- Binarios x64 de SocksOverRDP
- Binarios portátiles de Proxifier: podemos descargar "ProxifierPE.zip"


## Cargando SocksOverRDP.dll usando regsvr32.exe
Luego, podemos conectarnos al destino mediante xfreerdp y copiar el archivo SocksOverRDPx64.zip. 
Desde atacante, cargaremos SocksOverRDP.dll con regsvr32.exe:

```cmd-session
C:\Users\htb-student\Desktop\SocksOverRDP-x64> regsvr32.exe SocksOverRDP-Plugin.dll
```
![[socksoverrdpdll.webp]]

Ahora podemos conectarnos a 172.16.5.19 mediante RDP usando mstsc.exe. Deberíamos recibir un aviso indicando que el complemento SocksOverRDP está habilitado y que escuchará en 127.0.0.1:1080. Podemos usar las credenciales victor:pass@123 para conectarnos a 172.16.5.19.
![[pivotingtoDC.webp]]
Sobre la víctima, necesitaremos transferir SocksOverRDPx64.zip o simplemente SocksOverRDP-Server.exe a 172.16.5.19. 
En la víctima debemos iniciar SocksOverRDP-Server.exe con privilegios de administrador.

![[executingsocksoverrdpserver.webp]]

Cuando regresemos a nuestra máquina atacante y verificamos con Netstat, deberíamos ver que nuestro escucha SOCKS se inició en 127.0.0.1:1080:

Tras iniciar nuestro "listener", podemos transferir Proxifier portátil al destino de Windows 10 (en la red 10.129.x.x) y configurarlo para que reenvíe todos nuestros paquetes a 127.0.0.1:1080. Proxifier enrutará el tráfico a través del host y puerto indicados. Vea el vídeo a continuación para obtener una guía rápida sobre la configuración de Proxifier.
![[configuringproxifier.gif]]

Con Proxifier configurado y ejecutándose, podemos iniciar mstsc.exe, y usará Proxifier para pivotar todo nuestro tráfico a través de 127.0.0.1:1080, que lo tunelizará sobre RDP a 172.16.5.19, que luego lo enrutará a 172.16.6.155 usando SocksOverRDP-server.exe.
![[rdpsockspivot.webp]]

## Consideraciones sobre el rendimiento de RDP
Al interactuar con nuestras sesiones RDP en una interacción, es posible que experimentemos un rendimiento lento en una sesión determinada, especialmente si gestionamos varias sesiones RDP simultáneamente. En este caso, podemos acceder a la pestaña Experiencia en mstsc.exe y configurar Rendimiento en Módem.


**Target(s): 10.129.241.161**
**RDP to 10.129.133.113 with user "htb-student" and password "HTB_@cademy_stdnt!"**
**1  Use the concepts taught in this section to pivot to the Windows server at 172.16.6.155 (jason:WellConnected123!). Submit the contents of Flag.txt on Jason's Desktop.**

Nos conectamos a la máquina windows atacante, con las credenciales: 

```
xfreexxp /v:10.129.241.161 /u:htb-student /p:HTB_@cademy_stdnt!
```

Nos descargamos los binarios necesarios en nuestra máquina atacante Linux: 
- [Binarios SocksOverRDP x64](https://github.com/nccgroup/SocksOverRDP/releases)
- [Proxifier binario portátil](https://www.proxifier.com/download/#win-tab)

```
┌─[eu-academy-3]─[10.10.14.216]─[htb-ac-1876550@htb-xosmomgrna]─[~/Downloads]
└──╼ [★]$ ls
ProxifierSetup.exe  SocksOverRDP-x64.zip
```

Entramos en la máquina windows atacante (la de xfreerdp anterior) y nos descargamos de firefox dichos archivos: 
```
http://10.10.14.216:8080/
```

Nos detectará que SocksOverRDP-Plugin.dll tiene un virus.  Tenemos que entrar en el Antivirus de Windows y permitir dicha descarga.

Nos situamos en el directorio Windows que contiene dichos archivos:
```
PS C:\Users\htb-student\Downloads> dir


    Directory: C:\Users\htb-student\Downloads


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       10/27/2025   8:19 AM        5498000 ProxifierSetup (1).exe
-a----       10/27/2025   8:15 AM        5498000 ProxifierSetup.exe
-a----       10/27/2025   8:30 AM          67584 SocksOverRDP-Plugin.dll
-a----       10/27/2025   8:34 AM          31232 SocksOverRDP-Server.exe
-a----       10/27/2025   8:33 AM          44183 SocksOverRDP-x64 (1).zip
-a----       10/27/2025   8:33 AM          44183 SocksOverRDP-x64.zip
```


Desde la máquina Windows atacante, cargamos Socks Over RDP.dll usando regsvr32.exe con cmd como adminsitrador:


```
regsvr32.exe SocksOverRDP-Plugin.dll
```

