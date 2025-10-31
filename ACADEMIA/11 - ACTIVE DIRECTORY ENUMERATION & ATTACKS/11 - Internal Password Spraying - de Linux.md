Ahora que hemos generado un conjunto de usuarios, es el momento de iniciar el ataque.

## Ataque de fuerza bruta interna de contraseñas desde un host Linux

Una vez creada la lista de palabras mediante uno de los métodos mostrados en la sección anterior, es hora de ejecutar el ataque. 

Rpcclient es una excelente opción para realizar este ataque desde Linux. 

Es importante tener en cuenta que un inicio de sesión válido no es evidente de inmediato con rpcclient, ya que la respuesta «Authority Name» indica un inicio de sesión exitoso. 
Podemos filtrar los intentos de inicio de sesión no válidos buscando «Authority» en la respuesta. El siguiente comando Bash de una sola línea (adaptado de aquí) se puede usar para realizar el ataque.

```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

