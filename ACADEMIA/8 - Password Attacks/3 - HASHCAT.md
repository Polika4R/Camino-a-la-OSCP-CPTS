Para borrar HASHes previamente calculados se deberá acceder a la ruta: 
>home/polika4r/.local/share/hashcat/hashcat.potfile

Un punto fuerte de HASHCAT es que utiliza GPU para crackear.

La sintaxis general es:
```shell-session
Polika4RM@htb[/htb]$ hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]
```

- **-a**: se utiliza para especificar em odo de ataque. Los explicamos al final de esta hoja.
- **-m**: se utiliza para especificar el tipo de Hash
- **\<hashes>**: se utiliza para especificar la string o el archivo del HASH. 


**Tipos de HASH:**
HASHCAT soporta cientos de tipos de hashes, asociados cada uno a un ID.
En la siguiente página web ser resumen todos los existentes: 
>https://hashcat.net/wiki/doku.php?id=example_hashes

Para identificar el tipo de HASH, podremos realizar un:
```shell-session
Polika4RM@htb[/htb]$ hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'

Analyzing '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
[+] MD5 Crypt [Hashcat Mode: 500]
[+] Cisco-IOS(MD5) [Hashcat Mode: 500]
[+] FreeBSD MD5 [Hashcat Mode: 500]
```

---
## Modos de ataques: 
Principalmente utilizaremos dos modos de ataques concretos:
- **-a: Modo diccionario**
	- Utiliza un diccionario para crackear el hash. 
	- Como input necessita 
		- el tipo de HASH (-m): el cual se puede consultar mediante "hashid -j \<hash>"
		- El modo de ataque (-a 0; que es el modo diccionario).
		- El diccionario: /usr/share/wordlist/rockyou
		- El archivo o hash
		Ejemplo:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ hashcat -a 0 -m 0 hash /usr/share/wordlists/rockyou.txt 

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

e3e3ec5831ad5e7288241960e5d4fdb8:crazy!       <----------- hash crackeado                                                        
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: e3e3ec5831ad5e7288241960e5d4fdb8
Time.Started.....: Mon Oct  6 17:30:24 2025 (0 secs)
Time.Estimated...: Mon Oct  6 17:30:24 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   488.3 kH/s (0.13ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 28672/14344385 (0.20%)
Rejected.........: 0/28672 (0.00%)
Restore.Point....: 24576/14344385 (0.17%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: 280690 -> spongebob9
Hardware.Mon.#1..: Util: 10%

Started: Mon Oct  6 17:29:57 2025
Stopped: Mon Oct  6 17:30:26 2025
```

A esto, le podemos añadir "rules" para generar variantes del diccionario que estamos utilizando:
Estas rules se encuentran en:
```
Polika4RM@htb[/htb]$ ls -l /usr/share/hashcat/rules
```

Existen todas estas:
```shell-session
Polika4RM@htb[/htb]$ ls -l /usr/share/hashcat/rules

total 2852
-rw-r--r-- 1 root root 309439 Apr 24  2024 Incisive-leetspeak.rule
-rw-r--r-- 1 root root  35802 Apr 24  2024 InsidePro-HashManager.rule
-rw-r--r-- 1 root root  20580 Apr 24  2024 InsidePro-PasswordsPro.rule
-rw-r--r-- 1 root root  64068 Apr 24  2024 T0XlC-insert_00-99_1950-2050_toprules_0_F.rule
-rw-r--r-- 1 root root   2027 Apr 24  2024 T0XlC-insert_space_and_special_0_F.rule
-rw-r--r-- 1 root root  34437 Apr 24  2024 T0XlC-insert_top_100_passwords_1_G.rule
-rw-r--r-- 1 root root  34813 Apr 24  2024 T0XlC.rule
-rw-r--r-- 1 root root   1289 Apr 24  2024 T0XlC_3_rule.rule
-rw-r--r-- 1 root root 168700 Apr 24  2024 T0XlC_insert_HTML_entities_0_Z.rule
-rw-r--r-- 1 root root 197418 Apr 24  2024 T0XlCv2.rule
-rw-r--r-- 1 root root    933 Apr 24  2024 best64.rule
-rw-r--r-- 1 root root    754 Apr 24  2024 combinator.rule
-rw-r--r-- 1 root root 200739 Apr 24  2024 d3ad0ne.rule
-rw-r--r-- 1 root root 788063 Apr 24  2024 dive.rule
-rw-r--r-- 1 root root  78068 Apr 24  2024 generated.rule
-rw-r--r-- 1 root root 483425 Apr 24  2024 generated2.rule
drwxr-xr-x 2 root root   4096 Oct 19 15:30 hybrid
-rw-r--r-- 1 root root    298 Apr 24  2024 leetspeak.rule
-rw-r--r-- 1 root root   1280 Apr 24  2024 oscommerce.rule
-rw-r--r-- 1 root root 301161 Apr 24  2024 rockyou-30000.rule
-rw-r--r-- 1 root root   1563 Apr 24  2024 specific.rule
-rw-r--r-- 1 root root     45 Apr 24  2024 toggles1.rule
-rw-r--r-- 1 root root    570 Apr 24  2024 toggles2.rule
-rw-r--r-- 1 root root   3755 Apr 24  2024 toggles3.rule
-rw-r--r-- 1 root root  16040 Apr 24  2024 toggles4.rule
-rw-r--r-- 1 root root  49073 Apr 24  2024 toggles5.rule
-rw-r--r-- 1 root root  55346 Apr 24  2024 unix-ninja-leetspeak.rule
```

Y por ejemplo, la best64.rule genera este tipo de variantes:

```
- `password1` (append `1`)
- `Password` (capitalizar)
- `PASSWORD` (mayúsculas)
- `p@ssword` (sustitución tipo leet)
- `drowssap` (reversa)
- `P@ssw0rd1!` (combinación de varias transformaciones)
```



- **-a 3 Mask attack**
	- Un mask attack (ataque por máscara) es un tipo de ataque de fuerza bruta donde defines explícitamente la estructura del password que quieres probar, en vez de probar todas las combinaciones posibles. Es muy útil cuando tienes pistas sobre la contraseña (longitud, posiciones con letras mayúsculas, dígitos al final, símbolos, etc.).
		-  `?l` → `abcdefghijklmnopqrstuvwxyz` (minúsculas)
		- `?u` → `ABCDEFGHIJKLMNOPQRSTUVWXYZ` (mayúsculas)
		- `?d` → `0123456789` (dígitos)
		- `?s` → caracteres especiales (espacio y símbolos)
		- `?h` / `?H` → hex (`0123456789abcdef` / mayúsculas)
		- `?a` → `?l?u?d?s` (todo)
	    - `?b` → bytes `0x00-0xff` (todo en binario)

Por ejemplo, poniendo esta secuencia:
```shell-session
hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

?u ?l ?l ?l ?L ?d ?s:

- ?u: mayúscula  (M)
- ?l: minúscula    (o)
- ?l: minúscula    (u)
- ?l: minúscula    (s)
- ?l: minúscula    (e)
- ?d: dñigito       (5)
- ?s: caracter especial.  (!)
Haciendo fuerza bruta contra el diccionario seleccionado, vemos que la combinación es "Mouse5!"

--- 
**1. Use a dictionary attack to crack the first password hash. (Hash: e3e3ec5831ad5e7288241960e5d4fdb8)**

En primer lugar, guardamos el contenido del hash en un archivo llamado hash1.txt.
Con esto, ejecutamos un:
```
──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ hashid -j hash1.txt
--File 'hash1.txt'--
Analyzing 'e3e3ec5831ad5e7288241960e5d4fdb8'
[+] MD2 [JtR Format: md2]
[+] MD5 [JtR Format: raw-md5]
[+] MD4 [JtR Format: raw-md4]
[+] Double MD5 
[+] LM [JtR Format: lm]
[+] RIPEMD-128 [JtR Format: ripemd-128]
...

--End of file 'hash1.txt'--                        
```

Los apuntes, nos avanzan que este HASH correponde a un "ripemd-128". Como lo hemos ejecutado con el parámetro "-j", nos dá el tipo de formato de HASH para la librería hashcat, el cual es "md5".

Pues, ejecutaremos:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ hashcat --show -m 0 hash1.txt
e3e3ec5831ad5e7288241960e5d4fdb8:crazy!
```
Siendo la respuesta de este ejercicio: "crazy!"

**2. Use a dictionary attack with rules to crack the second password hash. (Hash: 1b0556a75770563578569ae21392630c)**

Utilizaremos un ataque por diccionario. 
Vamos a identificar que tipo de hash puede tener, pues, ejecutamos:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ hashid -j hash2.txt 
--File 'hash2.txt'--
Analyzing '1b0556a75770563578569ae21392630c'
[+] MD2 [JtR Format: md2]
[+] MD5 [JtR Format: raw-md5]
[+] MD4 [JtR Format: raw-md4]
...
[+] Domain Cached Credentials 2 [JtR Format: mscach2]
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x [JtR Format: radmin]
--End of file 'hash2.txt'--   
```
Entre otros, identificamos que seguramente será un hash del tipo md5
Yendo a la página: 
>https://hashcat.net/wiki/doku.php?id=example_hashes

Observamos que le corresponde el HASH-CODE 0.

El enunciado nos dice que utilicemos rules para descifrar este HASH, pues, ejecutaremos:
```
hashcat -a 0 -m 0 hash2.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

Nos devuelve la contraseña: *c0wb0ys1*.

En caso de que 

En caso de que nos apareciese un mensaje diciendo que este HASH ya ha sido crackeado anteriormente, deberemos realizar un:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ hashcat --show -m 0 hash2.txt
1b0556a75770563578569ae21392630c:c0wb0ys1
```

Respuesta: *c0wb0ys1*

**3. Use a mask attack to crack the third password hash. (Hash: 1e293d6912d074c0fd15844d803400dd)**


