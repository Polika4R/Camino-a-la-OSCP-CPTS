#whois #dig #nslookup #dnsenum

# WHOIS

| Comando                | Descripción                                           | Ejemplo |
| ---------------------- | ----------------------------------------------------- | ------- |
| whois <nombre_dominio> | devuelve un conjunto de información de una página web |         |

# DNS

#### DIG - NSLOOKUP

| Comando                              | Descripción                                                                                        | Ejemplo                                                                                             |
| ------------------------------------ | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `dig domain.com`                     | `nslookup domain.com`                                                                              | Consulta por defecto (registro A).                                                                  |
| `dig domain.com A`                   | `nslookup -type=A domain.com`                                                                      | Obtiene dirección IPv4.                                                                             |
| `dig domain.com AAAA`                | `nslookup -type=AAAA domain.com`                                                                   | Obtiene dirección IPv6.                                                                             |
| `dig domain.com MX`                  | `nslookup -type=MX domain.com`                                                                     | Muestra los servidores de correo.                                                                   |
| `dig domain.com NS`                  | `nslookup -type=NS domain.com`                                                                     | Identifica los servidores de nombres autoritativos.                                                 |
| `dig domain.com TXT`                 | `nslookup -type=TXT domain.com`                                                                    | Recupera registros TXT.                                                                             |
| `dig domain.com CNAME`               | `nslookup -type=CNAME domain.com`                                                                  | Obtiene el registro de nombre canónico.                                                             |
| `dig domain.com SOA`                 | `nslookup -type=SOA domain.com`                                                                    | Recupera el registro de inicio de autoridad.                                                        |
| `dig @1.1.1.1 domain.com`            | `nslookup domain.com 1.1.1.1`                                                                      | Especifica servidor DNS concreto (ejemplo 1.1.1.1).                                                 |
| `dig +trace domain.com`              | _(No equivalente directo en nslookup)_                                                             | `nslookup` no tiene opción de traza completa. Se usa **`dig +trace`** o `whois` como alternativa.   |
| `dig -x 192.168.1.1`                 | `nslookup 192.168.1.1`                                                                             | Búsqueda inversa (IP → nombre).                                                                     |
| `dig +short domain.com`              | _(No equivalente directo)_                                                                         | `nslookup` siempre muestra salida extendida. Para algo más limpio se usan scripts o `host`.         |
| `dig +noall +answer domain.com`      | _(No equivalente directo)_                                                                         | `nslookup` no permite mostrar solo la sección de respuesta.                                         |
| `dig domain.com ANY`                 | `nslookup -type=ANY domain.com`                                                                    | Consulta todos los registros disponibles (aunque muchos servidores bloquean este tipo de consulta). |
| `dig <dominio> +short`               | Devuelve únicamente la resolución DNS de dicho dominio (IP)                                        |                                                                                                     |
| `dig axfr @<servidor_dns> <dominio>` | Realiza transferencia de zona DNS para obtener todos los registros del dominio.                    | dig axfr @nsztm1.digi.ninja zonetransfer.me                                                         |
| `nslookup <dominio>`                 | Similar a dig \<dominio> +short. Consulta DNS que traduce dominio en dirección IP correspondiente. |                                                                                                     |
| `nslookup <dirección_ip>`            | Devuelve el dominio asociado a tal dirección IP                                                    |                                                                                                     |

# DNSENUM
| Comando                                                   | Descripción                                                                     | Ejemplo                                                                                                              |
| --------------------------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `dnsenum --enum <dominio> -f /ruta/al/diccionario.txt -r` | Herramienta para enumerar subdominios, registros DNS y servidores relacionados. | dnsenum --enum inlanefreight.com -f /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r |
|                                                           |                                                                                 |                                                                                                                      |

# Descubrimiento de Virtual Hosts

| Comando                                                                               | Descripción                                                      | Ejemplo                                                                                                                                                                  |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `gobuster vhost -u http://<target_IP_address> -w <wordlist_file.txt> --append-domain` | Busca virtual hosts ocultos en un dominio mediante fuerza bruta. | Busca virtual hosts ocultos en un dominio mediante fuerza bruta.                                                                                                         |
| `wfuzz -c -t 200 -w <wordlist_file.txt> -H 'Host: FUZZ.\<dominio>' \<URL>`            |                                                                  | wfuzz --hh=116 -c -t 200 -w /usr/share/wordlists/ seclists/Discovery/DNS/ subdomains-top1million-110000.txt -H 'Host: FUZZ.inlanefreight.htb' http://94.237.48.12:54545/ |
|                                                                                       |                                                                  |                                                                                                                                                                          |
# Fingerprinting
| Comando                                                                                                                                                                   | Descripción                                                                  | Ejemplo                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------ |
| `curl -I <dominio>`                                                                                                                                                       | Muestra únicamente los encabezados HTTP de la respuesta del servidor.        | curl -I inlanefreight.com            |
| >sudo apt update && sudo apt install -y perl<br>>git clone https\://github.com/sullo/nikto<br>>cd nikto/program<br>>chmod +x ./nikto.pl<br>>nikto -h \<dominio> -Tuning b | Escáner web Nikto para detectar vulnerabilidades comunes en servidores HTTP. | nikto -h inlanefreight.com -Tuning b |
| `>whatweb <url>`                                                                                                                                                          | Identifica tecnologías, frameworks y cabeceras usadas en un sitio web.       | whatweb inlanefreight.com            |
|                                                                                                                                                                           |                                                                              |                                      |
