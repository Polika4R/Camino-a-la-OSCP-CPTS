
**Enunciado:**
Ahora, vayamos a la práctica. Una empresa nos contrató para probar sus sistemas de seguridad informática, incluyendo sus sistemas IDS e IPS. Nuestro cliente desea aumentar su seguridad informática y, por lo tanto, implementará mejoras específicas en sus sistemas IDS/IPS después de cada prueba exitosa. Sin embargo, desconocemos las directrices según las cuales se implementarán estos cambios. Nuestro objetivo es obtener información específica de las situaciones dadas.
Solo se nos proporciona una máquina protegida por sistemas IDS/IPS para que podamos realizar las pruebas. Para fines de aprendizaje y para familiarizarnos con el comportamiento de los sistemas IDS/IPS, tenemos acceso a una página web de estado en:

>http://<\target>/status.php

Esta página nos muestra el número de alertas. Sabemos que si recibimos una cantidad específica de alertas, se nos prohibirá el acceso. Por lo tanto, debemos probar el sistema objetivo de la forma más discreta posible.

**Target(s): 10.129.2.80 (ACADEMY-NMAP-EASY)**
**1. Our client wants to know if we can identify which operating system their provided machine is running on. Submit the OS name as the answer.**
Ejecutando un:
```
nmap 10.129.2.80 -sSV --open --min-rate 2000 -Pn -n -vvv
```

Veo que existen 3 puertos abiertos:
```
Discovered open port 22/tcp on 10.129.2.80
Discovered open port 80/tcp on 10.129.2.80
Discovered open port 10001/tcp on 10.129.2.80
```

Vamos a recibir cabeceras de dichos servicios con netcat. Ejecutamos:
```
nc -nv 10.129.2.80 22
nc -nv 10.129.2.80 80
nc -nv 10.129.2.80 10001
```

Ya con este simple comando, veo que el puerto 22 está corriendo Ubuntu:
```
nc -nv 10.129.2.80 22
(UNKNOWN) [10.129.2.80] 22 (ssh) open
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.7
```
Respuesta: *Ubuntu*

