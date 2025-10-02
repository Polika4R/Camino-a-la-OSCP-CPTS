Cada sistema operativo tiene una shell como mínimo para interactuar con él.
Para utilizar una shell, deberemos contar con un "emulador de terminal".

Estos son algunos de los que existen:

| **Emulador de terminal**                                       | **Operating System**     |
| :------------------------------------------------------------- | :----------------------- |
| [Windows Terminal](https://github.com/microsoft/terminal)      | Windows                  |
| [cmder](https://cmder.app/)                                    | Windows                  |
| [PuTTY](https://www.putty.org/)                                | Windows                  |
| [kitty](https://sw.kovidgoyal.net/kitty/)                      | Windows, Linux and MacOS |
| [Alacritty](https://github.com/alacritty/alacritty)            | Windows, Linux and MacOS |
| [xterm](https://invisible-island.net/xterm/)                   | Linux                    |
| [GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal) | Linux                    |
| [MATE Terminal](https://github.com/mate-desktop/mate-terminal) | Linux                    |
| [Konsole](https://konsole.kde.org/)                            | Linux                    |
| [Terminal](https://en.wikipedia.org/wiki/Terminal_\(macOS\))   | MacOS                    |
| [iTerm2](https://iterm2.com/)                                  | MacOS                    |
## Intérpretes de Lenguaje de Comandos (CLI Interpreters):
- Funcionan como un intérprete humano: traducen las instrucciones del usuario para que el sistema operativo las ejecute.
- Una **interfaz de línea de comandos** combina tres elementos:
    1. Sistema operativo (OS)
    2. Emulador de terminal
    3. Intérprete de lenguaje de comandos (shell)
## Comandos útiles
#### Comando *ps*
```shell-session
Polika4RM@htb[/htb]$ ps

    PID TTY          TIME CMD
   4232 pts/1    00:00:00 bash
  11435 pts/1    00:00:00 ps
```
Indica los procesos actuales que están corriendo en el sistema. En el ejemplo anterior, vemos que existe una shell corriendo en BASH, y el proceso "ps" justo ejecutado.

#### Comando *env*
```shell-session
Polika4RM@htb[/htb]$ env

SHELL=/bin/bash
```
Muestra todas las variables de entorno definidas en la sesión actual del usuario, en este caso, una /bin/bash.

#### PowerShell vs Bash
- **Plataforma:** 
	- Bash → Linux/Unix
	- PowerShell → Windows (multiplataforma).
    
- **Salida:** 
	- Bash → texto; 
	- PowerShell → objetos.
    
- **Comandos:** 
	- Bash → comandos Unix (`ls`, `grep`)
	- PowerShell → cmdlets (`Get-Process`).
    
- **Scripting:** 
	- Bash → simple, texto
	- PowerShell → avanzado, manipula objetos.
	   
- **Uso:** 
	- Bash → administración Linux, pentesting
	- PowerShell → administración Windows, automatización.

---------
**QUESTIONS**
**1. Which two shell languages did we experiment with in this section? (Format: shellname&shellname)**
Respuesta: *bash&powershell*

**2. In Pwnbox issue the $PSversiontable variable using PowerShell. Submit the edition of PowerShell that is running as the answer.**
Al ejecutarlo, observo:
```
PS [10.10.14.95] /home/htb-ac-1876550 > $PSversiontable

Name                           Value
----                           -----
PSVersion                      7.5.0
PSEdition                      Core
GitCommitId                    7.5.0
OS                             Parrot Security 6.3 (lorikeet)
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0
```

Siendo la respuesta: *Core*

