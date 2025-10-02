Un **subdominio** es una extensión del dominio principal (ej. `blog.ejemplo.com`, `shop.ejemplo.com`), utilizado para organizar secciones o servicios de un sitio.

En **reconocimiento web**, los subdominios son importantes porque pueden exponer:
- **Entornos de desarrollo o pruebas**: menos seguros y con posibles vulnerabilidades.
- **Portales ocultos de inicio de sesión**: paneles de administración no destinados al público.
- **Aplicaciones heredadas**: software antiguo con fallos conocidos.
- **Información sensible**: documentos, configuraciones o datos internos.

### Enumeración de subdominios
La **enumeración de subdominios** busca identificar estos subdominios mediante registros DNS (A, AAAA, CNAME).
Existen dos enfoques:

1. **Enumeración activa**
    - Interactúa directamente con los servidores DNS.
    - Ejemplo: intentos de transferencia de zona (raro que funcionen).
    - Más común: fuerza bruta con herramientas como _dnsenum_, _ffuf_ o _gobuster_.
        
2. **Enumeración pasiva**
    - Usa fuentes externas sin consultar directamente al servidor.
    - Ejemplo: **logs de transparencia de certificados (CT)**, que listan subdominios.
    - Búsqueda en **motores de búsqueda** con operadores (`site:`).
    - Bases de datos y servicios que agregan datos DNS.

