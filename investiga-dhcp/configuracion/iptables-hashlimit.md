# Mitigación con iptables y hashlimit

## Objetivo

Limitar las ráfagas de peticiones DHCP desde una misma MAC de origen.

## Regla propuesta

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

## Comprobar reglas

```bash
sudo iptables -L INPUT -v -n
```

## Eliminar la regla

```bash
sudo iptables -D INPUT -i enp0s8 -p udp --dport 67 \
  -m hashlimit \
  --hashlimit-name dhcp_limit \
  --hashlimit-above 3/sec \
  --hashlimit-burst 5 \
  --hashlimit-mode srcmac \
  --hashlimit-htable-expire 10000 \
  -j DROP
```

## Nota

El límite debe ajustarse según el número real de clientes. Un valor demasiado bajo puede afectar a clientes legítimos que renuevan su lease.