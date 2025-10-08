John the Ripper (también conocido como JtR o john) es una  herramienta de pruebas de penetración que permite descifrar contraseñas mediante diversos ataques, como ataques de fuerza bruta y de diccionario. 
Se recomienda la versión **"JUMBO"** para nuestros usos, ya que ofrece optimizaciones de rendimiento, funciones adicionales como listas de palabras multilingües y compatibilidad con arquitecturas de 64 bits. 
Esta versión permite descifrar contraseñas con mayor precisión y velocidad. 
JtR incluye diversas herramientas para convertir diferentes tipos de archivos y hashes a formatos compatibles con JtR. 
Además, el software se actualiza periódicamente para mantenerse al día con las últimas tendencias y tecnologías de seguridad.

---
### SINGLE CRACK MODE

El modo de crackeo único genera candidatos a contraseñas a partir del nombre de usuario de la víctima, el nombre del directorio personal y los valores GECOS (nombre completo, número de habitación, número de teléfono, etc.).

Estas cadenas se comparan con un amplio conjunto de reglas que aplican modificaciones comunes de cadenas en las contraseñas (por ejemplo, un usuario cuyo nombre real es Bob Smith podría usar Smith1 como contraseña).

Imaginemos que, como atacantes, encontramos el archivo "passwd" con el siguiente contenido:

```
r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```
A partir del contenido del archivo, se puede inferir que la víctima tiene el nombre de usuario r0lf, el nombre real Rolf Sebastian y el directorio personal /home/r0lf. El modo de descifrado único utilizará esta información para generar contraseñas candidatas y compararlas con el hash.

Podemos ejecutar el ataque con el siguiente comando sobre el archivo "passwd" anterior:

```shell-session
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ john --single passwd
Created directory: /home/polika4r/.john
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
NAITSABES        (r0lf)     
1g 0:00:00:00 DONE (2025-10-06 14:54) 9.090g/s 4072p/s 4072c/s 4072C/s NAITSABESFL0R..rolfr0lffl0rflor
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

En este caso, la contraseña es "NAITSABES"  (que es SEBASTIAN escrito al revés).
Una vez que se descifra la contraseña, podríamos volver a verla sin operar cálculos con:

```
──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ john --show passwd
r0lf:NAITSABES:0:0:Rolf Sebastian:/home/r0lf:/bin/bash

```

--- 
## WORDLIST mode
El modo de lista de palabras se utiliza para descifrar contraseñas mediante un ataque de diccionario, lo que significa que intenta descifrar todas las contraseñas de una lista de palabras proporcionada contra el hash de la contraseña. La sintaxis básica del comando es la siguiente:

```shell-session
Polika4RM@htb[/htb]$ john --wordlist=<wordlist_file> <hash_file>
```
Si anteriormente ya crackeamos esta contraseña por el método "single", john no volverá a hacer fuerza bruta. 
Nos saldría este comentario:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt passwd
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
No password hashes left to crack (see FAQ)
```

Haríamos simplemente:
```
──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ john --show passwd
r0lf:NAITSABES:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```


---

## MODO INCREMENTAL
Es un modo de descifrado por fuerza bruta que no prueba combinaciones aleatorias ni toma palabras de una lista fija, sino que genera contraseñas candidatas según un modelo estadístico (cadenas de Markov).
Usa probabilidades aprendidas para construir con más prioridad las contraseñas que son más probables según los datos de entrenamiento, y luego prueba esas candidatas contra los hashes.

Es más eficiente que una fuerza bruta ingenua porque prueba primero las combinaciones más probables.
Pero sigue siendo muy exhaustivo y lento si el conjunto de caracteres o la longitud crecen mucho.

```shell-session
Polika4RM@htb[/htb]$ john --incremental <hash_file>
```

De forma predeterminada, johntheripper utiliza modos incrementales predefinidos, especificados en su archivo de configuración (john.conf), que definen los conjuntos de caracteres y la longitud de las contraseñas. 

---
## Identificando formato de Hash 

Hay veces que los Hashes se nos presentan de formas poco familiares y ni John The Ripper lo puede idientificar. 
Por ejemplo, el HASH: 
```
193069ceb0461e1d40d216e32c79c704
```

Esta página resume el formato de los Hashes más comunes: 
>https://pentestmonkey.net/cheat-sheet/john-the-ripper-hash-formats

Otra opción es usar una herramienta como hashID, que compara los hashes proporcionados con una lista integrada para sugerir posibles formatos. Al añadir la opción -j, hashID mostrará, además del formato hash, el formato JtR correspondiente:
```
┌──(polika4r㉿kali)-[~]
└─$ hashid -j 193069ceb0461e1d40d216e32c79c704
Analyzing '193069ceb0461e1d40d216e32c79c704'
[+] MD2 [JtR Format: md2]
[+] MD5 [JtR Format: raw-md5]
[+] MD4 [JtR Format: raw-md4]
[+] Double MD5 
[+] LM [JtR Format: lm]
[+] RIPEMD-128 [JtR Format: ripemd-128]
[+] Haval-128 [JtR Format: haval-128-4]
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 [JtR Format: lotus5]
[+] Skype 
[+] Snefru-128 [JtR Format: snefru-128]
[+] NTLM [JtR Format: nt]
[+] Domain Cached Credentials [JtR Format: mscach]
[+] Domain Cached Credentials 2 [JtR Format: mscach2]
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x [JtR Format: radmin]

```



Sin embargo, en este ejemplo aún no está claro el formato del hash. 
Muchas veces, el contexto de la procedencia del hash basta para fundamentar su formato. En este ejemplo específico, el formato del hash es RIPEMD-128.
JtR admite cientos de formatos de hash, algunos de los cuales se listan en la tabla a continuación. Se puede usar el argumento --format para indicar a JtR el formato de los hashes de destino.

  

| **Hash format**      | **Example command**                               | **Description**                                                      |
| -------------------- | ------------------------------------------------- | -------------------------------------------------------------------- |
| afs                  | `john --format=afs [...] <hash_file>`             | AFS (Andrew File System) password hashes                             |
| bfegg                | `john --format=bfegg [...] <hash_file>`           | bfegg hashes used in Eggdrop IRC bots                                |
| bf                   | `john --format=bf [...] <hash_file>`              | Blowfish-based crypt(3) hashes                                       |
| bsdi                 | `john --format=bsdi [...] <hash_file>`            | BSDi crypt(3) hashes                                                 |
| crypt(3)             | `john --format=crypt [...] <hash_file>`           | Traditional Unix crypt(3) hashes                                     |
| des                  | `john --format=des [...] <hash_file>`             | Traditional DES-based crypt(3) hashes                                |
| dmd5                 | `john --format=dmd5 [...] <hash_file>`            | DMD5 (Dragonfly BSD MD5) password hashes                             |
| dominosec            | `john --format=dominosec [...] <hash_file>`       | IBM Lotus Domino 6/7 password hashes                                 |
| EPiServer SID hashes | `john --format=episerver [...] <hash_file>`       | EPiServer SID (Security Identifier) password hashes                  |
| hdaa                 | `john --format=hdaa [...] <hash_file>`            | hdaa password hashes used in Openwall GNU/Linux                      |
| hmac-md5             | `john --format=hmac-md5 [...] <hash_file>`        | hmac-md5 password hashes                                             |
| hmailserver          | `john --format=hmailserver [...] <hash_file>`     | hmailserver password hashes                                          |
| ipb2                 | `john --format=ipb2 [...] <hash_file>`            | Invision Power Board 2 password hashes                               |
| krb4                 | `john --format=krb4 [...] <hash_file>`            | Kerberos 4 password hashes                                           |
| krb5                 | `john --format=krb5 [...] <hash_file>`            | Kerberos 5 password hashes                                           |
| LM                   | `john --format=LM [...] <hash_file>`              | LM (Lan Manager) password hashes                                     |
| lotus5               | `john --format=lotus5 [...] <hash_file>`          | Lotus Notes/Domino 5 password hashes                                 |
| mscash               | `john --format=mscash [...] <hash_file>`          | MS Cache password hashes                                             |
| mscash2              | `john --format=mscash2 [...] <hash_file>`         | MS Cache v2 password hashes                                          |
| mschapv2             | `john --format=mschapv2 [...] <hash_file>`        | MS CHAP v2 password hashes                                           |
| mskrb5               | `john --format=mskrb5 [...] <hash_file>`          | MS Kerberos 5 password hashes                                        |
| mssql05              | `john --format=mssql05 [...] <hash_file>`         | MS SQL 2005 password hashes                                          |
| mssql                | `john --format=mssql [...] <hash_file>`           | MS SQL password hashes                                               |
| mysql-fast           | `john --format=mysql-fast [...] <hash_file>`      | MySQL fast password hashes                                           |
| mysql                | `john --format=mysql [...] <hash_file>`           | MySQL password hashes                                                |
| mysql-sha1           | `john --format=mysql-sha1 [...] <hash_file>`      | MySQL SHA1 password hashes                                           |
| NETLM                | `john --format=netlm [...] <hash_file>`           | NETLM (NT LAN Manager) password hashes                               |
| NETLMv2              | `john --format=netlmv2 [...] <hash_file>`         | NETLMv2 (NT LAN Manager version 2) password hashes                   |
| NETNTLM              | `john --format=netntlm [...] <hash_file>`         | NETNTLM (NT LAN Manager) password hashes                             |
| NETNTLMv2            | `john --format=netntlmv2 [...] <hash_file>`       | NETNTLMv2 (NT LAN Manager version 2) password hashes                 |
| NEThalfLM            | `john --format=nethalflm [...] <hash_file>`       | NEThalfLM (NT LAN Manager) password hashes                           |
| md5ns                | `john --format=md5ns [...] <hash_file>`           | md5ns (MD5 namespace) password hashes                                |
| nsldap               | `john --format=nsldap [...] <hash_file>`          | nsldap (OpenLDAP SHA) password hashes                                |
| ssha                 | `john --format=ssha [...] <hash_file>`            | ssha (Salted SHA) password hashes                                    |
| NT                   | `john --format=nt [...] <hash_file>`              | NT (Windows NT) password hashes                                      |
| openssha             | `john --format=openssha [...] <hash_file>`        | OPENSSH private key password hashes                                  |
| oracle11             | `john --format=oracle11 [...] <hash_file>`        | Oracle 11 password hashes                                            |
| oracle               | `john --format=oracle [...] <hash_file>`          | Oracle password hashes                                               |
| pdf                  | `john --format=pdf [...] <hash_file>`             | PDF (Portable Document Format) password hashes                       |
| phpass-md5           | `john --format=phpass-md5 [...] <hash_file>`      | PHPass-MD5 (Portable PHP password hashing framework) password hashes |
| phps                 | `john --format=phps [...] <hash_file>`            | PHPS password hashes                                                 |
| pix-md5              | `john --format=pix-md5 [...] <hash_file>`         | Cisco PIX MD5 password hashes                                        |
| po                   | `john --format=po [...] <hash_file>`              | Po (Sybase SQL Anywhere) password hashes                             |
| rar                  | `john --format=rar [...] <hash_file>`             | RAR (WinRAR) password hashes                                         |
| raw-md4              | `john --format=raw-md4 [...] <hash_file>`         | Raw MD4 password hashes                                              |
| raw-md5              | `john --format=raw-md5 [...] <hash_file>`         | Raw MD5 password hashes                                              |
| raw-md5-unicode      | `john --format=raw-md5-unicode [...] <hash_file>` | Raw MD5 Unicode password hashes                                      |
| raw-sha1             | `john --format=raw-sha1 [...] <hash_file>`        | Raw SHA1 password hashes                                             |
| raw-sha224           | `john --format=raw-sha224 [...] <hash_file>`      | Raw SHA224 password hashes                                           |
| raw-sha256           | `john --format=raw-sha256 [...] <hash_file>`      | Raw SHA256 password hashes                                           |
| raw-sha384           | `john --format=raw-sha384 [...] <hash_file>`      | Raw SHA384 password hashes                                           |
| raw-sha512           | `john --format=raw-sha512 [...] <hash_file>`      | Raw SHA512 password hashes                                           |
| salted-sha           | `john --format=salted-sha [...] <hash_file>`      | Salted SHA password hashes                                           |
| sapb                 | `john --format=sapb [...] <hash_file>`            | SAP CODVN B (BCODE) password hashes                                  |
| sapg                 | `john --format=sapg [...] <hash_file>`            | SAP CODVN G (PASSCODE) password hashes                               |
| sha1-gen             | `john --format=sha1-gen [...] <hash_file>`        | Generic SHA1 password hashes                                         |
| skey                 | `john --format=skey [...] <hash_file>`            | S/Key (One-time password) hashes                                     |
| ssh                  | `john --format=ssh [...] <hash_file>`             | SSH (Secure Shell) password hashes                                   |
| sybasease            | `john --format=sybasease [...] <hash_file>`       | Sybase ASE password hashes                                           |
| xsha                 | `john --format=xsha [...] <hash_file>`            | xsha (Extended SHA) password hashes                                  |
| zip                  | `john --format=zip [...] <hash_file>`             | ZIP (WinZip) password hashes                                         |

---
## Crackeando archivos
También es posible descifrar archivos protegidos con contraseña o cifrados con JtR. JtR incluye varias herramientas "2john" que permiten procesar archivos y generar hashes compatibles con JtR. La sintaxis general de estas herramientas es:

La estructura es:
```shell-session
Polika4RM@htb[/htb]$ <tool> <file_to_crack> > file.hash
```

Siendo las posibles combinaciones: 

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
| ...SNIP...              | ...SNIP...                                    |


---
**1. Use single-crack mode to crack r0lf's password.**
Simplemente ejecuto: 
```
john --single passwd
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
No password hashes left to crack (see FAQ)
```

Y me debería dar el password. Pero como anteriormente hice un escaneo de fuerza bruta con el Wordlist mode, John no se pone a hacer fuerza bruta puesto que sería utilizar recursos de cálculo ya utizados anteriormente. 

En este caso, ejecutaremos:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ john -show passwd
r0lf:NAITSABES:0:0:Rolf Sebastian:/home/r0lf:/bin/bash

1 password hash cracked, 0 left

```

Dándonos el HASH crackeado.
Respuesta: "NAITSABES"

**2. Use wordlist-mode with rockyou.txt to crack the RIPEMD-128 password.**
Con:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ hashid -j 193069ceb0461e1d40d216e32c79c704
```

Debería identificar el tipo de HASH y el formato correspondiente para John, pero el enunciado y los apuntes ya me avanzan que se trata de un formato RIPEMD-128.

Pues, simplemente ejecuto:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ john --format=ripemd-128 --wordlist=/usr/share/wordlists/rockyou.txt pass
Using default input encoding: UTF-8
Loaded 1 password hash (ripemd-128, RIPEMD 128 [32/64])
Warning: no OpenMP support for this hash type, consider --fork=8
Press 'q' or Ctrl-C to abort, almost any other key for status
50cent           (?)     
1g 0:00:00:00 DONE (2025-10-06 17:14) 100.0g/s 32000p/s 32000c/s 32000C/s angelo..101010
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

Obteniendo la 
Respuesta: "50cent"
