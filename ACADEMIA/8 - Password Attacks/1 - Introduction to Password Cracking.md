
Ejemplos de funciones HASHES típicas son "md5" y "SHA-256".
En una terminal Kali, podemos encriptar un string de la siguiente forma:

```
┌──(polika4r㉿kali)-[~]
└─$ echo -n polika | md5sum 
f7f395d9e793b280dd6ac3625ecc7dab  -
```

El "-n" indica que no añada un salto de línea al final de la línea, pues, la salida de texto sería diferente:
```
┌──(polika4r㉿kali)-[~]
└─$ echo -n polika | md5sum
f7f395d9e793b280dd6ac3625ecc7dab  -

┌──(polika4r㉿kali)-[~]
└─$ echo polika | md5sum 
1009b47a8ccec3569205b1b862be064b  -

```

También podemos encriptar mediante el algoritmo SHA-256 con:
```
┌──(polika4r㉿kali)-[~]
└─$ echo -n polika | sha256sum
187b13cb3f4ddd7e085ddf4a63b90f05d881915421233253dcecf5bcea76e5c5  -

```


---
### Conceptos claves:
- **Rainbow tables**:Son tablas precomputadas que asocian contraseñas comunes con sus hashes para permitir búsquedas muy rápidas. Si la contraseña ya está en la tabla, su hash se puede **revertir** (identificar la contraseña) casi instantáneamente. Algunos ejemplos de RAINBOW TABLE:

| Password  | MD5 Hash                         |
| --------- | -------------------------------- |
| 123456    | e10adc3949ba59abbe56e057f20f883e |
| 12345     | 827ccb0eea8a706c4c34a16891f84e7b |
| 123456789 | 25f9e794323b453885f5181f1b624d0b |
| password  | 5f4dcc3b5aa765d61d8327deb882cf99 |
| iloveyou  | f25a2fc72690b780b2a14e140ef6a9e0 |
Un _salt_ es un texto aleatorio que se añade a la contraseña **antes** de crear el hash.  
Por ejemplo:
- Contraseña: `password`
- Salt: `Th1sIsTh3S@lt_`
- Lo que realmente se hashea: `Th1sIsTh3S@lt_password`

Si cada usuario tiene un _salt_ distinto, no sirve una rainbow table genérica, porque para cada salt habría que hacer una tabla nueva (y eso es imposible de escalar).

---
## Ataques por fuerza bruta
En un **brute-force** NO se generan “hashes aleatorios” y luego se intenta sacar la contraseña de esos hashes. Se **generan candidatos en texto plano** (pueden ser combinaciones sistemáticas o “legibles” según la estrategia), se **aplica la función de hash** a cada candidato y se **compara** el resultado con el hash objetivo. Si coinciden, encontraste la contraseña.

---
## Ataques por diccionario
Un ataque de diccionario, también conocido como ataque de lista de palabras, es una de las técnicas más eficientes para descifrar contraseñas, especialmente cuando se opera con limitaciones de tiempo, como suelen hacer los expertos en pruebas de penetración. En lugar de probar todas las combinaciones de caracteres posibles, se utiliza una lista con las contraseñas estadísticamente probables. Las listas de palabras más conocidas para descifrar contraseñas son rockyou.txt y las incluidas en SecLists.

Un diccionario muy utilizado es el "rockyou":
```shell-session
Polika4RM@htb[/htb]$ head --lines=20 /usr/share/wordlists/rockyou.txt 

123456
12345
123456789
password
iloveyou
...
michael
ashley
qwerty
```

---
**1. What is the SHA1 hash for `Academy#2025`?**
Ejecuto un:
```
┌──(polika4r㉿kali)-[~]
└─$ echo -n Academy#2025 | sha1sum
750fe4b402dc9f91cedf09b652543cd85406be8c  -
```
Nota importante!   Pongo "-n" para eliminar el salto de línea, de lo contrario, el HASH sería otro.

Y obtengo la respuesta: "750fe4b402dc9f91cedf09b652543cd85406be8c"

