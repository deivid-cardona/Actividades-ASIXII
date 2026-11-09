# Consecuencias y APIPA

## Impacto del ataque

Cuando el pool de direcciones del servidor DHCP legítimo se agota, los nuevos dispositivos que intentan unirse a la red no pueden obtener una dirección IP válida.

Según la configuración del sistema, pueden:

- Quedarse sin dirección IP.
- Reintentar continuamente la solicitud DHCP.
- Autoconfigurarse mediante APIPA.

## Qué es APIPA

APIPA (*Automatic Private IP Addressing*) es un mecanismo de autoconfiguración utilizado cuando un cliente no recibe respuesta de un servidor DHCP.

Las direcciones APIPA pertenecen al rango:

```text
169.254.0.0/16
```

## Limitaciones de APIPA

- Máscara `255.255.0.0`.
- No hay puerta de enlace predeterminada.
- No hay acceso a Internet.
- No hay acceso a otras redes.
- Solo permite comunicación local dentro del mismo segmento.

## Relación con DHCP Starvation

En un ataque de DHCP Starvation, es posible que un cliente obtenga una dirección APIPA porque el servidor DHCP legítimo ya no tiene direcciones disponibles.

Esto permite identificar que existe un problema de asignación DHCP, aunque no confirma por sí solo que se haya producido un ataque.

> **Captura pendiente:** ejemplo de dirección APIPA en un cliente.