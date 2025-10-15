
##  SMB (SERVER MESSAGE BLOCK)
Podemos acceder a servicios SMB en windows mediante "cmd" y "powershell":
Un recurso, por ejemplo, puede ubicarse en: \\\192.168.220.129\Finance\
#### 1. SMB & CMD windows
Si conocemos la ruta del servicio compartido, podemos acceder a este con:
```
C:\htb> dir \\192.168.220.129\Finance\
```


El comando net use conecta o desconecta una computadora de un recurso compartido, o muestra información sobre las conexiones de la computadora. Podemos conectarnos a un recurso compartido de archivos con el siguiente comando y asignar su contenido a la letra de unidad "n":
```cmd-session
C:\htb> net use n: \\192.168.220.129\Finance

The command completed successfully.
```
Lo que estamos haciendo es montar dicho recurso en el disco local "N" del ordenador. 
Hay veces en que nos tenemos que autenticar para poder acceder a dicho recurso compartido:
```
C:\htb> net use n: \\192.168.220.129\Finance /user:plaintext Password123

The command completed successfully.
```

Ahora, tenemos dicho recurso en la unidad "N". Vamos a descubrir cuantos archivos contiene la carpeta compartida y sus subdirectorios: 
```cmd-session
C:\htb> dir n: /a-d /s /b | find /c ":\"

29302
```
- `dir n:`  
    Lista el contenido de la unidad `N:`.
- `/a-d`  
    El modificador `/a` muestra archivos con ciertos atributos; cuando añades `-d` significa _excluir directorios_. Es decir: **mostrar solo archivos**, no carpetas.
- `/s`  
    Recorre **recursivamente** todos los subdirectorios bajo `N:`.
- `/b`  
    Modo **bare** (sencillo): imprime cada entrada en una línea con la ruta completa (sin encabezados, sumarios ni tamaños). Ejemplo de una línea:  
    `N:\Proyectos\2024\informe.pdf`
- `|`  
    El _pipe_ envía la salida del `dir` como entrada al siguiente comando.
- `find /c ":\\"`
    - `find` busca líneas que contienen el texto dado.
    - `/c` hace que `find` **cuente** las líneas que coinciden (en lugar de mostrarlas).
    - `" :\ "` (en el prompt suele escribirse como `":\"`) es la cadena buscada. Esa cadena aparece en las rutas completas porque las rutas comienzan con `N:\...` — por tanto **cada línea** generada por `dir /b` tendrá `:\` justo después de la letra de unidad.         
    - Resultado: `find /c ":\\"` devuelve el número de líneas que contienen `:\` — en la práctica, el número de archivos listados por `dir` (porque cada ruta contiene `N:\`).

Seguido a esto, con "dir" podemos buscar archivos con un nombre especifico. Nos interesa buscar archivos del tipo: 
- cred
- password
- users
- secrets
- key
- Common File Extensions for source code such as: .cs, .c, .go, .java, .php, .asp, .aspx, .html.

Buscaremos pues del tipo "cred" o "secret", con el comando:
```cmd-session
C:\htb>dir n:\*cred* /s /b

n:\Contracts\private\credentials.txt
```

- /s: busca en todos los subdirectorios recursivamente
- /b: Usa el formato "bare" (simple o limpio). Esto muestra solo la ruta completa del archivo o carpeta, sin encabezados, sin tamaños, sin fechas, ni sumarios.

```
C:\htb>dir n:\*secret* /s /b

n:\Contracts\private\secret.txt
```

Con el comando "Findstr" podemos buscar por palabras específicas dentro de un archivo de texto:
```cmd-session
c:\htb>findstr /s /i cred n:\*.*

n:\Contracts\private\secret.txt:file with all credentials
n:\Contracts\private\credentials.txt:admin:SecureCredentials!
```

- /i: ignora mayúsculas o minúsculas.
- n:\\*.*: busca dentro de todos los archivos en el disco N:.ç

---
## Con Windows Powershell

```powershell-session
PS C:\htb> Get-ChildItem \\192.168.220.129\Finance\

    Directory: \\192.168.220.129\Finance

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/23/2022   3:27 PM                Contracts
```

Lista los archivos y carpetas dentro del recurso compartido "Finance" en ese equipo remoto.
Podría escribir "Get-ChildItem" o simplemente "gci"

El siguiente comando sirve para **crear una unidad virtual (temporal)** en PowerShell, que apunta a una **carpeta de red compartida**. En vez de utilizar 'net use', utilizaremos 'New-PSDrive':
```powershell-session
PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"

Name           Used (GB)     Free (GB) Provider      Root                                               CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
N                                      FileSystem    \\192.168.220.129\Finance
```

En caso de tener que loggearnos con usuario y contraseña, deberemos crear un PSCredential Object.
```powershell-session
PS C:\htb> $username = 'plaintext'
PS C:\htb> $password = 'Password123'
PS C:\htb> $secpassword = ConvertTo-SecureString $password -AsPlainText -Force
PS C:\htb> $cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
PS C:\htb> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred

Name           Used (GB)     Free (GB) Provider      Root                                                              CurrentLocation
----           ---------     --------- --------      ----                                                              ---------------
N                                      FileSystem    \\192.168.220.129\Finance
```

#### Windows PowerShell - GCI

El comando:
```powershell-session
PS C:\htb> N:
PS N:\> (Get-ChildItem -File -Recurse | Measure-Object).Count

29302
```
- Recorre todas las carpetas y subcarpetas (-Recurse) de la ubicación actual (N:\\)
- Devuelve solo los archivos (-File, excluye las carpetas).
- | Measure-Object: cuenta cuántos objetos (en este caso, archivos) se obtuvieron.
- .Count: Extrae solo el número total del resultado.

Podemos utilizar la propiedad -Include para buscar elementos específicos del directorio especificado por el parámetro Path:
```powershell-session
PS C:\htb> Get-ChildItem -Recurse -Path N:\ -Include *cred* -File

    Directory: N:\Contracts\private

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2/23/2022   4:36 PM             25 credentials.txt
```


El cmdlet Select-String utiliza la coincidencia de expresiones regulares para buscar patrones de texto en cadenas de entrada y archivos. Podemos usar Select-String de forma similar a grep en UNIX o findstr.exe en Windows:
```powershell-session
PS C:\htb> Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List

N:\Contracts\private\secret.txt:1:file with all credentials
N:\Contracts\private\credentials.txt:1:admin:SecureCredentials!
```


---

## Linux
Con Linux también podemos explorar y montar recursos compartidos con SMB.

Creamos una carpeta y montamos dicha ruta:
```shell-session
Polika4RM@htb[/htb]$ sudo mkdir /mnt/Finance
Polika4RM@htb[/htb]$ sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

O para no dejar rastro de las credenciales en el historial, podemos hacerlo con un archivo de credenciales con la siguiente estructura: 
```txt
username=plaintext
password=Password123
domain=.
```

```shell-session
Polika4RM@htb[/htb]$ mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

Ahora, de la misma forma podemos encontrar palabras claves con:
```shell-session
Polika4RM@htb[/htb]$ find /mnt/Finance/ -name *cred*

/mnt/Finance/Contracts/private/credentials.txt
```
