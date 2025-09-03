
En Linux existen muchas formas de administrar sistemas remotamente. 
Una configuración incorrecta de estas aplicaciones o protocolos para acceder remotamente al servidor puede ser un honeypot para atacantes.

# SSH
**Secure Shell (SSH)** permite que dos ordenadores se conecten de forma segura a través de una red insegura, normalmente usando el puerto TCP 22, protegiendo los datos sensibles frente a interceptaciones. Funciona en todos los sistemas operativos principales: de forma nativa en Linux y macOS, y mediante software adicional en Windows.

La implementación más usada es **OpenSSH**, un fork de código abierto del servidor SSH comercial original. Existen dos versiones del protocolo:
- **SSH-1** – antigua y vulnerable a ataques _Man-in-the-Middle_.
- **SSH-2** – más segura, rápida, estable y con mejor cifrado.

Un **fork** es cuando se toma el código fuente de un proyecto existente y se crea una copia independiente para desarrollarlo por separado, siguiendo su propio camino.

SSH permite administrar sistemas de forma remota (CLI o GUI), enviar comandos, transferir archivos y hacer _port forwarding_.

**OpenSSH admite seis métodos de autenticación**:
1. Autenticación por contraseña
2. Autenticación por clave pública
3. Autenticación basada en host
4. Autenticación por teclado (_keyboard-interactive_)
5. Autenticación por reto–respuesta (_challenge–response_)
6. Autenticación GSSAPI

# Autenticación por clave pública (Public Key Authentication)

En la autenticación por clave pública de SSH:

- **Autenticación del servidor**:
    - Al iniciar una conexión SSH, el servidor envía su clave pública de host al cliente para verificar su identidad.
    - El riesgo de ataque "man-in-the-middle" solo existe en el primer contacto si el cliente no verifica la clave contra una fuente confiable.
        
- **Autenticación del cliente**:
    - Por defecto, el servidor valida al usuario pidiéndole la contraseña (comparada con el hash almacenado), lo que obliga a introducirla en cada conexión.
    - Como alternativa, se usan **pares de claves pública/privada**.
        - La clave privada se guarda solo en el equipo del usuario y está protegida con una **passphrase**.
        - La clave pública se guarda en el servidor.
        - El servidor envía un reto cifrado con la clave pública del cliente, que este resuelve con su clave privada para demostrar su identidad.
            
- **Ventajas de clave pública**:
    - Solo se introduce la passphrase una vez por sesión para acceder a varios servidores.
    - Al cerrar sesión en la máquina local, nadie más puede usarla para conectarse al servidor.

# Vulnerabilidad X11

El archivo sshd_config , responsable del servidor OpenSSH, solo incluye algunas de las opciones predeterminadas, sin embargo, entre ellas, en la versión 7.2p1 de OpenSSH se incluía por defecto el reenvío X11. 

**¿Qué es X11 forwarding?**:
- En OpenSSH, cuando un usuario se conecta con **X11 forwarding** activado, el servidor usa el comando `xauth` para configurar la autenticación gráfica.
- `xauth` se encarga de gestionar cookies de acceso X11 para que las aplicaciones gráficas remotas puedan mostrarse en tu pantalla.

**Problema**:
- En **OpenSSH 7.2p1** (y algunas versiones cercanas), la forma en que se pasaban los parámetros a `xauth` **no estaba correctamente saneada**.
- Esto permitía que **un usuario autenticado** (es decir, que ya tenía acceso por SSH) inyectara comandos arbitrarios a través de datos que OpenSSH pasaba a `xauth`.
- Básicamente, era un **Command Injection** en el proceso que invoca `xauth`.

Se recomienda siempre desactivar el X11Forwarding siempre.

## Configuraciones peligrosas

A pesar de que el protocolo SSH es uno de los más seguros disponibles actualmente, algunas configuraciones incorrectas pueden hacer que el servidor SSH sea vulnerable a ataques fáciles de ejecutar.

Sobre el archivo */etc/ssh/sshd_config*:

|**Configuración**|**Descripción**|
|---|---|
|`PasswordAuthentication yes`|Permite la autenticación basada en contraseña.|
|`PermitEmptyPasswords yes`|Permite el uso de contraseñas vacías.|
|`PermitRootLogin yes`|Permite iniciar sesión como usuario root.|
|`Protocol 1`|Utiliza una versión obsoleta de cifrado.|
|`X11Forwarding yes`|Permite el reenvío X11 para aplicaciones GUI.|
|`AllowTcpForwarding yes`|Permite el reenvío de puertos TCP.|
|`PermitTunnel`|Permite la tunelización.|
|`DebianBanner yes`|Muestra un banner específico al iniciar sesión.|

# Footprinting
Una de las herramientas que podemos usar para identificar el servidor SSH es **ssh-audit**. Esta herramienta verifica la configuración del cliente y del servidor, y muestra información general, así como los algoritmos de cifrado que aún utilizan el cliente y el servidor.

```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
Polika4RM@htb[/htb]$ ./ssh-audit.py 10.129.14.132

# general
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
(gen) compression: enabled (zlib@openssh.com)                                   

# key exchange algorithms
(kex) curve25519-sha256                     -- [info] available since OpenSSH 7.4, Dropbear SSH 2018.76                            
(kex) curve25519-sha256@libssh.org          -- [info] available since OpenSSH 6.5, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp256                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp384                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp521                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) diffie-hellman-group-exchange-sha256 (2048-bit) -- [info] available since OpenSSH 4.4
(kex) diffie-hellman-group16-sha512         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73
(kex) diffie-hellman-group18-sha512         -- [info] available since OpenSSH 7.3
(kex) diffie-hellman-group14-sha256         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73

# host-key algorithms
(key) rsa-sha2-512 (3072-bit)               -- [info] available since OpenSSH 7.2
(key) rsa-sha2-256 (3072-bit)               -- [info] available since OpenSSH 7.2
(key) ssh-rsa (3072-bit)                    -- [fail] using weak hashing algorithm
                                            `- [info] available since OpenSSH 2.5.0, Dropbear SSH 0.28
                                            `- [info] a future deprecation notice has been issued in OpenSSH 8.2: https://www.openssh.com/txt/release-8.2
(key) ecdsa-sha2-nistp256                   -- [fail] using weak elliptic curves
                                            `- [warn] using weak random number generator could reveal the key
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(key) ssh-ed25519                           -- [info] available since OpenSSH 6.5
...SNIP...
```

Lo primero que vemos en las primeras líneas del resultado es el banner que revela la versión del servidor OpenSSH. Las versiones anteriores presentaban vulnerabilidades, como [CVE-2020-14145](https://www.cvedetails.com/cve/CVE-2020-14145/) , que permitían al atacante realizar una intervención de intermediario (Man-in-the-Middle) y atacar el intento de conexión inicial. El resultado detallado de la configuración de la conexión con el servidor OpenSSH también suele proporcionar información importante, como los métodos de autenticación que puede utilizar el servidor.


#### Cambiar el método de autenticación

```shell-session
Polika4RM@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config 
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
```

Para posibles ataques de fuerza bruta, podemos especificar el método de autenticación con la opción de cliente SSH `PreferredAuthentications`.

  Protocolos de administración remota de Linux

```shell-session
Polika4RM@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
debug1: Next authentication method: password

cry0l1t3@10.129.14.132's password:
```


# RSYNC
- Rsync es una herramienta rápida y versátil para copiar archivos **localmente** o entre **hosts remotos**.
- Utiliza un **algoritmo delta**, que envía solo las diferencias entre archivos para reducir el tráfico de red.
- Se usa comúnmente para **copias de seguridad** y **mirroring**.
- Detecta archivos a transferir según cambios en **tamaño** o **fecha de modificación**.
- Puerto por defecto: **873**; puede usar **SSH** para transferencias seguras.

```
sudo nmap -sV -p 873 120.0.0.1
```

Este comando escanea el puerto 873 de la máquina actual, dando como resultado:

```shell-session
Polika4RM@htb[/htb]$ sudo nmap -sV -p 873 127.0.0.1

Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-19 09:31 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0058s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```

----

Con el siguiente comando puedo **conectarme a los servicios RSYNC y listar los módulos accesibles que están compartidos***:
```shell-session
Polika4RM@htb[/htb]$ nc -nv 127.0.0.1 873

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools
@RSYNCD: EXIT
```

Como veo disponible contenido en dev, voy a numerar dicho contenido con:
```shell-session
Polika4RM@htb[/htb]$ rsync -av --list-only rsync://127.0.0.1/dev

receiving incremental file list
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh

sent 25 bytes  received 221 bytes  492.00 bytes/sec
total size is 0  speedup is 0.00
```

# R services
Los **R-Services** son un conjunto de servicios diseñados para permitir acceso remoto o ejecutar comandos entre sistemas Unix a través de TCP/IP. Fueron desarrollados por el **Computer Systems Research Group (CSRG)** de la Universidad de California, Berkeley, y fueron estándar hasta ser reemplazados por **SSH** debido a fallos de seguridad inherentes. Al igual que **telnet**, transmiten información de manera **no cifrada**, lo que permite ataques tipo **MITM** para interceptar contraseñas o información de login.

Se ejecutan en los puertos **512, 513 y 514** y se usan mediante programas llamados **r-commands**, todavía presentes en sistemas como **Solaris, HP-UX y AIX**.

**Principales r-commands y su uso:**

|Comando|Daemon|Puerto|Protocolo|Descripción|
|---|---|---|---|---|
|`rcp`|`rshd`|514|TCP|Copia archivos/directorios entre sistemas, bidireccional. Similar a `cp`, pero sin advertencias de sobreescritura.|
|`rsh`|`rshd`|514|TCP|Abre un shell remoto sin procedimiento de login. Usa `/etc/hosts.equiv` y `.rhosts` para validar hosts confiables.|
|`rexec`|`rexecd`|512|TCP|Ejecuta comandos remotos. Requiere usuario y contraseña no cifrados; se puede sobrescribir con hosts confiables.|
|`rlogin`|`rlogind`|513|TCP|Permite login remoto sobre Unix, similar a telnet. También depende de hosts confiables.|
El archivo **`/etc/hosts.equiv`** contiene una lista de **hosts confiables** que pueden acceder automáticamente al sistema sin necesidad de autenticación adicional.

```shell-session
Polika4RM@htb[/htb]$ cat /etc/hosts.equiv

# <hostname> <local username>
pwnbox cry0l1t3
```

Significa que el host llamado **`pwnbox`** puede acceder a este sistema usando el usuario **`cry0l1t3`** sin que se le pida contraseña.

A partir de esto, los **r-commands** (como `rsh`, `rlogin` o `rexec`) podrían aprovechar esta confianza para abrir sesiones remotas automáticamente desde `pwnbox` como `cry0l1t3`.

