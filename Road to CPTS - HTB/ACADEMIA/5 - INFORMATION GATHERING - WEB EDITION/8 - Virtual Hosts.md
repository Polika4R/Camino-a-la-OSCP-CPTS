Una vez que el DNS dirige el tráfico al servidor correcto, la configuración del servidor web (Apache, Nginx, IIS, etc.) determina cómo se gestionan las solicitudes. Estos servidores pueden alojar múltiples sitios o aplicaciones en un mismo servidor mediante **virtual hosting**, lo que permite diferenciarlos por dominios, subdominios o sitios con contenido distinto.

El _virtual hosting_ permite que un servidor web aloje varios sitios o aplicaciones en la misma IP usando el encabezado **HTTP Host**.
- **Subdominios**: Extensiones de un dominio principal (ej: `blog.ejemplo.com`), definidos en DNS, usados para organizar secciones o servicios.
- **Virtual Hosts (VHosts)**: Configuraciones en el servidor que asignan dominios o subdominios a sitios o apps específicos, cada uno con reglas propias.

Si no existe un registro DNS, se puede acceder mediante el archivo _hosts_ local. Algunos subdominios no son públicos (no aparecen en DNS). La técnica de **VHost fuzzing** permite descubrir subdominios y VHosts ocultos probando distintos nombres de host contra una IP conocida.

Los sitios web suelen tener **subdominios no públicos**, que no aparecen en los registros DNS y solo son accesibles de forma interna o con configuraciones específicas. Para encontrarlos, se utiliza la técnica llamada **VHost fuzzing**, que consiste en probar distintos nombres de host contra una IP conocida para descubrir subdominios y VHosts ocultos.

Además, los **virtual hosts** no se limitan a subdominios: también pueden configurarse para distintos dominios completamente diferentes.  
Por ejemplo:

```apacheconf
# Example of name-based virtual host configuration in Apache
<VirtualHost *:80>
    ServerName www.example1.com
    DocumentRoot /var/www/example1
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example2.org
    DocumentRoot /var/www/example2
</VirtualHost>

<VirtualHost *:80>
    ServerName www.another-example.net
    DocumentRoot /var/www/another-example
</VirtualHost>
```

En este caso, **example1.com**, **example2.org** y **another-example.net** son dominios distintos alojados en el mismo servidor.  
El servidor web utiliza el **encabezado HTTP _Host_** para identificar a qué dominio pertenece la solicitud y entregar el contenido correcto según el nombre de dominio solicitado.


## Esquema Virtual Host
Un servidor web decide qué contenido entregar usando el **encabezado Host** del navegador. El proceso funciona así:

![[Pasted image 20250823131545.png]]

1. **El navegador solicita un sitio** → Cuando ingresas un dominio (ej. `www.inlanefreight.com`), el navegador envía una petición HTTP al servidor asociado con la IP del dominio.
    
2. **El encabezado Host indica el dominio** → La petición incluye el nombre de dominio en el encabezado _Host_, indicando al servidor qué sitio se está solicitando.
    
3. **El servidor consulta su configuración de Virtual Hosts** → El servidor analiza el encabezado _Host_ y busca la configuración correspondiente en sus _VirtualHosts_.
    
4. **Entrega del contenido correcto** → Una vez identificado el _VirtualHost_ adecuado, el servidor obtiene los archivos desde el _DocumentRoot_ asociado y los envía al navegador como respuesta HTTP.
    

El **Host header funciona como un interruptor**, permitiendo que el servidor elija dinámicamente qué sitio mostrar según el dominio solicitado.

### Tipos de _Virtual Hosting_

Existen **tres tipos principales** de _virtual hosting_, cada uno con ventajas y desventajas:

1. **Name-Based Virtual Hosting (basado en nombre)**
    - Usa el encabezado **HTTP Host** para diferenciar los sitios.        
    - Es el más común, flexible y económico (no requiere múltiples IPs).
    - Fácil de configurar y soportado por la mayoría de los servidores web.
    - Limitaciones: depende de que el servidor soporte _name-based hosting_ y puede presentar problemas con ciertos protocolos como **SSL/TLS** (aunque hoy en día SNI soluciona mucho de esto).
        
2. **IP-Based Virtual Hosting (basado en IP)**
    - Cada sitio web tiene una **IP única**.
    - El servidor identifica el sitio por la IP destino de la petición, sin depender del encabezado _Host_.
    - Ventajas: compatible con cualquier protocolo y ofrece mayor aislamiento entre sitios.
    - Desventajas: necesita múltiples IPs, lo cual puede ser costoso y poco escalable.

3. **Port-Based Virtual Hosting (basado en puertos)**
    - Los sitios se diferencian por **puertos distintos** en la misma IP.
    - Ejemplo: un sitio en el puerto **80** y otro en el **8080**.
    - Útil cuando hay limitación de direcciones IP.
    - Menos común y poco práctico, ya que el usuario debe especificar el puerto en la URL (ej: `http://ejemplo.com:8080`).

## Descubrimiento de Virtual Hosts
Aunque el análisis manual de encabezados HTTP y consultas _reverse DNS_ puede funcionar, existen herramientas especializadas que automatizan el proceso y permiten descubrir _virtual hosts_ de forma más rápida y completa. Estas herramientas prueban diferentes nombres de host contra el servidor objetivo para identificar dominios o subdominios ocultos.

|**Herramienta**|**Descripción**|**Características principales**|
|---|---|---|
|**Gobuster**|Herramienta multipropósito usada normalmente para _brute-forcing_ de directorios/archivos, también útil para descubrir _virtual hosts_.|Rápida, soporta múltiples métodos HTTP, acepta _wordlists_ personalizadas.|
|**Feroxbuster**|Similar a Gobuster, pero implementada en Rust, destacada por su velocidad y flexibilidad.|Soporta recursividad, detección de comodines (_wildcards_), varios filtros.|
|**ffuf**|_Web fuzzer_ muy rápido, puede usarse para descubrir _virtual hosts_ mediante _fuzzing_ del encabezado **Host**.|Permite _wordlists_ personalizadas y múltiples opciones de filtrado.|
### Uso de Gobuster para descubrir Virtual Hosts

**Cómo funciona:**  
Gobuster envía solicitudes HTTP al servidor objetivo modificando el encabezado **Host** con distintos valores (sacados de una _wordlist_). Después analiza las respuestas para identificar cuáles corresponden a _virtual hosts_ válidos.

**Preparación necesaria:**

1. **Identificación del objetivo** → Obtener la **IP del servidor** (mediante _DNS lookup_ o reconocimiento previo).
    
2. **Wordlist de posibles hosts** → Usar listas precompiladas (ej. [SecLists](https://github.com/danielmiessler/SecLists)

```
gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```
##### Explicación de parámetros:
- **`vhost`** → Indica que Gobuster trabajará en modo descubrimiento de _virtual hosts_.
- **`-u http://<target_IP_address>`** → Especifica la URL objetivo, normalmente la **IP del servidor** al que se harán las peticiones.
- **`-w <wordlist_file>`** → Ruta a la _wordlist_ que contiene los posibles nombres de host o subdominios a probar.
- **`--append-domain`** → Muy útil: añade automáticamente el dominio principal a cada entrada de la _wordlist_.
    - Ejemplo: si tu _wordlist_ contiene `admin`, `test`, `dev` y usas `--append-domain` con `example.com`, entonces Gobuster probará:
        - `admin.example.com`
        - `test.example.com`
        - `dev.example.com`


----------

**Questions**:

**1. Brute-force vhosts on the target system. What is the full subdomain that is prefixed with "web"? Answer using the full domain, e.g. "x.inlanefreight.htb"**

Añado al /etc/hosts la siguiente línea:

```
94.237.48.12 inlanefreight.htb     #(sin añadir los puertos!!)
```

Hago fuzzing en dicha dirección

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-exsw0f0ccf]─[~]
└──╼ [★]$ wfuzz --hh=116 -c -t 200 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.inlanefreight.htb' http://94.237.48.12:54545/
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://94.237.48.12:54545/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000018:   200        1 L      4 W        98 Ch       "blog"                   
000000023:   200        1 L      4 W        100 Ch      "forum"                  
000000024:   200        1 L      4 W        100 Ch      "admin"                  
000000034:   200        1 L      4 W        104 Ch      "support"                
000006389:   200        1 L      4 W        96 Ch       "vm5"                    
000015106:   200        1 L      4 W        102 Ch      "browse"                 
000016567:   200        1 L      4 W        106 Ch      "web17611"                                                                                                
Total time: 45.78917
Processed Requests: 114441
Filtered Requests: 114434
Requests/sec.: 2499.302

```

Respuesta: web17611.inlanefreight.htb

**2. Brute-force vhosts on the target system. What is the full subdomain that is prefixed with "vm"? Answer using the full domain, e.g. "x.inlanefreight.htb"**

Sobre la misma captura del ejercicio 1:
Respuesta: vm5.inlanefreight.htb

**3. Brute-force vhosts on the target system. What is the full subdomain that is prefixed with "br"? Answer using the full domain, e.g. "x.inlanefreight.htb":

Sobre la misma captura del ejercicio 1:
Respuesta: browse.inlanefreight.htb

**4. Brute-force vhosts on the target system. What is the full subdomain that is prefixed with "a"? Answer using the full domain, e.g. "x.inlanefreight.htb"**

Sobre la misma captura del ejercicio 1:
Respuesta: admin.inlanefreight.htb

**5. Brute-force vhosts on the target system. What is the full subdomain that is prefixed with "su"? Answer using the full domain, e.g. "x.inlanefreight.htb"**

Sobre la misma captura del ejercicio 1:
Respuesta: support.inlanefreight.htb


