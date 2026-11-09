# Fundamentos de DHCP

## Qué es DHCP

DHCP (*Dynamic Host Configuration Protocol*) es un protocolo de red que permite asignar automáticamente parámetros de configuración a los dispositivos que se conectan a una red.

Los parámetros más habituales son:

- Dirección IP.
- Máscara de subred.
- Puerta de enlace predeterminada.
- Servidores DNS.
- Nombre de dominio.
- Tiempo de concesión o lease.

## Funcionamiento básico

Cuando un equipo se conecta a la red y está configurado para obtener IP automáticamente, envía una solicitud DHCP. Un servidor DHCP responde ofreciendo una dirección disponible y, finalmente, confirma la asignación.

## Componentes principales

| Componente | Función |
|---|---|
| Cliente DHCP | Solicita y recibe la configuración de red |
| Servidor DHCP | Asigna direcciones IP y parámetros de red |
| Pool | Rango de direcciones IP disponibles para asignar |
| Lease | Tiempo durante el cual un cliente puede utilizar una IP |
| Relay agent | Reenvía peticiones DHCP entre redes diferentes |

## Importancia en una red

DHCP facilita la administración de una red porque evita tener que configurar manualmente cada dispositivo. Además, permite reutilizar direcciones IP cuando un equipo deja de utilizarlas.

Sin embargo, esta automatización también introduce riesgos: si no existen controles de acceso o autenticación, un atacante puede aprovecharse del proceso para obtener direcciones, suplantar clientes o actuar como servidor DHCP.

## Relación con la seguridad

La seguridad de DHCP no depende únicamente del servidor. También depende de quién puede acceder al segmento de red, qué dispositivos pueden enviar mensajes y qué controles existen para limitar el tráfico anómalo.