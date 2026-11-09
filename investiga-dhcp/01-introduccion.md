# Introducción

## Contexto

En esta actividad voy a analizar el protocolo DHCP (*Dynamic Host Configuration Protocol*) desde el punto de vista de la seguridad. DHCP es el servicio que permite a los dispositivos obtener automáticamente una dirección IP, una máscara de red, una puerta de enlace y otros parámetros necesarios para comunicarse dentro de una red.

Sin DHCP, tendría que configurar manualmente cada dispositivo. Esto sería viable en una red muy pequeña, pero en una red con muchos equipos, móviles, impresoras, cámaras o máquinas virtuales sería lento, propenso a errores y difícil de mantener.

## Problema de seguridad

DHCP fue diseñado priorizando la facilidad de uso, la compatibilidad y la automatización. Sin embargo, no incluye un mecanismo obligatorio de autenticación en su funcionamiento estándar.

Esto significa que cualquier equipo conectado al mismo segmento puede:

- Enviar peticiones DHCP.
- Intentar actuar como servidor DHCP.
- Capturar tráfico DHCP.
- Suplantar la identidad de otros clientes.
- Agotar el pool de direcciones IP.

## Ataque DHCP Starvation

El ataque de **DHCP Starvation** consiste en agotar el pool de direcciones IP de un servidor DHCP legítimo.

El atacante genera múltiples peticiones DHCP simulando clientes falsos. Para que el servidor las considere clientes diferentes, modifica el campo `chaddr` del mensaje DHCP y, en el caso más completo, también la dirección MAC de origen de la trama Ethernet.

Cuando el pool se agota, los nuevos dispositivos no pueden obtener una dirección IP. Esto provoca una denegación de servicio parcial, ya que los equipos legítimos que intentan conectarse no pueden acceder correctamente a la red.

## Objetivo de la práctica

Mi objetivo no es únicamente ejecutar el ataque, sino comprender:

- Cómo funciona el intercambio DHCP.
- Qué información utiliza el servidor para identificar a un cliente.
- Por qué es posible falsificar una identidad de cliente.
- Cómo se puede detectar el ataque mediante Wireshark.
- Qué medidas pueden reducir o impedir el impacto del ataque.

## Entorno controlado

Todas las pruebas se realizarán en una Red Interna de VirtualBox, utilizando máquinas virtuales propias. No se realizarán pruebas contra redes domésticas, educativas, corporativas ni redes de terceros.

> **Captura pendiente:** esquema o configuración de la Red Interna en VirtualBox.