### Escenario

- Se obtiene **ejecución remota de código (RCE)** en un servidor IIS mediante una vulnerabilidad de subida de archivos.
- Se logra un **shell inverso** para enumerar el sistema.
- PowerShell está bloqueado por la política de aplicaciones → no se puede usar PowerUp.ps1.
- Durante la enumeración manual se descubre **SeImpersonatePrivilege**, útil para escalar privilegios.
- Se intenta transferir la herramienta **PrintSpoofer** con distintos métodos:
    - **Certutil** → bloqueado por filtrado de contenido web.
    - **FTP** → bloqueado por firewall (puerto 21).
    - **SMB (puerto 445)** → permitido, se usa con **Impacket smbserver** y se transfiere con éxito el binario.
- Con PrintSpoofer se logra la **escalada a administrador**.
### Lecciones clave

- La **transferencia de archivos** es esencial en pentesting y puede estar limitada por:
    - Controles en el host (whitelisting, bloqueo de apps, AV/EDR).
    - Controles de red (firewalls, IDS/IPS, filtrado web).
- Es vital conocer múltiples métodos de transferencia para adaptarse a diferentes restricciones.
- Windows y Linux ofrecen herramientas integradas y técnicas alternativas que pueden explotarse.
- Practicar en entornos controlados (como **HTB Academy**) permite comprender qué método aplicar en cada situación.