El **reconocimiento DNS** utiliza herramientas especializadas para consultar servidores DNS y extraer información valiosa sobre dominios, subdominios e infraestructura relacionada. Las más usadas son:

- **dig**: Herramienta versátil para consultas detalladas y análisis de registros.
- **nslookup**: Sencilla, para comprobaciones rápidas de resolución y servidores de correo.
- **host**: Consultas básicas con salida concisa.
- **dnsenum**: Automatiza enumeración, fuerza bruta y descubrimiento de subdominios.
- **fierce**: Intuitiva, con búsquedas recursivas y detección de comodines.
- **dnsrecon**: Combina múltiples técnicas de reconocimiento en una sola herramienta.
- **theHarvester**: Orientada a OSINT, recolecta registros DNS, correos electrónicos e info de empleados.
- **Servicios DNS en línea**: Interfaces web para consultas rápidas cuando no se dispone de herramientas en consola.

## **El comando dig (Domain Information Groper)**

`dig` es una utilidad versátil y potente para consultar servidores DNS y obtener diferentes tipos de registros. 

#### **Comandos comunes de dig**:
(ver sección anterior para relacionar cada código)

| **Comando**                     | **Descripción**                                                                                                                                          |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dig domain.com`                | Consulta por defecto (A record) del dominio.                                                                                                             |
| `dig domain.com A`              | Obtiene la dirección IPv4 del dominio.                                                                                                                   |
| `dig domain.com AAAA`           | Obtiene la dirección IPv6 del dominio.                                                                                                                   |
| `dig domain.com MX`             | Muestra los servidores de correo (registros MX) del dominio.                                                                                             |
| `dig domain.com NS`             | Identifica los servidores de nombres autoritativos del dominio.                                                                                          |
| `dig domain.com TXT`            | Recupera registros TXT asociados al dominio.                                                                                                             |
| `dig domain.com CNAME`          | Obtiene el registro de nombre canónico (CNAME).                                                                                                          |
| `dig domain.com SOA`            | Recupera el registro de inicio de autoridad (SOA).                                                                                                       |
| `dig @1.1.1.1 domain.com`       | Especifica un servidor DNS concreto (ejemplo: 1.1.1.1).                                                                                                  |
| `dig +trace domain.com`         | Muestra la ruta completa de la resolución DNS.                                                                                                           |
| `dig -x 192.168.1.1`            | Realiza una búsqueda inversa (reverse lookup) para obtener el nombre asociado a una IP.                                                                  |
| `dig +short domain.com`         | Devuelve una respuesta breve y concisa.                                                                                                                  |
| `dig +noall +answer domain.com` | Muestra solo la sección de respuesta.                                                                                                                    |
| `dig domain.com ANY`            | Intenta recuperar todos los registros DNS disponibles (aunque muchos servidores ignoran esta consulta por razones de seguridad y carga, según RFC 8482). |

## Ejemplo de DIG

COMANDO **DIG**:

```shell-session
Polika4RM@htb[/htb]$ dig google.com

; <<>> DiG 9.18.24-0ubuntu0.22.04.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16449
;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             0       IN      A       142.251.47.142

;; Query time: 0 msec
;; SERVER: 172.23.176.1#53(172.23.176.1) (UDP)
;; WHEN: Thu Jun 13 10:45:58 SAST 2024
;; MSG SIZE  rcvd: 54
```

Salida del comando dig google.com:
#### 1. **Encabezado (Header)**

```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16449
;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available
```

- **opcode: QUERY** → Es una consulta normal.
- **status: NOERROR** → La consulta fue exitosa.
- **id: 16449** → Identificador único de esta consulta.
- **flags:**
    - **qr** → Es una respuesta (Query Response).
    - **rd** → Se pidió recursión (Recursion Desired).
    - **ad** → Datos auténticos (Autentic Data).

- **QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0** → Hubo 1 pregunta, 1 respuesta, y no se devolvieron registros de autoridad ni adicionales.
- **WARNING** → El servidor no admite recursión, aunque se solicitó.
    

#### 2. **Sección de Pregunta (Question Section)**

```
;google.com.                    IN      A
```

- Indica lo que se consultó: el **registro A** (IPv4) del dominio **google.com**.

####  3. **Sección de Respuesta (Answer Section)**

```
google.com.             0       IN      A       142.251.47.142
```

- El resultado: la IP de **google.com** es **142.251.47.142**.
    
- El **0** es el TTL (tiempo de vida en caché), en este caso indica que debe refrescarse inmediatamente.
    

---

#### 4. **Pie de la respuesta (Footer)**

```
;; Query time: 0 msec
;; SERVER: 172.23.176.1#53(172.23.176.1) (UDP)
;; WHEN: Thu Jun 13 10:45:58 SAST 2024
;; MSG SIZE  rcvd: 54
```

- **Query time: 0 msec** → La respuesta fue inmediata.
- **SERVER** → La consulta fue resuelta por el servidor DNS **172.23.176.1** en el puerto **53/UDP**.
- **WHEN** → Fecha y hora de la consulta.
- **MSG SIZE** → Tamaño total de la respuesta (54 bytes).
#### Nota sobre EDNS

En algunas consultas aparece una sección **OPT Pseudosection**. Esto se debe a **EDNS (Extension Mechanisms for DNS)**, que permite:
- Mensajes DNS más grandes.
- Soporte para **DNSSEC** (seguridad).

#### Versión resumida con `+short`

Si solo quieres la **IP directamente**, sin toda la salida:

```bash
dig google.com +short
```

Devuelve únicamente:

```
142.251.47.142
```

----
##### Questions:

**1. Which IP address maps to inlanefreight.com?**

Ejecuto el comando:

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-8uex6xx94q]─[~]
└──╼ [★]$dig inlanefreight.com +short
134.209.24.248
```

O puedo ejecutar un:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-8uex6xx94q]─[~]
└──╼ [★]$ nslookup inlanefreight.com
Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
Name:	inlanefreight.com
Address: 134.209.24.248
Name:	inlanefreight.com
Address: 2a03:b0c0:1:e0::32c:b001

```


**2. Which domain is returned when querying the PTR record for 134.209.24.248?**

Dos opciones: 
Puedo buscar directamente en google la dirección: 
134.209.24.248, y me redirige a la página:
`https://www.inlanefreight.com/`

O puedo ejecutar un:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-8uex6xx94q]─[~]
└──╼ [★]$ nslookup 134.209.24.248
248.24.209.134.in-addr.arpa	name = inlanefreight.com.
```

**Respuesta:**  inlanefreight.com

**What is the full domain returned when you query the mail records for facebook.com?**
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-8uex6xx94q]─[~]
└──╼ [★]$ nslookup -type=mx facebook.com
Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
facebook.com	mail exchanger = 10 smtpin.vvv.facebook.com.

Authoritative answers can be found from:
```

**Respuesta**: smtpin.vvv.facebook.com


