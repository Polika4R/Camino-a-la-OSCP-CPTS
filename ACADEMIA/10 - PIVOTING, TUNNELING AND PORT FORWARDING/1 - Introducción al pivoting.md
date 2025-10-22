## Lateral Movement

Movimiento lateral es la técnica para extender el control dentro de la misma red: buscar hosts, servicios y recursos adicionales a los que podamos acceder con las credenciales o privilegios ya obtenidos. Suele usarse para encontrar cuentas con más privilegios, robar hashes/credenciales, o llegar a servidores con datos sensibles.
Ejemplo práctico: obtenemos credenciales de administrador local en una máquina, escaneamos la subred, encontramos otros equipos y usamos esas mismas credenciales para autenticarnos en otro host y así avanzar hacia recursos de dominio.

---
## Pivoting

Pivoting es usar uno o varios hosts comprometidos como puentes para cruzar límites de red (segmentación física o lógica) y alcanzar redes o sistemas inaccesibles desde nuestro punto inicial. Es más dirigido: el objetivo es llegar a segmentos concretos (OT, DMZ, VLANs separadas).
Ejemplo práctico: comprometes una estación de ingeniería dual-homed (2 NICs), la cual tiene visibilidad tanto de la red empresarial como de la red operacional; a través de esa estación puedes atacar activos en la red operacional que antes no podías alcanzar.

Indicadores útiles: interfaces de red múltiples, enrutamiento entre subredes, reglas NAT/ACL que permitan tránsito, servicios que actúan como pasarela (jump hosts, bastion).

---
## Tunneling

Tunneling consiste en encapsular tráfico dentro de otro protocolo para transportarlo por canales que parezcan legítimos o menos vigilados. Se usa para ocultar C2, descargar payloads o exfiltrar datos, aprovechando protocolos comunes (HTTP/HTTPS, DNS, SSH) para evadir detección.
Ejemplo práctico: camuflas tráfico C2 en solicitudes HTTPS (GET/POST) hacia un servidor controlado por ti; a simple vista parece tráfico web normal y puede pasar por proxies y firewalls sin levantar sospechas.

Consideraciones: elegir protocolos aceptados por la red objetivo, mimetizar headers/patrones de tráfico normales, y gestionar latencia/tamaño de datos.