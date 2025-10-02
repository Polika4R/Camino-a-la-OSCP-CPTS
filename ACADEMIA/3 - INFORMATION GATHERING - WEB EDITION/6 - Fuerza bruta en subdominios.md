La enumeración de subdominios mediante **Brute-Force** es una técnica activa de descubrimiento de subdominios que utiliza listas predefinidas de nombres posibles para probar sistemáticamente cuáles existen en un dominio objetivo. Usar **wordlists** bien diseñadas aumenta la eficacia del proceso.

El proceso se divide en cuatro pasos:
1. **Selección de wordlist:**
    - **General:** Nombres comunes como `dev`, `staging`, `blog`, `mail`, `admin`. Útil si no se conoce la convención de nombres del objetivo.
    - **Específica:** Orientada a industrias, tecnologías o patrones del objetivo. Más eficiente y con menos falsos positivos.
    - **Personalizada:** Creada a partir de palabras clave o información obtenida de otras fuentes.
2. **Iteración y consulta:**  
    Cada palabra de la lista se combina con el dominio principal (por ejemplo, `dev.ejemplo.com`) y se prueba sistemáticamente.
3. **Búsqueda DNS:**  
    Se realiza una consulta DNS para cada subdominio potencial para verificar si resuelve a una dirección IP (tipo A o AAAA).
4. **Filtrado y validación:**  
    Los subdominios que resuelven correctamente se añaden a la lista de subdominios válidos. Se pueden realizar pasos adicionales de validación, como acceder a ellos mediante un navegador, para confirmar su existencia y funcionalidad.

Existen varias herramientas destacadas para la enumeración de subdominios mediante **brute-force**:
- **dnsenum:** Herramienta completa de enumeración DNS, soporta ataques de diccionario y brute-force.    
- **fierce:** Fácil de usar, realiza descubrimiento recursivo de subdominios e incluye detección de comodines.
- **dnsrecon:** Combina múltiples técnicas de reconocimiento DNS y permite formatos de salida personalizables.
- **amass:** Enfocada en descubrimiento de subdominios, integra otras herramientas y fuentes de datos extensas.
- **assetfinder:** Herramienta simple y rápida para encontrar subdominios mediante varias técnicas; ideal para escaneos ligeros.
- **puredns:** Potente y flexible para brute-force DNS, con capacidad de resolver y filtrar resultados eficazmente.

## DNS ENUM
**DNSEnum** es una herramienta de línea de comandos escrita en Perl, muy versátil y usada para el reconocimiento DNS. Permite recopilar información sobre la infraestructura DNS de un dominio y descubrir posibles subdominios. Sus funciones principales son:
1. **Enumeración de registros DNS:** Recupera registros A, AAAA, NS, MX y TXT para obtener un panorama completo de la configuración DNS del objetivo.
2. **Intentos de transferencia de zona:** Prueba transferencias de zona desde servidores de nombrs descubiertos; si tiene éxito, puede revelar mucha información DNS.
3. **Brute-force de subdominios:** Soporta enumeración de subdominios mediante listas de palabras, probando sistemáticamente nombres posibles.
4. **Scraping de Google:** Busca subdominios adicionales que pueden no estar directamente en los registros DNS.
5. **Búsqueda inversa:** Realiza consultas DNS inversas para identificar dominios asociados a una IP y descubrir otros sitios en el mismo servidor.
6. **Consultas WHOIS:** Obtiene información sobre la propiedad y registro del dominio.

### Ejemplo de uso de DNS ENUM


```bash
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

- `dnsenum --enum inlanefreight.com`: Especifica el dominio objetivo y activa opciones predeterminadas de enumeración (`--enum`).
- `-f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt`: Ruta a la wordlist de SecLists con los 20,000 subdominios más comunes. Ajusta la ruta según tu instalación.    
- `-r`: Activa el **brute-force recursivo**, es decir, si se encuentra un subdominio, dnsenum intentará enumerar subdominios de ese subdominio también.

Esto permite realizar un escaneo exhaustivo de los subdominios del objetivo, combinando wordlists conocidas y descubrimiento recursivo.

Salida esperada:

```shell-session
Polika4RM@htb[/htb]$ dnsenum --enum inlanefreight.com -f  /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt 

dnsenum VERSION:1.2.6

-----   inlanefreight.com   -----


Host's addresses:
__________________

inlanefreight.com.                       300      IN    A        134.209.24.248

[...]

Brute forcing with /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt:
_______________________________________________________________________________________

www.inlanefreight.com.                   300      IN    A        134.209.24.248
support.inlanefreight.com.               300      IN    A        134.209.24.248
[...]


done.
```

---------------
##### Questions:

**1. Using the known subdomains for inlanefreight.com (www, ns1, ns2, ns3, blog, support, customer), find any missing subdomains by brute-forcing possible domain names. Provide your answer with the complete subdomain, e.g., www.inlanefreight.com.**

Ejecuto el comando:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-nfypd10slq]─[~]
└──╼ [★]$ dnsenum --enum inlanefreight.com -f /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r

```

Utilizando el diccionario subdomains-top1million-22000 no aparece nada nuevo, con el 110000 (más extenso), sí !


