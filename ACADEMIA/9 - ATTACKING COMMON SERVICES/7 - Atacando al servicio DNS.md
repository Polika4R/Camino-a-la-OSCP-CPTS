DNS traduce nombres de dominio (ej. hackthebox.com) a direcciones IP (ej. 104.17.42.72). 
Usa UDP/53 por defecto y TCP/53 como reserva (p. ej. cuando la respuesta es demasiado grande para UDP).

Casi todas las aplicaciones dependen de DNS, por lo que atacar servidores DNS es una amenaza muy relevante.

Permite obtener datos sobre la organización (servicios, proveedores terceros, correos, etc.) durante la fase de enumeración/footprinting.


## Enumeración
Para numerar inicialmente un servicio DNS, podemos utilizar los parmámetros -sCV de nmap:
```shell-session
Polika4RM@htb[/htb]# nmap -p53 -Pn -sV -sC 10.10.110.213

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-29 03:47 EDT
Nmap scan report for 10.10.110.213
Host is up (0.017s latency).

PORT    STATE  SERVICE     VERSION
53/tcp  open   domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
```

---

## Transferencia de zona DNS
Una zona DNS es una parte del espacio de nombres DNS que gestiona una organización o un administrador específico. 

Dado que el DNS comprende varias zonas DNS, los servidores DNS utilizan transferencias de zona DNS para copiar una parte de su base de datos a otro servidor DNS. A menos que un servidor DNS esté configurado correctamente (lo que limita las IP que pueden realizar una transferencia de zona DNS), cualquiera puede solicitarle una copia de la información de su zona, ya que las transferencias de zona DNS no requieren autenticación. Además, el servicio DNS suele ejecutarse en un puerto UDP; sin embargo, al realizar la transferencia de zona DNS, utiliza un puerto TCP para una transmisión de datos fiable.

Un atacante podría aprovechar esta vulnerabilidad de transferencia de zona DNS para obtener más información sobre el espacio de nombres DNS de la organización objetivo, lo que aumenta la superficie de ataque. Para su explotación, podemos usar la `dig`utilidad con `AXFR`la opción de tipo de consulta DNS para volcar todos los espacios de nombres DNS de un servidor DNS vulnerable:

```shell-session
Polika4RM@htb[/htb]# dig AXFR @ns1.inlanefreight.htb inlanefreight.htb

; <<>> DiG 9.11.5-P1-1-Debian <<>> axfr inlanefrieght.htb @10.129.110.213
;; global options: +cmd
inlanefrieght.htb.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
inlanefrieght.htb.         604800  IN      AAAA    ::1
inlanefrieght.htb.         604800  IN      NS      localhost.
inlanefrieght.htb.         604800  IN      A       10.129.110.22
admin.inlanefrieght.htb.   604800  IN      A       10.129.110.21
hr.inlanefrieght.htb.      604800  IN      A       10.129.110.25
support.inlanefrieght.htb. 604800  IN      A       10.129.110.28
inlanefrieght.htb.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 28 msec
;; SERVER: 10.129.110.213#53(10.129.110.213)
;; WHEN: Mon Oct 11 17:20:13 EDT 2020
;; XFR size: 8 records (messages 1, bytes 289)
```

- @ns1.inlanefreight.htb:
	  Especifica el servidor DNS al que le hacemos la consulta.
- inlanefreight.htb:
	  Nombre de la zona que queremos transferir.

---
Fierce no empieza sabiendo cual es el servidor autoritativo (el servidor que almacena las respuestas oficiales para ese dominio). 

Fierce:
- descubre servidores DNS autoritativos
- cuando los tiene, consultará (si permite) una zona de transferencia

```shell-session
Polika4RM@htb[/htb]# fierce --domain zonetransfer.me

NS: nsztm2.digi.ninja. nsztm1.digi.ninja.
SOA: nsztm1.digi.ninja. (81.4.108.41)
Zone: success
{<DNS name @>: '@ 7200 IN SOA nsztm1.digi.ninja. robin.digi.ninja. 2019100801 '
               '172800 900 1209600 3600\n'
               '@ 300 IN HINFO "Casio fx-700G" "Windows XP"\n'
               '@ 301 IN TXT '
               '"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"\n'
               '@ 7200 IN MX 0 ASPMX.L.GOOGLE.COM.\n'
               '@ 7200 IN MX 10 ALT1.ASPMX.L.GOOGLE.COM.\n'
               '@ 7200 IN MX 10 ALT2.ASPMX.L.GOOGLE.COM.\n'
               '@ 7200 IN MX 20 ASPMX2.GOOGLEMAIL.COM.\n'
               '@ 7200 IN MX 20 ASPMX3.GOOGLEMAIL.COM.\n'
               '@ 7200 IN MX 20 ASPMX4.GOOGLEMAIL.COM.\n'
               '@ 7200 IN MX 20 ASPMX5.GOOGLEMAIL.COM.\n'
               '@ 7200 IN A 5.196.105.14\n'
               '@ 7200 IN NS nsztm1.digi.ninja.\n'
               '@ 7200 IN NS nsztm2.digi.ninja.',
 <DNS name _acme-challenge>: '_acme-challenge 301 IN TXT '
                             '"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"',
 <DNS name _sip._tcp>: '_sip._tcp 14000 IN SRV 0 0 5060 www',
 <DNS name 14.105.196.5.IN-ADDR.ARPA>: '14.105.196.5.IN-ADDR.ARPA 7200 IN PTR '
                                       'www',
 <DNS name asfdbauthdns>: 'asfdbauthdns 7900 IN AFSDB 1 asfdbbox',
 <DNS name asfdbbox>: 'asfdbbox 7200 IN A 127.0.0.1',
 <DNS name asfdbvolume>: 'asfdbvolume 7800 IN AFSDB 1 asfdbbox',
 <DNS name canberra-office>: 'canberra-office 7200 IN A 202.14.81.230',
 <DNS name cmdexec>: 'cmdexec 300 IN TXT "; ls"',
 <DNS name contact>: 'contact 2592000 IN TXT "Remember to call or email Pippa '
                     'on +44 123 4567890 or pippa@zonetransfer.me when making '
                     'DNS changes"',
 <DNS name dc-office>: 'dc-office 7200 IN A 143.228.181.132',
 <DNS name deadbeef>: 'deadbeef 7201 IN AAAA dead:beaf::',
 <DNS name dr>: 'dr 300 IN LOC 53 20 56.558 N 1 38 33.526 W 0.00m',
 <DNS name DZC>: 'DZC 7200 IN TXT "AbCdEfG"',
 <DNS name email>: 'email 2222 IN NAPTR 1 1 "P" "E2U+email" "" '
                   'email.zonetransfer.me\n'
                   'email 7200 IN A 74.125.206.26',
 <DNS name Hello>: 'Hello 7200 IN TXT "Hi to Josh and all his class"',
 <DNS name home>: 'home 7200 IN A 127.0.0.1',
 <DNS name Info>: 'Info 7200 IN TXT "ZoneTransfer.me service provided by Robin '
                  'Wood - robin@digi.ninja. See '
                  'http://digi.ninja/projects/zonetransferme.php for more '
                  'information."',
 <DNS name internal>: 'internal 300 IN NS intns1\ninternal 300 IN NS intns2',
 <DNS name intns1>: 'intns1 300 IN A 81.4.108.41',
 <DNS name intns2>: 'intns2 300 IN A 167.88.42.94',
 <DNS name office>: 'office 7200 IN A 4.23.39.254',
 <DNS name ipv6actnow.org>: 'ipv6actnow.org 7200 IN AAAA '
                            '2001:67c:2e8:11::c100:1332',
...SNIP...
```
---

# Adquisiciones de dominios y enumeración de subdominios

La toma de control de dominio (domain takeover) consiste en registrar un nombre de dominio inexistente para obtener el control sobre otro dominio. Si los atacantes encuentran un dominio expirado, pueden reclamarlo para realizar ataques adicionales, como alojar contenido malicioso en un sitio web o enviar correos de phishing utilizando el dominio reclamado.

Imaginemos que una empresa tiene un dominio llamado **empresa.com**.  
Dentro de ese dominio, puede haber subdominios como **blog.empresa.com** o **tienda.empresa.com**.

Ahora bien:
- Si el dominio **empresa.com** deja de pagarse o caduca, **cualquiera** puede volver a registrarlo.
- Si un atacante lo registra, pasa a tener **control total** sobre ese dominio.
- Con ese control, puede hacer cosas peligrosas, como:
    - Crear una web falsa con aspecto legítimo.

La toma de control de dominio también es posible con subdominios, lo que se conoce como toma de control de subdominio (subdomain takeover).
Un registro CNAME (Canonical Name) de DNS se utiliza para mapear diferentes dominios hacia un dominio principal.
Muchas organizaciones utilizan servicios de terceros como AWS, GitHub, Akamai, Fastly y otras redes de entrega de contenido (CDN) para alojar su contenido. 

En estos casos, normalmente crean un subdominio y lo hacen apuntar a esos servicios. 
Por ejemplo, “si buscas **sub.target.com**, en realidad ve a **anotherdomain.com**”.:
```shell-session
sub.target.com.   60   IN   CNAME   anotherdomain.com
```

El nombre de dominio (por ejemplo, **sub.target.com**) utiliza un registro **CNAME** que apunta a otro dominio (por ejemplo, **anotherdomain.com**).  
Supongamos que **anotherdomain.com** caduca y queda disponible para que cualquiera lo registre.  
Dado que el servidor DNS de **target.com** aún tiene el registro CNAME apuntando a ese dominio, **cualquiera que registre anotherdomain.com tendrá control total sobre sub.target.com** hasta que se actualice el registro DNS.

---

### Subdomain Enumeration

Antes de realizar un subdomain takeover, deberíamos enumerar los subdominios del dominio objetivo usando herramientas como Subfinder. Esta herramienta puede raspar subdominios de fuentes abiertas como DNSdumpster. Otras herramientas, como Sublist3r, también pueden usarse para forzar por fuerza bruta subdominios proporcionando una lista de palabras (wordlist) pre-generada.

```shell-session
Polika4RM@htb[/htb]# ./subfinder -d inlanefreight.com -v       
                                                                       
        _     __ _         _                                           
____  _| |__ / _(_)_ _  __| |___ _ _          
(_-< || | '_ \  _| | ' \/ _  / -_) '_|                 
/__/\_,_|_.__/_| |_|_||_\__,_\___|_| v2.4.5                                                                                                                                                                                                                                                 
                projectdiscovery.io                    
                                                                       
[WRN] Use with caution. You are responsible for your actions
[WRN] Developers assume no liability and are not responsible for any misuse or damage.
[WRN] By using subfinder, you also agree to the terms of the APIs used. 
                                   
[INF] Enumerating subdomains for inlanefreight.com
[alienvault] www.inlanefreight.com
[dnsdumpster] ns1.inlanefreight.com
[dnsdumpster] ns2.inlanefreight.com
...snip...
[bufferover] Source took 2.193235338s for enumeration
ns2.inlanefreight.com
www.inlanefreight.com
ns1.inlanefreight.com
support.inlanefreight.com
[INF] Found 4 subdomains for inlanefreight.com in 20 seconds 11 milliseconds
```

Una excelente alternativa es la herramienta Subbrute. Esta herramienta nos permite automatizar la búsqueda de subdominios de un dominio concreto.
```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
Polika4RM@htb[/htb]$ cd subbrute
Polika4RM@htb[/htb]$ echo "ns1.inlanefreight.com" > ./resolvers.txt
Polika4RM@htb[/htb]$ ./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt

Warning: Fewer than 16 resolvers per process, consider adding more nameservers to resolvers.txt.
inlanefreight.com
ns2.inlanefreight.com
www.inlanefreight.com
ms1.inlanefreight.com
support.inlanefreight.com

<SNIP>
```

Hemos encontrado 4 subdominios para el dominio "inlanefreight.htb".
Podemos numerar los CNAME records (los archivos que indican a donde apunta cada subdominio). Lo haremos ejecutando:
```shell-session
Polika4RM@htb[/htb]# host support.inlanefreight.com

support.inlanefreight.com is an alias for inlanefreight.s3.amazonaws.com
```

El subdominio de soporte tiene un registro de alias que apunta a un bucket de AWS S3. Sin embargo, la URL https\://support.inlanefreight.com muestra un error "NoSuchBucket", lo que indica que el subdominio es potencialmente vulnerable a una apropiación indebida. Ahora, podemos apropiarnos del subdominio creando un bucket de AWS S3 con el mismo nombre.

El repositorio can-i-take-over-xyz también es una excelente referencia para vulnerabilidades de toma de control de subdominios. Indica si los servicios objetivo son vulnerables a una toma de control de subdominios y proporciona directrices para evaluar la vulnerabilidad.

---

# DNS spoofing
Consiste en manipular entradas DNS legítimas para devolver direcciones falsas y redirigir tráfico a un servidor controlado por el atacante (también llamado DNS Cache Poisoning).

Diferentes rutas de ataques podrían ser:
- Ataque MITM (man in the middle):
	  Un atacante puede interceptar la comunicación entre un usuario y un servidor DNS para enrutar al usuario a un destino fraudulento.
- Una vulnerabilidad encontrada en un servidor DNS podría permitir que un atacante tome el control del servidor para modificar los registros DNS.

### DNS Cache Poisoning
Desde la perspectiva de una red local, un atacante también puede realizar envenenamiento de caché DNS utilizando herramientas MITM como [Ettercap](https://www.ettercap-project.org/) o [Bettercap](https://www.bettercap.org/) .

Para explotar el envenenamiento de caché DNS a través de `Ettercap`, primero debemos editar el archivo `/etc/ettercap/etter.dns` para asignar el nombre de dominio de destino (por ejemplo, `inlanefreight.com`) que quieren falsificar y la dirección IP del atacante (por ejemplo, `192.168.225.110`) a la que quieren redirigir a un usuario:

```shell-session
Polika4RM@htb[/htb]# cat /etc/ettercap/etter.dns

inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110
```

A continuación, iniciamos la herramienta Ettercap para escanear hosts activos dentro de la red. Para ello, utilizaremos: `Hosts> Scan for Hosts.`

- Ettercap permite seleccionar **dos objetivos** para lanzar el ataque MITM (Man-In-The-Middle).
    - **Target1** = la **víctima** (la máquina a la que quieres engañar). Ej.: `192.168.152.129`.
    - **Target2** = la **puerta de enlace** (gateway/router) de la red, o a veces otra máquina con la que la víctima se comunica. Ej.: `192.168.152.2`.

- Al colocar a la víctima en Target1 y al gateway en Target2, Ettercap hará que **ambos** (víctima y gateway) crean que la IP del otro corresponde a la MAC del atacante. Eso sitúa al atacante en medio del flujo de tráfico.

Dentro de `Plugins > Manage Plugins` activaremos la opción de dns_spoof.
Esto envia a la máquina víctima falsas respuestas de resolución DNS que convertirá la petición inlanefreight.com a la IP 192.168.225.110.

Después de un ataque de suplantación de DNS exitoso, si un usuario víctima que proviene de la máquina de destino 192.168.152.129 visita el dominio inlanefreight.com en un navegador web, será redirigido a una página falsa alojada en la dirección IP 192.168.225.110:![[etter_site.webp]]

Además, un ping proveniente de la dirección IP víctima 192.168.152.129 a inlanefreight.com también debería resolverse en 192.168.225.110:
```cmd-session
C:\>ping inlanefreight.com

Pinging inlanefreight.com [192.168.225.110] with 32 bytes of data:
Reply from 192.168.225.110: bytes=32 time<1ms TTL=64
Reply from 192.168.225.110: bytes=32 time<1ms TTL=64
Reply from 192.168.225.110: bytes=32 time<1ms TTL=64
Reply from 192.168.225.110: bytes=32 time<1ms TTL=64

Ping statistics for 192.168.225.110:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

---
**Target:10.129.130.73
**1. Find all available DNS records for the "inlanefreight.htb" domain on the target name server and submit the flag found as a DNS record as the answer.**


Realizo un escaneo básico en nmap con:
```
nmap -sCV --open --min-rate 2000 -Pn -n 10.129.203.6
```

Y me devuelve un servicio DNS corriendo en el puert TCP53: 
```
53/tcp   open  domain      ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
```
Añado al /etc/host la dirección del target: 10.129.203.6

Intento hacer una transferencia de zona con dicho servidor DNS pero esta se rechaza: 

```
dig AXFR @10.129.203.6 inlanefreight.htb

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> AXFR @10.129.203.6 inlanfefreight.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

Hacemos fuerza bruta de subdominios con "Subbrute":
```
Polika4RM@htb[/htb]$ git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
Polika4RM@htb[/htb]$ cd subbrute
Polika4RM@htb[/htb]$ echo "inlanefreight.htb" > ./resolvers.txt
Polika4RM@htb[/htb]$ ./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt

inlanefreight.htb
hr.inlanefreight.htb
```

Y encuentro este subdominio.

Ejecuto finalmente una petición de zona de transferencia a:
```
dig AXFR @10.129.203.6 hr.inlanefreight.htb

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> AXFR @10.129.203.6 hr.inlanefreight.htb
; (1 server found)
;; global options: +cmd
hr.inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
hr.inlanefreight.htb.	604800	IN	TXT	"HTB{LUIHNFAS2871SJK1259991}"
hr.inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
ns.hr.inlanefreight.htb. 604800	IN	A	127.0.0.1
hr.inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 3 msec
;; SERVER: 10.129.203.6#53(10.129.203.6) (TCP)
;; WHEN: Mon Oct 20 08:35:03 CDT 2025
;; XFR size: 5 records (messages 1, bytes 230)

```

Y encuentro la flag / repuesta: "HTB{LUIHNFAS2871SJK1259991}"
