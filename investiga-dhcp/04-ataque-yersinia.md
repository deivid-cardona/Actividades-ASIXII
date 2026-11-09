# Ataque con Yersinia

## Aviso ético

Este procedimiento debe ejecutarse únicamente en una Red Interna propia de VirtualBox. No debe utilizarse contra redes domésticas, educativas, corporativas ni redes de terceros.

## Objetivo

Utilizar Yersinia para generar múltiples peticiones DHCP y comprobar cómo puede agotarse el pool de direcciones IP de un servidor DHCP legítimo.

Yersinia es un framework utilizado para analizar y probar protocolos de capa 2. Entre los protocolos que puede manipular se encuentran STP, CDP, HSRP, 802.1Q, 802.1X y DHCP.

## Instalación

En Kali Linux:

```bash
sudo apt update
sudo apt install -y yersinia
```

## Ejecución en modo consola

```bash
sudo yersinia -I
```

Dentro de la interfaz:

1. Pulsar `g` para seleccionar el protocolo.
2. Seleccionar **DHCP**.
3. Pulsar `x` para listar los ataques disponibles.
4. Seleccionar la opción de envío de paquetes DHCP, por ejemplo *sending RAW packets* o *sending DISCOVER packets*.
5. Dejarlo activo durante unos 15 segundos.
6. Pulsar `L` para detener la ejecución.

## Ejecución en modo gráfico

También puede utilizarse el modo gráfico:

```bash
sudo yersinia -G
```

## Qué ocurre

Yersinia genera múltiples solicitudes DHCP con direcciones MAC aleatorias. El servidor DHCP interpreta que cada petición proviene de un cliente distinto y asigna una dirección IP diferente.

Si el pool es pequeño, puede agotarse rápidamente.

## Comprobación en el servidor

```bash
cat /var/lib/dhcp/dhcpd.leases | grep "lease " | wc -l
```

Este comando permite contar el número de leases asignados.

## Comprobación en el cliente legítimo

En la máquina cliente:

```bash
sudo dhclient -r enp0s3
sudo dhclient -v enp0s3
```

Si el pool está agotado, el cliente enviará `DHCPDISCOVER` repetidamente sin recibir una respuesta válida.

## Evidencias

- [ ] Captura de la instalación de Yersinia.
- [ ] Captura de la ejecución en modo consola.
- [ ] Captura de la ejecución en modo gráfico.
- [ ] Captura de las peticiones DHCP generadas.
- [ ] Captura del número de leases asignados.
- [ ] Captura del cliente legítimo sin IP.