
- **Qué es `.well-known`:**  
    Un directorio estándar en la raíz de un sitio web (`/.well-known/`) definido por **RFC 8615**, que centraliza metadatos importantes como archivos de configuración, políticas de seguridad y detalles de servicios o protocolos. Facilita que navegadores, aplicaciones y herramientas de seguridad descubran información automáticamente mediante URLs consistentes.
    
- **Ejemplos de URIs `.well-known` registrados por IANA:**
    
|URI Suffix|Descripción|Estado|
|---|---|---|
|`security.txt`|Información de contacto para reportar vulnerabilidades|Permanente|
|`change-password`|URL estándar para cambiar contraseñas|Provisional|
|`openid-configuration`|Configuración de OpenID Connect (autenticación)|Permanente|
|`assetlinks.json`|Verificación de propiedad de activos digitales|Permanente|
|`mta-sts.txt`|Políticas de seguridad para correo SMTP (MTA-STS)|Permanente|
- **Uso en web reconnaissance:**
    - Permite **descubrir endpoints y configuraciones** que pueden ser probados en pruebas de penetración.
    - Ejemplo: `.well-known/openid-configuration` devuelve un JSON con:
        - Endpoints de autorización, token y userinfo.
        - `jwks_uri` con claves criptográficas.
        - Tipos de respuestas y scopes soportados.
        - Algoritmos de firma utilizados.
            
- **Ventaja principal:**  
    `.well-known` ofrece **acceso estructurado a metadatos críticos**, ayudando a mapear la arquitectura y seguridad de un sitio web de manera sistemática y estandarizada.



Los archivos dentro de `.well-known` **no son secretos**, sino información que el sitio **pone intencionalmente para que sea accesible**. Por eso, cualquier persona que conozca la URL puede acceder a ellos. Por ejemplo:

- `/.well-known/security.txt` → Contiene contactos de seguridad.
- `/.well-known/openid-configuration` → Contiene información de endpoints de autenticación y claves públicas.

Esto **no es un “leak” accidental”**, sino datos que el sitio decidió hacer públicos para facilitar la interoperabilidad con navegadores, apps o herramientas de seguridad.
Sin embargo, desde la perspectiva de un atacante o auditor:

- Puede **usar estos datos para mapear el sitio** y descubrir endpoints, servicios o configuraciones.
- Por ejemplo, saber dónde está el endpoint de login o qué algoritmos de firma usa un servidor puede ayudar a planificar un ataque.
