Para borrar HASHes previamente calculados se deberá acceder a la ruta: 
>home/polika4r/.local/share/hashcat/hashcat.potfile

En muchas ocasiones, hay imposiciones por parte de las empresas para generar contraseñas con un determinado número de carácteres, mayúsculas, minúsculas, números y carácteres especiales. 
En el momento que te imponen hacer esto, muchas personas optan por la "simplicidad" antes que la "seguridad", pues, escogen contraseñas relacionadas normalmente con su organización.

Por ejemplo, podrían poner *ST3_3ng1ph4rm!*  para la empresa *STE ENGIPHARM*.

Las normas, entre otras muchas, se pueden resumir a:
```
:      # do nothing
l      # lowercase
u      # uppercase
c      # capitalize (Primera letra mayúscula, resto minúsculas)
s@4    # reemplaza todas las @ por 4
s a 4  # alternativa: reemplaza "a" por "4" (forma legible)
$!     # añade '!' al final
^1     # inserta '1' al inicio (prefijo)
$1     # añade '1' al final (sufijo)

```

Generaré un archivo custom.rules que contenga una serie de normas:
```shell-session
Polika4RM@htb[/htb]$ cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

Generaremos una lista de palabras patrones, en un archivo llamado password.list (en este caso, solo tendrá una entrada):
```shell-session
Polika4RM@htb[/htb]$ cat password.list

password
```

Podemos utilizar el siguiente comando para aplicar las normas a custom.rules para la palabra patrón "password":

```
Polika4RM@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

Y nos generará el siguiente contenido:
```shell-session
Polika4RM@htb[/htb]$ cat mut_password.list

password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
Password!
passw0rd!
p@ssword!
Passw0rd!
P@ssword!
p@ssw0rd!
P@ssw0rd!
```

---
## Exercise

For this sections exercise, imagine that we compromised the password hash of a `work email` belonging to `Mark White`. After performing a bit of OSINT, we have gathered the following information about Mark:

- He was born on `August 5, 1998`
- He works at `Nexura, Ltd.`
    - The company's password policy requires passwords to be at least 12 characters long, to contain at least one uppercase letter, at least one lowercase letter, at least one symbol and at least one number
- He lives in `San Francisco, CA, USA`
- He has a pet cat named `Bella`
- He has a wife named `Maria`
- He has a son named `Alex`
- He is a big fan of `baseball`

**1. What is Mark's password?**

Generamos una lista con palabras madres, llamada passwords.list, agregando el contenido:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ cat password.list 
august
5
1998
nexura
ltd
san francisco
ca
usa
bella
maria
alex
baseball
Mark1998Maria!
Maria0805Alex!
Bella0805White!
SanFrancisco98!
Nexura!1998
baseball1998!
MariaAlexBella!
NexuraBella0805!
NexuraBaseball1998!
NexuraMariaAlex!
Nexura08051998!
M@ria0805!
M@rk1998!
M@rkWh!te1998
```

Generamos unas rules, en custom.rules, con el siguiente contenido:
```
:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

Generamos la lista personalizada: 
```
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

Y atacamos el Hash contra esa lista:
```
hashcat -a 0 -m 0 97268a8ae45ac7d15c3cea4ce6ea550b mut_password.list               
```

Siendo la respuesta del ejercicio: *Baseball1998!*


