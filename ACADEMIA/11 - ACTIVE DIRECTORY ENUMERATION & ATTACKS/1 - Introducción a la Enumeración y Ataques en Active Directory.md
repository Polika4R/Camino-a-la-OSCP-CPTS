

## Active Directory (AD) Explicado

- **Definición:** Servicio de directorio jerárquico y distribuido para entornos empresariales de **Windows** (implementado desde Windows Server 2000).
- **Base:** Protocolos **x.500** y **LDAP**.
- **Función Clave:** Permite la **gestión centralizada** de recursos de la organización (usuarios, equipos, grupos, políticas, dispositivos, etc.).
- **Servicios Esenciales:** Proporciona funciones de **autenticación, contabilidad** (accounting) y **autorización**.

## Importancia de Active Directory

- **Dominio de Mercado:** AD tiene cerca del **43%** de la cuota de mercado en soluciones de **Gestión de Identidad y Acceso (IAM)** empresariales, y sigue evolucionando con Azure AD.
- **Riesgos de Seguridad:** Su gran tamaño y propósito de facilitar el acceso a la información lo hacen complejo de gestionar y proteger correctamente.
- **Vulnerabilidades:** Las **malas configuraciones** de permisos y servicios son una fuente común de vulnerabilidades y explotación.
- **Objetivo del Atacante:** El objetivo principal es **escalar privilegios** (movimiento lateral o vertical) hasta lograr el objetivo del compromiso, que a menudo es el **compromiso total del dominio** (acceso a nivel _Domain Admin_).
## Enfoque del Módulo y Tácticas

- **Enfoque:** Explorar problemas comunes, identificar, enumerar y aprovechar vulnerabilidades y malas configuraciones.
- **Herramientas y Técnicas a Practicar:**
    - **Enumeración:** Herramientas nativas como **Sysinternals, WMI, DNS**, entre otras.
    - **Ataques:** **Password Spraying, Kerberoasting, Pass-the-Ticket, DCSync, Shadow Credentials.**
    - **Utilidades:** **Responder, Kerbrute, Bloodhound, Rubeus, DomainPasswordSpray.**
- **Mentalidad Clave:** Ser capaz de enumerar y atacar desde **Windows** y **Linux**, con un conjunto de herramientas limitado o integrado ("**living off the land**").
- **Principio Fundamental:** Comprender el "**porqué**" detrás de las fallas y malas configuraciones para ser un atacante más efectivo y poder ofrecer mejores recomendaciones de remediación.

## Ejemplos del mundo real:

```
Escenario 1:
Durante la prueba de penetración, comprometí un único host y obtuve acceso a nivel SYSTEM. Dado que este era un host unido al dominio, pude usar este acceso para enumerar el dominio. Realicé toda la enumeración estándar, pero no encontré mucho. Había Nombres Principales de Servicio (SPNs) presentes en el entorno, y pude realizar un ataque de Kerberoasting y recuperar tickets TGS para algunas cuentas. Intenté crackearlos con Hashcat y algunas de mis listas de palabras y reglas estándar, pero al principio no tuve éxito. Terminé dejando un trabajo de crackeo ejecutándose durante la noche con una lista de palabras muy grande combinada con la regla d3ad0ne que viene con Hashcat. A la mañana siguiente, tuve un acierto en un ticket y recuperé la contraseña en texto claro de una cuenta de usuario. Esta cuenta no me dio un acceso significativo, pero sí me otorgó acceso de escritura en ciertos recursos compartidos de archivos. Utilicé este acceso para dejar caer archivos SCF alrededor de los recursos compartidos y dejé Responder ejecutándose. Después de un tiempo, obtuve un único acierto: el hash NetNTLMv2 de un usuario. Revisé la salida de BloodHound y noté que ¡este usuario era en realidad un administrador de dominio! Un día fácil a partir de ahí.
```
## Entornos de Práctica (Ejemplos)

| **Host**     | **Sistema Operativo** | **Acceso Principal** | **Propósito**                                                         |
| ------------ | --------------------- | -------------------- | --------------------------------------------------------------------- |
| **MS01**     | Windows               | RDP                  | Enumeración y ataque desde un host Windows.                           |
| **ATTACK01** | Parrot Linux          | SSH / Xfreerdp (GUI) | Enumeración y ataque desde un host Linux (Ej: uso de BloodHound GUI). |

- **Conexión RDP (Ej. a MS01/ATTACK01):**
```
xfreerdp /v:<IP_TARGET> /u:htb-student /p:<PASSWORD>
```

- **Conexión SSH (Ej. a ATTACK01):**
```
ssh htb-student@<IP_TARGET>
```
    
- **Toolkit:** Herramientas necesarias preinstaladas: `C:\Tools` en Windows (`MS01`) y `/opt` o en el PATH en Linux (`ATTACK01`). Se recomienda compilar herramientas propias para la práctica en entornos reales.