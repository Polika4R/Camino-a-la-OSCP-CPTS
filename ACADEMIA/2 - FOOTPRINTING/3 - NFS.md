# NFS

## Qué es NFS y cómo funciona

NFS (Network File System) es un protocolo desarrollado por Sun Microsystems que permite a los usuarios acceder y montar sistemas de archivos remotos a través de una red como si fueran locales. A diferencia de SMB, que se utiliza principalmente en entornos Windows, NFS se emplea sobre todo entre sistemas Linux y Unix. 

Debido a que ambos protocolos son fundamentalmente diferentes, los clientes NFS no pueden interactuar directamente con servidores SMB

Existen varias versiones de este protocolo:
- **NFSv2**: El más antiguo, funciona con UDP.
- **NFSv3**: No es totalmente compatible con clientes de NFSv2. Tiene un abanico de funcionalidades algo más amplio (incluyendo tamaño de archivos variable) y mejor informes de errores.
- **NFSv4**: Incluye Kerberos, funciona a través de firewalls y en Internet, ya no requiere mapeadores de puertos, admite ACL, aplica operaciones basadas en estado y ofrece mejoras de rendimiento y alta seguridad. Además, es la primera versión con un protocolo con estado.

## Configuración de serie

El archivo */etc/exports* contiene una table de los sistemas de archivos físicos accesibles para clientes de un servidor NFS. 
La "NFS Exports Table" muestra que opciones están disponibles para qué usuario.

La estructura es la siguiente:
- Carpeta a explortar
- Host / Subred con permisos
- Permisos y opciones extra
```
/ruta/al/share   cliente(rw,sync,no_subtree_check)
```

```shell-session
Polika4RM@htb[/htb]$ cat /etc/exports 

# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

Como opciones de configuración, encontramos:

|Opción|Descripción|
|---|---|
|`rw`|Lectura y escritura.|
|`ro`|Solo lectura.|
|`sync`|Transferencia síncrona (más seguro, más lento).|
|`async`|Transferencia asíncrona (más rápido, menos seguro).|
|`secure`|Solo puertos <1024.|
|`insecure`|Permite puertos >1024.|
|`no_subtree_check`|Desactiva verificación de subdirectorios.|
|`root_squash`|UID/GID 0 (root) mapeado a usuario anónimo.|
## Creación de un recurso compartido

Con el siguiente comando, la subred 10.129.14.0/24 podrá montar /mnt/nfs y ver su contenido:
```shell-session
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server 
root@nfs:~# exportfs

/mnt/nfs      	10.129.14.0/24
```

En Linux, **montar** significa **hacer accesible el contenido de un sistema de archivos** (como un disco duro, una memoria USB o una carpeta compartida por red como NFS) en un punto del sistema de archivos local.

### Opciones peligrosas

|Opción|Peligro|
|---|---|
|`rw`|Permite **lectura y escritura**: se puede **modificar** o **borrar** todo.|
|`insecure`|Permite conexiones desde **puertos >1024** (que puede usar cualquier usuario, no solo root). Esto **rompe el control de privilegios** de RPC.|
|`nohide`|Si hay otro sistema de archivos montado dentro de una carpeta exportada, también se exporta **sin ocultarse**. Puede exponer más datos de lo previsto.|
|`no_root_squash`|El root del cliente sigue siendo **root en el servidor** → puede acceder, modificar o borrar cualquier cosa si la carpeta es `rw`.|

## FOOTPRINTING de NFS

- Puertos clave:
    - `111` → RPC (Remote Procedure Call) #-p111
    - `2049` → NFS #-p2049
- Se puede usar **RPC** para averiguar qué servicios ofrece el servidor y qué carpetas tiene exportadas.

RPC (Remote Procedure Call) es un protocolo que permite a un programa pedirle a otro en una máquina diferente que ejecute una función o procedimiento.

El Portmapper es un servicio que funciona como un "registro" o directorio. 
Su trabajo es decir en qué puerto están corriendo otros servicios RPC en el servidor.
- Cuando un cliente quiere usar NFS, primero pregunta al portmapper en el puerto 111 qué puerto usa NFS.
- El portmapper responde: "NFS está en el puerto 2049".
- Entonces el cliente se conecta al puerto 2049 para usar NFS.
El puerto 111 es fundamental porque permite descubrir qué servicios RPC están activos y en qué puertos, lo cual es clave para hacer footprinting y enumerar servicios en pentesting.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap 10.129.14.128 -p111,2049 -sV -sC

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:12 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00018s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41982/udp6  mountd
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      47217/tcp6  mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      39542/udp   nlockmgr
|   100021  1,3,4      44629/tcp   nlockmgr
|   100021  1,3,4      45273/tcp6  nlockmgr
|   100021  1,3,4      47524/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
```

### rpcinfo NSE script (Nmap)
- Este script de Nmap consulta los servicios RPC activos en un objetivo.
- Te devuelve una lista de servicios RPC que están corriendo, con sus nombres, descripciones y puertos usados.
- Así puedes verificar si el servidor tiene abiertos todos los puertos necesarios para que funcione el servicio NFS o cualquier otro servicio RPC.

### Scripts NSE para NFS
- Nmap tiene scripts específicos para NFS que permiten escanear y obtener información más detallada.
- Por ejemplo, pueden mostrar:
    - El contenido de los shares exportados (carpetas compartidas).
    - Estadísticas del servicio NFS.

```shell-session
Polika4RM@htb[/htb]$ sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:37 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0      0      0     2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount: 
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs: 
|   Filesystem  1K-blocks   Used       Available   Use%  Maxfilesize  Maxlink
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%   16.0T        32000
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41982/udp6  mountd
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      47217/tcp6  mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      39542/udp   nlockmgr
|   100021  1,3,4      44629/tcp   nlockmgr
|   100021  1,3,4      45273/tcp6  nlockmgr
|   100021  1,3,4      47524/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```


### Descubrimiento de archivos NFS disponibles
Para descubrir sistemas de archivos disponibles en una máquina concreta, utilizaremos el comando:

```shell-session
Polika4RM@htb[/htb]$ showmount -e 10.129.14.128

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

Acto seguido, en nuestra máquina local, montaremos el recurso para poder acceder a él.
### Montar un share NFS en local
- **Cuando detectamos un servicio NFS en un servidor, podemos montar (conectar) ese recurso compartido en nuestra máquina local.**
- Para hacerlo, primero creamos una carpeta vacía en nuestro sistema.
- Luego montamos ahí el share NFS. Una vez montado, podemos navegar y ver los archivos como si fueran parte de nuestro disco local.
```shell-session
Polika4RM@htb[/htb]$ mkdir target-NFS
Polika4RM@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
Polika4RM@htb[/htb]$ cd target-NFS
Polika4RM@htb[/htb]$ tree .

.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files
```

---
**QUESTIONS**
**Target: 10.129.244.49**
**1. Enumerate the NFS service and submit the contents of the flag.txt in the "nfs" share as the answer.

Realizando un escaneo básico en nmap:
```
sudo nmap -sSV --min-rate 2000 --open -Pn -n 10.129.244.49
```
observo que existen los siguientes puertos abiertos:
```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 4.6.2
445/tcp  open  netbios-ssn Samba smbd 4.6.2
2049/tcp open  nfs         3-4 (RPC #100003)
```

Los puertos 111 y 2049 indican que está corriendo un servicio NFS.  

Lanzando un script más concreto contra servicios NFS:
```
sudo nmap 10.129.244.49 --script nfs* -sV -p111,2049
```
, observo dos volúmenes que contienen un archivo llamado flag.txt

```
| nfs-statfs: 
|   Filesystem     1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|   /var/nfs       4062912.0  3330672.0  506144.0   87%   16.0T        32000
|_  /mnt/nfsshare  4062912.0  3330672.0  506144.0   87%   16.0T        32000
| nfs-ls: Volume /var/nfs
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxr-xr-x   65534  65534  4096  2021-11-08T15:08:27  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   65534  65534  39    2021-11-08T15:08:27  flag.txt
| 
| 
| Volume /mnt/nfsshare
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxr-xr-x   65534  65534  4096  2021-11-08T14:06:40  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   65534  65534  59    2021-11-08T14:06:40  flag.txt
```

Dichos recursos también los puedo listar de la siguiente forma:
```
showmount -e 10.129.244.49
Export list for 10.129.244.49:
/var/nfs      10.0.0.0/8
/mnt/nfsshare 10.0.0.0/8
```

Nos creamos una carpeta en nuesta máquina atacante, que se llame *target-NFS* y montamos el recurso sobre dicha ruta: 
```
mkdir target-NFS
sudo mount -t nfs 10.129.244.49:/ ./target-NFS/ -o nolock
```
Entrado a:
cat target-NFS/mnt/nfsshare/flag.txt veo la flag del primer ejercicio:
```
cat target-NFS/var/nfs/flag.txt
HTB{hjglmvtkjhlkfuhgi734zthrie7rjmdze}
```

**2. Enumerate the NFS service and submit the contents of the flag.txt in the "nfsshare" share as the answer.**
Partiendo del ejercico anterior, entrando a:
```
cat target-NFS/mnt/nfsshare/flag.txt 
HTB{8o7435zhtuih7fztdrzuhdhkfjcn7ghi4357ndcthzuc7rtfghu34}
```

Encuentro la flag del directorio nfsshare.



