PHP (Hypertext Preprocessor) es un lenguaje de scripting open-source, ampliamente usado en servidores web.

Accediendo a una máquina ejemplo (no tenemos acceso, es hipotética), en la dirección:
```
https://10.129.201.101/login.php
```
Me encuentro un formulario php que requiere de usuario y contraseña.

Podemos utilizar PAYLOADS predefinidos para ganar acceso a traves de una *web shell* o *reverse shell*.

Trabajaremos con el siguiente repositorio:
>https://github.com/WhiteWinterWolf/wwwolf-php-webshell

Nos descargamos el código y lo copiamos en un archivo .php.

Nosotros vamos a subir una web shell a través del botón de "Browse" de la página web. Al subir dicha webshell.php, el servidor nos muestra un error conforme los archivos pemitidos solo son del tipo *.png .jpg .gif*.

Vamos a interceptar con *BURPSUITE* dicha subida de archivos.

Observamos, que al subir una dicho archivo, burpsuite detecta:
*Content-Type: application/x-php*

Haciendo una búsqueda rápida en google, observo como dicho valor de "Content Type" puede ser cambiado por otro. 
https://stackoverflow.com/questions/23714383/what-are-all-the-possible-values-for-http-content-type-header

Vamos a cambiar dicho texto por *image/gif*.  Al cambiarlo, dicho archivo puede ser subido al servidor. 
Accedemos al directorio: `/images/vendor/connect.php` y vemos como está disponible la webshell.

---
**QUESTIONS**
**Target: 10.129.206.192**
**1. In the example shown, what must the Content-Type be changed to in order to successfully upload the web shell? (Format: .../... )**
Respuesta: *image/gif*

**2. Use what you learned from the module to gain a web shell. What is the file name of the gif in the /images/vendor directory on the target? (Format: xxxx.gif)**
Ejercicio muy sencillo. 
Simplemente, nos descargamos la web-shell que nos indican, del repositorio: 
https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php

Accedemos a la máquina víctima, con credenciales admin:admin en el panel de autenticación. Accedemos al apartado Devices>Vendors (tarda 30s en cargar...). 
Nos deja subir un archivo pero con una extensión del tipo imagen. Abrimos Burpsuite e interceptamos dicha subida, cambiando el Content-Type a "image/gif". A esta imagen, le ponemos un nombre cualquiera, en mi caso, ha sido "pol", pero bueno, es un nombre de archivo que no servirá para nada.
Accedemos a la url:
https://10.129.206.192/images/vendor/webshell.php

Haciendo un simple "ls" desde la webshell, nos numera un conjunto de archivos, entre los cuales, me interesa el que acabar en .gif, llamado "ajax-loader.gif".
Respuesta: *ajax-loader.gif*

