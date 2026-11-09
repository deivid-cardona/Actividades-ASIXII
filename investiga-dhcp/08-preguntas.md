# Preguntas de análisis

## 1. Análisis de capas

**¿Por qué un atacante de DHCP Starvation debe alterar simultáneamente la MAC Ethernet y el campo `chaddr`?**

Debe alterar ambos campos porque cada capa utiliza una información diferente para identificar al cliente.

La MAC Ethernet de origen pertenece a la capa 2 y es la que observa el switch. El campo `chaddr` pertenece al mensaje DHCP, en capa 7, y es el que utiliza el servidor DHCP para identificar al cliente y asignarle una IP.

Si solo se modifica `chaddr`, el servidor puede asignar varias IP, pero el switch seguirá viendo la misma MAC de origen. Esto facilita la detección mediante Port Security.

Si solo se modifica la MAC Ethernet, el switch verá diferentes orígenes, pero el servidor DHCP puede seguir asociando las peticiones al mismo `chaddr` y no asignará nuevas IP.

Por ello, el ataque más eficaz modifica ambos campos de forma coherente.

## 2. Impacto en entornos virtuales

**Si no se hubiera cambiado el Modo Promiscuo en VirtualBox, ¿habría tenido éxito el ataque?**

El modo promiscuo afecta principalmente a la recepción de tráfico, no a la capacidad de enviar tramas con MAC origen falsificada.

El script puede enviar paquetes con MACs inventadas aunque la interfaz no esté en modo promiscuo. Sin embargo, la recepción y observación completa del tráfico puede verse afectada.

En una Red Interna de VirtualBox, habilitar **Permitir todo** facilita que Kali pueda enviar y recibir correctamente las tramas necesarias para la práctica.

## 3. Optimización con Linux Bridge

**¿Cómo podría utilizarse ebtables para validar la correspondencia entre la MAC de origen y el campo `chaddr`?**

`ebtables` permite filtrar tramas Ethernet en un bridge Linux. Actúa en capa 2 y puede aplicar reglas basadas en direcciones MAC, tipo de trama, VLAN o protocolo Ethernet.

Para validar que la MAC Ethernet coincide con el campo `chaddr`, sería necesario inspeccionar el contenido del paquete DHCP, no solo la cabecera Ethernet.

Una regla básica de `ebtables` puede limitar el número de MACs por puerto:

```bash
sudo ebtables -A FORWARD -p IPv4 --limit 10/min --limit-burst 15 -j DROP
```

Sin embargo, la comprobación estricta de `chaddr` requiere un análisis más profundo del payload DHCP, que puede realizarse con nftables, eBPF o herramientas de detección de intrusiones.

## 4. Resistencia de la mitigación

**¿Cómo se vería afectada la protección si el atacante falsifica completamente la MAC en capa física?**

Si el atacante consigue falsear completamente la MAC en capa 2, la regla de `hashlimit` basada en `srcmac` pierde eficacia, porque cada petición parecerá proceder de una MAC distinta.

En este caso, sería necesario utilizar mecanismos de capa 2 más robustos:

- Port Security con límite de MACs por puerto.
- DHCP Snooping.
- Dynamic ARP Inspection.
- 802.1X para autenticar el acceso a la red.
- Monitorización y alertas ante cambios frecuentes de MAC.

La defensa más eficaz combina controles de acceso físico, autenticación de red y limitación de tasa.