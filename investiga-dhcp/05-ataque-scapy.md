# Ataque con Scapy

## Aviso ético

Este procedimiento debe ejecutarse únicamente en una Red Interna propia de VirtualBox. No debe utilizarse contra redes domésticas, educativas, corporativas ni redes de terceros.

## Objetivo

Utilizar Scapy para generar peticiones DHCP controladas y comprender cómo se falsifican tanto la MAC Ethernet como el campo `chaddr`.

Scapy es una herramienta de manipulación interactiva de paquetes basada en Python. Permite construir, enviar, capturar y analizar tráfico de red en distintas capas del modelo OSI.

## Instalación

En Kali Linux:

```bash
sudo apt update
sudo apt install python3-scapy -y
```

## Script de laboratorio

El script incluido en `scripts/starvation.py` genera un número limitado de peticiones DHCP.

```bash
python3 scripts/starvation.py --interface eth0 --cantidad 10
```

El parámetro `--cantidad` limita el número de peticiones. Para esta práctica se recomienda utilizar un valor bajo, por ejemplo 10.

## Funcionamiento del script

El script realiza los siguientes pasos:

1. Genera una MAC aleatoria.
2. Crea una trama Ethernet con esa MAC como origen.
3. Crea un paquete IP con origen `0.0.0.0` y destino `255.255.255.255`.
4. Crea un paquete UDP desde el puerto 68 hacia el puerto 67.
5. Crea un mensaje BOOTP con el campo `chaddr` coincidente con la MAC generada.
6. Añade la opción DHCP `message-type: discover`.
7. Envía el paquete por la interfaz indicada.

## Diferencia con Yersinia

Yersinia puede fallar si no modifica de forma coherente el campo `chaddr` del payload DHCP. El script con Scapy permite controlar ambos valores:

- MAC de origen Ethernet, capa 2.
- Campo `chaddr` del mensaje DHCP, capa 7.

De esta forma, cada petición parece proceder de un cliente distinto tanto para el switch como para el servidor DHCP.

## Comprobación del ataque

En el servidor:

```bash
cat /var/lib/dhcp/dhcpd.leases | grep "lease " | wc -l
```

En el cliente legítimo:

```bash
sudo dhclient -r enp0s3
sudo dhclient -v enp0s3
```

Si el pool está agotado, el cliente no recibirá una dirección IP válida.

## Evidencias

- [ ] Captura de la instalación de Scapy.
- [ ] Captura de la ejecución del script.
- [ ] Captura del número de leases asignados.
- [ ] Captura de Wireshark mostrando múltiples `DHCPDISCOVER`.
- [ ] Captura del cliente legítimo sin IP.