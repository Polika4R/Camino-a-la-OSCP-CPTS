
# SNMP (Simple Network Management Protocol)

## Función principal
Protocolo estándar para **monitoreo y gestión remota** de dispositivos de red (routers, switches, servidores, IoT...).
Se creó para supervisar dispositivos de red. Además, este protocolo también permite gestionar tareas de configuración y modificar ajustes de forma remota.

## Versiones

| Versión     | Características principales                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------- |
| **SNMPv1**  | Sin autenticación ni cifrado. Todo en texto plano. Soporta traps.                                     |
| **SNMPv2c** | Usa *community strings* (tipo contraseña). Sin cifrado.                                               |
| **SNMPv3**  | Seguridad mejorada: autenticación (usuario/contraseña) + cifrado (clave precompartida). Más complejo. |

## Community Strings

- Actúan como **contraseñas** para acceder a la información SNMP.
- En **v1/v2c** se envían en **texto plano** → pueden ser interceptadas.
- Muchas organizaciones siguen usando v2c por simplicidad, aunque es inseguro.

## Funcionamiento

- Usa **UDP 161** para operaciones normales (get/set).
- Usa **UDP 162** para **traps**: alertas enviadas del dispositivo al cliente sin solicitud.
- Permite:
  - Consultar información.
  - Cambiar configuraciones remotamente.
  - Recibir alertas (traps).

## MIB (Management Information Base)

- Archivo de texto (.mib) en formato **ASN.1** que describe los objetos SNMP del dispositivo.
- Contiene:
  - OID (Object Identifier)
  - Nombre del objeto
  - Tipo de dato
  - Permisos de acceso
  - Descripción
- Es **esencial para interoperabilidad entre fabricantes**.

## OID (Object Identifier)

- Cadena de números separados por puntos → `.1.3.6.1.2.1...`
- Representa una **ruta jerárquica** dentro del árbol MIB.
- Cuanto más larga la cadena, más específica la información.
- Pueden consultarse en el *Object Identifier Registry*.


# Configuración del demonio SNMP

Archivo de configuración típico ubicado en: `/etc/snmp/snmpd.conf`

La configuración predeterminada del demonio SNMP define las configuraciones básicas para el servicio, que incluyen las direcciones IP, los puertos, el MIB, los OID, la autenticación y las cadenas de comunidad.

```shell-session
Polika4RM@htb[/htb]$ cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'

sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly```
```

Algunas configuraciones peligrosas que el administrador puede realizar con SNMP son:

|**Ajustes**|**Descripción**|
|---|---|
|`rwuser noauth`|Proporciona acceso al árbol OID completo sin autenticación.|
|`rwcommunity <community string> <IPv4 address>`|Proporciona acceso al árbol OID completo independientemente de dónde se enviaron las solicitudes.|
|`rwcommunity6 <community string> <IPv6 address>`|Mismo acceso que `rwcommunity`con la diferencia de utilizar IPv6.|


# FOOTPRINTING

Claro, aquí tienes los apuntes bien estructurados y limpios para Obsidian, con explicaciones claras y ejemplos prácticos de uso para cada herramienta SNMP:

# Herramientas para rastrear y enumerar SNMP

## 1. snmpwalk

- Herramienta para consultar recursivamente los OID de un dispositivo SNMP.
- Permite obtener toda la información disponible que el dispositivo expone vía SNMP.
- Muy útil para explorar y entender qué datos se pueden obtener.

### Ejemplo de uso

```shell
snmpwalk -v2c -c public 10.129.14.128
````

### Salida ejemplo

```
iso.3.6.1.2.1.1.1.0 = STRING: "Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64"
iso.3.6.1.2.1.1.4.0 = STRING: "mrb3n@inlanefreight.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "htb"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
...
```

Con un proceso concreto, puedo hacer:
```
snmpwalk -v2c -c public <ip_victima> <iso_number> 
```

```
snmpwalk -v2c -c public 10.129.100.248 iso.3.6.1.2.1.25.1.7
```


---

## 2. onesixtyone

- Herramienta para descubrir y forzar nombres de _community strings_ (cadenas de comunidad).
- Útil porque los administradores pueden asignar nombres arbitrarios a estas cadenas.
- La búsqueda puede ser lenta, ya que puede haber muchas combinaciones y nombres diferentes.

### Instalación y ejemplo de uso

```shell
sudo apt install onesixtyone

onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```
### Salida ejemplo

```
Scanning 1 hosts, 3220 communities
10.129.14.128 [public] Linux htb 5.11.0-37-generic #41~20.04.2-Ubuntu SMP Fri Sep 24 09:06:38 UTC 2021 x86_64
```

---

## 3. braa

- Herramienta alternativa para escanear servicios SNMP y enumerar información específica.
    
- Se usa para consultar OID individuales con una cadena de comunidad conocida.
    
### Instalación y ejemplo de uso

```shell
sudo apt install braa

braa public@10.129.14.128:.1.3.6.*
```

### Salida ejemplo

```
10.129.14.128:20ms:.1.3.6.1.2.1.1.1.0:Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64
10.129.14.128:20ms:.1.3.6.1.2.1.1.4.0:mrb3n@inlanefreight.htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.5.0:htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.6.0:US
...
```

---

# Resumen

- **snmpwalk:** Ideal para explorar todos los datos SNMP disponibles.
    
- **onesixtyone:** Útil para descubrir _community strings_ mediante fuerza bruta.
    
- **braa:** Perfecto para enumerar información concreta con una cadena conocida.
    

---------------

**\[+1 CUBE ] Enumerate the SNMP service and obtain the email address of the admin. Submit it as the answer.

Ejecuto el comando:
```
snmpwalk -v2c -c public 10.129.42.195 | grep "@"
```
el cual muestra toda la información disponible del servicio SNMP. En concreto, filtro por "@" y veo un email disponible.

```
PS [10.10.15.48] /home/htb-ac-1876550 > snmpwalk -v2c -c public 10.129.42.195 | grep "@"
iso.3.6.1.2.1.1.4.0 = STRING: "devadmin <devadmin@inlanefreight.htb>"
```

Solución: *devadmin\@inlanefreight.htb*

**\[+1 CUBE ] What is the customized version of the SNMP server?**

Ejecutando un:
```
PS [10.10.15.48] /home/htb-ac-1876550 > snmpwalk -v2c -c public 10.129.224.247 | grep "SNMP"
```

Veo como la versión es la:
*iso.3.6.1.2.1.1.6.0 = STRING: **"InFreight SNMP v0.91"***




**\[+1 CUBE ]  Enumerate the custom script that is running on the system and submit its output as the answer.**

Haciendo un:
```
snmpwalk -v2c -c public 10.129.224.247 | grep "flag.sh"
```

El agente snmpwalk me devuelve:
```
iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 = STRING: "/usr/share/flag.sh"
```

Busca en toda la información que proporciona el agente SNMP aquella línea que contenga `"flag.sh"`, lo que indica que el agente tiene almacenada (en alguna variable SNMP) la ruta al fichero `flag.sh`.

Ahora, ejecutando un:
```
`snmpwalk -v2c -c public 10.129.100.248 iso.3.6.1.2.1.25.1.7`
```


- Igual que antes, pero ahora la consulta se limita a partir del OID (identificador) `iso.3.6.1.2.1.25.1.7`.
    
- Esto significa que en lugar de recorrer todo el árbol SNMP, solo se obtiene la información relacionada a esa rama concreta.   He recortado parte del código original, pasando de:
> 	iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 

a
>	 **iso.3.6.1.2.1.25.1.7.**

¿Por qué usar este OID?
Porque en el paso 1 viste que el fichero `flag.sh` está dentro de esa rama, por eso directamente limitas la consulta a esa parte, que es más rápida y focalizada.


Resultado:

```
snmpwalk -v2c -c public 10.129.100.248 iso.3.6.1.2.1.25.1.7`

iso.3.6.1.2.1.25.1.7.1.4.1.2.4.70.76.65.71.1 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"`

```
Podría también hacer directamente un:
snmpwalk -v2c -c public 10.129.100.248 | grep "htb"

