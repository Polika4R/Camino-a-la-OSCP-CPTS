- **Shell:** Programa que permite al usuario interactuar con el sistema operativo mediante texto (ej. Bash, Zsh, cmd, PowerShell).
    - En pruebas de penetración, “conseguir un shell” significa tener acceso interactivo a un sistema tras explotar una vulnerabilidad.
    - Beneficios de un shell:
        - Acceso directo a comandos y sistema de archivos.
        - Permite enumerar vectores para escalar privilegios, pivotar, transferir archivos, etc.
        - Facilita mantener persistencia y automatizar acciones.
        - Es más sigiloso y rápido que interfaces gráficas como VNC o RDP.

- **Perspectivas del shell:**
    1. **Computación:** Interfaz de texto para administrar tareas (Bash, cmd…).
    2. **Explotación/Security:** Resultado de explotar vulnerabilidades (ej. EternalBlue en Windows).
    3. **Web:** Web shell que permite controlar un servidor a través de scripts subidos a la web.

- **Payloads:**
    - Código o datos diseñados para ejecutar acciones específicas o explotar vulnerabilidades.
    - Tipos según contexto:
        - **Networking:** Datos encapsulados en un paquete.
        - **Computación básica:** Parte de una instrucción que define la acción.
        - **Programación:** Datos transportados por instrucciones de código.
        - **Seguridad:** Código para explotar vulnerabilidades o desplegar malware.

- **Objetivo:** Combinar payloads y shells para ganar acceso, interactuar con el sistema objetivo y mantener persistencia de manera eficiente y discreta.