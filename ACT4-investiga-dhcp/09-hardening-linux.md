# Hardening en Linux

## Contexto

En un entorno virtual como VirtualBox no disponemos de un switch físico gestionable. Por este motivo, el servidor Linux debe protegerse mediante medidas propias.

## hashlimit

El módulo `hashlimit` de iptables permite limitar la tasa de paquetes por grupos, por ejemplo por IP de origen o MAC de origen.

Es útil para limitar ráfagas de mensajes `DHCPDISCOVER`, pero un límite basado únicamente en una identidad que se puede falsificar no detiene todas las variantes del abuso.

Ejemplo:

```bash
sudo iptables -A INPUT -i enp0s8 -p udp --dport 67 \
  -m hashlimit \
  --hashlimit-name dhcp_limit \
  --hashlimit-above 3/sec \
  --hashlimit-burst 5 \
  --hashlimit-mode srcmac \
  --hashlimit-htable-expire 10000 \
  -j DROP
```

## ebtables

`ebtables` es una herramienta de filtrado para bridges Linux. Actúa en capa 2, mientras que `iptables` actúa principalmente en capa 3 y superior.

Con `ebtables` se pueden aplicar reglas basadas en direcciones MAC, tipo de trama, VLAN o protocolo Ethernet.

La validación estricta de que la MAC Ethernet coincide con el campo `chaddr` requiere inspección del contenido del paquete DHCP.

Esto puede implementarse mediante herramientas más avanzadas como nftables, eBPF o sistemas de detección de intrusiones.

## Conclusión

En un entorno virtual, la protección debe combinar límites de tasa, políticas de lease y monitorización del tráfico.

> **Captura pendiente:** regla de `hashlimit` aplicada en el servidor.