# Preparación del entorno

## Máquinas virtuales

Para esta práctica utilizaré tres máquinas virtuales conectadas a la misma Red Interna de VirtualBox:

| VM | Rol | Sistema operativo |
|---|---|---|
| VM 1 | Atacante | Kali Linux |
| VM 2 | Servidor DHCP | Debian / Ubuntu Server |
| VM 3 | Cliente legítimo | Ubuntu / Debian Desktop |

## Configuración de red

### VM 1 — Kali Linux

- Adaptador 1: **Red Interna**.
- Nombre de la red: `DHCP-LAB`.
- Modo promiscuo: **Permitir todo**.

### VM 2 — Servidor Debian/Ubuntu

- Adaptador 1: **NAT**, necesario para descargar paquetes.
- Adaptador 2: **Red Interna**.
- Nombre de la red: `DHCP-LAB`.
- Modo promiscuo: **Permitir todo**.

### VM 3 — Cliente legítimo

- Adaptador 1: **Red Interna**.
- Nombre de la red: `DHCP-LAB`.
- Modo promiscuo: **Permitir todo**.

## Configuración del servidor

En el servidor Debian/Ubuntu, configuraré una IP estática en la interfaz conectada a la Red Interna.

```bash
sudo nano /etc/network/interfaces
```

Ejemplo:

```text
auto enp0s8
iface enp0s8 inet static
    address 192.168.10.1
    netmask 255.255.255.0
```

Aplicar la configuración:

```bash
sudo systemctl restart networking
```

Comprobar:

```bash
ip a
ip route
```

## Instalación del servidor DHCP

```bash
sudo apt update
sudo apt install isc-dhcp-server -y
```

Comprobar el nombre de la interfaz conectada a la Red Interna:

```bash
ip a
```

Editar:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Configurar:

```text
INTERFACESv4="enp0s8"
```

## Pool DHCP restringido

Para poder comprobar el ataque, configuraré un pool pequeño. Por ejemplo, utilizaré diez direcciones disponibles.

```conf
authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.100 192.168.10.110;
    option routers 192.168.10.1;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 8.8.8.8;

    default-lease-time 300;
    max-lease-time 600;
}
```

Reiniciar el servicio:

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

## Comprobación inicial

Desde el cliente legítimo:

```bash
sudo dhclient -v enp0s3
ip a
```

Desde el servidor:

```bash
cat /var/lib/dhcp/dhcpd.leases
```

## Captura de tráfico

En Kali Linux:

```bash
sudo tcpdump -i eth0 -n port 67 or port 68
```

En Wireshark, utilizaré el filtro:

```text
bootp || dhcp
```

## Evidencias pendientes

- [ ] Captura de la configuración de las tres máquinas virtuales.
- [ ] Captura de la configuración de red del servidor.
- [ ] Captura del estado de `isc-dhcp-server`.
- [ ] Captura del cliente legítimo obteniendo IP.
- [ ] Captura del archivo `dhcpd.leases`.