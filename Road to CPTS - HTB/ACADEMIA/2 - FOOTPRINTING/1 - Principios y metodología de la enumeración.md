
# Principios
La recopilación de información puede venir por parte de dominios, direcciones IP, servicios accesibles o muchos otros sitios.

Tenemos que **entender como se estructura la compañía o el sistema que se esté auditando**.
No hay que ir directo a probar fuerza bruta o acceder aisladamente a sistemas yendo casi a ciegas.

Si encontramos algún servicio de autentificación del tipo "SSH", probar fuerza bruta directamente podría hacer que incluyesen al atacante en una lista negra y no podamos realizar con las pruebas.

Un pirata que busca tesoros no se pondría a cavar en la playa con una pala porque sí, sino que planificaría, estudiaría mapas y buscaría más pistas. Cavar hoyos sin sentido haría **gastar tiempo y esfuerzos en vano por no realizar un estudio previo**

Tenemos que preguntarnos **qué vemos, por qué lo vemos, qué no vemos, por qué no lo vemos, etc.** Esto ayuda a ampliar la perspectiva y a profundizar en la investigación.


# Metodología de Enumeración en Pentesting

## Introducción
- Necesidad de metodología estandarizada para evitar omisiones.
- La enumeración es un proceso dinámico.
- Metodología propuesta: estática pero flexible y adaptable.

## Niveles de enumeración
- Infrastructure-based enumeration
- Host-based enumeration
- OS-based enumeration

## 6 Capas de Enumeración

| Capa | Descripción | Categorías de Información |
|-------|-------------|----------------------------|
| 1. Presencia en Internet | Identificar objetivos y sistemas accesibles públicamente | Dominios, subdominios, vHosts, ASN, Netblocks, IPs, nube, medidas de seguridad |
| 2. Puerta de enlace | Identificar medidas de seguridad perimetral | Firewalls, DMZ, IPS/IDS, EDR, proxies, NAC, segmentación, VPN, Cloudflare |
| 3. Servicios accesibles | Identificar servicios y su configuración | Tipo, funcionalidad, puerto, versión, interfaz |
| 4. Procesos | Procesos internos y tareas asociadas | PID, datos procesados, origen, destino |
| 5. Privilegios | Permisos y privilegios de usuarios y servicios | Grupos, usuarios, permisos, restricciones |
| 6. Configuración del SO | Configuración interna y seguridad del sistema operativo | Tipo SO, parches, configuración red, archivos confidenciales |

## Notas
- Capas 1 y 2 aplican diferente en redes internas (ej. Active Directory).
- No incluye información humana (OSINT sobre empleados) en capa 1.
- Prueba de penetración = laberinto → encontrar brechas y caminos útiles.
- No todas las brechas permiten acceso.
- Pruebas con tiempo limitado; no garantizan detectar todo.
- Caso SolarWinds: ejemplo de persistencia y complejidad.

## Proceso práctico
1. Presencia en Internet: encontrar todos los objetivos accesibles.
2. Puerta de enlace: analizar protección y seguridad.
3. Servicios accesibles: identificar y entender servicios.
4. Procesos, privilegios y configuración: entender detalles internos para explotar.

## Conceptos clave
- Metodología = resumen sistemático, no pasos fijos.
- Herramientas y comandos cambian; metodología es marco general.
- Objetivo: obtener información para avanzar en la prueba.






