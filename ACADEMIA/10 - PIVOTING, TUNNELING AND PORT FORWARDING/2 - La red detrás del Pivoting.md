## Direccionamiento IP y NIC's
Todo ordenador que se comunica en una red necesita un IP.
Si no la tiene, no estará en su red. 
La IP se suele asignar normalmente mediante un servidor DHCP (un servidor que asigna la IP libre automáticamente).
Pero también pueden existir IP's estáticas. 

El NIC es la interfaz de red. Un mismo equipo puede tener 1 o más (tantas interfaces de red como redes a las que se conecte).

Con el comando "ifconfig" puedo listar la totalidad de interfaces de red disponibles en un equipo.
```shell-session
Polika4RM@htb[/htb]$ ifconfig

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 134.122.100.200  netmask 255.255.240.0  broadcast 134.122.111.255
        inet6 fe80::e973:b08d:7bdf:dc67  prefixlen 64  scopeid 0x20<link>
        ether 12:ed:13:35:68:f5  txqueuelen 1000  (Ethernet)
        RX packets 8844  bytes 803773 (784.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5698  bytes 9713896 (9.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.106.0.172  netmask 255.255.240.0  broadcast 10.106.15.255
        inet6 fe80::a5bf:1cd4:9bca:b3ae  prefixlen 64  scopeid 0x20<link>
        ether 4e:c7:60:b0:01:8d  txqueuelen 1000  (Ethernet)
        RX packets 15  bytes 1620 (1.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18  bytes 1858 (1.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 19787  bytes 10346966 (9.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 19787  bytes 10346966 (9.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.10.15.54  netmask 255.255.254.0  destination 10.10.15.54
        inet6 fe80::c85a:5717:5e3a:38de  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef:2::1034  prefixlen 64  scopeid 0x0<global>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7  bytes 336 (336.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

- El comando muestra las interfaces de red (NICs): `eth0`, `eth1`, `lo`, `tun0` y sus estadísticas.
- `tun0` es una interfaz de túnel: indica que hay una VPN activa.
- Conectarse a los servidores VPN de HTB crea siempre una `tun0` con una IP asignada; sin ese túnel no se accede a las redes del laboratorio.
- La VPN cifra el tráfico y crea un túnel desde la red pública (Internet) —a menudo atravesando NAT— hasta la red interna/privada.
- `eth0` tiene una IP pública (ej. `134.122.100.200`): esa IP es enrutada por ISPs por Internet y suele corresponder a interfaces expuestas (DMZ).
- Las demás NICs usan IPs privadas: sólo enrutables dentro de la red interna, no directamente por Internet.
- NAT se usa para traducir direcciones privadas a una pública cuando es necesario comunicarse con Internet.
- `ipconfig` (o comandos equivalentes) permite ver estas asignaciones y confirmar si la VPN/túnel está activo.

---

## ipconfig

El comando **ipconfig** en un sistema Windows muestra todos los adaptadores de red del equipo. Generalmente, sólo uno de ellos tiene direcciones IP asignadas y activas. En muchos casos, ese adaptador presenta tanto una dirección **IPv4** como una **IPv6**, lo que se conoce como una configuración _dual-stack_. Sin embargo, la mayoría de las redes empresariales siguen utilizando principalmente **IPv4**, por lo que este módulo se centra en ese tipo de direccionamiento.

Cada dirección IPv4 viene acompañada de una **máscara de subred**, la cual determina qué parte de la dirección corresponde a la red y cuál al host. Si pensamos en la IP como un número de teléfono, la máscara sería el código de área. Cuando un equipo necesita comunicarse con una dirección que no pertenece a su misma red, envía el tráfico a su **puerta de enlace predeterminada** (_default gateway_), que normalmente es la dirección IP de un router dentro de la LAN.

En el contexto del **pivoting** dentro de una evaluación de seguridad, es importante entender a qué redes puede acceder un host comprometido. Por eso, resulta fundamental documentar toda la información de direccionamiento posible —como las direcciones IP, máscaras, gateways y adaptadores presentes—, ya que esto puede revelar nuevas rutas o segmentos de red a los que se puede llegar desde el sistema analizado.

```powershell-session
PS C:\Users\htb-student> ipconfig

Windows IP Configuration

Unknown adapter NordLynx:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . : .htb
   IPv6 Address. . . . . . . . . . . : dead:beef::1a9
   IPv6 Address. . . . . . . . . . . : dead:beef::f58b:6381:c648:1fb0
   Temporary IPv6 Address. . . . . . : dead:beef::dd0b:7cda:7118:3373
   Link-local IPv6 Address . . . . . : fe80::f58b:6381:c648:1fb0%8
   IPv4 Address. . . . . . . . . . . : 10.129.221.36
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:df81%8
                                       10.129.0.1

Ethernet adapter Ethernet:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
```

---

## Routing

En redes, un **router** no es necesariamente un aparato físico dedicado: cualquier equipo que tenga una tabla de enrutamiento y reenvíe paquetes entre interfaces puede actuar como router. Esa tabla de enrutamiento contiene reglas que dicen “si el destino está en esta red, envía el paquete por esta interfaz o hacia este gateway”. En el ejemplo de Pwnbox se ve precisamente eso: varias rutas aprendidas por las interfaces directamente conectadas (`eth0`, `eth1`, `tun0`) y una ruta por defecto (`default`) que se usa cuando no existe una ruta más específica. Así, si queremos alcanzar `10.129.10.25`, el sistema consulta la tabla y decide a qué gateway e interfaz mandar el paquete para que llegue a su destino.

En entornos de pentesting y pivoting esa tabla es muy valiosa porque revela qué redes puede alcanzar un host comprometido y por cuáles saltos habría que encaminar el tráfico. Los routers “de verdad” pueden además aprender rutas por protocolos dinámicos (OSPF, BGP, etc.) o por rutas estáticas añadidas por un administrador, pero en muchos laboratorios y en Pwnbox las rutas aparecen porque las interfaces están directamente conectadas o porque se han añadido rutas específicas (por ejemplo, AutoRoute en frameworks de ataque).

```shell-session
Polika4RM@htb[/htb]$ netstat -r

Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         178.62.64.1     0.0.0.0         UG        0 0          0 eth0
10.10.10.0      10.10.14.1      255.255.254.0   UG        0 0          0 tun0
10.10.14.0      0.0.0.0         255.255.254.0   U         0 0          0 tun0
10.106.0.0      0.0.0.0         255.255.240.0   U         0 0          0 eth1
10.129.0.0      10.10.14.1      255.255.0.0     UG        0 0          0 tun0
178.62.64.0     0.0.0.0         255.255.192.0   U         0 0          0 eth0
```


---

## Protocolos y puertos
Los **protocolos** y **puertos** son la otra pieza clave: un protocolo define reglas de comunicación y a menudo se asocia con un puerto lógico (por ejemplo, HTTP con el puerto 80). Ese puerto no es algo físico, sino un identificador asignado en software a la aplicación que escucha. Encontrar un puerto abierto en una IP significa encontrar un servicio al que podemos conectarnos; a menudo los administradores permiten ciertos puertos (p. ej. 80 o 443 para web), y esos mismos puertos pueden servir como vectores de entrada si se explotan servicios o se aprovechan canales permitidos por firewalls. También hay que recordar los **puertos de origen**: el cliente genera un puerto efímero para cada conexión y, al preparar payloads o listeners, debemos cuidar qué puertos usamos para asegurar que el retorno de la conexión funcione como esperamos.

En resumen, para pivotar eficazmente conviene: inspeccionar la tabla de enrutamiento del host comprometido (qué redes conoce y por dónde), identificar servicios accesibles por puertos permitidos, y diseñar rutas o reenvíos (locales o remotos) que permitan llevar tráfico a las subredes objetivo. Herramientas visuales (dibujar la topología en Draw.io u otra) y practicar múltiples técnicas de forwarding/port-forwarding ayudará mucho a consolidar estos conceptos.

---

**1. Reference the Using ifconfig output in the section reading. Which NIC is assigned a public IP address?**
Respuesta: eth0

**2. Reference the Routing Table on Pwnbox output shown in the section reading. If a packet is destined for a host with the IP address of 10.129.10.25, out of which NIC will the packet be forwarded?**
```shell-session
Polika4RM@htb[/htb]$ netstat -r

Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         178.62.64.1     0.0.0.0         UG        0 0          0 eth0
10.10.10.0      10.10.14.1      255.255.254.0   UG        0 0          0 tun0
10.10.14.0      0.0.0.0         255.255.254.0   U         0 0          0 tun0
10.106.0.0      0.0.0.0         255.255.240.0   U         0 0          0 eth1
10.129.0.0      10.10.14.1      255.255.0.0     UG        0 0          0 tun0
178.62.64.0     0.0.0.0         255.255.192.0   U         0 0          0 eth0
```

Respuesta: tun0

**3. Reference the Routing Table on Pwnbox output shown in the section reading. If a packet is destined for www.hackthebox.com what is the IP address of the gateway it will be sent to?**
- Primero el sistema resuelve `www.hackthebox.com` a una dirección IP (p. ej. una IP pública).
- Al enviar el paquete, el kernel consulta la **tabla de enrutamiento** para decidir por dónde enviarlo. La decisión se basa en la **coincidencia más específica** (longest-prefix match): busca una ruta cuya red abarque la IP de destino.
- En la tabla mostrada las rutas específicas son para redes `10.*.*.*` (tun0 y eth1) y para la red `178.62.64.0/18` (eth0). Las redes `10.*` son privadas; una IP pública de `www.hackthebox.com` **no** caerá en esas redes privadas.
- Si no hay ninguna ruta más específica que coincida con la IP destino, se usa la **ruta por defecto** (`Destination = default`, `Genmask = 0.0.0.0`). En la tabla la ruta por defecto tiene **Gateway = 178.62.64.1** y `Iface = eth0`. Las banderas `UG` indican que la ruta está **Up** y que es hacia un **Gateway**.
- Por tanto, como la IP de `www.hackthebox.com` no coincide con las redes 10.x ni con rutas más específicas, el paquete se envía al gateway **178.62.64.1** vía `eth0` (esa es la puerta de enlace hacia Internet en esa máquina).

Respuesta: 178.62.64.1