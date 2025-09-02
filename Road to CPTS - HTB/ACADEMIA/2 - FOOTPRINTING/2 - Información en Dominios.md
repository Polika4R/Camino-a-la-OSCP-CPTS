
En una prueba de penetración, recopilar información del dominio incluye toda la presencia online de la empresa, no solo subdominios, y se hace de forma pasiva para no ser detectados. Se empieza analizando la web principal para entender qué tecnologías y servicios ofrece la empresa. Esto ayuda a identificar la estructura y funcionamiento técnico detrás de esos servicios, incluso investigando aquellos que no conocemos, siempre desde una perspectiva técnica para obtener detalles útiles sin contacto directo.

### Certificados SLL
En ocasiones, el **certificado SSL** puede contener más de un subdominio. Debemos revisar dichos certificados siempre.

### crth.sh
Otra vía es consultar la página web http "crt.sh".
**crt.sh** es una interfaz web pública que consulta una base de datos llamada **Transparencia de Certificados** (Certificate Transparency, CT), que es un registro público y auditado donde todas las Autoridades de Certificación (CA) deben publicar los certificados SSL/TLS que emiten.

Esta página **recoge y almacena todos los certificados digitales emitidos públicamente** para cualquier dominio o subdominio.
- Permite buscar certificados asociados a un dominio concreto, incluyendo **subdominios**.
- Al buscar un dominio, muestra todos los certificados emitidos para ese dominio y sus subdominios, con fechas de emisión, expiración, la autoridad certificadora, etc.
- Esto permite encontrar subdominios que quizás no están documentados públicamente o que no se pueden descubrir fácilmente por otras vías.

Es útil para **revelar  subdominios**.

### SHODAN
Shodan es un motor de búsqueda, una página que sirve para encontrar cosas en Internet. Pero no es un buscador de páginas web o imágenes como pueden ser Google o Bing, sino que es **un buscador de sistemas y servicios conectados a internet**, lo que busca son máquinas conectadas a la red.

Por lo tanto, si haces una búsqueda como escribir el nombre de una ciudad, Shodan no te mostrará información relacionada con ese nomber que hayas escrito. En su lugar, te va a mostrar **servidores conectados en este lugar o que utilicen el término que hayas escrito**, y te dará información como su IP o su ubicación.

En este aspecto, **utilizar Shodan es completamente legal**, ya que se limita a mostrar información que ya está en Internet. En cambio, **lo que no es legal es acceder a los servidores que se muestran** en los resultados, ya que puedes estar cometiendo delitos de ciberdelincuencia.

**Shodan te va a permitir encontrar cualquier tipo de dispositivo conectado a Internet**, desde webcams, televisores inteligentes y dispositivos del hogar hasta semáforos, turbinas eólicas, y cualquier otro tipo de infraestructura que use la red para enviar los datos. 

