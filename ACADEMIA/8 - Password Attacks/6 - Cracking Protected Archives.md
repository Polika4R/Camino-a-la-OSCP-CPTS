Para borrar HASHes previamente calculados se deberá acceder a la ruta: 
>home/polika4r/.local/share/hashcat/hashcat.potfile


Existen diferentes extensiones para comprimir conjuntos de archivos, entre otros:
```
tar, gz, rar, zip, vmdb/vmx, cpt, truecrypt, bitlocker, kdbx, deb, 7z, and gzip.
```

### CRACKEANDO archivos ZIP
Extraeremos el HASH con:
```
Polika4RM@htb[/htb]$ zip2john ZIP.zip > zip.hash
Polika4RM@htb[/htb]$ cat zip.hash 

ZIP.zip/customers.csv:$pkzip2$1*2*2*0*2a*1e*490e7510*0*42*0*2a*490e*409b*ef1e7feb7c1cf701a6ada7132e6a5c6c84c032401536faf7493df0294b0d5afc3464f14ec081cc0e18cb*$/pkzip2$:customers.csv:ZIP.zip::ZIP.zip
```

Una vez tengamos el HASH extraido, podremos crackearlo con John y nuestra lista deseada. 
```shell-session
Polika4RM@htb[/htb]$ john --wordlist=rockyou.txt zip.hash

Polika4RM@htb[/htb]$ john zip.hash --show

ZIP.zip/customers.csv:1234:customers.csv:ZIP.zip::ZIP.zip

1 password hash cracked, 0 left
```

---
## CRACKEANDO archivos GZIP
Normalmente, un archivo .gzip es un archivo comprimido NO cifrado.
Sirve para reducir el tamaño de los datos pero cualquiera podría descomprimirlo.

Sin embargo *OpenSSL* puede usuarse para cifrar cualquier archivo (incluso un gzip) con contraseña.
Esto crea un archivo que parece un GZIP pero en realidad no es un GZIP normal, sino un archivo cifrado con OpenSSL.

El comando `file` de Linux analiza el formato del archivo a partir de los contenidos de este y no de su extensión.
Al mirar el tipo de archivo:
```shell-session
Polika4RM@htb[/htb]$ file GZIP.gzip 

GZIP.gzip: openssl enc'd data with salted password
```

Observamos que es un archivo openssl "salted".

Desencriptar con openssl puede darnos muchos falsos positivos, pues, el método más recomendado es utilizar un bucle for: 
```
Polika4RM@htb[/htb]$ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

Una vez que el Loop finalice, en el directorio desde donde se lanzó el comando anterior veríamos el archivo extraído:
```
Polika4RM@htb[/htb]$ ls

customers.csv  GZIP.gzip  rockyou.txt
```

---
## CRACKEANDO BITLOCKER
Podemos utilizar un script que se llama:
```
bitlocker2john
```

La herramienta puede generar hasta **4 hashes distintos** para un mismo volumen:

|Hash|Significado|Uso típico|
|---|---|---|
|`$bitlocker$0$...`|Contraseña de usuario (modo rápido, “fast”)|Se prueba primero porque es más rápido; puede dar falsos positivos, pero suele bastar para descifrar si la contraseña es débil.|
|`$bitlocker$1$...`|Contraseña de usuario (modo completo / verificación estricta)|Más seguro; verifica totalmente el descifrado. Se usa si `$0` falla o quieres confirmar.|
|`$bitlocker$2$...`|Recovery key (clave de 48 dígitos)|Muy larga, aleatoria, casi imposible de adivinar sin pistas.|
|`$bitlocker$3$...`|Recovery key (otra variante)|Igual que `$2`; solo otro formato interno.|


Con el comando:
```
Polika4RM@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes
Polika4RM@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash
Polika4RM@htb[/htb]$ cat backup.hash

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f

```

Desgranemos paso por paso:
```
Polika4RM@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes
```
- -i (input file): procesa el fichero Backup.vhd para extraer lso metadatos/VMK y generar los HASHES.

Seguimos:
```
Polika4RM@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash
```
- `grep` busca dentro del fichero `backup.hashes` todas las líneas que coinciden con el patrón `bitlocker$0`.
- se redirige todas estas entradas al fichero backup.hash

Una vez se ha generado, podemos utilizar John o Hashcat para crackearlo.

Una búsqueda rápida en:
>https://hashcat.net/wiki/doku.php?id=example_hashes

Nos indica que el HASH es 22100 | bitlocker;  pues utilizaremos con hashcat el -m 22100.

Le enviamos a HASHCAT la totalidad del archivo backup.hash a crackear, con el comando:
```
Polika4RM@htb[/htb]$ hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt
```
(tarda 2 minutos pero lo encuentra, be patient).

Y nos encuentra satisfactoriamente que la contraseña es: *1234qwer*

---
## Montando un BitLocker-encrypted drives en Windows
- Doble clic en el archivo `.vhd`.
- Windows mostrará un error inicial por estar cifrado.
- Después de montar, doble clic en la unidad BitLocker para ingresar la contraseña y desbloquearla.

---
## Montando un BitLocker-encrypted drives en Linux or macOS
- Instalar la herramienta dislocker (sudo apt-get install dislocker).
- Crear carpetas para montar la unidad: /media/bitlocker y /media/bitlockermount.
```shell-session
Polika4RM@htb[/htb]$ sudo mkdir -p /media/bitlocker
Polika4RM@htb[/htb]$ sudo mkdir -p /media/bitlockermount
```
- Luego usamos losetup para configurar el VHD como dispositivo de bucle, desciframos la unidad usando dislocker y finalmente montamos el volumen descifrado:
```shell-session
Polika4RM@htb[/htb]$ sudo losetup -f -P Backup.vhd
Polika4RM@htb[/htb]$ sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
Polika4RM@htb[/htb]$ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

- Acceder a los archivos:
```shell-session
Polika4RM@htb[/htb]$ cd /media/bitlockermount/
Polika4RM@htb[/htb]$ ls -la
```

- Y una vez acabado, desmontar con:
```
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```

---
Target(s): 94.237.57.211:47890 
**1. Run the above target then navigate to http:\//ip:port/download, then extract the downloaded file. Inside, you will find a password-protected VHD file. Crack the password for the VHD and submit the recovered password as your answer.**

Accedemos por firefox a: 
```
http://94.237.57.211:47890/download
```

Y abrimos una terminal en la carpeta donde se encuentra: 
```
cracking-protected-archives.zip
unzip cracking-protected-archives.zip
```
Y nos quedará el archivo:
```
Private.vhd
```

Al tratarse de una partición vhd (archivo de bitlocker), usaré:
```
bitlocker2john -i Private.vhd > backup.hashes
```

Seguidamente, filtraré por la opción 
```
grep "bitlocker\$0" backup.hashes > backup.hash
```
Dejándome solo con el HASH0 (contraseña del usuario):
```
┌─[eu-academy-3]─[10.10.15.0]─[htb-ac-1876550@htb-ccvfnbil3z]─[~/Downloads]
└──╼ [★]$ cat backup.hash
$bitlocker$0$16$b3c105c7ab7faaf544e84d712810da65$1048576$12$b020fe18bbb1db0103000000$60$e9c6b548788aeff190e517b0d85ada5daad7a0a3f40c4467307011ac17f79f8c99768419903025fd7072ee78b15a729afcf54b8c2e3af05bb18d4ba0
```
Y ejecuto:
```
hashcat -a 0 -m 22100 backup.hash /usr/share/wordlists/rockyou.txt 
```

Dando como respuesta la contraseña: "francisco"


**2. Mount the BitLocker-encrypted VHD and enter the contents of flag.txt as your answer.**

Siguiendo los apuntes anteriores, sigo este conjunto de comandos:
```
sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount
sudo losetup -f -P Private.vhd
sudo apt install dislocker
sudo dislocker /dev/loop0p1 -ufrancisco -- /media/bitlocker
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
cd /media/bitlockermount/
cat flag.txt
```

Siendo la respuesta: *43d95aeed3114a53ac66f01265f9b7af*

