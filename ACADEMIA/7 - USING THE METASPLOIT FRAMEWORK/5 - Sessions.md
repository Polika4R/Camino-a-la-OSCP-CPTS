## Listar sesiones de Metasploit
Podemos listar las sesiones activas de MSFCONSOLE con:
```
msf > sessions
```
Dando como salida: 
```shell-session
Active sessions
===============

  Id  Name  Type                     Information                 Connection
  --  ----  ----                     -----------                 ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ MS01  10.10.10.129:443 -> 10.10.10.205:50501 (10.10.10.205)
```

Podemos acceder a una sesión concreta con:
```shell-session
msf > sessions -i 1
[*] Starting interaction with 1...
```


### jobs (msfconsole)
Por ejemplo, si ejecutamos un exploit activo en un puerto específico y lo necesitamos para un módulo diferente, no podemos simplemente cerrar la sesión con \[Ctrl] + \[C]. Si lo hiciéramos, el puerto seguiría en uso, lo que afectaría el uso del nuevo módulo. Por lo tanto, necesitaríamos usar el comando jobs para revisar las tareas activas que se ejecutan en segundo plano y cerrar las antiguas para liberar el puerto.

Otros tipos de tareas dentro de las sesiones también pueden convertirse en trabajos para que se ejecuten en segundo plano sin problemas, incluso si la sesión finaliza o desaparece.

### job -h
Cuando ejecuto `exploit -j` Metasploit lanza el exploit en segundo plano como un “job” (un trabajo). Sustituiríamos el "run"/"exploit" por "exploit -j" Eso significa que el módulo queda corriendo (por ejemplo, un _handler_ que escucha conexiones reversas) y te devuelve el control del prompt para que puedas hacer otras cosas mientras tanto.

Por qué usar `-j` (ventajas):
- Mantienes el _handler_ escuchando en segundo plano sin bloquear tu consola.
- Puedes lanzar varios handlers/exploits a la vez.
- Te permite moverte por el framework, revisar sesiones, escanear, etc., mientras el job está activo.
