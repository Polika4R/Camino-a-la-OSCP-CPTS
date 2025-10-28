Llegados a este punto hemos completado la enumeración inicial:
Has el momento hemos podido:
- obtener información básica de usuario y grupos.
- enumerar hosts (como el de domain controller)

Vamos a utilizar dos técnicas diferentes en paralelo:
- network poisoning
- password sparying
El objetivo es encontrar credenciales en texto plano para un usuario del dominio.

Esta sección y la siguiente cubrirán una forma común de recopilar credenciales y obtener una base inicial durante una evaluación: 
**un ataque de intermediario (Man-in-the-Middle) contra las transmisiones de Resolución de Nombres de Multidifusión Local de Enlace (LLMNR) y Servicio de Nombres NetBIOS (NBT-NS).** 

```
Explicación:
Imagina que estás en una oficina y un ordenador necesita encontrar la dirección IP de un servidor llamado "ServidorArchivos".

1. **Pregunta Pública (LLMNR/NBT-NS):** Si el DNS (el directorio telefónico principal de la red) falla, el ordenador **grita la pregunta** a toda la red local: "¿Quién es ServidorArchivos?"
    
2. **El Ataque Man-in-the-Middle (MitM):** El atacante, que está escuchando la red, **se hace pasar por ese servidor** y responde **antes** que nadie: "**¡Soy yo! ¡Aquí tienes mi IP!**" (La IP del atacante).
    
3. **El Robo de Credenciales:** La máquina que preguntó intenta autenticarse con el atacante pensando que es el servidor real. En lugar de recibir la contraseña en texto claro, el atacante intercepta y captura el **hash de la contraseña** (una versión cifrada) que la máquina envía automáticamente al intentar conectarse.
    

**En resumen:** El atacante explota la forma en que los ordenadores buscan nombres en una red local (LLMNR/NBT-NS) para **engañarlos**, haciéndoles enviar sus credenciales cifradas (los _hashes_) al atacante. Esto se logra sin que el atacante tenga que interactuar con el servidor real, obteniendo así una base inicial para el _pentest_.
```


Dependiendo de la red, este ataque puede proporcionar hashes de contraseñas de bajo privilegio o de nivel administrativo que se pueden descifrar sin conexión, o incluso credenciales en texto plano. Aunque no se aborda en este módulo, estos hashes también se pueden usar para realizar un ataque de Retransmisión SMB para autenticarse en uno o varios hosts del dominio con privilegios administrativos sin tener que descifrar el hash de la contraseña sin conexión. ¡Comencemos!


---

## Manual básico LLMNR & NBT-NS
