# Actividad 8 — Seguridad en DNS: análisis con Wireshark y DNS sobre HTTPS

## Índice

1. [Objetivo](#1-objetivo)
2. [Mi escenario de trabajo](#2-mi-escenario-de-trabajo)
3. [Preparación del laboratorio](#3-preparación-del-laboratorio)
4. [Consultas DNS y análisis con Wireshark](#4-consultas-dns-y-análisis-con-wireshark)
5. [Recorrido de una consulta DNS](#5-recorrido-de-una-consulta-dns)
6. [DNS sobre HTTPS](#6-dns-sobre-https)
7. [Evidencias y conclusiones](#7-evidencias-y-conclusiones)
8. [Entrega](#8-entrega)

## 1. Objetivo

En esta actividad analizaré el tráfico DNS con Wireshark y compararé las consultas convencionales con las realizadas mediante DNS sobre HTTPS (DoH).

He ubicado esta práctica en ACT8 de mi repositorio. El documento del enunciado se denomina ACT2-DNS-Wireshark.

## 2. Mi escenario de trabajo

He decidido reutilizar las máquinas Deivid y dns2 de mi práctica DNSSEC y adaptar sus funciones al nuevo escenario.

| Máquina | Función en mi práctica anterior | Función que he previsto para esta actividad | IP de referencia anterior |
|---|---|---|---|
| Deivid | Servidor DNS principal con BIND9 | Servidor DNS con reenviador, DHCP y puerta de enlace | 192.168.6.100/24 |
| dns2 | Servidor DNS secundario con BIND9 | Cliente con interfaz gráfica y navegador para mis pruebas DNS y DoH | 192.168.6.101 |

Utilizo estas IP únicamente como referencia de mi laboratorio anterior. Todavía no he comprobado la configuración actual ni he definido la dirección final del cliente. Prepararé el cliente para recibir su configuración por DHCP.

### Topología que prepararé

Internet → adaptador NAT o puente → Deivid → red interna → dns2.

Configuraré Deivid con dos adaptadores: uno en la red interna y otro en NAT o puente. Conectaré dns2 a la misma red interna y comprobaré que sale a Internet a través de Deivid, sin utilizar una salida NAT propia.

En mi última práctica documentada utilicé una red solo-anfitrión en el adaptador 2 de ambas máquinas para consultar los servidores desde Windows. Antes de modificarla, comprobaré cómo están conectadas actualmente.

Todavía no he verificado DHCP, el reenviador DNS ni el encaminamiento para esta actividad.

## 3. Preparación del laboratorio

### Mi orden de trabajo

1. Comprobaré las interfaces, las IP, las rutas, la resolución DNS y los servicios de ambas máquinas.
2. Guardaré el estado de mi laboratorio DNSSEC antes de modificarlo. Todavía tengo pendiente elegir y realizar el respaldo o las instantáneas.
3. Adaptaré los adaptadores de VirtualBox a la red interna que exige el enunciado.
4. Revisaré y prepararé DHCP, DNS con reenviador y la salida a Internet mediante Deivid.
5. Adaptaré dns2 a su función de cliente y comprobaré que su configuración anterior como secundario no interfiere en las pruebas.
6. Verificaré la asignación DHCP, la conectividad y la resolución desde dns2.
7. Realizaré las capturas DNS convencionales y las pruebas DoH.
8. Incorporaré mis evidencias verificadas, el diagrama y las conclusiones.

### Mis primeras evidencias previstas

| Captura que prepararé | Contenido que mostraré | Nombre previsto |
|---|---|---|
| Estado inicial de Deivid | Nombre del equipo, interfaces, IP y rutas, con los comandos y sus resultados legibles | 01-estado-inicial-deivid.png |
| Estado inicial de dns2 | Nombre del equipo, interfaces, IP y rutas, con los comandos y sus resultados legibles | 02-estado-inicial-dns2.png |

Todavía no he realizado estas capturas. Si los resultados no caben de forma legible en una imagen, los separaré en varias evidencias. Documentaré la resolución DNS y los servicios en las capturas adicionales que necesite tras revisar las salidas.

### Mi estado de avance

| Elemento | Estado de mi trabajo |
|---|---|
| Máquinas y funciones | He decidido utilizar Deivid como servidor y dns2 como cliente |
| Apartado de documentación | He ubicado la actividad en dns-wireshark/ |
| Interfaces, IP, rutas y servicios actuales | Todavía no los he comprobado para esta actividad |
| Respaldo del laboratorio anterior | Todavía no lo he realizado |
| Adaptación a red interna | Todavía no la he realizado |
| DHCP, DNS con reenviador y encaminamiento | Todavía no los he verificado ni preparado para esta actividad |
| Navegador y Wireshark | Todavía no he comprobado su disponibilidad y configuración para las pruebas |
| Capturas y resultados | Todavía no los he incorporado |

## 4. Consultas DNS y análisis con Wireshark

Consultaré tres dominios de Internet poco habituales y controlaré la caché cuando sea necesario para evitar que una respuesta anterior oculte el tráfico que quiero analizar.

Utilizaré herramientas que generen consultas DNS o el navegador y capturaré el tráfico con Wireshark. Para cada petición explicaré el número real de paquetes observados, los protocolos, las direcciones, los puertos y si puedo leer el contenido de la consulta y de la respuesta.

No daré por supuesto un número fijo de paquetes: documentaré lo que obtenga en mis capturas.

### Mis resultados

Todavía no he seleccionado los dominios ni he realizado las consultas y capturas de esta actividad.

## 5. Recorrido de una consulta DNS

Prepararé un gráfico que muestre el recorrido real de mi consulta: desde dns2 hasta Deivid, el reenvío hacia Internet y la respuesta al cliente.

El esquema del apartado 2 representa únicamente mi topología prevista. Elaboraré el diagrama definitivo después de comprobar la configuración y el tráfico.

## 6. DNS sobre HTTPS

Activaré DNS sobre HTTPS en el navegador de dns2 y mostraré en una captura el navegador y la opción concreta que he utilizado.

Repetiré desde el navegador las peticiones a los mismos dominios de la primera parte. Explicaré qué he hecho para observarlas con Wireshark, qué información puedo ver y qué diferencias encuentro respecto al DNS convencional.

Distinguiré lo que observo en el tráfico cifrado de una posible inspección del contenido descifrado, si llego a realizarla.

### Mis resultados

Todavía no he configurado DoH ni he realizado las capturas y la comparación.

## 7. Evidencias y conclusiones

Guardaré las capturas de esta práctica en evidencias/. En cada una mostraré el comando o la configuración relevante junto con su resultado, de forma legible y con el equipo identificado.

Debajo de cada imagen explicaré en primera persona qué he comprobado, qué datos veo y qué conclusión puedo obtener. No repetiré en bloques de texto los comandos que ya se lean en las capturas. Solo añadiré información complementaria cuando sea necesaria para entender el procedimiento.

Incorporaré las evidencias después de revisar sus resultados. No presentaré una comprobación prevista como si ya la hubiera realizado ni reutilizaré capturas de otras actividades como resultados de esta práctica.

Mantendré separadas mi actividad anterior ACT2-dns-resolucion/ y mi documentación DNSSEC de dnssec/.

Redactaré mis conclusiones finales cuando haya completado y verificado las pruebas.

## 8. Entrega

Organizaré mi documentación y mis evidencias en GitHub y prepararé el informe en mi GitBook personal, ya que el enunciado solicita la URL de ese informe.

Todavía no he incorporado el enlace de entrega.
