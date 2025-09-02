

# 1. Escenario 1: *Phishing investigation.*

Un correo sospechoso llega a varios empleados haciéndose pasar por el banco de la empresa. El encargado de IT revisa el dominio del enlace con WHOIS y encuentra:

- El dominio se registró hace pocos días.
- La información del dueño está oculta.
- Los servidores están en un proveedor conocido por alojar actividades maliciosas.

Con esto, el analista confirma que es **phishing**, alerta al área de TI para bloquear el dominio y avisa a los empleados. Además, investigará más direcciones IP para detectar otros posibles sitios de phishing.


# 2. Escenario 2: *Análisis de malware.*

Un investigador analiza un nuevo malware que roba datos y se conecta a un servidor remoto (C2). Al revisar el dominio del servidor:
- Está registrado con un correo gratuito anónimo.
- La dirección es de un país con mucha actividad de ciberdelito.
- El registrador es conocido por no controlar bien los abusos.

Con estos datos, el investigador concluye que el servidor C2 está en un hosting sospechoso o comprometido. Luego, avisa al proveedor de hosting para reportar la actividad maliciosa.

# 3. Escenario 3: *Informe de inteligencia sobre amenazas*
Una empresa de ciberseguridad estudia a un grupo avanzado que ataca bancos. Revisando los dominios que usan, descubren patrones:

- Registran varios dominios poco antes de lanzar ataques.
- Usan alias o identidades falsas.
- Comparten servidores de nombres, lo que muestra una infraestructura común.
- Muchos de sus dominios ya fueron dados de baja tras ataques anteriores.

Con esta información, los analistas arman un perfil del grupo (sus tácticas y métodos) y generan **indicadores de compromiso (IOCs)** para que otras organizaciones puedan detectar y bloquear futuros ataques.




# Ejemplo de uso

El comando se instala con:

```shell-session
Polika4RM@htb[/htb]$ sudo apt update
Polika4RM@htb[/htb]$ sudo apt install whois -y
```


La forma más sencilla de acceder a los datos de WHOIS es mediante la herramienta de línea de comandos de WHOIS. Realicemos una búsqueda de WHOIS en facebook.com:Utilising WHOIS

```shell-session
Polika4RM@htb[/htb]$ whois facebook.com

   Domain Name: FACEBOOK.COM
   Registry Domain ID: 2320948_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.registrarsafe.com
   Registrar URL: http://www.registrarsafe.com
   Updated Date: 2024-04-24T19:06:12Z
   Creation Date: 1997-03-29T05:00:00Z
   Registry Expiry Date: 2033-03-30T04:00:00Z
   Registrar: RegistrarSafe, LLC
   Registrar IANA ID: 3237
   Registrar Abuse Contact Email: abusecomplaints@registrarsafe.com
   Registrar Abuse Contact Phone: +1-650-308-7004
   Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
   Domain Status: serverDeleteProhibited https://icann.org/epp#serverDeleteProhibited
   Domain Status: serverTransferProhibited https://icann.org/epp#serverTransferProhibited
   Domain Status: serverUpdateProhibited https://icann.org/epp#serverUpdateProhibited
   Name Server: A.NS.FACEBOOK.COM
   Name Server: B.NS.FACEBOOK.COM
   Name Server: C.NS.FACEBOOK.COM
   Name Server: D.NS.FACEBOOK.COM
   DNSSEC: unsigned
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of whois database: 2024-06-01T11:24:10Z <<<

[...]
Registry Registrant ID:
Registrant Name: Domain Admin
Registrant Organization: Meta Platforms, Inc.
[...]
```

I**nformación clave obtenida del WHOIS de facebook.com**:

- **Registro del dominio**
    - Registrador: _RegistrarSafe, LLC_
    - Creación: _1997-03-29_
    - Expira: _2033-03-30_  
        → Indica un dominio antiguo, legítimo y con larga vigencia.
- **Propietario (Registrant/Admin/Tech)**
    - Organización: _Meta Platforms, Inc._
    - Contacto: _Domain Admin_  
        → Confirma que el dominio pertenece oficialmente a Meta.
- **Estatus del dominio**
    - Protecciones: _clientDeleteProhibited, clientTransferProhibited, serverUpdateProhibited_, etc.  
        → Estas medidas evitan cambios no autorizados, mostrando altos niveles de seguridad.
- **Servidores de nombres (DNS)**
    - _A.NS.FACEBOOK.COM, B.NS.FACEBOOK.COM, C.NS.FACEBOOK.COM, D.NS.FACEBOOK.COM_  
        → Los DNS están bajo el propio dominio, lo que refleja control y confiabilidad.

 **Conclusiones**:

- El dominio es **legítimo, bien protegido y gestionado directamente por Meta**.
    
- WHOIS permite ver información útil para confirmar autenticidad y seguridad.
    
- Sin embargo, **no revela datos internos (empleados o vulnerabilidades)**, por lo que debe complementarse con otras técnicas de reconocimiento.

