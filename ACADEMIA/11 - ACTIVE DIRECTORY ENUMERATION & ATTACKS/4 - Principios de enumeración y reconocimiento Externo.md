Antes de comenzar cualquier **prueba de penetración (pentest)**, es fundamental realizar una labor de reconocimiento externo del objetivo. 
Este proceso se asemeja a "familiarizarse con el terreno" y tiene como objetivo garantizar que se realice la prueba más completa posible.

Sus funciones principales son:
- **Validar** la información y el alcance proporcionados por el cliente.
- **Asegurar** que las acciones se dirigen al alcance correcto, especialmente al trabajar de forma remota.
- **Identificar cualquier información de acceso público** que pueda influir en el resultado de la prueba, como **credenciales filtradas**, formatos de nombres de usuario o enlaces a sitios internos.

El reconocimiento puede ir desde tareas sencillas, como obtener el formato de los nombres de usuario de un sitio web público, hasta una búsqueda más profunda, como **escanear repositorios de GitHub** en busca de **credenciales** dejadas accidentalmente, o buscar en documentos enlaces a la **intranet** o sitios de acceso remoto. Esto ayuda a comprender la configuración del entorno empresarial del objetivo e identificar posibles fugas de información.

## ¿Qué estamos buscando?

La siguiente tabla resume el conjunto de conceptos que deberíamos buscar públicamente en internet para tener una mejor idea del objetivo:

| **Punto de Datos**          | **Descripción**                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Espacio IP**              | Incluye el **ASN** (Sistema Autónomo), los **bloques de red** utilizados para la infraestructura pública, la presencia en la **nube** y sus proveedores de _hosting_, y las entradas de **registros DNS**.                                                                                                                                                                        |
| **Información del Dominio** | Datos sobre la **administración del dominio**, subdominios asociados, y servicios de dominio de acceso público (servidores de correo, sitios web, portales VPN). También se busca identificar **defensas** en uso (SIEM, AV, IPS/IDS).                                                                                                                                            |
| **Formato de Esquema**      | Descubrir el formato de las **cuentas de correo electrónico**, los **nombres de usuario de AD** (Active Directory) e incluso las **políticas de contraseñas**. Esta información es vital para construir listas de usuarios para ataques como _password spraying_ (pulverización de contraseñas) o _credential stuffing_.                                                          |
| **Divulgaciones de Datos**  | Búsqueda de **archivos accesibles públicamente** (.pdf, .ppt, .docx, .xlsx, etc.) que puedan revelar información interna, como listados de sitios de **intranet**, metadatos de usuario, rutas a comparticiones (shares), o detalles de _software_/_hardware_ crítico. Esto incluye credenciales subidas a **GitHub** o formatos de nombre de usuario en los metadatos de un PDF. |
| **Datos de Brechas**        | Información divulgada públicamente, como **nombres de usuario, contraseñas** u otra información crítica, que pueda ser utilizada para obtener un **punto de apoyo** inicial.                                                                                                                                                                                                      |

## ¿Donde buscamos dicha información?
| **Categoría de Datos**                     | **Recurso/Ejemplo**                                      | **Descripción**                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------ | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Registros ASN / IP**                     | IANA, ARIN (Américas), RIPE (Europa), BGP Toolkit.       | Sitios para buscar la validez del Sistema Autónomo (ASN), los bloques de red y la presencia en la nube del objetivo.<br>Los registradores de direcciones IP y ASN son Registros Regionales de Internet (RIR), que son organizaciones que administran y asignan direcciones IP y Números de Sistema Autónomo (ASN) para regiones geográficas específicas. |
| **Registros de Dominio y DNS**             | Domaintools, PTRArchive, ICANN.                          | Uso de herramientas y solicitudes manuales a servidores DNS (como `8.8.8.8`) para obtener información del dominio, subdominios y quién lo administra.                                                                                                                                                                                                    |
| **Redes Sociales**                         | LinkedIn, Twitter, Facebook, sitios de noticias.         | Búsqueda de información relevante sobre la organización en plataformas sociales y artículos de noticias.                                                                                                                                                                                                                                                 |
| **Sitios Web Públicos**                    | La web principal de la empresa.                          | Análisis de páginas como "Acerca de nosotros" y "Contáctenos", así como documentos incrustados y artículos de noticias para obtener información relevante.                                                                                                                                                                                               |
| **Almacenamiento en la Nube y Desarrollo** | GitHub, _buckets_ de AWS S3, contenedores de Azure Blob. | Uso de estos repositorios y almacenamiento público, a menudo mediante búsquedas avanzadas en Google ("**Dorks**"), para encontrar credenciales o código filtrado.                                                                                                                                                                                        |
| **Fuentes de Datos de Brechas**            | HaveIBeenPwned, Dehashed.                                | Búsqueda de cuentas de correo corporativas en datos de brechas para encontrar contraseñas o hashes que puedan probarse contra portales de inicio de sesión externos (VPN, OWA, Citrix, etc.) que utilicen autenticación AD.                                                                                                                              |

---

# Descubriendo "Address Spaces"

La herramienta BGP (de Hurricane Electric) sirve para numerar direcciones asignadas a una organización y muestra en que ASN residen:
>https://bgp.he.net/?gad_source=1&gad_campaignid=21110334175&gbraid=0AAAAAD-rri-qwNUAbPRFKFdIeaLC7kK_4&gclid=Cj0KCQjwsPzHBhDCARIsALlWNG1MUaCgNRWMX5Tu6Ghl6v4_cN84L0lDASOqHIuFcIOpj4oACw3AU64aAu5hEALw_wcB
- ASN:
	  Imaginemos Internet como un mapa de carreteras gigantesco. Un ASN es el número que identifica a un Sistema Autónomo (AS), que es una colección de routers y redes controladas por una sola entidad (una empresa grande, una universidad, un proveedor de servicios de Internet como Telefónica o Vodafone, etc.).
	  - El ASN es un número único que esa entidad registra para que el resto de Internet sepa por dónde enviar tráfico a sus direcciones IP.
	  - Ruteo: El AS usa ese número para "anunciar" qué direcciones IP controla. Los routers de todo el mundo usan esta información para decidir la ruta más eficiente para que un paquete de datos llegue desde el origen al destino.

Para utilizar dicha herramienta, tan solo tendremos que introducir una dirección IP y se nos retornará todos los resultados posibles. 
Muchas empresas, al alojar una infraestructura tan grande tendrás su propio ASN.
Empresas más pequeñas suelen alojar sus sitios web y infraestructuras en un espacio del tipo CloudFare, Google Cloud, AWS o Azure. 

# DNS

El DNS es una forma muy buena de validar nuestro alcance y descubrir hosts accesibles que el cliente no indicó en su documento de alcance.
- Sitios web como *domaintools* y *viewdns.info* son excelentes puntos de partida.

El DNS es una excelente manera de validar nuestro alcance y descubrir hosts accesibles que el cliente no indicó en su documento de alcance. Sitios como domaintools y viewdns.info son excelentes puntos de partida. Podemos obtener numerosos registros y otros datos, desde la resolución DNS hasta las pruebas de DNSSEC y si el sitio es accesible en países con restricciones. A veces, podemos encontrar hosts adicionales fuera del alcance, pero que parecen interesantes. En ese caso, podríamos presentar esta lista a nuestro cliente para ver si alguno de ellos debería incluirse en el alcance. También podemos encontrar subdominios interesantes que no figuraban en los documentos de alcance, pero que residen en direcciones IP dentro del alcance y, por lo tanto, son válidos.

# Información Pública
Las redes sociales y los sitios de empleo pueden revelar información valiosa sobre una empresa, como su estructura, tecnología y seguridad. Por ejemplo, una oferta para un administrador de SharePoint muestra que la empresa tiene un sistema consolidado con medidas de seguridad, copias de respaldo y recuperación ante desastres. Además, menciona el uso de SharePoint 2013 y 2016, lo que sugiere actualizaciones que podrían generar vulnerabilidades y la coexistencia de distintas versiones del software.

Por otro lado, información pública, como publicaciones en redes sociales o sitios de empleo, puede revelar mucho sobre una organización: su estructura interna, tecnologías utilizadas e incluso detalles de seguridad. Los sitios web corporativos también son una fuente valiosa, ya que suelen incluir correos de contacto, números telefónicos, organigramas y documentos publicados que, en ocasiones, contienen enlaces a la infraestructura interna o a intranets. Además, plataformas como GitHub, AWS u otros servicios en la nube pueden exponer información sensible por error, como credenciales o notas de desarrolladores. Detectar estos datos puede ofrecer una ventaja significativa frente a métodos más lentos como el _password spraying_ o la fuerza bruta. Herramientas como Trufflehog o sitios como Greyhat Warfare permiten localizar estas filtraciones con mayor eficacia. En conjunto, estas actividades forman parte de la fase de reconocimiento externo dentro de una auditoría de seguridad, la cual comienza con una recolección pasiva de información y avanza hacia una enumeración activa, donde se valida y profundiza lo obtenido para entender mejor el entorno del objetivo.


# Principios generales de enumeración
Nuestro objetivo es comprender mejor al objetivo y encontrar todas las vías posibles que puedan ofrecer una ruta hacia el interior. La enumeración es un proceso iterativo que se repetirá varias veces a lo largo de una prueba de penetración. Además del documento de alcance proporcionado por el cliente, la información recopilada en esta fase será nuestra principal fuente de datos, por lo que debemos asegurarnos de no dejar piedra sin remover. Al iniciar la enumeración empezamos por recursos pasivos, abarcando un espectro amplio que iremos afinando progresivamente. Una vez agotada la primera tanda de técnicas pasivas, analizaremos los resultados y pasaremos a la fase de enumeración activa para validar y profundizar lo descubierto.

----

# **Ejemplo de enumeración**
Ya hemos cubierto varios conceptos relacionados con la enumeración. Comencemos a organizar todo. 
Practicaremos nuestras tácticas de enumeración en el dominio **inlanefreight.com** sin realizar análisis exhaustivos (como Nmap o análisis de vulnerabilidades, que quedan fuera de nuestro alcance). Primero, revisaremos nuestros datos de Netblocks y veremos qué podemos encontrar.


### Revisando ASN/IP & Domain Data
![[Pasted image 20251028080252.png]]

En este primer vistazo, ya hemos obtenido información interesante. BGP.he informa:

- Dirección IP: 134.209.24.248
- Servidor de correo: mail1.inlanefreight.com
- Servidores de nombres: NS1.inlanefreight.com y NS2.inlanefreight.com

Por ahora, esto es lo que nos interesa de su resultado. 
Inlanefreight no es una gran corporación, por lo que no esperábamos encontrar su propio ASN. Ahora, validemos parte de esta información con Viewdns result:

![[Pasted image 20251028080539.png]]

En la solicitud anterior, utilizamos viewdns.info para validar la dirección IP de nuestro objetivo. Ambos resultados coinciden, lo cual es una buena señal. Ahora, probemos otra ruta para validar los dos servidores de nombres en nuestros resultados:

```shell-session
Polika4RM@htb[/htb]$ nslookup ns1.inlanefreight.com

Server:		192.168.186.1
Address:	192.168.186.1#53

Non-authoritative answer:
Name:	ns1.inlanefreight.com
Address: 178.128.39.165

nslookup ns2.inlanefreight.com
Server:		192.168.86.1
Address:	192.168.86.1#53

Non-authoritative answer:
Name:	ns2.inlanefreight.com
Address: 206.189.119.186 
```

Ahora tenemos dos nuevas direcciones IP para añadir a nuestra lista para validación y pruebas:
- `ns1.inlanefreight.com → 178.128.39.165`
- `ns2.inlanefreight.com → 206.189.119.186`

Antes de realizar cualquier otra acción con ellas, asegúrese de que estén dentro del alcance de la prueba. Para nuestros fines, las direcciones IP reales no estarían dentro del alcance del escaneo, pero podríamos explorar pasivamente cualquier sitio web para buscar datos interesantes. Por ahora, esto es todo con la enumeración de la información del dominio del DNS. Analicemos la información disponible públicamente.

Inlanefreight es una empresa ficticia que utilizamos para este módulo, por lo que no tiene presencia real en redes sociales. Sin embargo, revisaríamos sitios como LinkedIn, Twitter, Instagram y Facebook para obtener información útil si fuera real. En su lugar, examinaremos el sitio web inlanefreight.com.

- **Búsqueda de documentos:** se realizó una búsqueda con `filetype:pdf inurl:inlanefreight.com` y apareció un PDF titulado _corporate goals and strategy_. Se recomienda descargar y guardar inmediatamente cualquier archivo, captura o salida de herramientas para registro y análisis posterior.
    
- **Búsqueda de correos:** con la dork `intext:"@inlanefreight.com" inurl:inlanefreight.com` se localizó la página de contacto, que contiene varias direcciones de personal. Esto permite conocer el patrón de correos (por ejemplo `first.last`) y la existencia de empleados activos en distintas oficinas.
    
- **Uso de la información de contacto:** los correos y nombres (p. ej. Emma Williams, John Smith, David Jones) pueden servir para tareas posteriores como generación de listas de usuarios, phishing o pruebas de credenciales (siempre dentro del alcance permitido).
    
- **Recolección de nombres de usuario:** se puede usar una herramienta (p. ej. `linkedin2username`) para raspar LinkedIn y generar variaciones de nombres de usuario (flast, first.last, f.last, etc.) para añadir a la lista de objetivos.
    
- **Búsqueda de credenciales:** herramientas como DeHashed permiten buscar contraseñas o hashes filtrados por dominio; el ejemplo muestra resultados ficticios que ilustran cómo se pueden obtener credenciales antiguas para crear listas de prueba. Se advierte revisar y entender los scripts/API antes de usarlos.
    
- **Principios operativos:** mantener siempre el alcance y las reglas del compromiso; guardar evidencia; usar fuentes externas (Google, LinkedIn, DeHashed) para construir wordlists y, si corresponde al alcance, realizar password spraying u otras pruebas que permitan obtener credenciales. Con credenciales se puede avanzar a la enumeración interna del dominio `INLANEFREIGHT.LOCAL`.
    
- **Siguiente paso:** pasar a la enumeración interna (pasiva/activa) del dominio interno una vez se disponga de credenciales válidas y dentro del alcance del test.



---
**1. While looking at inlanefreights public records; A flag can be seen. Find the flag and submit it. ( format == HTB{\*\*\*\*\*\*} )**


```
┌──(polika4r㉿kali)-[~]
└─$ dig inlanefreight.com ANY

; <<>> DiG 9.20.11-4+b1-Debian <<>> inlanefreight.com ANY
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24476
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1fe8132679aea5e70100000069006ec71731a1c3861dcf7f (good)
;; QUESTION SECTION:
;inlanefreight.com.             IN      ANY

;; ANSWER SECTION:
inlanefreight.com.      300     IN      AAAA    2a03:b0c0:1:e0::32c:b001
inlanefreight.com.      300     IN      TXT     "HTB{5Fz6UPNUFFzqjdg0AzXyxCjMZ}"
inlanefreight.com.      300     IN      MX      10 mail1.inlanefreight.com.
inlanefreight.com.      900     IN      SOA     ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
inlanefreight.com.      300     IN      A       134.209.24.248
inlanefreight.com.      60      IN      NS      ns1.inlanefreight.com.
inlanefreight.com.      60      IN      NS      ns2.inlanefreight.com.
inlanefreight.com.      300     IN      SPF     "v=spf1 include:_spf.google.com include:mail1.inlanefreight.com include:google.com ~all"

;; Query time: 0 msec
;; SERVER: 100.90.1.1#53(100.90.1.1) (TCP)
;; WHEN: Tue Oct 28 08:20:49 CET 2025
;; MSG SIZE  rcvd: 402
```

Respuesta: HTB{5Fz6UPNUFFzqjdg0AzXyxCjMZ}
