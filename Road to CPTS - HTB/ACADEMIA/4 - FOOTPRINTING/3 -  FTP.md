## Introducción al FTP

Se utiliza para transferir archivos entre un cliente y un servidor, ya sea subiendo o descargando archivos.
Para establecer una conexión FTP, se abren dos canales TCP:

- **Canal de control** en el puerto 21, donde el cliente envía comandos y el servidor responde con códigos de estado.
- **Canal de datos** en el puerto 20, exclusivo para la transferencia de archivos, con mecanismos para retomar la transferencia si se interrumpe.

Existen dos modos de FTP:
- **Activo:** el cliente inicia la conexión al puerto 21 y comunica al servidor el puerto cliente para la transferencia, pero los firewalls pueden bloquear la respuesta del servidor.
- **Pasivo:** el servidor anuncia un puerto para que el cliente establezca el canal de datos, facilitando el paso de firewalls ya que el cliente inicia ambas conexiones.

FTP utiliza varios comandos y códigos de estado para gestionar operaciones como subir, bajar, eliminar archivos o manejar directorios, aunque no todos los comandos son soportados por todos los servidores.

Normalmente, FTP requiere credenciales, aunque algunos servidores permiten acceso anónimo con permisos limitados. Al ser un protocolo en texto claro, puede ser vulnerable a ataques de sniffing, lo que representa un riesgo de seguridad.

## TFTP (Trivial File Transfer Protocol)

- **Capa**: Aplicación (UDP).
- **Objetivo**: transferencias simples de archivos entre cliente y servidor.
- **Características**:
  - **No requiere autenticación**.
  - Sin gestión de usuarios ni contraseñas.
  - Acceso controlado solo por los permisos del sistema de archivos.
  - Solo opera sobre archivos/directorios globalmente accesibles.
  - Sin funcionalidades como listar directorios.
  - Solo recomendado en **redes locales y seguras**.
  - Utiliza **UDP** → no confiable, pero incluye recuperación a nivel de aplicación.
- **Comandos comunes**:
  - `connect`: establecer host remoto y puerto.
  - `get`: descargar archivo(s).
  - `put`: subir archivo(s).
  - `quit`: salir de la sesión.
  - `status`: estado de la sesión y modo de transferencia.
  - `verbose`: activar/desactivar salida detallada.

## Comparativa entre FTP y TFTP

| Característica        | FTP                     | TFTP                      |
|-----------------------|-------------------------|---------------------------|
| Protocolo de transporte | TCP (puertos 21 y 20)  | UDP                      |
| Autenticación         | Sí (usuario/contraseña) | No                       |
| Seguridad             | Baja (texto claro)      | Muy baja (sin control de acceso) |
| Comandos avanzados    | Sí                      | No                       |
| Transferencias seguras| No (usa texto claro)    | No                       |
| Uso típico            | Internet / redes LAN    | Solo redes locales seguras |
| Listado de directorios| Sí                      | No                       |
# vsFTPd

Uno de los servidores FTP más utilizados en distribuciones Linux es **vsFTPd**.

La configuración por defecto de vsFTPd se encuentra en:
>/etc/vsftpd.conf

Se instala con:
```shell-session
Polika4RM@htb[/htb]$ sudo apt install vsftpd 
```

Si revisamos el archivo de configuración de vsFTPd, veremos muchas opciones y configuraciones comentadas o no. Sin embargo, el archivo de configuración no contiene todas las configuraciones posibles. Las existentes y las que faltan se pueden encontrar en la página de --help de vsFTPd

Algunos parámetros a configurar son:

| Parámetro                          | Descripción                                                                 |
|-----------------------------------|-----------------------------------------------------------------------------|
| `listen=NO`                       | Ejecuta vsftpd desde inetd en lugar de como demonio independiente.         |
| `listen_ipv6=YES`                 | Habilita escucha en conexiones IPv6.                                       |
| `anonymous_enable=NO`             | Desactiva el acceso anónimo.                                               |
| `local_enable=YES`                | Permite el acceso a usuarios locales del sistema.                          |
| `dirmessage_enable=YES`           | Muestra mensajes personalizados al entrar en ciertos directorios.          |
| `use_localtime=YES`               | Usa la hora local del sistema en los registros.                            |
| `xferlog_enable=YES`              | Activa el registro de transferencias (subidas y descargas).                |
| `connect_from_port_20=YES`        | Usa el puerto 20 para las conexiones de datos en modo activo.              |
| `secure_chroot_dir=/var/run/vsftpd/empty` | Directorio vacío necesario para el aislamiento seguro del proceso. |
| `pam_service_name=vsftpd`         | Define el nombre del servicio PAM que usa vsftpd para autenticación.       |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem` | Ruta al certificado SSL.                                    |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` | Ruta a la clave privada del certificado SSL.              |
| `ssl_enable=NO`                   | Desactiva el cifrado SSL/TLS en las conexiones FTP.                        |
### Usuarios FTP denegados
Hay un archivo llamado /etc/ftpusers que lista los usuarios denegados al servicio FTP.
Los aparecientes en dicha lista no podrán hacer uso del protocolo.

## Configuraciónes peligrosas en FTP

| Parámetro                          | Descripción                                                                 |
|-----------------------------------|-----------------------------------------------------------------------------|
| `anonymous_enable=YES`            | Permite el inicio de sesión como usuario anónimo (sin cuenta local).       |
| `anon_upload_enable=YES`          | Permite que usuarios anónimos suban archivos al servidor FTP.              |
| `anon_mkdir_write_enable=YES`     | Permite que usuarios anónimos creen nuevos directorios.                    |
| `no_anon_password=YES`            | No solicita contraseña al usuario anónimo (entra directamente).            |
| `anon_root=/home/username/ftp`    | Define el directorio raíz para usuarios anónimos.                          |
| `write_enable=YES`                | Permite comandos FTP de escritura: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE. |

## Iniciar conexión como anónimo

Simplemente me conecto al servicio con:

```
ftp 1.2.3.4
```

y cuando pida nombre de usuario escribo ***anonymous*.**


## Comandos varios FTP

#### Comando "status"
Dentro de una sesión FTP, podemos ejecutar "status" para que nos muestre información de:
- Modo de transferencia (por ejemplo, binario o ASCII.
- Si se está en modo activo o pasivo
- Usuario actual.
- Servidor conectado.
- Dirección IP remota.
- Puerto utilizado.
- Tiempo de inactividad.
- Modo de autenticación, si aplica.

#### Comando "debug"
- Activa la **depuración** en la sesión FTP.
- Muestra mensajes detallados sobre los comandos enviados y las respuestas recibidas.
- Útil para diagnosticar problemas o entender el comportamiento del servidor FTP.

```
ftp> debug
Debugging on (debug=1).

ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002 1002 8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002 1002 4096 Sep 14 17:03 Clients
226 Directory send OK.
```



#### Comando "trace"

- Activa el **seguimiento de paquetes** en la sesión FTP.
- Muestra en detalle cada comando FTP que se envía y la respuesta que devuelve el servidor.
- Ayuda a ver la comunicación exacta a bajo nivel entre cliente y servidor FTP.

```
ftp> trace
Packet tracing on.

ftp> ls
---> PORT 10,10,14,4,188,195
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002 1002 8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002 1002 4096 Sep 14 17:03 Clients
226 Directory send OK.
```
- Al activar `trace`, cada comando que el cliente FTP envía aparece precedido por `--->`.
- Por ejemplo, el comando `PORT` y luego `LIST` se muestran con sus respuestas del servidor.
- Esto permite seguir en detalle el diálogo entre cliente y servidor.

**PORT** establece dónde el servidor debe enviar la conexión de datos para la transferencia.

## Configuraciones importantes de vsFTPd

| Configuración           | Descripción                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------------- |
| dirmessage_enable=YES   | ¿Mostrar un mensaje cuando ingresan por primera vez a un nuevo directorio?                      |
| chown_uploads=YES       | ¿Cambiar la propiedad de archivos cargados anónimamente?                                        |
| chown_username=username | Usuario al que se le otorga la propiedad de los archivos cargados de forma anónima.             |
| local_enable=YES        | ¿Permitir que los usuarios locales inicien sesión?                                              |
| chroot_local_user=YES   | ¿Colocar a los usuarios locales en su directorio de inicio?                                     |
| chroot_list_enable=YES  | ¿Utilizar una lista de usuarios locales que se colocarán en su directorio de inicio?            |
| hide_ids=YES            | Toda la información de usuarios y grupos en los listados de directorios se mostrará como "ftp". |
| ls_recurse_enable=YES   | Permite el uso de listados recursivos.                                                          |

En el siguiente ejemplo, podemos ver que si la `hide_ids=YES`configuración está presente, la representación UID y GUID del servicio se sobrescribirá, lo que hará que nos resulte más difícil identificar con qué derechos se escriben y cargan estos archivos.

**con hide_ids=YES**:
```
-rw-rw-r-- 1 user1 group1 1024 Sep 14 12:00 archivo.txt  
drwxr-xr-x 2 user2 group2 4096 Sep 14 12:05 carpeta  
-rw-r--r-- 1 user3 group3 2048 Sep 14 12:10 documento.pdf
```

**sin hide_ids=YES**:

```
-rw-rw-r-- 1 ftp ftp 1024 Sep 14 12:00 archivo.txt  
drwxr-xr-x 2 ftp ftp 4096 Sep 14 12:05 carpeta  
-rw-r--r-- 1 ftp ftp 2048 Sep 14 12:10 documento.pdf
```

Con estos nombres de usuario, en teoría podríamos atacar servicios como FTP, SSH y muchos otros con un ataque de fuerza bruta.


#### Comando "ls -R"
El comando `ls -R` dentro de una sesión FTP sirve para:
- **Listar directorios de forma recursiva**, es decir:
    - Muestra el contenido del directorio actual.
    - Luego entra en cada subdirectorio y lista también su contenido
    - Y así sucesivamente para todos los niveles de subdirectorios.

Salida esperada:
```
ftp> ls -R
200 PORT command successful.
150 Here comes the directory listing.
./:
archivo1.txt  carpeta1  carpeta2

./carpeta1:
archivo2.txt  subcarpeta1

./carpeta1/subcarpeta1:
archivo3.txt

./carpeta2:
archivo4.txt
226 Directory send OK.

```


#### Comando "get <nombre_archivo"

Dentro de una sesión FTP, podemos descargarnos un archivo con "get <nombre_archivo>":
```
```shell-session
ftp> get Important\ Notes.txt

local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)

ftp> exit
```

Al salir de la sesión, lo veremos en el directorio desde donde se haya ejecutado el servicio ftp anteriormente:

```shell-session
Polika4RM@htb[/htb]$ ls | grep Notes.txt

'Important Notes.txt'
```

#### Descarga masiva de archivos FTP con wget

- Se utiliza el comando `wget -m --no-passive ftp://anonymous:anonymous@IP` para descargar **todos los archivos disponibles** desde un servidor FTP accesible como usuario anónimo.
```shell-session
Polika4RM@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

- `wget` crea automáticamente una carpeta con el nombre de la IP del servidor y guarda dentro todos los archivos y carpetas descargados, respetando la estructura remota.
- Esto permite inspeccionar localmente todo el contenido del servidor FTP.

Después, con "tree ." podemos ver todo lo descargado:

```shell-session
Polika4RM@htb[/htb]$ tree .

.
└── 10.129.14.136
    ├── Calendar.pptx
    ├── Clients
    │   └── Inlanefreight
    │       ├── appointments.xlsx
    │       ├── contract.docx
    │       ├── meetings.txt
    │       └── proposal.pptx
    ├── Documents
    │   ├── appointments-template.xlsx
    │   ├── contract-template.docx
    │   └── contract-template.pdf
    ├── Employees
    └── Important Notes.txt

5 directories, 9 files
```


#### Comando "put <nombre_archivo"
Sube un archivo desde el cliente local al servidor FTP.

```
ftp> put documento.txt
```

El "documento.txt" se debe encontrar en el directorio desde donde se ha iniciado la sesión de FTP. Si no es el caso, se puede hacer un.

```
ftp> put /ruta/documento.txt
```


