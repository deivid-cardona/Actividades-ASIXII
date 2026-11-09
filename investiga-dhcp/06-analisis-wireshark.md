# Análisis con Wireshark

## Objetivo

Analizar el tráfico DHCP para identificar el patrón del ataque y comprender la diferencia entre la MAC de origen Ethernet y el campo `chaddr`.

## Filtro recomendado

En Wireshark utilizaré el siguiente filtro:

```text
bootp || dhcp
```

También puedo utilizar:

```text
udp.port == 67 || udp.port == 68
```

## Mensajes esperados

Durante el funcionamiento normal, deberían aparecer los cuatro mensajes DORA:

- `DHCPDISCOVER`
- `DHCPOFFER`
- `DHCPREQUEST`
- `DHCPACK`

Durante el ataque, deberían aparecer múltiples mensajes `DHCPDISCOVER` procedentes de diferentes direcciones MAC.

## Análisis de capa 2

En la cabecera Ethernet debo comprobar:

- **Source:** MAC de origen de la trama.
- **Destination:** `ff:ff:ff:ff:ff:ff`, broadcast.

## Análisis de capa 7

Dentro del mensaje DHCP debo comprobar:

- **Client MAC Address (`chaddr`):** dirección hardware declarada por el cliente.
- **Transaction ID (`xid`):** identificador de la transacción.
- **Options:** tipo de mensaje DHCP.

## Comparación

| Campo | Capa | Función |
|---|---|---|
| MAC Ethernet de origen | Capa 2 | Identifica el origen de la trama |
| `chaddr` | Capa 7 | Identifica al cliente para el servidor DHCP |

En un cliente legítimo, ambos valores coinciden. En un ataque de Starvation complejo, el atacante modifica ambos campos.

## Patrón de ataque

El patrón típico del ataque es:

- Muchas peticiones `DHCPDISCOVER` en poco tiempo.
- Diferentes MACs Ethernet de origen.
- Diferentes valores de `chaddr`.
- Ausencia de mensajes `DHCPREQUEST` o `DHCPACK` posteriores.
- Agotamiento rápido del pool DHCP.

## Evidencias

- [ ] Captura del tráfico normal DORA.
- [ ] Captura del tráfico durante el ataque.
- [ ] Captura mostrando diferentes MACs Ethernet.
- [ ] Captura mostrando diferentes valores de `chaddr`.
- [ ] Captura comparando ambos campos.