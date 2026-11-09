# ACT4 — Investigación: seguridad en el servicio DHCP

**Módulo:** 037 — Seguridad en servicios  
**Ciclo formativo:** ASIX  
**Alumno:** Deivid Cardona  

## Índice

1. Introducción
2. Fundamentos de DHCP
3. Proceso DORA
4. Anatomía del paquete DHCP
5. Autenticación y RFC
6. DHCP Starvation teórico
7. Consecuencias y APIPA
8. Hardening de infraestructura
9. Hardening en Linux
10. Políticas de lease
11. Modo promiscuo
12. Aspectos legales
13. Conclusiones

# 1. Introducción

En esta actividad voy a analizar el protocolo DHCPv4 desde el punto de vista de la seguridad. DHCP (*Dynamic Host Configuration Protocol*) permite a los dispositivos obtener automáticamente una dirección IP, una máscara de red, una puerta de enlace, servidores DNS y otros parámetros necesarios para comunicarse en una red.

Sin DHCP, tendría que configurar manualmente cada dispositivo. Esto sería viable en una red muy pequeña, pero en una red con muchos equipos, móviles, impresoras, cámaras o máquinas virtuales sería lento, propenso a errores y difícil de mantener.

## Problema de seguridad

DHCP fue diseñado priorizando la facilidad de uso, la compatibilidad y la automatización. Sin embargo, el intercambio estándar no incorpora autenticación obligatoria de los mensajes.

Esto significa que cualquier equipo conectado al mismo segmento puede:

- Enviar peticiones DHCP.
- Intentar actuar como servidor DHCP.
- Capturar tráfico DHCP.
- Suplantar la identidad de otros clientes.
- Agotar el pool de direcciones IP.

## Ataque DHCP Starvation

El ataque de **DHCP Starvation** consiste en agotar el pool de direcciones IP de un servidor DHCP legítimo.

El atacante genera múltiples peticiones DHCP simulando clientes falsos. Para que el servidor las considere clientes diferentes, modifica el campo `chaddr` del mensaje DHCP y, en el caso más completo, también la dirección MAC de origen de la trama Ethernet.

Cuando el pool se agota, los nuevos dispositivos no pueden obtener una dirección IP. Esto provoca una denegación de servicio parcial.

## Entorno controlado

Cualquier prueba debe realizarse únicamente en una Red Interna propia de VirtualBox. No deben realizarse pruebas contra redes domésticas, educativas, corporativas ni redes de terceros.

> **Captura pendiente:** esquema o configuración de la Red Interna en VirtualBox.

# 2. Fundamentos de DHCP

DHCP (*Dynamic Host Configuration Protocol*) es un protocolo de red que permite asignar automáticamente parámetros de configuración a los dispositivos que se conectan a una red.

Los parámetros más habituales son:

- Dirección IP.
- Máscara de subred.
- Puerta de enlace predeterminada.
- Servidores DNS.
- Nombre de dominio.
- Tiempo de concesión o lease.

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

# 3. Proceso DORA

El funcionamiento básico de DHCPv4 se basa en cuatro mensajes conocidos como **DORA**:

| Paso | Mensaje | Emisor | Función |
|---|---|---|---|
| 1 | `DHCPDISCOVER` | Cliente | Busca servidores DHCP disponibles |
| 2 | `DHCPOFFER` | Servidor | Propone una dirección IP y parámetros de red |
| 3 | `DHCPREQUEST` | Cliente | Solicita formalmente una oferta recibida |
| 4 | `DHCPACK` | Servidor | Confirma la asignación y establece el lease |

## DHCPDISCOVER

Cuando un cliente se conecta a la red y no tiene una dirección IP, envía un mensaje `DHCPDISCOVER` en broadcast.

El destino es `255.255.255.255` y el cliente normalmente utiliza `0.0.0.0` como dirección IP de origen.

## DHCPOFFER

Los servidores DHCP que reciben la petición pueden responder con un `DHCPOFFER`. En este mensaje proponen una dirección IP, máscara, puerta de enlace, servidores DNS y otras opciones.

Si existen varios servidores DHCP, el cliente puede recibir varias ofertas.

## DHCPREQUEST

El cliente envía un `DHCPREQUEST` para aceptar una oferta concreta. Este mensaje también se envía en broadcast para informar a los demás servidores de que no se ha aceptado su propuesta.

## DHCPACK

El servidor seleccionado responde con un `DHCPACK`, confirmando la asignación de la dirección IP y estableciendo el tiempo de concesión.

## Esquema del intercambio

```text
Cliente                         Servidor DHCP
  │                                 │
  │──── DHCPDISCOVER ──────────────▶│
  │                                 │
  │◀──────────── DHCPOFFER ─────────│
  │                                 │
  │──── DHCPREQUEST ───────────────▶│
  │                                 │
  │◀────────────── DHCPACK ─────────│
```

## Broadcast y seguridad

El uso de broadcast es necesario porque el cliente no conoce inicialmente la dirección del servidor DHCP. Sin embargo, cualquier equipo del mismo segmento puede observar el tráfico DHCP.

Un atacante puede utilizar esta información para identificar clientes, servidores y configuraciones de red. También puede responder como servidor DHCP no autorizado, conocido como Rogue DHCP.

> **Captura pendiente:** esquema o captura del proceso DORA.

# 4. Anatomía del paquete DHCP

DHCP se basa en BOOTP y funciona en la capa de aplicación. Un mensaje DHCP incluye campos como:

- `op`: tipo de operación.
- `htype`: tipo de hardware.
- `hlen`: longitud de la dirección hardware.
- `xid`: identificador de transacción.
- `secs`: segundos transcurridos.
- `flags`: opciones de difusión.
- `ciaddr`: dirección IP del cliente.
- `yiaddr`: dirección IP ofrecida.
- `siaddr`: dirección del servidor.
- `giaddr`: dirección del agente relay.
- `chaddr`: dirección hardware del cliente.
- `sname`: nombre del servidor.
- `file`: nombre de archivo de arranque.
- Opciones DHCP.

## Campo `chaddr`

El campo `chaddr` (*Client Hardware Address*) contiene la dirección hardware que el cliente declara dentro del mensaje DHCP.

La MAC de origen Ethernet es la dirección incluida en la cabecera de la trama de capa 2.

En un cliente legítimo, ambos valores suelen coincidir:

```text
MAC Ethernet de origen = chaddr del mensaje DHCP
```

## Diferencia entre capa 2 y capa 7

| Campo | Capa | Función |
|---|---|---|
| MAC Ethernet de origen | Capa 2 | Identifica el origen de la trama |
| `chaddr` | Capa 7 | Identifica al cliente para el servidor DHCP |

## Riesgo de suplantación

Un atacante puede modificar `chaddr` en cada petición y hacer creer al servidor que se trata de un cliente nuevo. Si además modifica la MAC Ethernet, cada petición puede parecer proceder de un equipo físico diferente.

Esta posibilidad es la base teórica del ataque de DHCP Starvation.

> **Captura pendiente:** análisis de un paquete DHCP en Wireshark mostrando el campo `chaddr`.

# 5. Autenticación y RFC

## RFC 2131

El RFC 2131 define el funcionamiento básico de DHCP. El intercambio ordinario no incorpora autenticación ni cifrado obligatorios.

Esto tiene dos consecuencias importantes:

- Cualquier host puede enviar un `DHCPDISCOVER` y solicitar una dirección IP.
- Cualquier host puede intentar responder como servidor DHCP.

No existe una comprobación inherente que permita al servidor saber si el cliente es legítimo, ni al cliente saber si el servidor es de confianza.

## RFC 3118

El RFC 3118 propone un mecanismo de autenticación para mensajes DHCP. Su objetivo es permitir que cliente y servidor validen la procedencia e integridad de los mensajes.

A pesar de ser una solución teóricamente útil, su adopción generalizada ha sido limitada.

## Razones de la baja adopción

- Requiere configuración adicional en clientes y servidores.
- Muchos dispositivos antiguos no lo implementan.
- No resuelve por sí solo el acceso físico a la red.
- Las organizaciones suelen utilizar controles de capa 2 como DHCP Snooping, Port Security y 802.1X.

> **Captura pendiente:** extracto o referencia del RFC 2131 y RFC 3118.

# 6. DHCP Starvation teórico

El ataque de **DHCP Starvation** consiste en agotar el pool de direcciones IP de un servidor DHCP legítimo.

El atacante envía múltiples peticiones DHCP utilizando diferentes valores en el campo `chaddr`. El servidor interpreta que cada petición proviene de un cliente distinto y asigna una dirección IP diferente.

## Mecánica del ataque

1. El atacante se conecta al mismo segmento de red que el servidor DHCP.
2. Genera una identidad falsa modificando el campo `chaddr`.
3. Envía un mensaje `DHCPDISCOVER`.
4. El servidor ofrece una dirección IP.
5. El atacante solicita la oferta mediante `DHCPREQUEST`.
6. El servidor confirma la asignación con `DHCPACK`.
7. El atacante repite el proceso con otra identidad falsa.
8. El pool se agota.

## Ataque simple

En el ataque simple, el atacante modifica únicamente el campo `chaddr` del mensaje DHCP. La MAC Ethernet puede permanecer igual.

Este tipo de ataque es más fácil de detectar porque la MAC real de la trama se repite.

## Ataque complejo

En el ataque complejo, el atacante modifica tanto la MAC de origen Ethernet como el campo `chaddr`.

De esta forma, cada petición parece proceder de un equipo físico diferente. Esto dificulta la detección, porque el switch observa muchas MACs distintas aunque el origen real sea el mismo.

## Comparación

| Tipo de ataque | Campo modificado | Dificultad de detección |
|---|---|---|
| Simple | `chaddr` | Más fácil de detectar |
| Complejo | MAC Ethernet y `chaddr` | Más difícil de detectar |

> **Captura pendiente:** esquema del ataque de DHCP Starvation.

# 7. Consecuencias y APIPA

Cuando el pool de direcciones del servidor DHCP legítimo se agota, los nuevos dispositivos que intentan unirse a la red no pueden obtener una dirección IP válida.

Según la configuración del sistema, pueden:

- Quedarse sin dirección IP.
- Reintentar continuamente la solicitud DHCP.
- Autoconfigurarse mediante APIPA.

## Qué es APIPA

APIPA (*Automatic Private IP Addressing*) es un mecanismo de autoconfiguración utilizado cuando un cliente no recibe respuesta de un servidor DHCP.

Las direcciones APIPA pertenecen al rango:

```text
169.254.0.0/16
```

## Limitaciones de APIPA

- Máscara `255.255.0.0`.
- No hay puerta de enlace predeterminada.
- No hay acceso a Internet.
- No hay acceso a otras redes.
- Solo permite comunicación local dentro del mismo segmento.

> **Captura pendiente:** ejemplo de dirección APIPA en un cliente.

# 8. Hardening de infraestructura

## DHCP Snooping

DHCP Snooping es una función de seguridad de capa 2 presente en switches de acceso. Su objetivo es proteger la red frente a servidores DHCP no autorizados.

El switch diferencia entre dos tipos de puertos:

| Tipo de puerto | Uso | Comportamiento |
|---|---|---|
| Trusted | Conecta al servidor DHCP legítimo | Permite mensajes de servidor DHCP |
| Untrusted | Conecta a clientes | Bloquea respuestas de servidores no autorizados |

Por defecto, los puertos de acceso son untrusted. Solo el puerto que conecta con el servidor DHCP legítimo debe configurarse como trusted.

## Base de vinculación

La base de vinculación o *DHCP Snooping Binding Database* registra información sobre las asignaciones legítimas:

- Dirección MAC del cliente.
- Dirección IP asignada.
- VLAN.
- Puerto de acceso.
- Tipo y tiempo de lease.

Esta información puede utilizarse posteriormente por otras funciones de seguridad, como Dynamic ARP Inspection o IP Source Guard.

## Port Security

Port Security limita el número de direcciones MAC que pueden aprenderse por un puerto físico de un switch.

Si un atacante intenta enviar tramas con muchas MAC origen distintas desde un mismo puerto, el switch puede:

- Descartar las tramas adicionales.
- Generar una alerta.
- Desactivar el puerto.

> **Captura pendiente:** esquema de puertos trusted y untrusted.

# 9. Hardening en Linux

En un entorno virtual como VirtualBox no disponemos de un switch físico gestionable. Por este motivo, el servidor Linux debe protegerse mediante medidas propias.

## hashlimit

El módulo `hashlimit` de iptables permite limitar la tasa de paquetes por grupos, por ejemplo por IP de origen o MAC de origen.

Es útil para limitar ráfagas de mensajes `DHCPDISCOVER`, pero un límite basado únicamente en una identidad que se puede falsificar no detiene todas las variantes del abuso.

Ejemplo:

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

## ebtables

`ebtables` es una herramienta de filtrado para bridges Linux. Actúa en capa 2, mientras que `iptables` actúa principalmente en capa 3 y superior.

Con `ebtables` se pueden aplicar reglas basadas en direcciones MAC, tipo de trama, VLAN o protocolo Ethernet.

La validación estricta de que la MAC Ethernet coincide con el campo `chaddr` requiere inspección del contenido del paquete DHCP.

Esto puede implementarse mediante herramientas más avanzadas como nftables, eBPF o sistemas de detección de intrusiones.

> **Captura pendiente:** regla de `hashlimit` aplicada en el servidor.

# 10. Políticas de lease

Un lease o concesión es el tiempo durante el cual un cliente puede utilizar una dirección IP asignada por el servidor DHCP.

## Impacto del tiempo de lease

| Tipo de lease | Ventaja | Inconveniente |
|---|---|---|
| Corto | Libera IP antes tras un ataque | Aumenta el tráfico de renovación |
| Largo | Reduce el tráfico de renovación | Mantiene IP reservadas más tiempo |

## Ejemplo

- Con un lease de 7 días, el efecto de un ataque puede durar hasta 7 días.
- Con un lease de 5 minutos, las direcciones vuelven a estar disponibles rápidamente.

## Recomendación

El administrador debe equilibrar el tiempo de lease según el tipo de red:

- Redes de clientes temporales: 1 a 8 horas.
- Redes corporativas estables: 1 a 3 días.
- Dispositivos fijos o críticos: reservas DHCP o IP estática.

## Detección de conflictos

La detección de conflictos, como el parámetro `ping-check` o `conflict-detection`, permite comprobar si una IP ya está en uso antes de asignarla.

Esto ayuda a evitar asignar direcciones que ya están siendo utilizadas por otro equipo, aunque el pool esté corrupto o incompleto.

> **Captura pendiente:** configuración de lease y detección de conflictos.

# 11. Modo promiscuo

El modo promiscuo permite que una interfaz de red acepte y observe tramas que no están dirigidas específicamente a su dirección MAC.

En VirtualBox, esta opción está restringida por defecto porque puede exponer tráfico de otras máquinas virtuales o del host.

En una Red Interna de VirtualBox, habilitar **Permitir todo** facilita que Kali pueda enviar y recibir paquetes DHCP durante las pruebas.

Un script puede enviar tramas con MAC origen inventada aunque la interfaz no esté en modo promiscuo. Sin embargo, la recepción y observación completa del tráfico puede verse afectada si esta opción no está habilitada.

> **Captura pendiente:** configuración del adaptador en modo promiscuo.

# 12. Aspectos legales

El ataque de DHCP Starvation es una variante de denegación de servicio (*Denial of Service* o DoS). Su objetivo es impedir que nuevos dispositivos obtengan una dirección IP válida.

Las pruebas solo están permitidas cuando:

- Se realizan sobre sistemas propios.
- Se dispone de autorización expresa del propietario.
- Se utiliza un entorno aislado de laboratorio.
- No se afectan servicios de terceros.

Realizar ataques de denegación de servicio sin autorización puede tener consecuencias legales. Para describir artículos, penas o sanciones concretas del Código Penal español, debo consultar una versión oficial y actualizada del texto legal y evitar presentar una interpretación como asesoramiento jurídico.

> **Pendiente de investigación:** añadir referencias oficiales al BOE y explicar de forma prudente los artículos aplicables, con fecha de consulta.

# 13. Conclusiones

He analizado el funcionamiento del protocolo DHCPv4 y sus principales riesgos de seguridad. DHCP facilita la administración de una red, pero su falta de autenticación obligatoria permite que un atacante suplante clientes o servidores.

La principal vulnerabilidad es la confianza del servidor DHCP en el campo `chaddr`, que puede ser modificado por el cliente. Esto permite generar múltiples identidades falsas y agotar el pool de direcciones IP.

Cuando el pool se agota, los nuevos clientes no pueden obtener una dirección IP válida. Esto provoca una denegación de servicio parcial y puede derivar en el uso de direcciones APIPA.

Las mitigaciones principales son DHCP Snooping, Port Security, límites de tasa con `hashlimit`, políticas de lease adecuadas, detección de conflictos de IP y monitorización del tráfico.

La seguridad del servicio DHCP no depende únicamente del servidor. Debe combinarse con controles de acceso de capa 2, límites de tasa, políticas de lease adecuadas y monitorización del tráfico de red.