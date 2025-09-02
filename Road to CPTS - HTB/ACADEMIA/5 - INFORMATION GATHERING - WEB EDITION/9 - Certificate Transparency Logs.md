
En Internet, la **confianza** es un recurso frágil, y uno de sus pilares es el protocolo **SSL/TLS**, que cifra la comunicación entre el navegador y los sitios web. En el centro de SSL/TLS están los **certificados digitales**, que verifican la identidad de un sitio y permiten comunicaciones seguras.

Sin embargo, la emisión de certificados puede ser vulnerable. Un atacante podría aprovechar certificados **mal emitidos o falsos** para suplantar sitios, interceptar datos o propagar malware. Aquí es donde entran los **Certificate Transparency (CT) logs**.

### ¿Qué son los CT Logs?

Los **CT logs** son registros públicos y de solo anexado que registran la emisión de certificados SSL/TLS. Cada vez que una Autoridad Certificadora (CA) emite un certificado, debe enviarlo a varios CT logs. Cualquiera puede inspeccionar estos registros, lo que genera **transparencia y trazabilidad**.

**Beneficios principales:**

- **Detección temprana de certificados falsos:** Permite a investigadores y propietarios de sitios identificar certificados sospechosos o mal emitidos.
    
- **Responsabilidad de las CAs:** Si una CA emite un certificado incorrecto, queda visible públicamente, fomentando buenas prácticas.
    
- **Fortalecimiento de la PKI web:** Aporta supervisión pública, aumentando la seguridad del sistema de confianza online.
    

---

### CT Logs y Reconocimiento Web

Los CT logs son muy útiles para **enumerar subdominios**. A diferencia de métodos de fuerza bruta o listas de palabras, los CT logs muestran **todos los certificados emitidos**, revelando subdominios históricos, activos o antiguos. Esto incluye subdominios con certificados caducados, que podrían ser vulnerables.

**Ventaja:** permite descubrir subdominios de forma **más confiable y eficiente**, sin depender de adivinanzas o listas incompletas.

---

### Herramientas para buscar en CT Logs

| Herramienta | Características                                                                       | Uso                                                  | Pros                                | Contras                               |
| ----------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------- | ----------------------------------- | ------------------------------------- |
| **crt.sh**  | Interfaz web sencilla, busca por dominio, muestra certificados y SANs                 | Buscar subdominios, historial de certificados        | Gratis, fácil de usar, sin registro | Opciones de filtrado limitadas        |
| **Censys**  | Buscador avanzado de dispositivos conectados, filtrado por dominio, IP y certificados | Análisis profundo, detectar errores de configuración | Datos extensos, API disponible      | Requiere registro (gratis disponible) |

