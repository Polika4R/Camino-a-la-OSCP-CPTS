El fingerprinting consiste en identificar las tecnologías y software que usa un sitio web o aplicación. Permite detectar vulnerabilidades, configuraciones incorrectas o sistemas desactualizados, facilitando ataques dirigidos, priorización de objetivos valiosos y la creación de un perfil completo de la infraestructura del objetivo.**Técnicas de Fingerprinting**

Existen varias técnicas para identificar tecnologías y servidores web:

- **Banner Grabbing:** Consiste en analizar los banners que muestran los servidores y otros servicios, los cuales suelen revelar el software del servidor, versiones y otros detalles.
    
- **Análisis de Cabeceras HTTP:** Las cabeceras enviadas en cada petición y respuesta HTTP contienen mucha información. La cabecera `Server` suele indicar el software del servidor, y `X-Powered-By` puede mostrar tecnologías adicionales como lenguajes de scripting o frameworks.
    
- **Sondeo de Respuestas Específicas:** Enviar peticiones diseñadas especialmente al objetivo puede generar respuestas únicas que delatan tecnologías o versiones concretas, como ciertos mensajes de error o comportamientos característicos.
    
- **Análisis del Contenido de la Página:** La estructura, scripts y otros elementos de la página pueden ofrecer pistas sobre las tecnologías usadas. Por ejemplo, un encabezado de copyright puede indicar software específico.

Existen diversas herramientas que automatizan el fingerprinting, combinando técnicas para identificar servidores web, sistemas operativos, CMS y otras tecnologías:

|Herramienta|Descripción|Características|
|---|---|---|
|**Wappalyzer**|Extensión de navegador y servicio online para identificar tecnologías de sitios web.|Detecta CMS, frameworks, herramientas de análisis y más.|
|**BuiltWith**|Perfilador de tecnologías web que ofrece informes detallados del stack tecnológico de un sitio.|Planes gratuitos y de pago con distintos niveles de detalle.|
|**WhatWeb**|Herramienta de línea de comandos para fingerprinting web.|Usa una amplia base de firmas para identificar tecnologías web.|
|**Nmap**|Escáner de red versátil para diversas tareas de reconocimiento, incluyendo fingerprinting de servicios y SO.|Permite usar scripts (NSE) para fingerprinting más especializado.|
|**Netcraft**|Ofrece servicios de seguridad web, incluyendo fingerprinting y reportes de seguridad.|Genera informes detallados sobre tecnologías, proveedor de hosting y postura de seguridad.|
|**wafw00f**|Herramienta de línea de comandos para identificar firewalls de aplicaciones web (WAF).|Determina si hay un WAF presente y su tipo/configuración.|

### Fingerprinting en inlanefreight.com

- *TÉCINA 1*: Búsqueda por cabezera web:
```
curl -I inlanefreight.com
```

Como salida obtendremos: 

```shell-session
Polika4RM@htb[/htb]$ curl -I inlanefreight.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:07:44 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: https://inlanefreight.com/
Content-Type: text/html; charset=iso-8859-1
```

Vemos que inlanefreight.com está corriendo Apache/2.4.41, bajo la versión de Ubuntu. La salida anterior también nos muestra la página https\://inlanefreight.com/.

Volviendo a hacer una llamada a la nueva página web, obtenemos: 

```shell-session
Polika4RM@htb[/htb]$ curl -I https://inlanefreight.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:12:12 GMT
Server: Apache/2.4.41 (Ubuntu)
X-Redirect-By: WordPress
Location: https://www.inlanefreight.com/
Content-Type: text/html; charset=UTF-8
```

Ahora vemos que es un WordPress quien está haciendo esta redirección.

## WEB APPLICATION FIREWALLS (WAFs)
Son soluciones de seguridad diseñadas para proteger las aplicaciones web de ataques. Antes de seguir con el fingerprinting, es crucial determinar si inlanefreight.com utiliza una WAF.  Para detectar la presencia de un WAF, utilizaremos la herramienta **waf00f**:
```shell-session
Polika4RM@htb[/htb]$ wafw00f inlanefreight.com
```

Como salida, se obtiene: 
```shell-session
Polika4RM@htb[/htb]$ wafw00f inlanefreight.com

                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.2.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://inlanefreight.com
[+] The site https://inlanefreight.com is behind Wordfence (Defiant) WAF.
[~] Number of requests: 2
```

El escaneo wafw00f muestra que el escaneo a inlanefreight.com es un sitio web protegido por *Wordfence Web Application Firewall*. Esta tecnología puede evitar ataques.

## Nikto
Nikto es una herramienta open-source que escanea servidores web. 
Además de su función principal como herramienta de evaluación de vulnerabilidades, sus capacidades de identificación de vulnerabilidades proporcionan información sobre la tecnología utilizada en un sitio web.

Se instala con:

```shell-session
Polika4RM@htb[/htb]$ sudo apt update && sudo apt install -y perl
Polika4RM@htb[/htb]$ git clone https://github.com/sullo/nikto
Polika4RM@htb[/htb]$ cd nikto/program
Polika4RM@htb[/htb]$ chmod +x ./nikto.pl
```

Para escanear *inlanefreight.com* utilizaremos nikto con el parámetro -Tuning b:

```shell-session
Polika4RM@htb[/htb]$ nikto -h inlanefreight.com -Tuning b
```

La opción `-Tuning` permite seleccionar qué tipo de pruebas realizar.

- La letra `b` se refiere específicamente a los **módulos de fingerprinting** o identificación de software. Esto significa que Nikto solo intentará identificar las tecnologías y software que corre el servidor, como servidores web, versiones de aplicaciones, frameworks o CMS, sin ejecutar otros tests más invasivos.

```shell-session
Polika4RM@htb[/htb]$ nikto -h inlanefreight.com -Tuning b

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Multiple IPs found: 134.209.24.248, 2a03:b0c0:1:e0::32c:b001
+ Target IP:          134.209.24.248
+ Target Hostname:    www.inlanefreight.com
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /CN=inlanefreight.com
                   Altnames: inlanefreight.com, www.inlanefreight.com
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Let's Encrypt/CN=R3
+ Start Time:         2024-05-31 13:35:54 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: Link header found with value: ARRAY(0x558e78790248). See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link
+ /: The site uses TLS and the Strict-Transport-Security HTTP header is not defined. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /index.php?: Uncommon header 'x-redirect-by' found, with contents: WordPress.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /: The Content-Encoding header is set to "deflate" which may mean that the server is vulnerable to the BREACH attack. See: http://breachattack.com/
+ Apache/2.4.41 appears to be outdated (current is at least 2.4.59). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /license.txt: License file found may identify site software.
+ /: A Wordpress installation was found.
+ /wp-login.php?action=register: Cookie wordpress_test_cookie created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /wp-login.php:X-Frame-Options header is deprecated and has been replaced with the Content-Security-Policy HTTP header with the frame-ancestors directive instead. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /wp-login.php: Wordpress login found.
+ 1316 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2024-05-31 13:47:27 (GMT0) (693 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

- **IPs:** Se resolvió en IPv4 (134.209.24.248) e IPv6 (2a03:b0c0:1:e0::32c:b001).
- **Tecnología del servidor:** Apache/2.4.41 en Ubuntu.
- **Presencia de WordPress:** Se detectó una instalación de WordPress, incluyendo la página de login (/wp-login.php), lo que indica posible riesgo de exploits específicos de WordPress.
- **Divulgación de información:** El archivo `license.txt` podría revelar detalles adicionales sobre los componentes de software.    
- **Cabeceras HTTP:** Se encontraron cabeceras no estándar o inseguras, como la ausencia de `Strict-Transport-Security` y un `x-redirect-by` potencialmente inseguro.

-----------

**QUESTIONS**

**Answer the question(s) below to complete this Section and earn cubes!
Target(s): 10.129.42.195 (ACADEMY-ATCKAPPS-APP01)
Life Left: 7 minute(s)
vHosts needed for these questions:
- `app.inlanefreight.local`
- `dev.inlanefreight.local`**

**1. Determine the Apache version running on app.inlanefreight.local on the target system. (Format: 0.0.0)**

Lanzo un:
```
nikto -h app.inlanefreight.local -Tuning b
```

Y me devuelve:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-yedztzq61z]─[~/nikto/program]
└──╼ [★]$ nikto -h app.inlanefreight.local -Tuning b
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.129.42.195
+ Target Hostname:    app.inlanefreight.local
+ Target Port:        80
+ Start Time:         2025-08-23 08:55:59 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
```

**2. Which CMS is used on app.inlanefreight.local on the target system? Respond with the name only, e.g., WordPress.**

Ejecuto un *whatweb <sitio_web>* y obtengo de salida: 
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-yedztzq61z]─[~/nikto/program]
└──╼ [★]$ whatweb app.inlanefreight.local
http://app.inlanefreight.local [200 OK] Apache[2.4.41], Bootstrap, Cookies[72af8f2b24261272e581a49f5c56de40], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], HttpOnly[72af8f2b24261272e581a49f5c56de40], IP[10.129.42.195], JQuery, MetaGenerator[Joomla! - Open Source Content Management], OpenSearch[http://app.inlanefreight.local/index.php/component/search/?layout=blog&amp;id=9&amp;Itemid=101&amp;format=opensearch], Script, Title[Home], UncommonHeaders[permissions-policy]
```

Veo como está corriendo un Joomla.

**3. On which operating system is the dev.inlanefreight.local webserver running in the target system? Respond with the name only, e.g., Debian.**

Ejecuto un *whatweb <sitio_web>* y obtengo de salida: 
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-yedztzq61z]─[~/nikto/program]
└──╼ [★]$ whatweb dev.inlanefreight.local
http://dev.inlanefreight.local [200 OK] Apache[2.4.41], Bootstrap, Cookies[02a93f6429c54209e06c64b77be2180d], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], HttpOnly[02a93f6429c54209e06c64b77be2180d], IP[10.129.42.195], JQuery, MetaGenerator[Joomla! - Open Source Content Management], OpenSearch[http://dev.inlanefreight.local/index.php/component/search/?layout=blog&amp;id=9&amp;Itemid=101&amp;format=opensearch], Script, Title[Home]
```
Corriendo bajo un Ubuntu.




