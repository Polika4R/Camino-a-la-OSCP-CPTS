El proceso de autenticación del cliente de Windows involucra múltiples módulos responsables del inicio de sesión, la recuperación de credenciales y la verificación. Entre los diversos mecanismos de autenticación de Windows, Kerberos es uno de los más utilizados y complejos. La Autoridad de Seguridad Local (LSA) es un subsistema protegido que autentica a los usuarios, administra los inicios de sesión locales, supervisa todos los aspectos de la seguridad local y proporciona servicios de traducción entre nombres de usuario e identificadores de seguridad (SID).

El subsistema de seguridad mantiene las políticas de seguridad y las cuentas de usuario en un sistema informático. En un controlador de dominio, estas políticas y cuentas se aplican a todo el dominio y se almacenan en Active Directory. Además, el subsistema LSA proporciona servicios de control de acceso, comprobación de permisos y generación de mensajes de auditoría de seguridad.
### Flujo de inicio de sesión (resumen paso a paso)
![[Pasted image 20251008115447.png]]

# lsass.exe
El lsass.exe se encarga de verificar las credenciales del usuario. WinLogon.exe le pasa las credenciales al "lsass.exe" y el propio lsass.exe las verifica contra la base de datos de cuentas de usuario.
	- Si no estoy en un dominio / Local (un ordenador privado): las credenciales se comprueban contra la lista de usuarios locales del propio equipo. 
	- Si estoy en un dominio / Remoto: El lsass le pregunta al servidor de dominio si las credenciales son correctas. *¿Oye, este usuario puede acceder?.* El servidor de dominio es "AD Directory Services".

### Local / Non Domain Joined
- NTLM: es un protocolo de autenticación que usa windows cuando no estoy en un dominio o no puedo usar kerberos. 
- Del NTLM al SAM: NTLM compara las credenciales obtenidas con la base de datos SAM. 
- msv1_0.dll Y SAMSRV.DLL son librerías que utilizan tanto NTLM como SAM para hacer su trabajo.

### Remote / domain joined
- NTLM: me puedo autenticar por NTLM porque hay veces que kerberos no está disponible y NTLM actúa como plan B. NTLM también se puede utilizar en una red coorporativa. Si tengo el ordenador aislado de la red, NTLM no se comunica con el ACTIVE DIRECTORY y dentro de ese ordenador aislado funcionaria todo como si fuese un equipo conectado a la red. Lo único que no tendría acceso a los recursos que dependan del dominio (carpetas compartidas de la empresa, etc.). Todo lo local lo podría utilizar sin problema.
- Kerberos: es otro protocolo de autenticación que utiliza un sistema de tickets para asegurarse que las identidades están identificadas de forma segura. Se otorga un ticket a un usuario (pase o entrada) que me dan cuando me autentico. Con ese ticket puedo acceder a otros servicios del sistema sin tener que autenticarme de nuevo. El ticket tiene un tiempo de vida y va caducando eventualmente. 
	- Ejemplo real: Una empresa grande. Inicio sesión en mi ordenador por la mañana y de ahí puedo acceder a mi correo electrónico y carpetas compartidas de la red, etc.

- NETLOGON: es un servicio que ayuda que la comunicación con el servidor de dominio. Tanto KERBEROS como NTLM puede pasar por él para hacer uso del AD.

---
#### LSASS

Los archivos .dll son librerías.

| **Authentication Packages** | **Description**                                                                                                                                                                                                                                             |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Lsasrv.dll`                | El servicio Servidor LSA implementa políticas de seguridad y actúa como administrador de paquetes de seguridad para el LSA. El LSA contiene la función Negociar, que selecciona el protocolo NTLM o Kerberos tras determinar cuál es el protocolo adecuado. |
| `Msv1_0.dll`                | Paquete de autenticación para inicios de sesión en máquinas locales que no requieren autenticación personalizada.                                                                                                                                           |
| `Samsrv.dll`                | El Administrador de cuentas de seguridad (SAM) almacena cuentas de seguridad locales, aplica políticas almacenadas localmente y admite API.                                                                                                                 |
| `Kerberos.dll`              | Paquete de seguridad cargado por LSA para la autenticación basada en Kerberos en una máquina.                                                                                                                                                               |
| `Netlogon.dll`              | Servicio de inicio de sesión basado en red.                                                                                                                                                                                                                 |
| `Ntdsa.dll`                 | Esta biblioteca se utiliza para crear nuevos registros y carpetas en el registro de Windows.                                                                                                                                                                |
Cada sesión de inicio de sesión interactiva crea una instancia independiente del servicio WinLogon. La arquitectura de Identificación y Autenticación Gráfica (GINA) se carga en el área de proceso utilizada por WinLogon, recibe y procesa las credenciales e invoca las interfaces de autenticación mediante la función LSALogonUser.

---
#### Base de datos SAM
El Administrador de Cuentas de Seguridad (SAM) es un archivo de base de datos en los sistemas operativos Windows que almacena las credenciales de las cuentas de usuario. Se utiliza para autenticar usuarios locales y remotos y utiliza protecciones criptográficas para evitar el acceso no autorizado. Las contraseñas de usuario se almacenan como hashes en el registro, generalmente en forma de hashes LM o NTLM. El archivo SAM se encuentra en %SystemRoot%\system32\config\SAM y se monta en HKLM\SAM. Para ver o acceder a este archivo se requieren privilegios de nivel SISTEMA.

Los sistemas Windows pueden asignarse a un grupo de trabajo o a un dominio durante la configuración. Si el sistema se ha asignado a un grupo de trabajo, gestiona la base de datos SAM localmente y almacena todos los usuarios existentes en ella. Sin embargo, si el sistema se ha unido a un dominio, el Controlador de Dominio (DC) debe validar las credenciales de la base de datos de Active Directory (ntds.dit), que se almacena en %SystemRoot%\ntds.dit.

Para mejorar la protección contra el acceso no autorizado a la base de datos SAM, Microsoft introdujo una función en Windows NT 4.0 llamada SYSKEY (syskey.exe). Al estar habilitada, SYSKEY cifra parcialmente el archivo SAM en el disco, garantizando que los hashes de contraseñas de todas las cuentas locales se cifren con una clave generada por el sistema.

---
## Credential Manager
El Administrador de Credenciales es una función integrada en todos los sistemas operativos Windows que permite a los usuarios almacenar y administrar las credenciales utilizadas para acceder a recursos de red, sitios web y aplicaciones. Estas credenciales se guardan por perfil
de usuario en el Casillero de Credenciales. Las credenciales se cifran y se almacenan en la siguiente ubicación:
```
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```


![[Pasted image 20251009075459.png]]

---
### NTDS
Es muy común encontrar entornos de red donde los sistemas Windows están unidos a un dominio Windows. Esta configuración simplifica la administración centralizada, permitiendo a los administradores supervisar eficientemente todos los sistemas de su organización. En estos entornos, las solicitudes de inicio de sesión se envían a los controladores de dominio dentro del mismo bosque de Active Directory. Cada controlador de dominio aloja un archivo llamado NTDS.dit, que se sincroniza en todos los controladores de dominio, excepto en los controladores de dominio de solo lectura (RODC).

NTDS.dit es un archivo de base de datos que almacena datos de Active Directory, incluyendo, entre otros:
- Cuentas de usuario (hash de nombre de usuario y contraseña)
- Cuentas de grupo
- Cuentas de equipo
- Objetos de directiva de grupo

Más adelante en este módulo, exploraremos métodos para extraer credenciales del archivo NTDS.dit.

Ahora que hemos repasado los conceptos básicos del almacenamiento de credenciales, estudiemos los diversos ataques que podemos realizar para extraer credenciales y mejorar nuestro acceso durante las evaluaciones.