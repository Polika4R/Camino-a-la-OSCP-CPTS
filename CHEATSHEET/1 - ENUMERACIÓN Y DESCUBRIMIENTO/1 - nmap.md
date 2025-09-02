---

# **Descubrimiento de Hosts y Puertos con Nmap**

---
##  1. Descubrimiento de Hosts Activos

| Comando                                                                | Descripción                                                                                                                         | Ejemplo                                                      |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `sudo nmap <IP>/24 -sn -oA tnet`                                       | Escanea toda la red `/24` buscando **hosts activos**, sin escanear puertos (`-sn`). Guarda la salida en `.nmap`, `.gnmap` y `.xml`. | `sudo nmap 10.129.2.0/24 -sn -oA tnet`                       |
| `sudo nmap -sn -iL hosts.lst -oA tnet`                                 | Escanea los **hosts activos** de una lista (`-iL`).                                                                                 | `sudo nmap -sn -iL hosts.lst -oA tnet`                       |
| `sudo nmap -sn <ip1> <ip2> <ip3> -oA tnet`                             | Escanea múltiples **IPs individuales**.                                                                                             | `sudo nmap -sn 10.129.2.18 10.129.2.19 10.129.2.20 -oA tnet` |
| `sudo nmap -sn 10.129.2.18-20 -oA tnet`                                | Escanea un **rango de IPs**.                                                                                                        |                                                              |
| `sudo nmap 10.129.2.18 -sn -oA host`                                   | Escanea un **único host** para ver si está activo.                                                                                  |                                                              |
| `sudo nmap 1.2.3.4 -sn -PE --packet-trace -oA host`                    | Fuerza ping con ICMP (`-PE`) y muestra detalle de los paquetes (`--packet-trace`). Útil para **analizar filtrados/firewalls**.      | Usa también ARP por defecto en LAN.                          |
| `sudo nmap 1.2.3.4 -sn -PE --packet-trace --disable-arp-ping -oA host` | Igual que el anterior, pero desactiva ARP ping. Solo usa ICMP puro.                                                                 | Útil para depurar ICMP puro en redes que bloquean ARP.       |

---
##  2. Descubrimiento de Puertos TCP

| Comando                                                                              | Descripción                                                                                                                                              | Ejemplo                                                       |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `sudo nmap 1.2.3.4 --top-ports=10`                                                   | Escanea los **10 puertos TCP más comunes**.                                                                                                              |                                                               |
| `sudo nmap 1.2.3.4 -p 22,25,80,139,445`                                              | Escanea **puertos específicos**.                                                                                                                         |                                                               |
| `sudo nmap 1.2.3.4 -p-`                                                              | Escanea **todos los puertos TCP (1-65535)**.                                                                                                             |                                                               |
| `sudo nmap 1.2.3.4 -p 22-445`                                                        | Escanea un **rango de puertos TCP**.                                                                                                                     |                                                               |
| `sudo nmap 1.2.3.4 -p <puerto> -Pn -n --disable-arp-ping --packet-trace`             | Escanea 1 puerto sin ping (-Pn) ni resolución DNS (-n). Muestra paquetes y **estado real** del puerto. Envia SYN y analiza respuesta (**RA = cerrado**). | Útil para análisis de filtrados/firewalls.                    |
| `sudo nmap 10.129.2.28 -p 443 -sT -Pn -n --disable-arp-ping --packet-trace --reason` | Escaneo completo con TCP Connect (`-sT`). Útil cuando SYN scan no es posible.                                                                            | Usa handshake completo. Muestra motivo del estado del puerto. |

---

##  3. Descubrimiento de Puertos UDP

| Comando                 | Descripción                                           | Ejemplo                                       |
| ----------------------- | ----------------------------------------------------- | --------------------------------------------- |
| `sudo nmap 1.2.3.4 -sU` | Escaneo rápido (`-F`) de **puertos UDP más comunes**. | Detecta servicios UDP activos más habituales. |

---

##  4. Escaneo con Detección de Servicio / Versión

| Comando                                                                               | Descripción                                                                                                                        | Ejemplo                                                          |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `sudo nmap 1.2.3.4 -p <puerto> -sV -Pn -n --disable-arp-ping --packet-trace --reason` | Escanea puerto con detección de versión (`-sV`). Muestra paquetes enviados, razones y **servicio corriendo**.                      | Útil para saber exactamente qué servicio está detrás del puerto. |
| `nc -nv 1.2.3.4 <port_a_atacar>`                                                      | Para detectar contenido que a veces nc obvia en los banners                                                                        |                                                                  |
| `nc -nv -p <num_port_origen> 1.2.3.4 <num_port_a_atacar>`                             | Para detectar contenido que a veces nc obvia en los banners haciéndome pasar por un puerto confiable para firewalls (53, 80, 443). | nc -nv -p 53 1.2.3.4 192.168.1.55                                |
| `ncat -nv --source-port <num_port_origen> 1.2.3.4 <puerto_a_atacar>`                  | Alternativa detectar contenido que a veces nc obvia en los banners haciéndome pasar por un puerto confiable para firewalls.        | ncat -nv --source-port 53 192.168.1.55                           |


---

## 5. SCRIPTING ENGINE

| Comando                                           | Descripción                                 | Ejemplo / Nota                                                       |
| ------------------------------------------------- | ------------------------------------------- | -------------------------------------------------------------------- |
| `sudo nmap <target> -sC`                          | Ejecuta los scripts por defecto             | Info básica, versión de servicios, posibles vulnerabilidades ligeras |
| `sudo nmap <target> --script <category>`          | Ejecuta scripts de una categoría específica | `--script vuln` para comprobar vulnerabilidades                      |
| `sudo nmap <target> --script <script1>,<script2>` | Ejecuta scripts específicos por nombre      | Ver nombres en `/usr/share/nmap/scripts/`                            |
| `ls /usr/share/nmap/scripts/`                     | Muestra todos los scripts disponibles       | Para conocer los nombres exactos                                     |


**Categorías:**

|**Categoría**|**Función principal**|
|---|---|
|`auth`|Descubrir credenciales de autenticación|
|`broadcast`|Descubrir hosts vía broadcast|
|`brute`|Ataques de fuerza bruta a servicios de login|
|`default`|Scripts básicos por defecto (`-sC`)|
|`discovery`|Descubrimiento de servicios|
|`dos`|Verificación de vulnerabilidades de denegación de servicio|
|`exploit`|Explotación de vulnerabilidades conocidas|
|`external`|Usa servicios externos para análisis adicional|
|`fuzzer`|Envío de campos modificados para detectar fallos|
|`intrusive`|Scripts potencialmente perjudiciales para el sistema objetivo|
|`malware`|Detecta signos de infección por malware|
|`safe`|Scripts seguros (no intrusivos ni destructivos)|
|`version`|Mejora la detección de versiones de servicios (`-sV`)|
|`vuln`|Busca vulnerabilidades conocidas en el puerto objetivo|

##  6. Parámetros de Nmap importantes

| Opción                                           | Descripción                                                                                                                                                                                                                                  |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-sn`                                            | Ping scan: detecta si un host está activo (no escanea puertos).                                                                                                                                                                              |
| `-p <puertos>`                                   | Especifica puertos a escanear (`-p 22,80`, `-p 1-1000`, `-p-` todos).                                                                                                                                                                        |
| `--top-ports <n>`                                | Escanea los `n` puertos más comunes.                                                                                                                                                                                                         |
| `-Pn`                                            | No hace ping; **asume que el host está activo**.                                                                                                                                                                                             |
| `-n`                                             | No resuelve nombres DNS (solo usa IPs).                                                                                                                                                                                                      |
| `-oA <nombre>`                                   | Guarda salida en `.nmap`, `.gnmap`, `.xml`.                                                                                                                                                                                                  |
| `--packet-trace`                                 | Muestra todos los paquetes enviados y recibidos.                                                                                                                                                                                             |
| `--reason`                                       | Explica **por qué** un puerto está en cierto estado (abierto/cerrado).                                                                                                                                                                       |
| `--disable-arp-ping`                             | Desactiva ARP ping (útil si se quiere evitar descubrimiento por MAC).                                                                                                                                                                        |
| `-sS`                                            | Escaneo TCP SYN (stealth scan / sigiloso); no completa handshake (requiere root). Lo hace por defecto si eres root. De lo contrario, hará un escaneo -sT.                                                                                    |
| `-sT`                                            | Escaneo TCP Connect (completo); útil si no tienes permisos para `-sS` (el cual requiere ROOT). Completa el three-way handshake por completo (poco sigiloso)                                                                                  |
| `-sU`                                            | Escaneo UDP.                                                                                                                                                                                                                                 |
| `-sV`                                            | Detección de versiones de servicios.                                                                                                                                                                                                         |
| `-O`                                             | Detección de sistema operativo.                                                                                                                                                                                                              |
| `-A`                                             | Detección avanzada: SO, versión, scripts, traceroute.                                                                                                                                                                                        |
| `-F`                                             | Escaneo rápido: solo puertos más comunes.                                                                                                                                                                                                    |
| `-T<0-5>`                                        | Velocidad/agresividad del escaneo (`0`: sigiloso, `5`: muy rápido).                                                                                                                                                                          |
| `-iL <archivo.lst>`                              | Lee las IPs o rangos desde un archivo.                                                                                                                                                                                                       |
| `-v`, `-vv`, `-vvv`                              | Modo verboso; más detalles en salida.                                                                                                                                                                                                        |
| `--stats-every=5s`                               | Muestra cada 5s el % de progreso del escaneo                                                                                                                                                                                                 |
| (pulsar espacio mientras hay un escaneo de nmap) | Muestra información del progreso del proceso de búsqueda.                                                                                                                                                                                    |
| -D RND:5                                         | Genera 5 IPs atacantes aleatorias como señuelos (-D RND:5). Entre otras, está la mía, porque sino incluyese mi IP de red el servidor respondería solo a las IP randoms (no existentes), y no a mi.                                           |
| -S \<ip_falsa_atacante>                          | Similar a -D RND:5 pero en este caso yo fuerzo la IP de origen. De la misma forma, si pongo una IP falsa que yo no tenga configurada en mi máquina no tendré respuesta por parte del servidor (debido a que se enviará a una IP inexistente) |
