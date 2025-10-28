

Nos solicitan:
![[scenario-email.webp]]

Este módulo nos permitirá practicar nuestras habilidades (tanto las previas como las nuevas) en estas tareas. 

La evaluación final consiste en la ejecución de dos pruebas de penetración internas contra la empresa Inlanefreight. 
Durante estas evaluaciones, realizaremos una prueba de penetración interna simulando una brecha externa y una segunda con un cuadro de ataque dentro de la red interna, como suelen solicitar los clientes. 

Completar las evaluaciones de habilidades significa completar con éxito las tareas mencionadas en el documento de alcance y el correo electrónico de asignación de tareas. 
De esta manera, demostraremos un dominio sólido de diversos conceptos de ataques y enumeración de AD, tanto automatizados como manuales, conocimiento y experiencia con una amplia gama de herramientas, y la capacidad de interpretar los datos recopilados en un entorno de AD para tomar decisiones cruciales que permitan avanzar en la evaluación. 
El contenido de este módulo abarca los conceptos básicos de enumeración necesarios para realizar con éxito pruebas de penetración internas en entornos de Active Directory. También profundizaremos en muchas de las técnicas de ataque más comunes, a la vez que profundizaremos en algunos conceptos más avanzados como introducción al material centrado en AD que se abordará en módulos más avanzados.


## Assessment Scope
A continuación encontrará un documento de alcance completo para el compromiso que contiene toda la información pertinente proporcionada por el cliente.
Las siguientes IP, hosts y dominios definidos a continuación conforman el alcance de la evaluación:

| **Range/Domain**                | **Description**                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------- |
| `INLANEFREIGHT.LOCAL`           | Customer domain to include AD and web services.                                           |
| `LOGISTICS.INLANEFREIGHT.LOCAL` | Customer subdomain                                                                        |
| `FREIGHTLOGISTICS.LOCAL`        | Subsidiary company owned by Inlanefreight. External forest trust with INLANEFREIGHT.LOCAL |
| `172.16.5.0/23`                 | In-scope internal subnet.                                                                 |

## Fuera del alcance
- Cualquier otro subdominio de INLANEFREIGHT.LOCAL
- Cualquier subdominio de FREIGHTLOGISTICS.LOCAL
- Cualquier ataque de phishing o ingeniería social
- Cualquier otro IPS/dominio/subdominio no mencionado explícitamente
- Cualquier tipo de ataque contra el sitio web real inlanefreight.com fuera de la enumeración pasiva que se muestra en este módulo

---
## Métodos utilizados
Los siguientes métodos están autorizados para evaluar a Inlanefreight y sus sistemas:

##### Recopilación de información externa (verificaciones pasivas)
La recopilación de información externa está autorizada para demostrar los riesgos asociados a la información que se puede obtener sobre la empresa a través de internet. Para simular un ataque real, CAT-5 y sus evaluadores realizarán una recopilación de información externa de forma anónima en internet, sin proporcionar información previa sobre Inlanefreight más allá de la que se proporciona en este documento.

Cat-5 realizará una enumeración pasiva para descubrir información que pueda ayudar en las pruebas internas. Las pruebas emplearán diversos grados de recopilación de información de fuentes abiertas para identificar datos de acceso público que puedan suponer un riesgo para Inlanefreight y contribuir a la prueba de penetración interna. No se realizarán enumeraciones activas, escaneos de puertos ni ataques contra direcciones IP reales con acceso a internet ni contra el sitio web https://www.inlanefreight.com.

##### Pruebas internas
La parte de la evaluación interna está diseñada para demostrar los riesgos asociados a las vulnerabilidades en hosts y servicios internos (específicamente, Active Directory) mediante el intento de emular vectores de ataque dentro del área de operaciones de Inlanefreight. El resultado permitirá a Inlanefreight evaluar los riesgos de las vulnerabilidades internas y el impacto potencial de una vulnerabilidad explotada con éxito.

Para simular un ataque real, Cat-5 realizará la evaluación desde la perspectiva de un usuario interno no confiable, sin información previa más allá de la proporcionada en esta documentación y la obtenida mediante pruebas externas. Las pruebas comenzarán desde una posición anónima en la red interna con el objetivo de obtener las credenciales de usuario del dominio, enumerar el dominio interno, establecerse y moverse lateral y verticalmente para comprometer todos los dominios internos dentro del alcance. Los sistemas informáticos y las operaciones de red no se interrumpirán intencionalmente durante la prueba.

##### Pruebas de contraseñas
Los archivos de contraseñas capturados de los dispositivos de Inlanefreight, o proporcionados por la organización, pueden cargarse en estaciones de trabajo fuera de línea para su descifrado y utilizarse para obtener más acceso y cumplir con los objetivos de la evaluación. En ningún momento se revelará el archivo de contraseñas capturado ni las contraseñas descifradas a personas que no participen oficialmente en la evaluación. Todos los datos se almacenarán de forma segura en sistemas Cat-5 propios y aprobados, y se conservarán durante el plazo definido en el contrato oficial entre Cat-5 e Inlanefreight.