**Crawling** (o **spidering**) es un proceso automático que hace un programa llamado **web crawler** o “araña web” para **recorrer páginas web y recopilar información**.
- Piensa en una **araña recorriendo su telaraña**: empieza en un punto (una página) y va siguiendo todos los hilos (enlaces) hacia otras páginas.
- Mientras sigue los enlaces, el crawler **guarda información** sobre cada página, como su contenido, enlaces, imágenes, etc.
- Esta información se usa, por ejemplo, para:
    - **Motores de búsqueda** (como Google) que indexan las páginas para que se puedan buscar.
    - **Análisis de datos** o estudios de seguridad (reconocimiento web).

En pocas palabras: **crawling = navegar automáticamente por un sitio web recogiendo datos de todas sus páginas**.

## ¿Cómo trabaja un Web Crawlers?
1. **Inicio con una URL semilla**
    - El crawler empieza en una página inicial, por ejemplo, la **homepage**.
    - Extrae todos los enlaces de esa página y los pone en una **cola** para visitar después.

```
Homepage
├── link1
├── link2
└── link3
```

2. **Visitar cada enlace**
    - El crawler visita `link1`.
    - Extrae todos los enlaces que encuentra allí y los añade a la cola.
    - Por ejemplo, `link1` tiene enlaces a `Homepage`, `link2`, `link4` y `link5`.

```
link1 Page
├── Homepage
├── link2
├── link4
└── link5
```

3. **Repetir el proceso**
    - El crawler sigue visitando los enlaces de la cola, extrayendo más enlaces, y así sucesivamente.
    - Puede recorrer **todo un sitio web** o incluso una gran parte de la web, dependiendo de su configuración.



### **Depth-First Crawling:**  
Explora un solo camino de enlaces hasta el final antes de retroceder y seguir otras rutas. Es útil para acceder a contenido profundo dentro de un sitio web.

- **Tipos de información que puede extraer un crawler:**
    - **Enlaces internos y externos:** Para mapear la estructura del sitio y descubrir páginas ocultas.
    - **Comentarios:** Pueden revelar información sensible o pistas sobre vulnerabilidades.
    - **Metadatos:** Títulos, descripciones, autores y fechas que dan contexto sobre las páginas.
    - **Archivos sensibles:** Backups, configuraciones, logs o archivos con contraseñas y claves.
        
- **Importancia del contexto:**
    - Los datos aislados pueden parecer irrelevantes, pero al combinarlos se puede descubrir información crítica.
    - Por ejemplo, un directorio `/files/` expuesto y comentarios sobre un "file server" pueden indicar acceso público a información sensible.
        

