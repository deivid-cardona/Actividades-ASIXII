# Modo promiscuo

## Qué es

El modo promiscuo permite que una interfaz de red acepte y observe tramas que no están dirigidas específicamente a su dirección MAC.

## Por qué está restringido

En VirtualBox, esta opción está restringida por defecto porque puede exponer tráfico de otras máquinas virtuales o del host.

## Impacto en la práctica

En una Red Interna de VirtualBox, habilitar **Permitir todo** facilita que Kali pueda enviar y recibir paquetes DHCP durante las pruebas.

## Envío sin modo promiscuo

Un script puede enviar tramas con MAC origen inventada aunque la interfaz no esté en modo promiscuo.

Sin embargo, la recepción y observación completa del tráfico puede verse afectada si esta opción no está habilitada.

## Conclusión

El modo promiscuo es necesario para observar correctamente el tráfico DHCP en el laboratorio, pero debe utilizarse únicamente en entornos controlados.

> **Captura pendiente:** configuración del adaptador en modo promiscuo.