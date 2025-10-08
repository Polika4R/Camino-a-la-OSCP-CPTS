El proceso de autenticación del cliente de Windows involucra múltiples módulos responsables del inicio de sesión, la recuperación de credenciales y la verificación. Entre los diversos mecanismos de autenticación de Windows, Kerberos es uno de los más utilizados y complejos. La Autoridad de Seguridad Local (LSA) es un subsistema protegido que autentica a los usuarios, administra los inicios de sesión locales, supervisa todos los aspectos de la seguridad local y proporciona servicios de traducción entre nombres de usuario e identificadores de seguridad (SID).

El subsistema de seguridad mantiene las políticas de seguridad y las cuentas de usuario en un sistema informático. En un controlador de dominio, estas políticas y cuentas se aplican a todo el dominio y se almacenan en Active Directory. Además, el subsistema LSA proporciona servicios de control de acceso, comprobación de permisos y generación de mensajes de auditoría de seguridad.

### Flujo de inicio de sesión (resumen paso a paso)
![[Pasted image 20251008115447.png]]
El inicio de sesión interactivo local se gestiona mediante la coordinación de varios componentes: el proceso de inicio de sesión (WinLogon), el proceso de interfaz de usuario de inicio de sesión (LogonUI), los proveedores de credenciales, el Servicio de Subsistema de Autoridad de Seguridad Local (LSASS), uno o más paquetes de autenticación y el Administrador de Cuentas de Seguridad (SAM) o Active Directory. En este contexto, los paquetes de autenticación son bibliotecas de vínculos dinámicos (DLL) responsables de realizar las comprobaciones de autenticación. Por ejemplo, para inicios de sesión interactivos y no vinculados a un dominio, se suele utilizar el paquete de autenticación Msv1_0.dll.

WinLogon es un proceso de sistema de confianza responsable de gestionar las interacciones del usuario relacionadas con la seguridad, como:
- Iniciar LogonUI para solicitar credenciales al iniciar sesión
- Gestionar cambios de contraseña
- Bloquear y desbloquear la estación de trabajo

Para obtener el nombre de usuario y la contraseña, WinLogon utiliza proveedores de credenciales instalados en el sistema. Estos proveedores de credenciales son objetos COM implementados como DLL.

WinLogon es el único proceso que intercepta las solicitudes de inicio de sesión desde el teclado, enviadas mediante mensajes RPC desde Win32k.sys. Al iniciar sesión, inicia inmediatamente la aplicación LogonUI para presentar la interfaz gráfica de usuario. Una vez que el proveedor de credenciales recopila las credenciales del usuario, WinLogon las envía al Servicio del Subsistema de Autoridad de Seguridad Local (LSASS) para autenticar al usuario.

### LSASS
El Servicio del Subsistema de Autoridad de Seguridad Local (LSASS) consta de varios módulos y gestiona todos los procesos de autenticación. Ubicado en %SystemRoot%\System32\Lsass.exe en el sistema de archivos, se encarga de aplicar la política de seguridad local, autenticar usuarios y reenviar registros de auditoría de seguridad al Registro de Eventos. En esencia, **LSASS actúa como el guardián en los sistemas operativos Windows**. 

| **Authentication Packages** | **Description**                                                                                                                                                                                                                                              |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Lsasrv.dll`                | El servicio del servidor LSA aplica las políticas de seguridad y actúa como administrador de paquetes de seguridad para el LSA. El LSA contiene la función Negotiate, que selecciona el protocolo NTLM o Kerberos tras determinar cuál debe ser el correcto. |
| `Msv1_0.dll`                | Paquete de autenticación para inicios de sesión en equipos locales que no requieren autenticación personalizada.<br>                                                                                                                                         |
| `Samsrv.dll`                | El Administrador de cuentas de seguridad (SAM) almacena cuentas de seguridad locales, aplica políticas almacenadas localmente y admite API.                                                                                                                  |
| `Kerberos.dll`              | Paquete de seguridad cargado por el LSA para la autenticación basada en Kerberos en un equipo.                                                                                                                                                               |
| `Netlogon.dll`              | Servicio de inicio de sesión basado en red.                                                                                                                                                                                                                  |
| `Ntdsa.dll`                 | Esta biblioteca se utiliza para crear nuevos registros y carpetas en el registro de Windows.                                                                                                                                                                 |

## SAM DATABASES

El Administrador de Cuentas de Seguridad (SAM) es un archivo de base de datos en los sistemas operativos Windows que almacena las credenciales de las cuentas de usuario. Se utiliza para autenticar usuarios locales y remotos y utiliza protecciones criptográficas para evitar el acceso no autorizado. Las contraseñas de usuario se almacenan como hashes en el registro, generalmente en forma de hashes LM o NTLM. El archivo SAM se encuentra en %SystemRoot%\system32\config\SAM y se monta en HKLM\SAM. Para ver o acceder a este archivo se requieren privilegios de nivel SISTEMA.

Los sistemas Windows pueden asignarse a un grupo de trabajo o a un dominio durante la configuración. Si el sistema se ha asignado a un grupo de trabajo, gestiona la base de datos SAM localmente y almacena todos los usuarios existentes en ella. Sin embargo, si el sistema se ha unido a un dominio, el Controlador de Dominio (DC) debe validar las credenciales de la base de datos de Active Directory (ntds.dit), que se almacena en %SystemRoot%\ntds.dit.

Para mejorar la protección contra el acceso no autorizado a la base de datos SAM, Microsoft introdujo una función en Windows NT 4.0 llamada SYSKEY (syskey.exe). Al estar habilitada, SYSKEY cifra parcialmente el archivo SAM en el disco, garantizando que los hashes de contraseñas de todas las cuentas locales se cifren con una clave generada por el sistema.

### NTDS
Es muy común encontrar entornos de red donde los sistemas Windows están unidos a un dominio Windows. Esta configuración simplifica la administración centralizada, permitiendo a los administradores supervisar eficientemente todos los sistemas de su organización. En estos entornos, las solicitudes de inicio de sesión se envían a los controladores de dominio dentro del mismo bosque de Active Directory. Cada controlador de dominio aloja un archivo llamado NTDS.dit, que se sincroniza en todos los controladores de dominio, excepto en los controladores de dominio de solo lectura (RODC).

NTDS.dit es un archivo de base de datos que almacena datos de Active Directory, incluyendo, entre otros:

- Cuentas de usuario (hash de nombre de usuario y contraseña)
- Cuentas de grupo
- Cuentas de equipo
- Objetos de directiva de grupo
Más adelante en este módulo, exploraremos métodos para extraer credenciales del archivo NTDS.dit.

Ahora que hemos repasado los conceptos básicos del almacenamiento de credenciales, estudiemos los diversos ataques que podemos realizar para extraer credenciales y mejorar nuestro acceso durante las evaluaciones.

