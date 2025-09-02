**robots.txt** es un archivo de texto simple que se coloca en la raíz de un sitio web (por ejemplo: `www.example.com/robots.txt`).

Funciona como una **guía de etiqueta para los bots**, indicando qué partes del sitio pueden explorar y cuáles deben evitar. Está basado en el **Robots Exclusion Standard** y contiene **directivas** que los crawlers respetan para saber qué páginas o directorios no deben rastrear.

En pocas palabras: **robots.txt le dice a los bots “puedes entrar aquí, pero evita estas áreas”**.

Ejemplo del código *robots.txt*:

```txt
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

User-agent: Googlebot
Crawl-delay: 10

Sitemap: https://www.example.com/sitemap.xml
```

**Estructura básica:**
- **User-agent:** define a qué bot se aplican las reglas (`*` = todos los bots, o nombres específicos como Googlebot).

| Directiva   | Descripción                                                       | Ejemplo                                        |     |
| ----------- | ----------------------------------------------------------------- | ---------------------------------------------- | --- |
| Disallow    | Bloquea rutas que el bot no debe rastrear                         | `Disallow: /admin/`                            |     |
| Allow       | Permite rastrear rutas específicas, aunque estén bajo un Disallow | `Allow: /public/`                              |     |
| Crawl-delay | Establece un retraso entre solicitudes al servidor                | `Crawl-delay: 10`                              |     |
| Sitemap     | Indica la ubicación del sitemap XML                               | `Sitemap: https://www.example.com/sitemap.xml` |     |
- **Importancia de respetarlo:**
    - Evita sobrecargar el servidor.
    - Protege información sensible de ser indexada.
    - Cumple con normas éticas y legales.
        
- **Uso en reconnaissance:**
    - **Descubrir directorios ocultos:** rutas prohibidas pueden indicar paneles administrativos o archivos sensibles.
    - **Mapear la estructura del sitio:** ayuda a entender secciones no visibles desde la navegación normal.
    - **Detectar trampas para bots:** algunos sitios colocan honeypots para bots maliciosos.

