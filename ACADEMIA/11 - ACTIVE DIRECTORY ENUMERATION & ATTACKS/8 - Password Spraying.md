
### Concepto y Objetivo

El "Password Spraying** es una técnica de ataque que busca obtener acceso a sistemas, logrando potencialmente una base (foothold) en una red objetivo. A diferencia del **fuerza bruta** (que prueba muchas contraseñas en una sola cuenta), este método es más sutil.
- **Técnica:** Intentar iniciar sesión en un servicio expuesto utilizando **una única contraseña común** contra una **larga lista de nombres de usuario o correos electrónicos**.
- **Ventaja:** Es **menos probable que bloquee cuentas** de usuario que la fuerza bruta, ya que se envían menos intentos de inicio de sesión por cuenta en un corto período de tiempo.
- **Momento de Uso:** Se realiza a menudo en paralelo con otras tareas largas de la prueba de penetración (como el envenenamiento de caché o el escaneo) para optimizar el tiempo limitado de la evaluación.

### Obtención de la Lista de Objetivos
Los nombres de usuario se obtienen en la fase de **OSINT (Inteligencia de Fuentes Abiertas)** o durante la **enumeración** inicial.
- **Fuentes Comunes:**
    - Listas de nombres de usuario estadísticamente probables (ej. `jsmith.txt`).  
    - Resultados de **scraping de LinkedIn**.
    - Enumeración de usuarios válidos del dominio con herramientas como **Kerbrute**.
    
- **Técnicas Creativas (Caso de estudio):**
    - Buscar **documentos PDF** públicos de la organización en Google.
    - Revisar los **metadatos del documento** (campo _Autor_) para descubrir el formato de nombre de usuario interno (ej. GUIDs predecibles como `F9L8`).
    - Usar scripts Bash para **generar combinaciones** basadas en el formato descubierto, permitiendo enumerar _todas_ las cuentas del dominio, superando la eficacia de las listas genéricas.
###  Consideraciones de Seguridad (Riesgo de Bloqueo)

Aunque es más sutil que la fuerza bruta, la pulverización de contraseñas **aún puede bloquear cientos de cuentas** de producción.

- **Mitigación de Riesgos:**
    - Es **esencial introducir una demora** (DELAY) entre los intentos de inicio de sesión o entre el uso de diferentes contraseñas.
    - El objetivo ideal es **obtener la política de contraseñas del dominio** (número de intentos fallidos antes del bloqueo y tiempo de restablecimiento) antes de realizar el ataque.
    - **Regla general (si se desconoce la política):** Esperar unas **pocas horas** entre intentos para permitir que el umbral de bloqueo se reinicie automáticamente.
    - **Opción de "último recurso":** Realizar un solo intento dirigido con una contraseña débil/común ("hail mary") si se han agotado otras opciones para obtener acceso.


---

### Casos de Uso (Ejemplos del Redactor)

1. **Encontrar Usuarios de Bajo Privilegio:** Uso de Kerbrute y una lista combinada de usuarios para pulverizar con `Welcome1`. El acceso de cuentas de bajo privilegio fue suficiente para ejecutar **BloodHound** e identificar rutas de ataque al dominio.
    
2. **Explotar Formato Predecible:** Descubrimiento del formato de nombre de usuario GUID (`F9L8`) a través de metadatos de PDF. Generación masiva de usuarios y posterior ataque de pulverización. Esto condujo a una cadena de ataque complicada (**Resource-Based Constrained Delegation (RBCD)** y **Shadow Credentials**) que resultó en el compromiso total del dominio.