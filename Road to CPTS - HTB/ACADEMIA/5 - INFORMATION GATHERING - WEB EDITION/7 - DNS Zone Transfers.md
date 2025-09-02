### **¿Qué es un DNS Zone Transfer?**

Un **DNS zone transfer** (transferencia de zona DNS) es básicamente un mecanismo mediante el cual un servidor DNS **copia todos los registros de un dominio y sus subdominios** de un servidor principal (primary) a un servidor secundario (secondary).

El propósito principal de esto es **mantener consistencia y redundancia**: si el servidor principal falla, el secundario ya tiene toda la información actualizada.

Si está mal configurado, un atacante podría aprovecharlo para obtener **toda la lista de subdominios y sus IPs**, lo cual es información sensible.

![[Pasted image 20250822171003.png]]### **Cómo funciona paso a paso**

1. **Solicitud de Transferencia de Zona (AXFR)**  
    El servidor secundario inicia la transferencia enviando una solicitud al servidor primario.
    - AXFR es el tipo de solicitud que pide **toda la zona completa**.
2. **Transferencia del Registro SOA**
    - El servidor primario responde enviando su **registro SOA** (_Start of Authority_).
    - Este registro contiene información clave sobre la zona, como un **número de serie** que indica si los datos del secundario están actualizados.
3. **Transmisión de los Registros DNS**
    - El servidor primario envía **todos los registros DNS**, como
        - `A` → IP de los subdominios
        - `AAAA` → IP versión IPv6
        - `MX` → servidores de correo
        - `CNAME` → alias de otros dominios
        - `NS` → otros servidores de nombres
    - Esto le permite al servidor secundario tener un **espejo exacto de la zona**.
4. **Finalización de la Transferencia**
    - El servidor primario indica que **ha terminado de enviar todos los registros**.
5. **Confirmación (ACK)**
    - El servidor secundario envía un **mensaje de confirmación** al primario, indicando que recibió y procesó correctamente los datos.


### **La vulnerabilidad de las transferencias de zona (Zone Transfer Vulnerability)**
Aunque las **transferencias de zona** son esenciales para la gestión legítima de DNS, un **servidor DNS mal configurado** puede convertir este mecanismo en un riesgo de seguridad grave.
El problema principal es: **quién puede solicitar la transferencia de zona**.
#### **Historia y contexto**
- En los primeros días de Internet, muchos servidores DNS **permitían transferencias de zona a cualquier cliente**.
- Esto facilitaba la administración, pero también creaba un **agujero de seguridad enorme**.
- Cualquier persona, incluso un atacante, podía obtener una **copia completa de la zona DNS**, revelando información muy sensible.

---

### **Qué información puede obtener un atacante**
Una transferencia de zona no autorizada puede dar un **mapa completo de la infraestructura DNS**:

1. **Subdominios**
    - Incluye todos los subdominios, incluso los **ocultos** que no aparecen en la web pública.
        
    - Esto puede incluir: servidores de desarrollo, entornos de prueba, paneles administrativos o recursos sensibles.
2. **Direcciones IP**
    - Cada subdominio tiene una IP asociada.
    - Con esta información, un atacante puede planificar ataques o exploraciones más precisas.
3. **Registros de servidores de nombres (NS Records)**
    - Muestra qué servidores son responsables del dominio.
    - Puede revelar información sobre el proveedor de hosting y posibles **configuraciones erróneas**.

---

### **Cómo se mitiga esta vulnerabilidad**
- Hoy en día, la mayoría de los administradores **restringen la transferencia de zona solo a servidores secundarios confiables**.
- Esto garantiza que los datos sensibles **no se filtren a atacantes**.
    
#### **Pero aún hay riesgos**
- Los errores humanos o configuraciones antiguas todavía pueden permitir transferencias de zona no autorizadas.
- Por eso, intentar un **AXFR (transferencia de zona) con autorización** sigue siendo una técnica útil de **reconocimiento**.
    - Incluso si falla, puede revelar información sobre cómo está configurado el servidor y su postura de seguridad.

### Ejemplo práctico de cómo funciona la explotación de una **transferencia de zona DNS**.

####  Comando utilizado

```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

- `dig` → es una herramienta de línea de comandos para consultar servidores DNS.
- `axfr` → indica que se solicita una **transferencia completa de la zona**.
- `@nsztm1.digi.ninja` → se especifica el servidor DNS que se quiere consultar.
- `zonetransfer.me` → el dominio del que se quiere obtener la zona completa.


💡 **Nota:** Este dominio está intencionalmente configurado para permitir transferencias de zona, **solo para demostración y pruebas**.

Salida del comando:
```shell-session
Polika4RM@htb[/htb]$ dig axfr @nsztm1.digi.ninja zonetransfer.me

; <<>> DiG 9.18.12-1~bpo11+1-Debian <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
...
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
_acme-challenge.zonetransfer.me. 301 IN	TXT	"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
...
;; Query time: 10 msec
;; SERVER: 81.4.108.41#53(nsztm1.digi.ninja) (TCP)
;; WHEN: Mon May 27 18:31:35 BST 2024
;; XFR size: 50 records (messages 1, bytes 2085)
```

---

#### Qué devuelve

Si el servidor está mal configurado y permite la transferencia, el resultado incluye **todos los registros DNS** del dominio:

- **SOA Record (Start of Authority)**
    
    - Información de control del dominio, como el número de serie y el servidor principal.    
    ```
    zonetransfer.me. 7200 IN SOA nsztm1.digi.ninja. robin.digi.ninja. 2019100801
    ```
    
- **HINFO Record**
    
    - Información sobre el sistema operativo o hardware del servidor.
    
    ```
    zonetransfer.me. 300 IN HINFO "Casio fx-700G" "Windows XP"
    ```
    
- **MX Records (Mail eXchange)**
    
    - Servidores de correo asociados al dominio.    
    ```
    zonetransfer.me. 7200 IN MX 0 ASPMX.L.GOOGLE.COM.
    ```
    
- **A Records**
    
    - Direcciones IPv4 de los subdominios.
    
    ```
    zonetransfer.me. 7200 IN A 5.196.105.14
    ```
    
- **NS Records (Name Server)**
    
    - Servidores autoritativos del dominio.
    
    ```
    zonetransfer.me. 7200 IN NS nsztm1.digi.ninja.
    zonetransfer.me. 7200 IN NS nsztm2.digi.ninja.
    ```
    
- **Otros registros**
    
    - `TXT`, `SRV`, `PTR`, `AFSDB` y más, según la configuración del dominio.
    - Por ejemplo:
    
    
    ```
    _acme-challenge.zonetransfer.me. 301 IN TXT "..."
    canberra-office.zonetransfer.me. 7200 IN A 202.14.81.230
    ```
    

Cuando un servidor DNS permite una transferencia de zona, un atacante o investigador puede obtener información muy valiosa del dominio. Esto incluye subdominios que no son visibles públicamente, direcciones IP y servicios asociados, así como detalles sobre la configuración del DNS que podrían revelar vulnerabilidades o errores.

Es importante destacar que el dominio **zonetransfer.me** está especialmente diseñado para demostrar estos riesgos de manera segura y legal. Sin embargo, intentar un AXFR en dominios reales sin autorización se considera intrusión y es ilegal.

En resumen, una transferencia de zona expone un mapa completo de la infraestructura DNS, lo que hace evidente la importancia de restringir este tipo de solicitudes solo a servidores autorizados.

-----
### Questions
**1. After performing a zone transfer for the domain inlanefreight.htb on the target system, how many DNS records are retrieved from the target system's name server? Provide your answer as an integer, e.g, 123.**

Ejecutando:

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-0dkehkdn0e]─[~]
└──╼ [★]$ dig axfr @10.129.33.186 inlanefreight.htb

```

- Siendo 10.129.33.186 la dirección del servidor DNS  que estoy consultando.
- inlanefreight.htb:
	- Este es el **dominio que quieres consultar**.
	- `dig axfr` le indica al servidor DNS: “envíame **todos los registros de esta zona** dominio y subdominios)”.
	- En otras palabras, estás pidiendo al servidor `10.129.33.186` que te devuelva **toda la información DNS de `inlanefreight.htb`**.



```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-0dkehkdn0e]─[~]
└──╼ [★]$ dig axfr @10.129.33.186 inlanefreight.htb

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr @10.129.33.186 inlanefreight.htb
; (1 server found)
;; global options: +cmd
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
admin.inlanefreight.htb. 604800	IN	A	10.10.34.2
ftp.admin.inlanefreight.htb. 604800 IN	A	10.10.34.2
careers.inlanefreight.htb. 604800 IN	A	10.10.34.50
dc1.inlanefreight.htb.	604800	IN	A	10.10.34.16
dc2.inlanefreight.htb.	604800	IN	A	10.10.34.11
internal.inlanefreight.htb. 604800 IN	A	127.0.0.1
admin.internal.inlanefreight.htb. 604800 IN A	10.10.1.11
wsus.internal.inlanefreight.htb. 604800	IN A	10.10.1.240
ir.inlanefreight.htb.	604800	IN	A	10.10.45.5
dev.ir.inlanefreight.htb. 604800 IN	A	10.10.45.6
ns.inlanefreight.htb.	604800	IN	A	127.0.0.1
resources.inlanefreight.htb. 604800 IN	A	10.10.34.100
securemessaging.inlanefreight.htb. 604800 IN A	10.10.34.52
test1.inlanefreight.htb. 604800	IN	A	10.10.34.101
us.inlanefreight.htb.	604800	IN	A	10.10.200.5
cluster14.us.inlanefreight.htb.	604800 IN A	10.10.200.14
messagecenter.us.inlanefreight.htb. 604800 IN A	10.10.200.10
ww02.inlanefreight.htb.	604800	IN	A	10.10.34.112
www1.inlanefreight.htb.	604800	IN	A	10.10.34.111
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 9 msec
;; SERVER: 10.129.33.186#53(10.129.33.186) (TCP)
;; WHEN: Fri Aug 22 10:18:04 CDT 2025
;; XFR size: 22 records (messages 1, bytes 594)
```

Respuesta: 22

**Within the zone record transferred above, find the ip address for ftp.admin.inlanefreight.htb. Respond only with the IP address, eg 127.0.0.1**

Respuesta: 10.10.34.2

**Within the same zone record, identify the largest IP address allocated within the 10.10.200 IP range. Respond with the full IP address, eg 10.10.200.1**

Respuesta: 10.10.200.14
