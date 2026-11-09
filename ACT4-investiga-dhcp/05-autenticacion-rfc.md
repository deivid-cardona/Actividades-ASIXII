# Autenticación y RFC

## RFC 2131

El RFC 2131 define el funcionamiento básico de DHCP. El intercambio ordinario no incorpora autenticación ni cifrado obligatorios.

Esto tiene dos consecuencias importantes:

- Cualquier host puede enviar un `DHCPDISCOVER` y solicitar una dirección IP.
- Cualquier host puede intentar responder como servidor DHCP.

No existe una comprobación inherente que permita al servidor saber si el cliente es legítimo, ni al cliente saber si el servidor es de confianza.

## RFC 3118

El RFC 3118 propone un mecanismo de autenticación para mensajes DHCP. Su objetivo es permitir que cliente y servidor validen la procedencia e integridad de los mensajes.

A pesar de ser una solución teóricamente útil, su adopción generalizada ha sido limitada.

## Razones de la baja adopción

- Requiere configuración adicional en clientes y servidores.
- Muchos dispositivos antiguos no lo implementan.
- No resuelve por sí solo el acceso físico a la red.
- Las organizaciones suelen utilizar controles de capa 2 como DHCP Snooping, Port Security y 802.1X.

## Conclusión

La falta de autenticación nativa obligatoria es una de las principales debilidades de DHCP. Por ello, la seguridad debe complementarse con controles de acceso y monitorización en la infraestructura de red.

> **Captura pendiente:** extracto o referencia del RFC 2131 y RFC 3118.