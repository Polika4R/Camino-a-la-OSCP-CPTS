
Cuando realizamos escaneos con Nmap, es muy recomendable guardar los resultados para poder analizarlos después o comparar distintos métodos de escaneo.
Nmap permite guardar resultados en 3 formatos principales:

|Opción|Extensión|Descripción|
|---|---|---|
|`-oN`|`.nmap`|Salida normal legible para humanos|
|`-oG`|`.gnmap`|Salida en formato grepable (para procesamiento con scripts)|
|`-oX`|`.xml`|Salida en formato XML (para procesamiento y generación de informes)|
También existe la opción:  
`-oA <nombre>` — Guarda la salida en los tres formatos simultáneamente con el prefijo dado.

>sudo nmap 10.129.218.42 -p- -oA target

Guarda resultados en los tres formatos con prefijo `target`:
- target.nmap (normal)
- target.gnmap (grepable)
- target.xml (XML)
### Visualización de archivos XML generados
El archivo `.xml` de Nmap no es un HTML listo para visualizar bonito, es un archivo estructurado de datos. Para verlo claro en un navegador, necesitas transformarlo en un archivo `.html` que el navegador entienda bien.
Nmap incluye una hoja de estilo llamada **XSL** que transforma ese XML en un reporte HTML legible. Para aplicar esa transformación usamos la herramienta `xsltproc`.

```
xsltproc target.xml -o target.html
xdg-open target.html
```

