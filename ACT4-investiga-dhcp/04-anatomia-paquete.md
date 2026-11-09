# Anatomía del paquete DHCP

## Estructura del mensaje

DHCP se basa en BOOTP y funciona en la capa de aplicación. Un mensaje DHCP incluye campos como:

- `op`: tipo de operación.
- `htype`: tipo de hardware.
- `hlen`: longitud de la dirección hardware.
- `xid`: identificador de transacción.
- `secs`: segundos transcurridos.
- `flags`: opciones de difusión.
- `ciaddr`: dirección IP del cliente.
- `yiaddr`: dirección IP ofrecida.
- `siaddr`: dirección del servidor.
- `giaddr`: dirección del agente relay.
- `chaddr`: dirección hardware del cliente.
- `sname`: nombre del servidor.
- `file`: nombre de archivo de arranque.
- Opciones DHCP.

## Campo `chaddr`

El campo `chaddr` (*Client Hardware Address*) contiene la dirección hardware que el cliente declara dentro del mensaje DHCP.

La MAC de origen Ethernet es la dirección incluida en la cabecera de la trama de capa 2.

En un cliente legítimo, ambos valores suelen coincidir:

```text
MAC Ethernet de origen = chaddr del mensaje DHCP
```

## Diferencia entre capa 2 y capa 7

| Campo | Capa | Función |
|---|---|---|
| MAC Ethernet de origen | Capa 2 | Identifica el origen de la trama |
| `chaddr` | Capa 7 | Identifica al cliente para el servidor DHCP |

## Uso por el servidor

El servidor DHCP puede utilizar `chaddr` como identificador del cliente para asociar la concesión y gestionar el lease. Al tratarse de un campo incluido en el mensaje, su validez depende de la confianza y de los controles aplicados por la infraestructura.

## Riesgo de suplantación

Un atacante puede modificar `chaddr` en cada petición y hacer creer al servidor que se trata de un cliente nuevo. Si además modifica la MAC Ethernet, cada petición puede parecer proceder de un equipo físico diferente.

Esta posibilidad es la base teórica del ataque de DHCP Starvation.

> **Captura pendiente:** análisis de un paquete DHCP en Wireshark mostrando el campo `chaddr`.