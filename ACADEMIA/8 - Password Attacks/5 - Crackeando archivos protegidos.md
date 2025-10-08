Para borrar HASHes previamente calculados se deberá acceder a la ruta: 
>home/polika4r/.local/share/hashcat/hashcat.potfile

Existen muchas extensiones que corresponden a archivos cifrados.
Como ejemplo, vamos a tomar el siguiente comando que lo que hace es ubicar archivos encriptados en nuestro sistema Linux:
```shell-session
Polika4RM@htb[/htb]$ for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

Teniendo como salida: 
```shell-session
Polika4RM@htb[/htb]$ for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

File extension:  .xls

File extension:  .xls*

File extension:  .xltx

File extension:  .od*
/home/cry0l1t3/Docs/document-temp.odt
/home/cry0l1t3/Docs/product-improvements.odp
/home/cry0l1t3/Docs/mgmt-spreadsheet.ods
...SNIP...
```

Muchos archivos de claves SSH comienzan por:
```
-----BEGIN [...SNIP...] PRIVATE KEY-----
```

Entonces, podemos buscar estos haciendo uso del siguiente comando:
``` 
Polika4RM@htb[/htb]$ grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null

/home/jsmith/.ssh/id_ed25519:1:-----BEGIN OPENSSH PRIVATE KEY-----
/home/jsmith/.ssh/SSH.private:1:-----BEGIN RSA PRIVATE KEY-----
/home/jsmith/Documents/id_rsa:1:-----BEGIN OPENSSH PRIVATE KEY-----
<SNIP>
```

Teniendo dichos archivos el siguiente contenido:
```shell-session
Polika4RM@htb[/htb]$ cat /home/jsmith/.ssh/SSH.private

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2109D25CC91F8DBFCEB0F7589066B2CC

8Uboy0afrTahejVGmB7kgvxkqJLOczb1I0/hEzPU1leCqhCKBlxYldM2s65jhflD
4/OH4ENhU7qpJ62KlrnZhFX8UwYBmebNDvG12oE7i21hB/9UqZmmHktjD3+OYTsD
<SNIP>
```

Para saber si un archivo está encriptado o no, podemos utilizar la herramienta *ssh-heygen* de la siguiente forma:
```shell-session
Polika4RM@htb[/htb]$ ssh-keygen -yf ~/.ssh/id_ed25519 

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIpNefJd834VkD5iq+22Zh59Gzmmtzo6rAffCx2UtaS6
```

En caso de que está encriptado, nos pedirá un passphrase para dicho archivo:
```shell-session
Polika4RM@htb[/htb]$ ssh-keygen -yf ~/.ssh/id_rsa

Enter passphrase for "/home/jsmith/.ssh/id_rsa":
```

---

## Crackeando claves SSH encriptadas.
John the Ripper tiene varios scripts para extrar hashes de archivos. 
Podemos localizarlos con el siguiente comando:
```shell-session
Polika4RM@htb[/htb]$ locate *2john*

/usr/bin/bitlocker2john
/usr/bin/dmg2john
/usr/bin/gpg2john
/usr/bin/hccap2john
/usr/bin/keepass2john
/usr/bin/putty2john
/usr/bin/racf2john
/usr/bin/rar2john
/usr/bin/uaf2john
/usr/bin/vncpcap2john
/usr/bin/wlanhcx2john
/usr/bin/wpapcap2john
/usr/bin/zip2john
/usr/share/john/1password2john.py
/usr/share/john/7z2john.pl
/usr/share/john/DPAPImk2john.py
/usr/share/john/adxcsouf2john.py
/usr/share/john/aem2john.py
/usr/share/john/aix2john.pl
/usr/share/john/aix2john.py
/usr/share/john/andotp2john.py
/usr/share/john/androidbackup2john.py
...
```


Por ejemplo, respecto los scripts anteriores, podemos utilizar el "ssh2john.py" para obtener el correspondiente hash de una clave SSH encriptada. 

Ejecutaremos:
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ locate *2john* | grep ssh
/usr/bin/ssh2john              <-------------------- utilizaremos este
/usr/share/john/ssh2john.py
/usr/share/john/__pycache__/ssh2john.cpython-313.pyc
```

Sobre el archivo SSH.private que contiene una clave SSH encriptada, ejecutaremos:
```shell-session
Polika4RM@htb[/htb]$ ssh2john.py SSH.private > ssh.hash
Polika4RM@htb[/htb]$ john --wordlist=rockyou.txt ssh.hash

Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
1234         (SSH.private)
1g 0:00:00:00 DONE (2022-02-08 03:03) 16.66g/s 1747Kp/s 1747Kc/s 1747KC/s Knightsing..Babying
Session completed
```

La contraseña es SSH.private:1234, pero lo podemos ver mejor haciendo:
```shell-session
Polika4RM@htb[/htb]$ john ssh.hash --show

SSH.private:1234

1 password hash cracked, 0 left
```

---
## Cracking password-protected documents

Buscando por los scripts de John, uno muy útil para documentos de oficina es "office2john":
```
┌──(polika4r㉿kali)-[~/Escritorio/trabajo]
└─$ locate *2john* | grep office
/usr/bin/libreoffice2john
/usr/bin/office2john              <-----------------
/usr/bin/staroffice2john
/usr/share/john/libreoffice2john.py
/usr/share/john/office2john.py
/usr/share/john/staroffice2john.py
/usr/share/john/__pycache__/libreoffice2john.cpython-313.pyc
/usr/share/john/__pycache__/office2john.cpython-313.pyc
/usr/share/john/__pycache__/staroffice2john.cpython-313.pyc

```

Tanto si tenemos un word como un pdf, podemos desencriptarlos con:

- Caso word:
```shell-session
Polika4RM@htb[/htb]$ office2john.py Protected.docx > protected-docx.hash
Polika4RM@htb[/htb]$ john --wordlist=rockyou.txt protected-docx.hash
Polika4RM@htb[/htb]$ john protected-docx.hash --show

Protected.docx:1234

1 password hash cracked, 0 left
```

- Caso pdf:
```shell-session
Polika4RM@htb[/htb]$ pdf2john.py PDF.pdf > pdf.hash
Polika4RM@htb[/htb]$ john --wordlist=rockyou.txt pdf.hash
Polika4RM@htb[/htb]$ john pdf.hash --show

PDF.pdf:1234

1 password hash cracked, 0 left
```

---
**1.  Download the attached ZIP archive (cracking-protected-files.zip), and crack the file within. What is the password?**

Tengo un archivo llamado: cracking-protected-files.zip.gz

Primero tengo que descomprimir el .gz y despues el .zip; pues, ejecuto:
```
gunzip cracking-protected-files.zip.gz
unzip cracking-protected-files.zip
```

Quedando como resultado un archivo con extension *xlsx* (es decir, un archivo de oficina).

Ejecuto:
```
┌──(polika4r㉿kali)-[~/Descargas]
└─$ /usr/bin/office2john Confidential.xlsx > hash.xlsx      
```

Y obteno el archivo hash.xlsx con el siguiente contenido:
```
Confidential.xlsx:$office$*2013*100000*256*16*cb0e251cdec92e97eeb38e595cd4eb09*58758c88f3bb25e43e1e21adbd4b6e50*0057c1ae71b0023424ba705607dc0df1d9a786974bb957a821cfd7e39129eb15

```

Ejecuto pues:
```
┌──(polika4r㉿kali)-[~/Descargas]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.xlsx
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2013 for all loaded hashes
Cost 2 (iteration count) is 100000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
beethoven        (Confidential.xlsx)     
1g 0:00:00:21 DONE (2025-10-07 08:55) 0.04688g/s 315.0p/s 315.0c/s 315.0C/s 98765432..BITCH
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

Obteniendo así la constraseña "beethoven".

