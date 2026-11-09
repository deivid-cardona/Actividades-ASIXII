# ACT11 — Investigación DHCP: simulación de DHCP Starvation y mitigaciones

**Módulo:** 037 — Seguridad en servicios  
**Ciclo formativo:** ASIX  
**Alumno:** Deivid Cardona  
**Formato de entrega:** URL al informe documentado en GitBook personal.

## Objetivo

En esta actividad voy a comprender el impacto del agotamiento del direccionamiento IP dinámico en una red local y aprenderé a proteger la infraestructura frente a este tipo de ataque.

Concretamente, voy a:

- Comprender el intercambio de mensajes del protocolo DHCP: `DISCOVER`, `OFFER`, `REQUEST` y `ACK`.
- Demostrar cómo un atacante puede agotar un pool de direcciones IPv4 mediante la simulación de múltiples clientes falsos.
- Analizar el tráfico malicioso con Wireshark, identificando la variación de la dirección MAC en la trama Ethernet frente al campo `chaddr` del payload DHCP.
- Implementar mitigaciones a nivel de host en Linux mediante `iptables` y el módulo `hashlimit`.
- Documentar las contramedidas de capa 2 utilizadas en entornos empresariales: DHCP Snooping, limitación de tasa y Port Security.

## Entorno de práctica

La práctica se realizará en un entorno aislado de VirtualBox utilizando una **Red Interna** con soporte promiscuo.

| Máquina virtual | Rol | Sistema operativo |
|---|---|---|
| VM 1 | Atacante | Kali Linux |
| VM 2 | Servidor DHCP | Debian / Ubuntu Server |
| VM 3 | Cliente legítimo | Ubuntu / Debian Desktop |

## Topología

```text
┌────────────────────┐     ┌────────────────────┐     ┌────────────────────┐
│     Kali Linux     │     │  Debian/Ubuntu     │     │  Cliente legítimo  │
│      Atacante      │────▶│   Servidor DHCP    │◀────│   Cliente DHCP     │
│   Yersinia/Scapy   │     │  isc-dhcp-server   │     │     dhclient       │
└────────────────────┘     └────────────────────┘     └────────────────────┘
                         Red interna VirtualBox
```

## Índice

1. [Introducción](01-introduccion.md)
2. [Preparación del entorno](02-preparacion-entorno.md)
3. [Funcionamiento de DHCP](03-funcionamiento-dhcp.md)
4. [Ataque con Yersinia](04-ataque-yersinia.md)
5. [Ataque con Scapy](05-ataque-scapy.md)
6. [Análisis con Wireshark](06-analisis-wireshark.md)
7. [Mitigaciones](07-mitigaciones.md)
8. [Preguntas](08-preguntas.md)
9. [Conclusiones](09-conclusiones.md)

## Estructura del proyecto

```text
investiga-dhcp/
├── README.md
├── 01-introduccion.md
├── 02-preparacion-entorno.md
├── 03-funcionamiento-dhcp.md
├── 04-ataque-yersinia.md
├── 05-ataque-scapy.md
├── 06-analisis-wireshark.md
├── 07-mitigaciones.md
├── 08-preguntas.md
├── 09-conclusiones.md
├── scripts/
│   └── starvation.py
├── configuracion/
│   ├── dhcpd.conf
│   ├── interfaces.md
│   └── iptables-hashlimit.md
├── evidencias/
│   └── README.md
└── gitbook/
    └── README.md
```

## Checklist de entrega

- [ ] Configurar las tres máquinas virtuales en la misma Red Interna.
- [ ] Habilitar el modo promiscuo cuando sea necesario.
- [ ] Configurar `isc-dhcp-server` con un pool restringido.
- [ ] Comprobar el funcionamiento normal del cliente DHCP.
- [ ] Capturar el intercambio DORA con Wireshark.
- [ ] Ejecutar el ataque con Yersinia en la red interna.
- [ ] Ejecutar el ataque con Scapy mediante el script preparado.
- [ ] Comprobar el agotamiento del pool DHCP.
- [ ] Comprobar que el cliente legítimo deja de recibir IP.
- [ ] Analizar la diferencia entre MAC Ethernet y `chaddr`.
- [ ] Aplicar mitigación con `iptables` y `hashlimit`.
- [ ] Documentar DHCP Snooping, Port Security y `ebtables`.
- [ ] Responder las preguntas de análisis.
- [ ] Añadir todas las capturas de evidencia.
- [ ] Redactar las conclusiones finales.

## Aviso ético

Las pruebas de este ataque deben realizarse exclusivamente en una Red Interna propia de VirtualBox. No deben ejecutarse contra redes domésticas, educativas, corporativas ni redes de terceros.

El objetivo académico es comprender la vulnerabilidad, reproducirla en un entorno controlado y aplicar medidas de mitigación.