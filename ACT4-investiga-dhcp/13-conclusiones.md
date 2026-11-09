# Conclusiones

## Resultado del análisis

He analizado el funcionamiento del protocolo DHCPv4 y sus principales riesgos de seguridad. DHCP facilita la administración de una red, pero su falta de autenticación obligatoria permite que un atacante suplante clientes o servidores.

## Vulnerabilidad identificada

La principal vulnerabilidad es la confianza del servidor DHCP en el campo `chaddr`, que puede ser modificado por el cliente. Esto permite generar múltiples identidades falsas y agotar el pool de direcciones IP.

## Impacto

Cuando el pool se agota, los nuevos clientes no pueden obtener una dirección IP válida. Esto provoca una denegación de servicio parcial y puede derivar en el uso de direcciones APIPA.

## Mitigaciones

- DHCP Snooping.
- Port Security.
- Límites de tasa con `hashlimit`.
- Políticas de lease adecuadas.
- Detección de conflictos de IP.
- Monitorización del tráfico.

## Conclusión final

La seguridad del servicio DHCP no depende únicamente del servidor. Debe combinarse con controles de acceso de capa 2, límites de tasa, políticas de lease adecuadas y monitorización del tráfico de red.