La **automatización** del reconocimiento web permite realizar tareas de recolección de información de forma **eficiente, escalable y consistente**, reduciendo errores humanos y permitiendo analizar múltiples objetivos simultáneamente.

**Ventajas de automatizar:**

- **Eficiencia:** Completa tareas repetitivas más rápido que manualmente.
- **Escalabilidad:** Permite trabajar con múltiples dominios o targets.
- **Consistencia:** Resultados reproducibles y menos propensos a errores.
- **Cobertura completa:** DNS, subdominios, crawling, puertos, etc.
- **Integración:** Compatible con otros frameworks y herramientas de seguridad.

## **Frameworks de Reconocimiento Populares**

1. **FinalRecon:** Python, modular, incluye SSL, Whois, header info, crawling y más.
2. **Recon-ng:** Framework Python con módulos para DNS, subdominios, escaneo de puertos y explotación.
3. **theHarvester:** CLI para emails, subdominios, hosts y banners desde fuentes públicas.
4. **SpiderFoot:** OSINT automatizado integrando múltiples fuentes (IP, dominios, emails, redes sociales).
5. **OSINT Framework:** Colección de herramientas y recursos para inteligencia de código abierto.

## **FinalRecon – Funcionalidades Clave**

- **Header Information:** Info de servidores y posibles vulnerabilidades.
- **Whois Lookup:** Datos del dominio y contacto del registrante.
- **SSL Info:** Validez, emisor y detalles de certificados SSL/TLS.
- **Crawler:** HTML, CSS, JS, enlaces internos/externos, imágenes, `robots.txt`, `sitemap.xml`, links históricos de Wayback Machine.
- **DNS Enumeration:** Más de 40 tipos de registros DNS, incluyendo DMARC.
- **Subdomain Enumeration:** Usa múltiples fuentes (crt.sh, VirusTotal, Shodan, Facebook API, etc.).  
- **Directory Enumeration:** Soporte de wordlists y extensiones personalizadas.
- **Wayback Machine:** URLs de los últimos 5 años.
## **Instalación Rápida de FinalRecon**

```
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help
```

## **Opciones Principales del Script**

- `--url URL` → Target.
- `--headers` → Info de cabeceras.
- `--sslinfo` → SSL.
- `--whois` → Whois.
- `--crawl` → Crawling.
- `--dns` → Enumeración DNS.
- `--sub` → Subdominios.
- `--dir` → Directorios.
- `--wayback` → URLs históricas.
- `--ps` → Escaneo rápido de puertos.
- `--full` → Recon completo.

Para obtener información de cabeceras y Whois de `inlanefreight.com`:
```
./finalrecon.py --url inlanefreight.com --headers --whois`
```


