# Hardening de infraestructura

## DHCP Snooping

DHCP Snooping es una función de seguridad de capa 2 presente en switches de acceso. Su objetivo es proteger la red frente a servidores DHCP no autorizados.

El switch diferencia entre dos tipos de puertos:

| Tipo de puerto | Uso | Comportamiento |
|---|---|---|
| Trusted | Conecta al servidor DHCP legítimo | Permite mensajes de servidor DHCP |
| Untrusted | Conecta a clientes | Bloquea respuestas de servidores no autorizados |

Por defecto, los puertos de acceso son untrusted. Solo el puerto que conecta con el servidor DHCP legítimo debe configurarse como trusted.

## Base de vinculación

La base de vinculación o *DHCP Snooping Binding Database* registra información sobre las asignaciones legítimas:

- Dirección MAC del cliente.
- Dirección IP asignada.
- VLAN.
- Puerto de acceso.
- Tipo y tiempo de lease.

Esta información puede utilizarse posteriormente por otras funciones de seguridad, como Dynamic ARP Inspection o IP Source Guard.

## Port Security

Port Security limita el número de direcciones MAC que pueden aprenderse por un puerto físico de un switch.

Si un atacante intenta enviar tramas con muchas MAC origen distintas desde un mismo puerto, el switch puede:

- Descartar las tramas adicionales.
- Generar una alerta.
- Desactivar el puerto.

Esta medida dificulta el ataque de Starvation complejo.

> **Captura pendiente:** esquema de puertos trusted y untrusted.