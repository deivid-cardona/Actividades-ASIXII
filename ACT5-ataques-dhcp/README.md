# ACT5 — Simulación DHCP Starvation

**Módulo:** 037 — Seguridad en servicios  
**Ciclo formativo:** ASIX  
**Alumno:** Deivid Cardona  
**Formato de entrega:** Informe documentado en GitBook personal.

> **Alcance:** esta actividad es independiente de ACT4. Aquí se documenta una práctica de laboratorio sobre agotamiento de direcciones DHCP y sus mitigaciones. Todas las pruebas se limitan a una red aislada propia en VirtualBox, con autorización y sin tráfico hacia redes externas.

## Índice

1. [Introducción y objetivos](#1-introducción-y-objetivos)
2. [Topología del laboratorio](#2-topología-del-laboratorio)
3. [Comprobación inicial](#3-comprobación-inicial)
4. [Práctica con Yersinia](#4-práctica-con-yersinia)
5. [Práctica con Scapy](#5-práctica-con-scapy)
6. [Validación del impacto](#6-validación-del-impacto)
7. [Mitigaciones](#7-mitigaciones)
8. [Prueba de las defensas](#8-prueba-de-las-defensas)
9. [Preguntas de análisis](#9-preguntas-de-análisis)
10. [Conclusiones](#10-conclusiones)
11. [Capturas y evidencias](#11-capturas-y-evidencias)
12. [Referencias](#12-referencias)

## 1. Introducción y objetivos

En esta práctica voy a estudiar el impacto que puede tener el agotamiento del pool DHCP de una LAN y cómo se puede reducir el riesgo mediante controles de red y de host. Observaré el proceso DORA (Discover, Offer, Request, Acknowledge), la identificación del cliente mediante la dirección MAC Ethernet y el campo `chaddr` de BOOTP/DHCP, y los registros de concesiones del servidor.

Los objetivos son:

- Comprender el intercambio de mensajes DHCP.
- Observar en un entorno cerrado cómo una sucesión de identidades de cliente puede consumir concesiones del pool.
- Analizar el tráfico y distinguir la MAC de la trama Ethernet del campo `chaddr` del mensaje DHCP.
- Evaluar controles defensivos como DHCP Snooping, limitación de tasa y Port Security.
- Registrar resultados y evidencias sin afectar redes ajenas.

**Importante:** el agotamiento de un pool es una denegación de servicio. No ejecutar pruebas de inundación fuera de un laboratorio aislado ni contra sistemas sin autorización expresa.

## 2. Topología del laboratorio

La topología indicada para la práctica contiene tres máquinas virtuales conectadas a la misma **Red Interna** de VirtualBox:

| VM | Función | Adaptadores |
|---|---|---|
| Kali Linux | Equipo de pruebas | Red Interna |
| Debian/Ubuntu Server | Servidor DHCP (ISC DHCP Server o Kea) | NAT para administración/descargas y Red Interna para el laboratorio |
| Ubuntu/Debian Client | Cliente DHCP legítimo | Red Interna |

Configurar el nombre de la Red Interna igual en las tres máquinas. En los adaptadores conectados a esa red, revisar la política de modo promiscuo según las necesidades de observación del laboratorio. No conectar la interfaz de laboratorio a una red puenteada o a la LAN real.

> **Evidencia pendiente:** captura de la configuración de VirtualBox que muestre la Red Interna y los adaptadores de cada VM, ocultando datos ajenos al laboratorio.

### Preparación del servidor

El servidor DHCP debe tener una interfaz conectada a la Red Interna y un pool pequeño, reservado exclusivamente para la práctica. Antes de comenzar, anotar el nombre de la interfaz de laboratorio, la subred y el rango configurado. No reutilizar configuraciones de una red real.

El servidor debe estar instalado y configurado previamente, tal como pide el enunciado. Confirmar el estado del servicio y revisar su configuración antes de iniciar cualquier prueba:

```bash
sudo systemctl status isc-dhcp-server
ip -br address
```

Los nombres de interfaces y rutas de los ficheros de concesiones varían entre distribuciones y versiones. Confirmarlos en la máquina antes de utilizar comandos de comprobación.

## 3. Comprobación inicial

Primero comprobaré que el servicio asigna correctamente una dirección al cliente legítimo y que el cliente puede renovarla. En la máquina cliente, identificar la interfaz correcta con `ip -br link`; el enunciado utiliza `enp0s3` como ejemplo, pero el nombre puede variar.

```bash
sudo dhclient -v enp0s3
ip -br address
```

En el servidor revisaré el fichero de concesiones indicado por la instalación. En ISC DHCP suele ser `/var/lib/dhcp/dhcpd.leases`:

```bash
sudo tail -n 40 /var/lib/dhcp/dhcpd.leases
```

Anotaré la IP asignada, la MAC del cliente, la hora y el estado del servicio. Para observar el intercambio DORA se puede capturar únicamente la interfaz interna del laboratorio con Wireshark y utilizar el filtro de visualización `dhcp` o `bootp`.

> **Evidencia pendiente:** concesión inicial en el servidor y captura de DORA del cliente legítimo.

## 4. Práctica con Yersinia

Yersinia puede utilizarse en laboratorio para estudiar protocolos de capa 2, incluido DHCP. El enunciado propone su modo interactivo (`-I`) o gráfico (`-G`) y la selección de DHCP y de una opción de envío de solicitudes.

Antes de iniciar, confirmar que Kali está conectada únicamente a la Red Interna y que el servidor usa un pool de laboratorio pequeño. Realizar la prueba únicamente durante el intervalo y con los límites autorizados por el docente; detenerla inmediatamente si el tráfico sale del laboratorio o afecta a otra máquina.

La observación debe centrarse en el patrón de solicitudes, las identidades declaradas y los cambios en el estado del pool, no en lanzar tráfico contra una red real. El nombre de teclas y opciones puede variar por versión; seguir la documentación de la versión instalada y las instrucciones del profesor.

> **Evidencia pendiente:** captura de Yersinia mostrando la selección de protocolo/función en el laboratorio, sin datos de redes reales.

## 5. Práctica con Scapy

El enunciado plantea analizar un script de Scapy que genera mensajes DHCP Discover con identidades de cliente variables. Como este tipo de código puede provocar una denegación de servicio si se ejecuta en una red compartida, no incluyo aquí un bucle de inundación ni instrucciones para acelerar o ampliar el ataque.

Para documentar el concepto de forma segura, estudiaré en una captura controlada un único intercambio DHCP generado por un cliente legítimo y compararé:

- La dirección MAC de origen de la trama Ethernet.
- El campo `chaddr` dentro del mensaje BOOTP/DHCP.
- El tipo de mensaje DHCP y el identificador de transacción.

Si el docente requiere la demostración de agotamiento, debe proporcionarse un entorno efímero e aislado y parámetros de ejecución limitados. La prueba se realizará bajo supervisión, con un mecanismo de parada y recuperación del servicio y sin conexión a redes externas.

> **Evidencia pendiente:** captura de Wireshark con Ethernet y BOOTP/DHCP expandidos para comparar origen L2 y `chaddr`.

## 6. Validación del impacto

Tras la demostración autorizada, comprobaré desde el servidor cuántas concesiones existen y si el pool conserva direcciones disponibles. En ISC DHCP, revisar el fichero de leases y los registros del servicio; no asumir que contar líneas equivale siempre al número de concesiones activas, porque el fichero puede guardar estados históricos.

En el cliente legítimo, liberar y solicitar de nuevo una concesión solo si el docente ha confirmado que el laboratorio está preparado y que no se afectará a otros servicios:

```bash
sudo dhclient -r enp0s3
sudo dhclient -v enp0s3
```

Registraré el resultado: asignación normal, ausencia de oferta, mensaje de pool agotado o dirección de autoconfiguración si el sistema la utiliza. Un fallo de DHCP, por sí solo, no demuestra que exista un ataque; también puede deberse a errores de configuración, servicio detenido, interfaz incorrecta o problemas de conectividad.

> **Evidencia pendiente:** estado del pool y resultado de la solicitud del cliente, antes y después, en la red aislada.

## 7. Mitigaciones

### DHCP Snooping

DHCP Snooping es una función de switches gestionables que distingue puertos confiables (hacia el servidor/relay autorizado) de puertos no confiables (acceso de clientes). En general, filtra mensajes de servidor DHCP recibidos por puertos no confiables y puede mantener una tabla de bindings. La configuración exacta depende del fabricante y modelo.

Debe habilitarse cuidadosamente en la VLAN correspondiente, marcar como trusted únicamente el enlace legítimo hacia el servidor DHCP y verificar el funcionamiento de clientes y relays. Una asignación incorrecta de puertos confiables puede interrumpir el servicio.

### Port Security y límites de tasa

Port Security puede restringir cuántas direcciones MAC se aprenden en un puerto de acceso y generar acciones ante excesos. El límite debe ajustarse a la realidad del puerto (por ejemplo, teléfonos IP, puntos de acceso o hipervisores pueden necesitar más de una MAC).

También se pueden aplicar límites de tasa de mensajes DHCP en los puertos de acceso, cuando el switch lo soporta. Estas medidas actúan cerca del origen y suelen ser más apropiadas que depender únicamente de un filtro en el servidor.

### Linux e iptables

El enunciado propone `hashlimit` para limitar tráfico dirigido al puerto UDP 67. Es importante entender sus límites: una regla de firewall debe ubicarse en la interfaz y cadena correctas, y un límite por MAC/IP puede ser evadido si el identificador puede falsificarse. Además, las reglas no reemplazan DHCP Snooping ni controles de acceso de capa 2.

La sintaxis concreta depende de la versión de iptables y de los módulos disponibles. Antes de aplicar reglas en un servidor, validar en una máquina de laboratorio, conservar una vía de administración alternativa y comprobar los contadores y el servicio. Evitar copiar reglas a un servidor de producción sin revisión.

### Validación del campo `chaddr`

No debe asumirse que `deny unknown-clients;` en ISC DHCP valida la coincidencia entre la MAC Ethernet de origen y `chaddr`: esa directiva controla clientes no declarados según la configuración, y no equivale por sí sola a inspección de capa 2.

La validación de correspondencia L2/L7 requiere un mecanismo que pueda inspeccionar ambos campos, por ejemplo una función del switch, bridge o sistema de filtrado compatible. Verificar compatibilidad y sintaxis con la documentación de la versión instalada antes de proponer una regla.

## 8. Prueba de las defensas

Para evaluar una mitigación en el laboratorio:

1. Guardar la configuración inicial y confirmar que existe una ruta de recuperación.
2. Habilitar una sola defensa por vez (por ejemplo, DHCP Snooping en el switch virtual/gestionable, o una política de tasa en el entorno aislado).
3. Verificar que el cliente legítimo obtiene una concesión normal.
4. Observar registros, contadores y tabla de leases durante la prueba aprobada.
5. Confirmar que la defensa reduce las solicitudes anómalas sin bloquear clientes legítimos.
6. Restaurar el estado del laboratorio y documentar los cambios.

No borrar a mano el fichero de leases mientras el servicio está activo ni reiniciar el servicio de forma destructiva sin seguir el procedimiento de recuperación del laboratorio y la documentación del servidor DHCP.

> **Evidencia pendiente:** regla/ajuste defensivo, contadores antes y después y prueba exitosa del cliente legítimo.

## 9. Preguntas de análisis

### ¿Por qué se deben comparar la MAC Ethernet y `chaddr`?

La MAC Ethernet pertenece a la cabecera de capa 2 y `chaddr` es un dato del mensaje BOOTP/DHCP. Analizar ambos ayuda a detectar inconsistencias y a entender cómo el servidor identifica al cliente. Si solo cambia uno, los controles que inspeccionan el otro pueden observar una identidad repetida o discrepante; la respuesta exacta depende del servidor y de la infraestructura.

### ¿Qué papel tiene el modo promiscuo en VirtualBox?

El modo promiscuo afecta a qué tramas puede recibir/observar una interfaz virtual. No sustituye el aislamiento de red ni es una defensa contra el agotamiento. La visibilidad depende de la topología, del tipo de red virtual y de la configuración del hipervisor; debe probarse solo en la Red Interna del laboratorio.

### ¿Cómo podría ayudar un bridge Linux?

Un bridge permite aplicar controles de capa 2 al tráfico que atraviesa sus puertos. Para validar que la MAC Ethernet coincide con `chaddr`, se necesita una función capaz de inspeccionar el contenido DHCP y comparar ambos valores; no conviene inventar una regla genérica de `ebtables` sin verificar que la herramienta y versión soporten esa inspección.

### ¿Qué limitación tiene un límite por MAC de origen?

Un límite agrupado por MAC puede ser menos eficaz si la identidad L2 se falsifica o si hay muchos clientes detrás de un mismo enlace. En switches gestionables se complementa con DHCP Snooping, límites de tasa por puerto, Port Security, autenticación de acceso (por ejemplo, 802.1X cuando corresponda) y monitorización.

## 10. Conclusiones

La práctica permite relacionar el proceso DORA con el uso de leases y comprender por qué un pool limitado puede afectar a nuevos clientes si se consumen sus direcciones. La inspección conjunta de la trama Ethernet, `chaddr`, logs del servidor y tabla de concesiones aporta más información que observar únicamente la dirección IP del cliente.

La defensa debe aplicarse en varias capas: controles de switch como DHCP Snooping y Port Security, limitación de tasa, configuración correcta del servidor y monitorización. Los controles deben verificarse con un cliente legítimo para evitar que la mitigación provoque una interrupción del servicio.

## 11. Capturas y evidencias

Guardar aquí las capturas del laboratorio o añadirlas en esta carpeta si se crea posteriormente. No incluir credenciales, datos personales ni tráfico de redes reales.

| Nº | Evidencia | Estado |
|---:|---|---|
| 1 | Topología y adaptadores de VirtualBox en Red Interna | Pendiente |
| 2 | Concesión DHCP inicial del cliente legítimo | Pendiente |
| 3 | Intercambio DORA observado con Wireshark | Pendiente |
| 4 | Campos Ethernet y `chaddr` del paquete | Pendiente |
| 5 | Estado del pool/leases durante la prueba | Pendiente |
| 6 | Mitigación aplicada y contadores | Pendiente |
| 7 | Cliente legítimo funcionando tras la mitigación | Pendiente |

## 12. Referencias

- [Scapy — documentación oficial](https://scapy.readthedocs.io/)
- [Kali Linux — herramienta Scapy](https://www.kali.org/tools/scapy/)
- [RFC 2131 — Dynamic Host Configuration Protocol](https://www.rfc-editor.org/rfc/rfc2131)
- [RFC 3118 — Authentication for DHCP Messages](https://www.rfc-editor.org/rfc/rfc3118)

---

[← Volver al panel de actividades](../README.md#actividades)