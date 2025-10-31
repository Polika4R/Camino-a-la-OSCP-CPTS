### En Linux:

| Comando                          | Descripción                                                                                                 | Ejemplo                  |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------ |
| sudo responder -I <interfaz_red> | Recibe intentos de log-on, obteniendo Hashes de usuarios de la red. Resultados en: <br>/usr/share/responder | sudo responder -I ens224 |

### En Windows:

| Comando       | Descripción                                                               | Ejemplo |
| ------------- | ------------------------------------------------------------------------- | ------- |
| .\Inveigh.exe | Ejecuta Inveigh, el cual recibe intentos de solicitud log-on de usuarios. |         |
Con Inveigh, entramos en consola interactiva presionando "ESC":
Seguidamente, se puede ejecutar:
- "GET NTLMV2UNIQUE":
	  Para obtener HASHES de contraseñas. 
- "GET NTLMV2USERNAMES":
	  Para obtener direcciones IP, Hosts y Usuarios

