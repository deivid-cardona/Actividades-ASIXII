# DHCP Starvation teórico

## Definición

El ataque de **DHCP Starvation** consiste en agotar el pool de direcciones IP de un servidor DHCP legítimo.

El atacante envía múltiples peticiones DHCP utilizando diferentes valores en el campo `chaddr`. El servidor interpreta que cada petición proviene de un cliente distinto y asigna una dirección IP diferente.

## Mecánica del ataque

1. El atacante se conecta al mismo segmento de red que el servidor DHCP.
2. Genera una identidad falsa modificando el campo `chaddr`.
3. Envía un mensaje `DHCPDISCOVER`.
4. El servidor ofrece una dirección IP.
5. El atacante solicita la oferta mediante `DHCPREQUEST`.
6. El servidor confirma la asignación con `DHCPACK`.
7. El atacante repite el proceso con otra identidad falsa.
8. El pool se agota.

## Ataque simple

En el ataque simple, el atacante modifica únicamente el campo `chaddr` del mensaje DHCP. La MAC Ethernet puede permanecer igual.

Este tipo de ataque es más fácil de detectar porque la MAC real de la trama se repite.

## Ataque complejo

En el ataque complejo, el atacante modifica tanto la MAC de origen Ethernet como el campo `chaddr`.

De esta forma, cada petición parece proceder de un equipo físico diferente. Esto dificulta la detección, porque el switch observa muchas MACs distintas aunque el origen real sea el mismo.

## Comparación

| Tipo de ataque | Campo modificado | Dificultad de detección |
|---|---|---|
| Simple | `chaddr` | Más fácil de detectar |
| Complejo | MAC Ethernet y `chaddr` | Más difícil de detectar |

## Consecuencia

Cuando el pool se agota, los nuevos clientes no pueden obtener una dirección IP válida. Esto provoca una denegación de servicio parcial.

> **Captura pendiente:** esquema del ataque de DHCP Starvation.