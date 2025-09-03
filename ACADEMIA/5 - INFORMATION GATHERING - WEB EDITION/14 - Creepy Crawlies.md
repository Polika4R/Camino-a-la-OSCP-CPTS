Existen múltiples herramientas que facilitan el web crawling, cada una con distintas fortalezas:

- **Burp Suite Spider**: Incluido en Burp Suite, permite mapear aplicaciones web, descubrir contenido oculto y detectar posibles vulnerabilidades.
- **OWASP ZAP**: Escáner de seguridad gratuito y de código abierto, con un módulo de _spider_ para explorar aplicaciones y encontrar fallos.
- **Scrapy**: Framework en Python, muy flexible y escalable, ideal para crear _crawlers_ personalizados y extraer datos estructurados.
- **Apache Nutch**: Crawler en Java, altamente extensible y preparado para rastreos a gran escala, aunque requiere mayor experiencia técnica.

## SCRAPY
- **Scrapy** es una herramienta para crear spiders (arañas) que permiten hacer **crawling** o **reconocimiento automatizado** de sitios web, en este caso sobre `inlanefreight.com`.
- Para aprender más sobre técnicas de crawling/spidering, se recomienda revisar el módulo **"Using Web Proxies"** del curso **CBBH**.
- **Instalación de Scrapy**:
    - Se instala con pip:
```
pip3 install scrapy`
```

## RECON SPYDER
La **utilidad de ReconSpider** es servir como una **araña personalizada con Scrapy** diseñada para:
- **Automatizar el reconocimiento web** sobre un dominio objetivo.
- **Rastrear** (crawl) el sitio web siguiendo enlaces internos.
- **Recolectar información útil**, como:
    - Estructura de directorios y páginas.
    - Enlaces internos y externos.
    - Recursos expuestos (imágenes, JS, CSS)        
    - Posibles puntos de entrada para pruebas de seguridad.
    
- **Ahorrar tiempo** en la fase de recolección de información dentro de un pentest o auditoría.
- **Descargar ReconSpider** (spider personalizado con Scrapy):

```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
    unzip ReconSpider.zip
```

- **Ejecutar ReconSpider** sobre el dominio objetivo:
```bash
python3 ReconSpider.py http://inlanefreight.com
```
   
- Se puede reemplazar `inlanefreight.com` por cualquier dominio a analizar.
    
- El spider rastrea el sitio y recolecta información útil para el reconocimiento.
 
 
Tras ejecutar **ReconSpider.py**, los resultados se guardan en `results.json`.
    
- Cada clave del JSON corresponde a un tipo de dato extraído del sitio:
    - **emails** → direcciones de correo encontradas.
    - **links** → URLs internas y externas.
    - **external_files** → archivos externos (ej. PDFs).
    - **js_files** → scripts JS usados por el sitio.
    - **form_fields** → campos de formularios (si hay).
    - **images** → URLs de imágenes.
    - **videos / audio** → medios encontrados (si hay).
    - **comments** → comentarios HTML en el código fuente.
- Explorar este archivo permite obtener **información útil sobre la arquitectura, contenido y posibles puntos de interés** para un análisis más profundo del sitio web.

-------
**QUESTIONS**
**After spidering inlanefreight.com, identify the location where future reports will be stored. Respond with the full domain, e.g., files.inlanefreight.com.**

Instalo scrapy con pip3 para poder utilizar

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-oqnbolf5og]─[~]
└──╼ [★]$ pip3 install scrapy
```

Instalo ReconSpyder con:
```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
    unzip ReconSpider.zip
```

Y lo ejecuto contra tal dirección:

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-oqnbolf5og]─[~]
└──╼ [★]$ python3 ReconSpider.py http://inlanefreight.com
```

Hago un cat a los resultados en formato JSON y observo:

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-oqnbolf5og]─[~]
└──╼ [★]$ python3 ReconSpider.py http://inlanefreight.com
```

Y observo en el apartado de los comentarios (dentro del propio JSON), el siguiente comentario:
```
    "<!-- TO-DO: change the location of future reports to inlanefreight-comp133.s3.amazonaws.htb -->",
```

Siendo la respuesta *inlanefreight-comp133.s3.amazonaws.htb*

