# Funcionamiento de DHCP

## Proceso DORA

El protocolo DHCPv4 utiliza el intercambio DORA para asignar configuración de red a los clientes.

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

## Campo `chaddr`

El campo `chaddr` (*Client Hardware Address*) es la dirección hardware que el cliente declara dentro del mensaje DHCP.

La MAC de origen Ethernet es la dirección incluida en la cabecera de la trama de capa 2.

En un cliente legítimo, ambos valores coinciden:

```text
MAC Ethernet de origen = chaddr del mensaje DHCP
```

Sin embargo, el servidor DHCP utiliza principalmente `chaddr` para identificar al cliente, asignarle una IP y mantener el lease.

Esto permite que un atacante modifique `chaddr` y haga creer al servidor que se trata de un cliente nuevo.

## Riesgo de suplantación

La diferencia entre la capa 2 y la capa 7 permite realizar suplantación o *spoofing*.

Un atacante puede generar muchas identidades falsas modificando el campo `chaddr` en cada petición. Si además modifica la MAC Ethernet, cada petición parecerá proceder de un equipo físico diferente.

> **Captura pendiente:** análisis de un paquete DHCP en Wireshark mostrando el campo `chaddr`.