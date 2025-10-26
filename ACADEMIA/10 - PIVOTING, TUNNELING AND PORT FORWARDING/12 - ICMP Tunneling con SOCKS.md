
La tunelización ICMP encapsula el tráfico en paquetes ICMP que contienen "echo" solicitudes y respuestas.
Cuando un host dentro de una red con cortafuegos puede hacer ping a un servidor externo, puede encapsular su tráfico dentro de la solicitud de "echo" de ping y enviarlo a dicho servidor. 
El servidor externo puede validar este tráfico y enviar una respuesta adecuada, lo cual resulta extremadamente útil para la exfiltración de datos y la creación de túneles pivote a un servidor externo.

Usaremos la herramienta *ptunnel-ng* para crear un túnel entre nuestro servidor Ubuntu y el host atacante. 
Una vez creado el túnel, podremos redirigir nuestro tráfico a través del cliente ptunnel-ng. Podemos iniciar el servidor ptunnel-ng en el host pivote de destino. 

## Configurando "ptunnel-ng"
Comencemos configurando ptunnel-ng.
En nuestra máquina atacante, lo clonamos de github con:

```shell-session
Polika4RM@htb[/htb]$ git clone https://github.com/utoni/ptunnel-ng.git
```

Una vez que el repositorio ptunnel-ng se clona en nuestra máquina atacante, podemos ejecutar el script autogen.sh ubicado en la raíz del directorio ptunnel-ng:

```shell-session
Polika4RM@htb[/htb]$ sudo ./autogen.sh 
```

Tras ejecutar "autogen.sh", ptunnel-ng se puede usar tanto en el lado del cliente como del servidor. Ahora debemos transferir el repositorio desde nuestro host de ataque al host objetivo. Como en secciones anteriores, podemos usar SCP para transferir los archivos. 
Si queremos transferir el repositorio completo y los archivos que contiene, debemos usar la opción -r con SCP:

