

Los motores de búsqueda no solo sirven para consultas cotidianas, también son herramientas clave de OSINT (Open Source Intelligence). Permiten extraer información pública útil para seguridad, investigación y análisis.

---

## Importancia de Search Engine Discovery

- Open Source: Toda la información es pública → legal y ética.
    
- Amplitud: Indexan gran parte de la web.
    
- Fácil de usar: No requiere conocimientos técnicos avanzados.
    
- Gratis: Recurso accesible para todos.
    

### Aplicaciones principales

- Seguridad: Detectar vulnerabilidades, datos expuestos.
    
- Inteligencia competitiva: Productos, servicios, estrategias de competidores.
    
- Periodismo investigativo: Conexiones ocultas, finanzas, prácticas cuestionables.
    
- Threat Intelligence: Rastreo de actores maliciosos y amenazas emergentes.
    

Limitación: No todo está indexado (contenido oculto o protegido).

---

## Operadores de búsqueda

Permiten búsquedas precisas dentro del índice de los buscadores.

|Operador|Descripción|Ejemplo|Uso|
|---|---|---|---|
|`site:`|Limita a un dominio|`site:example.com`|Páginas de example.com|
|`inurl:`|Término en la URL|`inurl:login`|Buscar páginas de login|
|`filetype:`|Tipo de archivo|`filetype:pdf`|Documentos PDF|
|`intitle:`|Término en el título|`intitle:"confidential report"`|Documentos con título “confidential report”|
|`intext:` / `inbody:`|Término en el cuerpo|`intext:"password reset"`|Páginas con “password reset”|
|`cache:`|Versión en caché|`cache:example.com`|Ver contenido previo|
|`link:`|Páginas que enlazan a un sitio|`link:example.com`|Backlinks hacia example.com|
|`related:`|Sitios similares|`related:example.com`|Páginas relacionadas|
|`info:`|Información básica de un sitio|`info:example.com`|Datos del dominio|
|`define:`|Definiciones|`define:phishing`|Definición de “phishing”|
|`numrange:`|Rango numérico|`site:example.com numrange:1000-2000`|Páginas con números entre 1000-2000|
|`allintext:`|Varias palabras en el cuerpo|`allintext:admin password reset`|Texto con ambas palabras|
|`allinurl:`|Varias palabras en la URL|`allinurl:admin panel`|Páginas con “admin panel”|
|`allintitle:`|Varias palabras en el título|`allintitle:confidential report 2023`|Documentos con esas palabras en el título|
|`AND`|Obliga a que estén todos los términos|`site:example.com AND (inurl:admin OR inurl:login)`|Admin o login en example.com|
|`OR`|Cualquiera de los términos|`"linux" OR "ubuntu" OR "debian"`|Páginas con alguno de ellos|
|`NOT`|Excluye resultados|`site:bank.com NOT inurl:login`|Excluye login|
|`*` (comodín)|Cualquier palabra|`site:socialnetwork.com filetype:pdf user* manual`|Manuales de usuario|
|`..`|Rango numérico|`site:ecommerce.com "price" 100..500`|Productos entre 100-500|
|`""`|Frase exacta|`"information security policy"`|Coincidencia exacta|
|`-`|Excluir términos|`site:news.com -inurl:sports`|Noticias sin deportes|