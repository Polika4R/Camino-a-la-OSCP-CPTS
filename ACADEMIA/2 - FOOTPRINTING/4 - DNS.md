El Sistema de Nombres de Dominio (DNS) es fundamental para Internet, ya que permite traducir nombres de dominio (como academy.hackthebox.com) en direcciones IP específicas asignadas a servidores web. 

DNS no tiene una base de datos central, sino que funciona de manera distribuida a través de miles de servidores de nombres en todo el mundo. 

Estos servidores traducen los nombres de dominio en IPs y dirigen a los usuarios al servidor correcto. Existen varios tipos de servidores DNS, entre ellos: servidores raíz, servidores autoritativos, no autoritativos, de caché, de reenvío y resolutores.

- **Root DNS:** Gestionan los dominios de nivel superior y actúan como última instancia. Hay 13 servidores coordinados por ICANN.  No saben la dirección de www\.ejemplo.com_, pero sí saben quién tiene la lista de los dominios .com, .org, etc.
- **Autoritativos:** Responden con información definitiva para su zona DNS.  El servidor autoritativo de esa zona dice:   “La IP de _www\.ejemplo.com_ es 192.0.2.1”. 
- **No autoritativos:** Obtienen información de otras fuentes sin tener autoridad.  Si tú preguntas a un servidor de tu proveedor de Internet y él no es autoritativo, va buscando la respuesta por la red
- **Caché:** Guardan respuestas temporales para acelerar consultas. Si ya alguien preguntó por _www\.ejemplo.com_ hace poco, el servidor guarda la respuesta y te la da rápido, sin volver a consultar todo
- **Reenvío:** Solo envían consultas a otros servidores.
- **Resolvers:** Realizan la resolución de nombres localmente en equipos o routers.

El DNS no está cifrado en su mayoría. Por lo tanto, los dispositivos de la red local WLAN y los proveedores de internet pueden acceder y espiar las consultas DNS.

Dado que esto supone un riesgo para la privacidad, existen soluciones para el cifrado DNS. Por defecto, los profesionales de seguridad informática utilizan `DNS over TLS`( `DoT`) o `DNS over HTTPS`( `DoH`) aquí. Además, el protocolo de red `DNSCrypt`también cifra el tráfico entre el ordenador y el servidor de nombres.

Sin embargo, el DNS no solo vincula nombres de ordenadores y direcciones IP. También almacena y genera información adicional sobre los servicios asociados a un dominio. **Por lo tanto, una consulta DNS también puede utilizarse, por ejemplo, para determinar qué ordenador sirve como servidor de correo electrónico para el dominio en cuestión o cómo se llaman los servidores de nombres del dominio.

## Registros de DNS
**Registros DNS** son entradas que indican diferentes tipos de información sobre un dominio. Cada registro es como una “ficha” que le dice a Internet cómo manejar un dominio

| Registro  | ¿Qué hace?                                                                        | Ejemplo real                                               |
| --------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **A**     | Asocia un dominio con una dirección IPv4.                                         | `midominio.com → 192.168.1.1`                              |
| **AAAA**  | Igual que A, pero con IPv6 (IPs largas).                                          | `midominio.com → 2001:0db8::1`                             |
| **MX**    | Dice qué servidor recibe el correo.                                               | `correo.midominio.com` maneja el email de `@midominio.com` |
| **NS**    | Indica qué servidores DNS son los “jefes” del dominio.                            | `ns1.midominio.com`, `ns2.midominio.com`                   |
| **TXT**   | Texto libre. Muy usado para seguridad (SPF, DKIM, verificación de Google, etc.).  | `"v=spf1 include:_spf.google.com ~all"`                    |
| **CNAME** | Alias: un nombre apunta a otro dominio.                                           | `www.midominio.com → midominio.com`                        |
| **PTR**   | Traducción inversa: de IP a nombre de dominio.                                    | `192.168.1.1 → midominio.com`                              |
| **SOA**   | Datos de administración: quién gestiona el dominio, tiempo de actualización, etc. | Nombre del responsable, intervalos de refresco, etc.       |
**Metáfora rápida:**
- **A/AAAA** → Dirección de la casa.
- **MX** → El buzón de correo.
- **NS** → Los administradores de la casa.
- **TXT** → Notas en la puerta (“Aquí vive Google, no spam”).
- **CNAME** → Un cartel que dice “si buscas esta casa, en realidad está en otra dirección”.
- **PTR** → Preguntar por una dirección para saber quién vive ahí.
- **SOA** → El contrato oficial del terreno, con las reglas de gestión.

Cuando se realiza una consulta con `dig soa dominio`, te muestra información sobre el servidor que administra el dominio y el contacto del administrador. La dirección de correo que aparece tiene un punto (.) en vez de arroba (@), que es una convención en DNS.

```shell-session
Polika4RM@htb[/htb]$ dig soa <dominio>
```

```shell-session
Polika4RM@htb[/htb]$ dig soa www.inlanefreight.com

; <<>> DiG 9.16.27-Debian <<>> soa www.inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15876
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.inlanefreight.com.         IN      SOA

;; AUTHORITY SECTION:
inlanefreight.com.      900     IN      SOA     ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

;; Query time: 16 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jan 05 12:56:10 GMT 2023
;; MSG SIZE  rcvd: 128
```

El punto (.) se sustituye por una arroba (@) en la dirección de correo electrónico. En este ejemplo, la dirección de correo electrónico del administrador es `awsdns-hostmaster@amazon.com`.

## Configuración predeterminada

Los servidores DNS usan tres tipos principales de archivos de configuración:
1. **Archivos de configuración local:** Ajustan parámetros generales del servidor DNS.
2. **Archivos de zona:** Definen las zonas específicas (dominios) que gestiona el servidor.
3. **Archivos de resolución inversa:** Relacionan direcciones IP con nombres de dominio (búsqueda inversa).

| Tipo de Archivo                     | Función Principal                                              | Ejemplo / Descripción                                               |
| ----------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Archivos de configuración local** | Ajustan parámetros generales y definen zonas DNS               | `named.conf.local` — define qué dominios gestionar                  |
| **Archivos de zona**                | Contienen datos de un dominio: IPs, servidores, alias          | `/etc/bind/db.domain.com` — registros SOA, NS, A, CNAME, MX         |
| **Archivos de resolución inversa**  | Asocian direcciones IP a nombres de dominio (búsqueda inversa) | `/etc/bind/db.10.129.14` — registros PTR que convierten IP a nombre |

El servidor **Bind9** (muy usado en Linux) divide su configuración local en archivos como `named.conf.local`, `named.conf.options` y `named.conf.log`.

El archivo principal `named.conf` contiene dos secciones:
- **Opciones globales:** Aplican a todo el servidor y todas las zonas.
- **Opciones de zona:** Configuraciones específicas para cada dominio.

Si una opción está definida tanto globalmente como para una zona, la configuración de la zona tiene prioridad.

Veamos estos tres:

#### 1. Archivos de "Configuración de DNS local"
```shell-session
root@bind9:~# cat /etc/bind/named.conf.local

//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

En los archivos de configuración DNS se definen las zonas, que suelen corresponder a un solo dominio (excepto en servidores públicos o de ISP). Cada zona tiene un archivo de zona en formato BIND, que describe toda la información del dominio.

Un archivo de zona debe contener un registro SOA (autoridad de la zona) y al menos un registro NS (servidores de nombres). El SOA siempre va al inicio y es clave para la gestión del dominio. Es como la "tarjeta de identidad" del dominio. Indica quién es el responsable del dominio y cómo se maneja la información.

Estos archivos funcionan como una “guía telefónica” para que el servidor DNS resuelva nombres de dominio a direcciones IP. Un error en la sintaxis del archivo puede invalidar toda la zona, haciendo que el servidor no responda correctamente.

#### 2. Archivos de Zona
```shell-session
root@bind9:~# cat /etc/bind/db.domain.com

;
; BIND reverse data file for local loopback interface
;
$ORIGIN domain.com
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

      IN     MX     10     mx.domain.com.
      IN     MX     20     mx2.domain.com.

             IN     A       10.129.14.5

server1      IN     A       10.129.14.5
server2      IN     A       10.129.14.7
ns1          IN     A       10.129.14.2
ns2          IN     A       10.129.14.3

ftp          IN     CNAME   server1
mx           IN     CNAME   server1
mx2          IN     CNAME   server2
www          IN     CNAME   server2
```

Para que un servidor DNS pueda traducir una dirección IP a un nombre de dominio (búsqueda inversa), necesita un **archivo de zona inversa**.

En este archivo se asigna el **nombre completo del equipo (FQDN)** al último octeto de la dirección IP mediante un registro **PTR**.
Es decir, mientras que normalmente DNS convierte nombres en IPs, el archivo de zona inversa y los registros PTR hacen el proceso contrario: convierten IPs en nombres.

#### 3. Archivos de resolución inversa (Reverse Name Resolution Zone Files)
Relacionan direcciones IP con nombres de dominio (búsqueda inversa).

```shell-session
root@bind9:~# cat /etc/bind/db.10.129.14

;
; BIND reverse data file for local loopback interface
;
$ORIGIN 14.129.10.in-addr.arpa
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

5    IN     PTR    server1.domain.com.
7    IN     MX     mx.domain.com.
...SNIP...
```

Los servidores DNS, como BIND9, pueden ser vulnerables a ataques si no se configuran correctamente. Muchas veces, por facilitar el funcionamiento, los administradores priorizan la funcionalidad sobre la seguridad, lo que puede generar errores y vulnerabilidades.

Algunas opciones importantes que afectan la seguridad son:
- **allow-query:** Define quién puede hacer consultas al servidor DNS.
- **allow-recursion:** Define quién puede hacer consultas recursivas (que implican buscar información en otros servidores).
- **allow-transfer:** Define quién puede copiar (transferir) las zonas DNS completas, algo sensible porque permite replicar toda la configuración.
- **zone-statistics:** Permite recolectar datos estadísticos sobre las zonas, útil para monitoreo.


## FOOTPRINTING

Cuando hacemos **footprinting** en servidores DNS, queremos obtener información sobre ellos enviando consultas.
- Primero, podemos preguntar qué otros servidores de nombres (NS) conoce el servidor DNS, usando el **registro NS**.

#### DIG - Version Query
Para eso usamos el comando `dig ns dominio @ip_del_servidor`, donde `@ip_del_servidor` indica a qué servidor DNS le preguntamos.
- Así, obtenemos los servidores DNS que gestionan ese dominio, y sus direcciones IP.

Por ejemplo:
```
dig ns inlanefreight.htb @10.129.14.128
```

preguntamos al servidor DNS en la IP 10.129.14.128 cuáles son los servidores de nombres para el dominio inlanefreight.htb.

El resultado muestra que el servidor de nombres es `ns.inlanefreight.htb` con IP 10.129.34.136.

```shell-session
Polika4RM@htb[/htb]$ dig ns inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> ns inlanefreight.htb @10.129.14.128
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45010
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: ce4d8681b32abaea0100000061475f73842c401c391690c7 (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      NS

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:04:03 CEST 2021
;; MSG SIZE  rcvd: 107
```

`ns.inlanefreight.htb` es el servidor **autoridad (master/primary NS)** para el dominio `inlanefreight.htb`.  El servidor autoridad contiene los **archivos de zona** con todos los registros DNS del dominio (por ejemplo, las IPs asociadas a los nombres, registros MX para correo, etc.).
- Es el encargado de **responder con datos válidos y confiables** cuando alguien consulta por ese dominio.

#### DIG - ANY Query
- La consulta con la opción **ANY** pide al servidor DNS que muestre **todos los registros DNS disponibles** para un dominio.
- Es como pedir una lista completa con toda la información que el servidor tiene sobre ese dominio (registros A, MX, NS, TXT, CNAME, etc.).
- Sin embargo, **no siempre el servidor devuelve toda la información**, ya que por seguridad o configuración puede limitar qué datos muestra.

**`dig ANY dominio.com`** es una forma rápida de ver muchos registros DNS de un dominio, pero no garantiza que aparezcan absolutamente todos.

```shell-session
Polika4RM@htb[/htb]$ dig any inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> any inlanefreight.htb @10.129.14.128
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7649
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 064b7e1f091b95120100000061476865a6026d01f87d10ca (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      ANY

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:42:13 CEST 2021
;; MSG SIZE  rcvd: 437
```


#### DIG - AXFR Zone Transfer

- **¿Qué es?**  
    La transferencia de zona es el proceso por el cual un servidor DNS (normalmente un servidor secundario o slave) copia la información completa de una zona (dominio) desde el servidor primario (master).
    
- **¿Para qué sirve?**  
    Sirve para mantener sincronizados los datos DNS entre varios servidores que gestionan la misma zona, aumentando así la **fiabilidad**, facilitando la **distribución de carga** y protegiendo el servidor primario de ataques o fallos.
    
- **¿Cómo funciona?**
    - El servidor primario es el que tiene la copia original y autorizada de la zona.
    - Los servidores secundarios (slaves) consultan periódicamente el **registro SOA** del primario para comparar el número de serie.
    - Si el número de serie del SOA del primario es mayor, significa que hay cambios, y el secundario inicia una transferencia completa de la zona (AXFR) para actualizar su copia.
        
- **Seguridad:**
    - La transferencia se suele proteger con una clave secreta (`rndc-key`), para que solo los servidores autorizados puedan sincronizar la información.
    - Si la configuración de `allow-transfer` es demasiado permisiva (como una subnet amplia o `any`), cualquiera podría descargar la zona completa, revelando información sensible como direcciones IP internas o nombres de host.

El comando `dig axfr`  se usa para forzar una transferencia de zona completa desde un servidor DNS.
```
dig axfr dominio @servidor
```

```shell-session
Polika4RM@htb[/htb]$ dig axfr inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr inlanefreight.htb @10.129.14.128
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 4 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:51:19 CEST 2021
;; XFR size: 9 records (messages 1, bytes 520)
```

- La salida muestra todos los registros que conforman la zona `inlanefreight.htb` que el servidor DNS tiene.
- Incluye registros **SOA**, **TXT**, **NS**, **A**, etc.
- El número de serie del SOA es `2`, lo que indica la versión de la zona.

## Transferencia de Zona (AXFR)

- Permite descargar **todo el contenido** de una zona DNS desde un servidor autorizado.
- Si la opción `allow-transfer` está mal configurada (ej. permitiendo cualquier IP o una subred amplia), **cualquiera puede obtener la zona completa**.
- Esto revela:
    - Registros SOA, NS, TXT, y A.
    - Nombres de hosts internos y sus IP privadas.
    - Información sensible de la infraestructura DNS.
```
dig axfr internal.inlanefreight.htb @10.129.14.128`
```

```shell-session
Polika4RM@htb[/htb]$ dig axfr internal.inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr internal.inlanefreight.htb @10.129.14.128
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN   TXT     "MS=ms97310371"
internal.inlanefreight.htb. 604800 IN   TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN   TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN   NS      ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb. 604800 IN A     10.129.34.16
dc2.internal.inlanefreight.htb. 604800 IN A     10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A   10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A      10.129.34.136
vpn.internal.inlanefreight.htb. 604800 IN A     10.129.1.6
ws1.internal.inlanefreight.htb. 604800 IN A     10.129.1.34
ws2.internal.inlanefreight.htb. 604800 IN A     10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A    10.129.18.2
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:53:11 CEST 2021
;; XFR size: 15 records (messages 1, bytes 664)
```


dnsenum --dnsserver 10.129.221.106 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt dev.inlanefreight.htb


---
**QUESTIONS
**Target: 10.129.221.106**
**1. Interact with the target DNS using its IP address and enumerate the FQDN (domain) of it for the "inlanefreight.htb" domain.**

Ejecutando un: 
```
dig ns inlanefreight.htb @10.129.221.106
```
Conseguimos preguntar al servidor DNS en tal IP cuáles son los servidores de nombres para el dominio inlanefreight.htb.

Como respuesta, entre demás, encontramos: 
```
;; ANSWER SECTION:
inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
```

Siendo la respuesta *ns.inlanefreight.htb*

**2. Identify if its possible to perform a zone transfer and submit the TXT record as the answer. (Format: HTB{...})**

Ejecutamos un: 
```
dig axfr inlanefreight.htb @10.129.221.106
```

Y nos muestra: 
```
dig axfr inlanefreight.htb @10.129.221.106
; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr inlanefreight.htb @10.129.221.106
;; global options: +cmd
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.	604800	IN	TXT	"MS=ms97310371"
inlanefreight.htb.	604800	IN	TXT	"atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.	604800	IN	TXT	"v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
app.inlanefreight.htb.	604800	IN	A	10.129.18.15
dev.inlanefreight.htb.	604800	IN	A	10.12.0.1
internal.inlanefreight.htb. 604800 IN	A	10.129.1.6         <------------- [!]
mail1.inlanefreight.htb. 604800	IN	A	10.129.18.201
ns.inlanefreight.htb.	604800	IN	A	127.0.0.1
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 9 msec
;; SERVER: 10.129.221.106#53(10.129.221.106) (TCP)
;; WHEN: Wed Sep 03 05:17:18 CDT 2025
;; XFR size: 11 records (messages 1, bytes 560)
```

- El servidor DNS 10.129.221.106 permite realizar zone transfer.
- Esto devuelve todos los registros DNS del dominio inlanefreight.htb (A, TXT, NS, SOA).
- Dentro de estos registros aparece el subdominio `internal.inlanefreight.htb` con su correspondiente IP (`10.129.1.6`).

Al ejecutar: 
```
dig axfr internal.inlanefreight.htb @10.129.221.106
```

Nos muestra, entre más información, la flag:
```
internal.inlanefreight.htb. 604800 IN	TXT	"MS=ms97310371"
internal.inlanefreight.htb. 604800 IN	TXT	"HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}"
internal.inlanefreight.htb. 604800 IN	TXT	"atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN	TXT	"v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN	NS	ns.inlanefreight.htb.
```
Respuesta: *HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}*

**3. What is the IPv4 address of the hostname DC1?**
Sobre la misma salida anterior:
```
dig axfr internal.inlanefreight.htb @10.129.221.106

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr internal.inlanefreight.htb @10.129.221.106
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN	TXT	"MS=ms97310371"
internal.inlanefreight.htb. 604800 IN	TXT	"HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}"
internal.inlanefreight.htb. 604800 IN	TXT	"atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN	TXT	"v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN	NS	ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb.	604800 IN A	10.129.34.16   <---------- [!]
dc2.internal.inlanefreight.htb.	604800 IN A	10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A	10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A	127.0.0.1
vpn.internal.inlanefreight.htb.	604800 IN A	10.129.1.6
ws1.internal.inlanefreight.htb.	604800 IN A	10.129.1.34
ws2.internal.inlanefreight.htb.	604800 IN A	10.129.1.35
wsus.internal.inlanefreight.htb. 604800	IN A	10.129.18.2
internal.inlanefreight.htb. 604800 IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 6 msec
;; SERVER: 10.129.221.106#53(10.129.221.106) (TCP)
;; WHEN: Wed Sep 03 05:27:30 CDT 2025
;; XFR size: 15 records (messages 1, bytes 677)
```

Vemos como el dirección IP asociada a dc1.internal.inlanefreight.htb es la *10.129.34.16*

**4. What is the FQDN of the host where the last octet ends with "x.x.x.203"?**

Forzando una zona de transferencia sobre el dominio inlanefreight.htb, observo como existen varios subdominios e IP's internas:

```
dig axfr inlanefreight.htb @10.129.221.106
; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr inlanefreight.htb @10.129.221.106
;; global options: +cmd
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.	604800	IN	TXT	"MS=ms97310371"
inlanefreight.htb.	604800	IN	TXT	"atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.	604800	IN	TXT	"v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
app.inlanefreight.htb.	604800	IN	A	10.129.18.15
dev.inlanefreight.htb.	604800	IN	A	10.12.0.1
internal.inlanefreight.htb. 604800 IN	A	10.129.1.6         <------------- [!]
mail1.inlanefreight.htb. 604800	IN	A	10.129.18.201
ns.inlanefreight.htb.	604800	IN	A	127.0.0.1
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 9 msec
;; SERVER: 10.129.221.106#53(10.129.221.106) (TCP)
;; WHEN: Wed Sep 03 05:17:18 CDT 2025
;; XFR size: 11 records (messages 1, bytes 560)
```

Intentando forzar zonas de transferencias sobre:
- inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
- app.inlanefreight.htb.	604800	IN	A	10.129.18.15
- dev.inlanefreight.htb.	604800	IN	A	10.12.0.1
- internal.inlanefreight.htb. 604800 IN	A	10.129.1.6        
- mail1.inlanefreight.htb. 604800	IN	A	10.129.18.201
- ns.inlanefreight.htb.	604800	IN	A	127.0.0.1

Observo que la única que me funciona es:
```
dig axfr internal.inlanefreight.htb @10.129.221.106
dig axfr ns.inlanefreight.htb @10.129.221.106
dig axfr dev.inlanefreight.htb @10.129.221.106
```


Ejecutando fuerza bruta contra el dominio dev.inlanefreight.htb con dicho diccionario concreto, observo:
```
dnsenum --dnsserver 10.129.221.106 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/fierce-hostlist.txt dev.inlanefreight.htb

```

Observo esta salida de datos:
```
Brute forcing with /opt/useful/seclists/Discovery/DNS/fierce-hostlist.txt:
___________________________________________________________________________

dev1.dev.inlanefreight.htb.              604800   IN    A         10.12.3.6
ns.dev.inlanefreight.htb.                604800   IN    A         127.0.0.1
win2k.dev.inlanefreight.htb.             604800   IN    A        10.12.3.203

```
