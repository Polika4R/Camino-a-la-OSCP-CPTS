El Motor de Scripts de Nmap (NSE) permite crear scripts en Lua para interactuar con ciertos servicios. 
**Estos scripts se dividen en 14 categorías:**

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