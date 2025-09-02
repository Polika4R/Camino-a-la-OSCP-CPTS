
# WFUZZ

| Comando                                                                                                                                                  | Descripción                                     | Ejemplo                                                                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `wfuzz -c -z file,/usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 http://<dominio>/FUZZ`                        | Fuzzing de **directorios** (paths)              | `wfuzz -c -z file,/usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 http://cozyhosting.htb/FUZZ`                        |
| `wfuzz -c -z file,/usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,php,html,txt --hc 404 http://<dominio>/FUZZ` | Fuzzing de **archivos** con extensiones comunes | `wfuzz -c -z file,/usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,php,html,txt --hc 404 http://cozyhosting.htb/FUZZ` |
| `wfuzz -c -z file,/usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "Host: FUZZ.example.com" http://example.com/`         | Fuzzing de **subdominios / vhosts**             | `wfuzz -c -z file,/usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "Host: FUZZ.example.com" http://example.com/`               |

# DIRBUSTER 

| Comando                                          | Descripción                            | Ejemplo |
| ------------------------------------------------ | -------------------------------------- | ------- |
| `gobuster dir -u <url> -w /ruta/diccionario.txt` | Busca directorios de tal dirección web |         |
|                                                  |                                        |         |
# FFUF
| Comando                                                         | Descripción                                                         | Ejemplo                                                                                                                                      |
| --------------------------------------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| ffuf -w /ruta/a/wordlist.txt:FFUZ -u <\dominio>/FFUZ -ic -t 100 | Fuzzing de **directorios** (paths) ocultando falsos positivos (-ic) | ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-<br>medium.txt:FFUZ -u http://cozyhosting.htb/FFUZ -ic -t 100 |
|                                                                 |                                                                     |                                                                                                                                              |
