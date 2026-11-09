# Mitigaciones

## 1. Rate limiting con hashlimit

El módulo `hashlimit` de iptables permite limitar la tasa de peticiones DHCP desde una misma MAC de origen.

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

Esta regla limita a un máximo de 3 peticiones DHCP por segundo, con un margen de hasta 5, identificadas por la MAC de origen.

## Comprobación

```bash
sudo iptables -L INPUT -v -n
```

Al ejecutar nuevamente el ataque, el contador de paquetes bloqueados debería incrementarse rápidamente.

## 2. Impedir el spoofing de chaddr

Un ataque avanzado puede utilizar la misma MAC de capa 2 pero modificar el campo `chaddr` del mensaje DHCP.

El servidor DHCP estándar puede endurecerse añadiendo:

```conf
deny unknown-clients;
```

Esta opción descarta peticiones de clientes desconocidos. Sin embargo, puede impedir que nuevos dispositivos legítimos obtengan una IP, por lo que debe utilizarse con cuidado.

## 3. ebtables

`ebtables` es análogo a iptables, pero opera en la capa 2 de enlace de datos.

Puede utilizarse en un bridge Linux para limitar el número de MACs origen que atraviesan un puerto.

La validación estricta de que la MAC Ethernet coincide con el campo `chaddr` requiere inspección del contenido del paquete DHCP. Esto puede implementarse mediante herramientas más avanzadas como nftables, eBPF o sistemas de detección de intrusiones.

## 4. DHCP Snooping

En un switch físico gestionable, DHCP Snooping permite diferenciar entre:

- **Puertos trusted:** conectados al servidor DHCP legítimo.
- **Puertos untrusted:** conectados a clientes.

Los mensajes de servidor DHCP recibidos por puertos untrusted son descartados, impidiendo la actuación de un Rogue DHCP.

## 5. Port Security

Port Security limita el número de MACs que pueden aprenderse por un puerto físico.

Esto dificulta el ataque de Starvation complejo, ya que impide que un único equipo genere masivamente muchas identidades de capa 2.

## 6. Políticas de lease

Un lease corto reduce el tiempo durante el cual las IP permanecen reservadas tras un ataque.

```conf
default-lease-time 300;
max-lease-time 600;
```

Sin embargo, un lease demasiado corto aumenta el tráfico de renovación.

## Evidencias

- [ ] Captura de la regla iptables aplicada.
- [ ] Captura del contador de paquetes bloqueados.
- [ ] Captura del cliente legítimo obteniendo IP tras la mitigación.
- [ ] Captura de la configuración de `deny unknown-clients`.