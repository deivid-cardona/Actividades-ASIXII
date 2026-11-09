# Proceso DORA

El funcionamiento básico de DHCPv4 se basa en cuatro mensajes conocidos como **DORA**:

| Paso | Mensaje | Emisor | Función |
|---|---|---|---|
| 1 | `DHCPDISCOVER` | Cliente | Busca servidores DHCP disponibles |
| 2 | `DHCPOFFER` | Servidor | Propone una dirección IP y parámetros de red |
| 3 | `DHCPREQUEST` | Cliente | Solicita formalmente una oferta recibida |
| 4 | `DHCPACK` | Servidor | Confirma la asignación y establece el lease |

## DHCPDISCOVER

Cuando un cliente se conecta a la red y no tiene una dirección IP, envía un mensaje `DHCPDISCOVER` en broadcast.

El destino es `255.255.255.255` y el cliente normalmente utiliza `0.0.0.0` como dirección IP de origen.

## DHCPOFFER

Los servidores DHCP que reciben la petición pueden responder con un `DHCPOFFER`. En este mensaje proponen una dirección IP, máscara, puerta de enlace, servidores DNS y otras opciones.

Si existen varios servidores DHCP, el cliente puede recibir varias ofertas.

## DHCPREQUEST

El cliente envía un `DHCPREQUEST` para aceptar una oferta concreta. Este mensaje también se envía en broadcast para informar a los demás servidores de que no se ha aceptado su propuesta.

## DHCPACK

El servidor seleccionado responde con un `DHCPACK`, confirmando la asignación de la dirección IP y estableciendo el tiempo de concesión.

## Esquema del intercambio

```text
Cliente                         Servidor DHCP
  │                                 │
  │──── DHCPDISCOVER ──────────────▶│
  │                                 │
  │◀──────────── DHCPOFFER ─────────│
  │                                 │
  │──── DHCPREQUEST ───────────────▶│
  │                                 │
  │◀────────────── DHCPACK ─────────│
```

## Broadcast y seguridad

El uso de broadcast es necesario porque el cliente no conoce inicialmente la dirección del servidor DHCP. Sin embargo, cualquier equipo del mismo segmento puede observar el tráfico DHCP.

Un atacante puede utilizar esta información para identificar clientes, servidores y configuraciones de red. También puede responder como servidor DHCP no autorizado, conocido como Rogue DHCP.

> **Captura pendiente:** esquema o captura del proceso DORA.