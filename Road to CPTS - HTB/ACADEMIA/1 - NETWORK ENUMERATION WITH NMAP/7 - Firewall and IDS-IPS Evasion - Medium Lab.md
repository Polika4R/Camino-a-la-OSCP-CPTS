**Enunciado:**
Tras realizar la primera prueba y enviar los resultados a nuestro cliente, los administradores realizaron algunos cambios y mejoras en el IDS/IPS y el firewall. Durante la reunión, pudimos observar que los administradores no estaban satisfechos con sus configuraciones anteriores y observaron que el tráfico de red podría filtrarse de forma más estricta.

Nota: Para realizar el ejercicio correctamente, debemos usar el protocolo UDP en la VPN.


**Target**: 10.129.2.48
**1. After the configurations are transferred to the system, our client wants to know if it is possible to find out our target's DNS server version. Submit the DNS server version of the target as the answer.**

Simplemente realizo un:
```
 sudo nmap -sCV --min-rate 2000 --open -Pn -n -oA target 10.129.2.48 -vvv
```

Y entre bastante texto de la respuesta, observo esta parte de la salida:
```
53/tcp  open  domain      syn-ack ttl 63 (unknown banner: HTB{GoTtgUnyze9Psw4vGjcuMpHRp})
| dns-nsid: 
|_  bind.version: HTB{GoTtgUnyze9Psw4vGjcuMpHRp}
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    HTB{GoTtgUnyze9Psw4vGjcuMpHRp}

```
Siendo la respuesta: *HTB{GoTtgUnyze9Psw4vGjcuMpHRp}*

