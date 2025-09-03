**Enunciado**:
Con la ayuda de nuestra segunda prueba, nuestro cliente obtuvo nuevos conocimientos y envió a uno de sus administradores a un curso de capacitación sobre sistemas IDS/IPS. Según nos informó, la capacitación duraría una semana. Ahora, el administrador ha tomado todas las precauciones necesarias y quiere que volvamos a realizar la prueba, ya que es necesario cambiar servicios específicos y modificar la comunicación del software proporcionado.

**Target: 
**1. Now our client wants to know if it is possible to find out the version of the running services. Identify the version of service our client was talking about and submit the flag as the answer.**

Haciendo un escaneo básico de nmap, a priori, solo encuentro estos puertos:
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Pero poniendo como puerto de origen el -p 53 (el del servicio DNS), observo que existe otro. Lanzo la búsqueda con el comando:
```
nmap -p- -sSV 10.129.120.150 --source-port 53
```

Dándome como resultado de puertos abiertos, los siguientes: 
```
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http       Apache httpd 2.4.29 ((Ubuntu))
50000/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:l
```

Vemos que el puerto 5000 está abierto, pues, ejecuto:
```
sudo ncat -nv --source-port 53 10.129.120.150 50000
Ncat: Version 7.94SVN ( https://nmap.org/ncat )
Ncat: Connected to 10.129.120.150:50000.
220 HTB{kjnsdf2n982n1827eh76238s98di1w6}
```

Dándome como respuesta: *HTB{kjnsdf2n982n1827eh76238s98di1w6}*