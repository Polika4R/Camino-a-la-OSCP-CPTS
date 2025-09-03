El Motor de Scripts de Nmap (NSE) permite crear scripts en Lua para interactuar con ciertos servicios. 
Estos scripts se dividen en 14 categorías:

- **auth**: Determinación de credenciales de autenticación.
- **broadcast**: Scripts usados para descubrir hosts mediante broadcast; los hosts descubiertos pueden añadirse automáticamente a otros escaneos.
- **brute**: Ejecuta scripts que intentan iniciar sesión en el servicio por fuerza bruta con credenciales.
- **default**: Scripts por defecto que se ejecutan usando la opción `-sC`.
- **discovery**: Evaluación de servicios accesibles
- **dos**: Scripts para comprobar vulnerabilidades de denegación de servicio (DoS), usados con precaución porque pueden dañar los servicios.
- **exploit**: Scripts que intentan explotar vulnerabilidades conocidas en el puerto escaneado.
- **external**: Scripts que usan servicios externos para procesamiento adicional.
- **fuzzer**: Scripts para identificar vulnerabilidades enviando diferentes campos, lo que puede llevar mucho tiempo.
- **intrusive**: Scripts intrusivos que podrían afectar negativamente al sistema objetivo.
- **malware**: Comprueba si el sistema está infectado con malware.
- **safe**: Scripts defensivos que no realizan accesos intrusivos ni destructivos.
- **version**: Extensión para la detección de versiones de servicios.
- **vuln**: Identificación de vulnerabilidades específicas.

## Scripts por defecto
Estos scripts por defecto realizan una variedad de comprobaciones básicas y útiles para obtener información sobre los servicios abiertos, versiones, configuraciones y posibles vulnerabilidades leves sin ser demasiado intrusivos:
```shell-session
Polika4RM@htb[/htb]$ sudo nmap <target> -sC
```
## Scripts por categoría
Nmap ejecutará **solo los scripts** que pertenecen a la categoría específica

```shell-session
Polika4RM@htb[/htb]$ sudo nmap <target> --script <category>
```
## Scripts concretos
Ejecuta **scripts específicos** de Nmap que tú eliges, listados separados por comas (`<script-name>,<script-name>,...`).
```shell-session
Polika4RM@htb[/htb]$ sudo nmap <target> --script <script-name>,<script-name>,...
```

Para saber los nombres de los scripts de Nmap:
ls /usr/share/nmap/scripts/

# Escaneo agresivo de nmap

```
sudo nmap 10.129.2.28 -p 80 -A
```

**`-A`**: Escaneo agresivo, que incluye:
- Detección de versión del servicio (versión exacta del software que corre en el puerto).
- Detección del sistema operativo.
- Ejecución de scripts Nmap Scripting Engine (NSE) por defecto para obtener información extra (como el CMS WordPress).
# Evaluación de vulnerabilidad

Con:
```shell-session
sudo nmap 10.129.2.28 -p 80 -sV --script vuln 
```

Me reporta las vulnerabilidades conocidas del servicio que corre en dicho puerto. 

---
**QUESTIONS**
**Target: 10.129.63.156**
**1. Use NSE and its scripts to find the flag that one of the services contain and submit it as the answer.**
En primer lugar, lanzamos un:
```
nmap -sC --min-rate 2000 --open -Pn -n -vvv 10.129.63.156 -oA target
```
Y al filtrar con grep por "HTB" no encontramos resultados de ninguna flag. 

Vemos que existen los siguientes puertos abiertos:
```
PORT      STATE SERVICE      REASON
22/tcp    open  ssh          syn-ack ttl 63
80/tcp    open  http         syn-ack ttl 63
110/tcp   open  pop3         syn-ack ttl 63
139/tcp   open  netbios-ssn  syn-ack ttl 63
143/tcp   open  imap         syn-ack ttl 63
445/tcp   open  microsoft-ds syn-ack ttl 63
31337/tcp open  Elite        syn-ack ttl 63
```

Vamos a probar scripts dedicados uno por uno:
Empezaremos haciendo todas las combinaciones de puertos con categorías de script:
```
sudo nmap 10.129.63.156 -p <puerto> --script <categoría>
```

Siendo las categorías:
- auth
- broadcast
- brute
- default
- discovery
- dos
- exploit
- external
- fuzzer
- intrusive
- malware
- safe
- version
- vuln

Y los puertos: 
- 22
- 80
- 110
- 139
- 143
- 445
- 31337

Encuentro algo interesante con:
```
sudo nmap 10.129.63.156 -p 80 -script vuln
```

Dándome como salida:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-09-02 12:18 CDT
Nmap scan report for 10.129.63.156
Host is up (0.0080s latency).

PORT   STATE SERVICE
80/tcp open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-enum: 
|_  /robots.txt: Robots file
```

Veo que existe robots.txt, pues, accedo a dicha página desde chrome con:
```
http://10.129.63.156/robots.txt
``` 

Viendo la flag: *HTB{873nniuc71bu6usbs1i96as6dsv26}*





