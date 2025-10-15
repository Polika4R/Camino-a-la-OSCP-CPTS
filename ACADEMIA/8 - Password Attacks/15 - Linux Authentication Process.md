Las distribuciones basadas en Linux admiten diversos mecanismos de autenticación. 
Uno de los más utilizados son los Módulos de Autenticación Conectables (PAM):

Los módulos responsables de esta funcionalidad, como pam_unix.so o pam_unix2.so, suelen ubicarse en /usr/lib/x86_64-linux-gnu/security/ en sistemas basados ​​en Debian. 
Estos módulos gestionan la información del usuario, la autenticación, las sesiones y los cambios de contraseña. 
Por ejemplo, cuando un usuario cambia su contraseña con el comando passwd, se invoca PAM, que toma las precauciones necesarias para gestionar y almacenar la información correctamente.

El módulo pam_unix.so utiliza llamadas API estandarizadas de las bibliotecas del sistema para actualizar la información de la cuenta. Los archivos principales que lee y escribe son:
>**/etc/passwd y /etc/shadow.** 

PAM también incluye muchos otros módulos de servicio, como los de LDAP, operaciones de montaje y autenticación Kerberos.

---
## Passwd file
El archivo /etc/passwd contiene información sobre cada usuario del sistema y es legible para todos los usuarios y servicios. Cada entrada del archivo corresponde a un solo usuario y consta de siete campos que almacenan datos relacionados con el usuario en un formato estructurado. Estos campos se separan con dos puntos (:). Por lo tanto, una entrada típica podría tener un aspecto similar a este:
```
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

|Field|Value|
|---|---|
|Username|`htb-student`|
|Password|`x`|
|User ID|`1000`|
|Group ID|`1000`|
|[GECOS](https://en.wikipedia.org/wiki/Gecos_field)|`,,,`|
|Home directory|`/home/htb-student`|
|Default shell|`/bin/bash`|
El campo más relevante para nuestros propósitos es el campo "Contraseña", ya que puede contener diferentes tipos de entradas. En casos excepcionales (generalmente en sistemas muy antiguos), este campo puede contener el hash de la contraseña. Sin embargo, en sistemas modernos, los hashes de las contraseñas se almacenan en el archivo /etc/shadow, que examinaremos más adelante. A pesar de esto, el archivo /etc/passwd es legible para todo el mundo, lo que permite a los atacantes descifrar las contraseñas si los hashes se almacenan aquí.

Normalmente, encontraremos el valor x en este campo, lo que indica que las contraseñas se almacenan en formato hash dentro del archivo /etc/shadow. Sin embargo, también puede ocurrir que el archivo /etc/passwd sea escribible por error. Esto nos permitiría eliminar por completo el campo de contraseña del usuario root.
```shell-session
Polika4RM@htb[/htb]$ head -n 1 /etc/passwd

root::0:0:root:/root:/bin/bash
```

This results in no password prompt being displayed when attempting to log in as `root`:
```shell-session
Polika4RM@htb[/htb]$ su

root@htb[/htb]#
```

## Shadow file
Dado que leer los valores hash de las contraseñas puede poner en riesgo todo el sistema, se introdujo el archivo /etc/shadow. Tiene un formato similar a /etc/passwd, pero es el único responsable del almacenamiento y la gestión de contraseñas. Contiene toda la información de contraseñas de los usuarios creados. Por ejemplo, si no hay ninguna entrada en el archivo /etc/shadow para un usuario que figura en /etc/passwd, dicho usuario se considera inválido. El archivo /etc/shadow solo es legible para usuarios con privilegios administrativos. El formato de este archivo se divide en los siguientes nueve campos:

```shell-session
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

|Field|Value|
|---|---|
|Username|`htb-student`|
|Password|`$y$j9T$3QSBB6CbHEu...SNIP...f8Ms`|
|Last change|`18955`|
|Min age|`0`|
|Max age|`99999`|
|Warning period|`7`|
|Inactivity period|`-`|
|Expiration date|`-`|
|Reserved field|`-`|
Si el campo Contraseña contiene un carácter como ! o \*, el usuario no puede iniciar sesión con una contraseña Unix. Sin embargo, se pueden usar otros métodos de autenticación, como Kerberos o la autenticación basada en claves. Lo mismo ocurre si el campo Contraseña está vacío, lo que significa que no se requiere contraseña para iniciar sesión. Esto puede provocar que algunos programas denieguen el acceso a funciones específicas. El campo Contraseña también sigue un formato específico, del cual podemos extraer información adicional:

- `$<id>$<salt>$<hashed>`

Como podemos ver aquí, las contraseñas hash se dividen en tres partes. El valor del ID especifica el algoritmo hash criptográfico utilizado, generalmente uno de los siguientes:
  
|ID|Cryptographic Hash Algorithm|
|---|---|
|`1`|[MD5](https://en.wikipedia.org/wiki/MD5)|
|`2a`|[Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))|
|`5`|[SHA-256](https://en.wikipedia.org/wiki/SHA-2)|
|`6`|[SHA-512](https://en.wikipedia.org/wiki/SHA-2)|
|`sha1`|[SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)|
|`y`|[Yescrypt](https://github.com/openwall/yescrypt)|
|`gy`|[Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1)|
|`7`|[Scrypt](https://en.wikipedia.org/wiki/Scrypt)|
Muchas distribuciones de Linux, incluyendo Debian, ahora usan yescrypt como algoritmo de hash predeterminado. Sin embargo, en sistemas más antiguos, aún podemos encontrar otros métodos de hash que podrían ser descifrados. En breve, explicaremos cómo funciona el proceso de descifrado.


Sobre el ejemplo anterior:
```
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

- id = y
- salt = j9T
- hashed = 3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::

---
## Opasswd
La biblioteca PAM (pam_unix.so) puede impedir que los usuarios reutilicen contraseñas antiguas. Estas contraseñas se almacenan en el archivo /etc/security/opasswd. Se requieren privilegios de administrador (root) para leer este archivo, siempre que sus permisos no se hayan modificado manualmente.

```shell-session
Polika4RM@htb[/htb]$ sudo cat /etc/security/opasswd

cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1
```

Al observar el contenido de este archivo, podemos ver que contiene varias entradas para el usuario cry0l1t3, separadas por una coma (,). Un detalle crucial a tener en cuenta es el tipo de hash utilizado. Esto se debe a que el algoritmo MD5 ($1$) es mucho más fácil de descifrar que el SHA-512. Esto es especialmente importante para identificar contraseñas antiguas y reconocer patrones, ya que los usuarios suelen reutilizar contraseñas similares en varios servicios o aplicaciones. Reconocer estos patrones puede mejorar considerablemente nuestras posibilidades de acertar la contraseña.
## Cracking Credenciales de Linux 
Una vez que tengamos acceso root a una máquina Linux, podemos recopilar los hashes de las contraseñas de los usuarios e intentar descifrarlos mediante diversos métodos para recuperar las contraseñas en texto plano. Para ello, podemos usar una herramienta llamada unshadow, incluida con John the Ripper (JtR). Funciona combinando los archivos passwd y shadow en un solo archivo apto para descifrar.

```shell-session
Polika4RM@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
Polika4RM@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
Polika4RM@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

Este archivo "unshadowed.hashes" ahora puede ser atacado con JtR o hashcat.

```shell-session
Polika4RM@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

---
**1. Download the attached ZIP file (linux-authentication-process.zip), and use single crack mode to find martin's password. What is it?**

Me descargo el contenido de los zips y los descomprimo con gzip.
Me quedan dos archivos, el "passwd" y el "shadow".

Los uno con el comando:
```
unshadow passwd shadow > unshadowed.hashes
```

Realizo un grep para filtrar por el HASH de Martin:
```
cat unshadowed.hashes| grep martin
martin:$6$0XiU8Oe/pGpxWvdq$n6TgiYUVAXBUOO11C155Ea8nNpSVtFFVQveY6yExlOdPu99hY4V9Chi1KEy/lAluVFuVcvi8QCO1mCG6ra70A1:1000:1000:Martin Mendes:/home/martin:/usr/bin/zsh
```

Y me guardo dicha salida en:
```
cat martin.hash                   
martin:$6$0XiU8Oe/pGpxWvdq$n6TgiYUVAXBUOO11C155Ea8nNpSVtFFVQveY6yExlOdPu99hY4V9Chi1KEy/lAluVFuVcvi8QCO1mCG6ra70A1:1000:1000:Martin Mendes:/home/martin:/usr/bin/zsh
```

Ejecuto finalmente
```
 hashcat -m 1800 -a 0 martin.hash /usr/share/wordlists/rockyou.txt -o unshadowed.cracked 
```

Y me devuelve en el documento "unshadow.cracked"
```
cat unshadowed.cracked 
$6$0XiU8Oe/pGpxWvdq$n6TgiYUVAXBUOO11C155Ea8nNpSVtFFVQveY6yExlOdPu99hY4V9Chi1KEy/lAluVFuVcvi8QCO1mCG6ra70A1:Martin1
```
Respuesta: Martin1

**2. Use a wordlist attack to find sarah's password. What is it?**:

Realizo exactamente el mismo procedimiento pero grepeando "sarah"
```
┌──(polika4r㉿kali)-[~/Descargas]
└─$ cat unshadowed.hashes| grep "sarah"                        
sarah:$6$EBOM5vJAV1TPvrdP$LqsLyYkoGzAGt4ihyvfhvBrrGpVjV976B3dEubi9i95P5cDx1U6BrE9G020PWuaeI6JSNaIDIbn43uskRDG0U/:1001:1001:Sarah Saragaday:/home/sarah:/usr/bin/bash
                                                  
┌──(polika4r㉿kali)-[~/Descargas]
└─$ cat sarah.hash                     
sarah:$6$EBOM5vJAV1TPvrdP$LqsLyYkoGzAGt4ihyvfhvBrrGpVjV976B3dEubi9i95P5cDx1U6BrE9G020PWuaeI6JSNaIDIbn43uskRDG0U/:1001:1001:Sarah Saragaday:/home/sarah:/usr/bin/bash
┌──(polika4r㉿kali)-[~/Descargas]
└─$ hashcat -m 1800 -a 0 sarah.hash /usr/share/wordlists/rockyou.txt -o sara.cracked      
hashcat (v6.2.6) starting

```

Obteniendo como respuesta:
```
cat sara.cracked 
$6$EBOM5vJAV1TPvrdP$LqsLyYkoGzAGt4ihyvfhvBrrGpVjV976B3dEubi9i95P5cDx1U6BrE9G020PWuaeI6JSNaIDIbn43uskRDG0U/:mariposa
```
Respuesta: mariposa