### ¿Qué es un servidor DNS?
El DNS funciona como el GPS de internet. Traduce nombres fáciles de recordar (ej. www.example.com) en direcciones numéricas (IP, ej. 192.0.2.1) que las computadoras entienden.

Sin DNS, tendríamos que memorizar las direcciones IP de cada sitio web, lo cual sería muy complicado.
Cuando escribes un dominio en el navegador, DNS busca y entrega la IP correcta, guiando tu conexión hasta el servidor adecuado.

En pocas palabras: DNS = traductor y mapa de internet.


### Como funciona:

1. **Consulta inicial (tu computadora pregunta)** → Verifica primero su memoria (caché). Si no lo tiene, pregunta al **DNS resolver** (normalmente de tu proveedor de internet).
2. **Resolver busca (recursivo)** → Si tampoco lo tiene, inicia el recorrido por la jerarquía DNS.
3. **Servidor raíz (Root)** → No sabe la IP exacta, pero indica a qué **TLD server** (ej. .com, .org) acudir.
4. **Servidor TLD** → Señala al **servidor autoritativo** que conoce el dominio específico.
5. **Servidor autoritativo** → Devuelve la **IP correcta** del dominio solicitado.
6. **Resolver responde** → Envía la IP a tu computadora y la guarda temporalmente en caché.
7. **Conexión establecida** → Tu computadora usa esa IP para conectarse al sitio web.

### Archivo /etc/hosts
**Dónde está:**
- Windows → `C:\Windows\System32\drivers\etc\hosts`
- Linux/MacOS → `/etc/hosts`

**Formato:**

```
127.0.0.1   localhost
192.168.1.10 devserver.local
```


El archivo _hosts_ es un **atajo manual para resolver nombres**, útil en desarrollo, pruebas y seguridad.


### Conceptos básicos
- **Zona (Zone):** Parte del espacio de nombres DNS administrada por una entidad.  
  Ejemplo: `example.com` y sus subdominios.
- **Archivo de zona (Zone File):** Archivo de texto que contiene los **registros DNS** de una zona (IP, servidores, correo, etc.).

##### Ejemplo simplificado de Zone File (example.com)
```zone
$TTL 3600
@   IN SOA   ns1.example.com. admin.example.com. (
            2024060401 ; Serial
            3600       ; Refresh
            900        ; Retry
            604800     ; Expire
            86400 )    ; Minimum TTL

@   IN NS    ns1.example.com.
@   IN NS    ns2.example.com.
@   IN MX 10 mail.example.com.
www IN A     192.0.2.1
mail IN A    198.51.100.1
ftp  IN CNAME www.example.com.

```

##### Infraestructura DNS:
| Concepto DNS             | Descripción                                                                 | Ejemplo / Nota                                      |
|---------------------------|-----------------------------------------------------------------------------|----------------------------------------------------|
| Nombre de Dominio         | Etiqueta legible para humanos que identifica un sitio web o recurso en internet | www.ejemplo.com                                     |
| Dirección IP              | Identificador numérico único de un dispositivo conectado a internet        | 192.0.2.1                                         |
| Resolutor DNS             | Servidor que traduce nombres de dominio en direcciones IP                  | Servidor DNS de tu ISP o públicos como Google DNS (8.8.8.8) |
| Servidor Raíz (Root)      | Servidores principales que dirigen las solicitudes DNS a los TLD correctos | Existen 13 servidores raíz globales, A-M: a.root-servers.net |
| Servidor TLD              | Servidores responsables de dominios de nivel superior (.com, .org, etc.)   | Verisign para .com, PIR para .org                 |
| Servidor Autoritativo     | Servidor que tiene la información final de la IP de un dominio            | Gestionado por proveedores de hosting o registradores |
| Tipos de Registros DNS    | Diferentes tipos de información que guarda el DNS sobre un dominio         | A, AAAA, CNAME, MX, NS, TXT, etc.               |

# Tipos de Registros DNS

Los registros DNS almacenan información sobre los dominios, cada uno con un propósito específico. La etiqueta "IN" indica que el registro aplica a protocolos de Internet.

| Tipo  | Nombre Completo          | Descripción                                                        | Ejemplo de Zona DNS                                      |
|-------|-------------------------|-------------------------------------------------------------------|---------------------------------------------------------|
| A     | Registro de Dirección    | Asocia un nombre de host con su dirección IPv4                     | www.ejemplo.com. IN A 192.0.2.1                        |
| AAAA  | Registro de Dirección IPv6 | Asocia un nombre de host con su dirección IPv6                   | www.ejemplo.com. IN AAAA 2001:db8:85a3::8a2e:370:7334 |
| CNAME | Registro Canónico        | Crea un alias para un nombre de host que apunta a otro hostname  | blog.ejemplo.com. IN CNAME webserver.ejemplo.net       |
| MX    | Registro de Intercambio de Correo | Especifica los servidores de correo del dominio           | ejemplo.com. IN MX 10 mail.ejemplo.com                |
| NS    | Registro de Servidor de Nombres | Indica qué servidor autoritativo gestiona la zona             | ejemplo.com. IN NS ns1.ejemplo.com                    |
| TXT   | Registro de Texto         | Almacena información de texto, usado para verificación o políticas de seguridad | ejemplo.com. IN TXT "v=spf1 mx -all"                 |
| SOA   | Registro de Autoridad    | Información administrativa de la zona (servidor principal, email responsable, etc.) | ejemplo.com. IN SOA ns1.ejemplo.com admin.ejemplo.com ... |
| SRV   | Registro de Servicio     | Define el hostname y puerto de un servicio específico            | _sip._udp.ejemplo.com. IN SRV 10 5 5060 sipserver.ejemplo.com |
| PTR   | Registro de Puntero      | Permite búsquedas inversas, mapeando una IP a un hostname       | 1.2.0.192.in-addr.arpa. IN PTR www.ejemplo.com        |
# Importancia del DNS en Reconocimiento Web

El DNS no solo traduce nombres de dominio a direcciones IP, sino que también es una fuente clave de información para pruebas de penetración y análisis de seguridad.

## Usos principales:

- **Descubrir activos**
  - Los registros DNS pueden revelar subdominios, servidores de correo y servidores de nombres.
  - Ejemplo: Un registro CNAME que apunta a un servidor antiguo (`dev.ejemplo.com CNAME oldserver.ejemplo.net`) podría indicar un sistema vulnerable.

- **Mapear la infraestructura de red**
  - Analizando los registros DNS se puede construir un mapa de la infraestructura del objetivo.
  - Ejemplo: 
    - Registros NS muestran los proveedores de hosting.
    - Un registro A para `loadbalancer.ejemplo.com` indica la ubicación de un balanceador de carga.
  - Esto ayuda a entender cómo se conectan los sistemas, el flujo de tráfico y posibles puntos débiles.

- **Monitorear cambios**
  - Cambios en los registros DNS pueden revelar nuevas configuraciones o subdominios.
  - Ejemplo: 
    - La aparición de `vpn.ejemplo.com` podría indicar un nuevo punto de entrada.
    - Un registro TXT con `_1password=...` sugiere uso de 1Password, información útil para ingeniería social o phishing dirigido.