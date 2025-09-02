**La empresa Inlanefreight Ltd nos encargó probar tres servidores diferentes en su red interna. La empresa utiliza diversos servicios, y el departamento de seguridad informática consideró necesario realizar una prueba de penetración para comprender mejor su estrategia de seguridad general.**

**El primer servidor es un servidor DNS interno que debe investigarse. En particular, nuestro cliente desea saber qué información podemos obtener de estos servicios y cómo podría utilizarse contra su infraestructura. Nuestro objetivo es recopilar la mayor cantidad de información posible sobre el servidor y encontrar maneras de utilizarla en su contra. Sin embargo, nuestro cliente ha dejado claro que está prohibido atacar los servicios agresivamente mediante exploits, ya que se encuentran en producción.**

**Además, nuestros compañeros de equipo encontraron las siguientes credenciales "ceil:qwer1234" y señalaron que algunos empleados de la empresa estaban hablando de claves SSH en un foro.**

**Los administradores han almacenado un archivo flag.txt en este servidor para realizar un seguimiento de nuestro progreso y medir el éxito. Enumere completamente el objetivo y envíe el contenido de este archivo como prueba.**

--------------

***Resolución:***
Partimos de unas credenciales que podemos utilizar seguramente en algun servicio FTP,SSH,DNS o similar.

Realizamos un escaneo básico de nmap y vemos:

-p21 y -p2121 corren un servicio FTP.
-p22 corre un servicio ssh.

Probamos a ingresar por FTP al puerto 21:
```
ftp 10.129.249.89 21
```

Y sí, nos podemos conectar a dicho servicio con las credenciales eil:qwer1234 pero exite ningún archivo útil (ni "ls" ni "ls -la" lista nada).

Probamos el mismo comando pero para el puerto 2121:
```
ftp 10.129.249.89 2121
```

Y aqui de nuevo nos podemos loggear con ceil:qwer1234 y con "ls -la" vemos un directorio llamado **"".ssh""**. Ingresamos en dicho directorio y nos descargamos la clave pública con:
```
get id_rsa
```

Nos probamos de conectar al servicio ssh con dicha clave pública, con el comando:

```
ssh -i id_rsa ceil@10.129.23.49
```

Y efectivamente ingresamos, encontrando la flag en:
```
ceil@NIXEASY:~$ cat /home/flag/flag.txt 
```


