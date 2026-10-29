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

He reutilizado las máquinas Deivid y dns2 de mi práctica DNSSEC para adaptar sus funciones al nuevo escenario.

| Máquina | Función anterior | Función para esta actividad | Dirección de referencia |
|---|---|---|---|
| Deivid | Servidor DNS principal con BIND9 | Servidor DNS con reenviador, DHCP y puerta de enlace | 192.168.6.100/24, comprobada en la ruta de la red interna |
| dns2 | Servidor DNS secundario con BIND9 | Cliente con interfaz gráfica y navegador | 192.168.6.101 en la práctica anterior; pendiente de adaptar a DHCP |

He conectado el servidor mediante NAT para su salida a Internet y mediante una red interna para comunicarse con el cliente. He preparado dns2 sin una salida NAT propia para que utilice Deivid como puerta de enlace cuando complete su configuración.

Mi topología prevista es:

Internet → NAT de VirtualBox → Deivid → red interna internet → dns2.

El nombre internet identifica mi red interna de VirtualBox; no significa que esa red proporcione acceso a Internet por sí sola.

## 3. Preparación del laboratorio

### 3.1 Conservación de mi práctica anterior

Antes de adaptar el laboratorio he creado una instantánea de cada máquina para conservar el estado de mi práctica DNSSEC. He mantenido separada su documentación en dnssec/.

### 3.2 Red interna del servidor Deivid

![Configuración de la red interna de Deivid](evidencias/01-red-interna-deivid.png)

He configurado el adaptador 2 de Deivid en modo Red interna con el nombre internet. He mantenido el adaptador 1 en NAT para la conexión exterior. De esta forma he separado la conexión hacia el cliente de la conexión hacia Internet.

### 3.3 Red interna del cliente dns2

![Configuración de la red interna de dns2](evidencias/02-red-interna-dns2.png)

He conectado el adaptador 2 de dns2 a la misma red interna internet y he deshabilitado su adaptador 1. Con esta distribución he preparado el cliente para utilizar el servidor como salida, sin una conexión NAT independiente. Todavía no he comprobado la conectividad final desde el cliente.

### 3.4 Diagnóstico del servidor

He revisado el estado del reenvío IPv4 y he obtenido inicialmente el valor 0, por lo que no estaba activado. También he comprobado que nftables estaba instalado y que el listado de reglas activas no mostraba ninguna regla.

En la tabla de rutas he identificado enp0s3 como interfaz de salida, con dirección 10.0.2.15 y puerta de enlace 10.0.2.2. He identificado enp0s8 como interfaz de la red 192.168.6.0/24, con dirección 192.168.6.100.

Al revisar BIND he encontrado configurados los reenviadores 1.1.1.1 y 1.0.0.1, la opción forward only, la recursión y el permiso de consulta para localhost y 192.168.6.0/24. He conservado esta configuración sin modificarla. Todavía no he probado la resolución externa para esta actividad.

En el listado de paquetes consultado no he encontrado isc-dhcp-server ni kea-dhcp4-server. He encontrado dnsmasq-base, pero no he considerado su presencia como una prueba de que DHCP esté configurado y funcionando.

### 3.5 Activación del reenvío y configuración NAT

![Comprobación del reenvío IPv4 y de la regla NAT en Deivid](evidencias/03-reenvio-nat-deivid.png)

He activado el reenvío IPv4 y he guardado el ajuste en /etc/sysctl.d/99-act8-forward.conf. En la comprobación posterior he obtenido el valor 1.

He creado la tabla act8_nat y una cadena postrouting con una regla de enmascaramiento para las conexiones de 192.168.6.0/24 que salen por enp0s3. He comprobado que nftables muestra la regla con masquerade y la prioridad srcnat.

Los contadores de la regla muestran 0 paquetes y 0 bytes en la salida que he obtenido. Por tanto, he comprobado que la regla está cargada, pero todavía no he demostrado que el cliente la utilice ni que tenga salida a Internet.

### 3.6 Configuración de la persistencia del NAT

![Archivo de reglas NAT y habilitación del servicio nftables](evidencias/04-persistencia-nat-deivid.png)

He guardado una copia del archivo anterior y he almacenado las reglas en /etc/nftables.conf. He validado el archivo sin obtener errores de sintaxis y he habilitado el servicio nftables para que cargue la configuración al arrancar.

En la comprobación final muestro el archivo con la tabla act8_nat y su regla de enmascaramiento, junto con el resultado enabled del servicio. He dejado configurada la persistencia del NAT, aunque todavía no he comprobado la carga de las reglas después de reiniciar. Esta evidencia tampoco demuestra por sí sola que el cliente tenga acceso a Internet.

### 3.7 Mi estado de avance

| Elemento | Estado de mi trabajo |
|---|---|
| Máquinas y funciones | He elegido Deivid como servidor y dns2 como cliente |
| Conservación del laboratorio DNSSEC | He creado las instantáneas de ambas máquinas |
| Conexiones de VirtualBox | He configurado la red interna y he subido las dos evidencias |
| Interfaces y rutas del servidor | He identificado enp0s3 como salida y enp0s8 como red interna |
| Reenvío IPv4 | He activado el ajuste y he comprobado el valor 1 |
| NAT | He cargado y listado la regla; todavía no he probado tráfico del cliente |
| Persistencia del NAT | He guardado y validado el archivo y he habilitado el servicio; todavía no he comprobado la carga tras reiniciar |
| DNS con reenviador | He revisado la configuración existente; todavía no he probado la resolución externa |
| DHCP | Todavía no lo he preparado ni probado |
| Adaptación de dns2 y salida a Internet | Todavía no las he completado ni verificado |
| Navegador, Wireshark y pruebas DNS/DoH | Todavía no los he preparado y probado para esta actividad |

### 3.8 Mis siguientes pasos

1. Prepararé DHCP en el servidor y adaptaré dns2 para recibir la configuración necesaria.
2. Comprobaré el DNS que utiliza dns2 y evitaré que su antigua configuración como secundario interfiera en las pruebas.
3. Verificaré la conectividad, la resolución externa y la salida a Internet desde el cliente.
4. Comprobaré la carga de las reglas después de reiniciar cuando realice la verificación de persistencia.
5. Realizaré las capturas DNS y DoH e incorporaré sus resultados.

## 4. Consultas DNS y análisis con Wireshark

Consultaré tres dominios de Internet poco habituales y controlaré la caché cuando sea necesario para evitar que una respuesta anterior oculte el tráfico que quiero analizar.

Utilizaré herramientas que generen consultas DNS o el navegador y capturaré el tráfico con Wireshark. Para cada petición explicaré el número real de paquetes observados, los protocolos, las direcciones, los puertos y si puedo leer el contenido de la consulta y de la respuesta.

No daré por supuesto un número fijo de paquetes: documentaré lo que obtenga en mis capturas.

### Mis resultados

Todavía no he seleccionado los dominios ni he realizado las consultas y capturas de esta actividad.

## 5. Recorrido de una consulta DNS

Prepararé un gráfico que muestre el recorrido real de mi consulta: desde dns2 hasta Deivid, el reenvío hacia Internet y la respuesta al cliente.

El esquema del apartado 2 representa mi topología de trabajo. Elaboraré el diagrama definitivo de la consulta después de comprobar la configuración y el tráfico.

## 6. DNS sobre HTTPS

Activaré DNS sobre HTTPS en el navegador de dns2 y mostraré en una captura el navegador y la opción concreta que he utilizado.

Repetiré desde el navegador las peticiones a los mismos dominios de la primera parte. Explicaré qué he hecho para observarlas con Wireshark, qué información puedo ver y qué diferencias encuentro respecto al DNS convencional.

Distinguiré lo que observo en el tráfico cifrado de una posible inspección del contenido descifrado, si llego a realizarla.

### Mis resultados

Todavía no he configurado DoH ni he realizado las capturas y la comparación.

## 7. Evidencias y conclusiones

He incorporado mis cuatro primeras capturas en evidencias/, junto con la explicación de la configuración realizada. Mantendré este formato para las siguientes pruebas: imagen y explicación en primera persona, sin duplicar los comandos que ya se lean en la captura.

| Evidencia | Archivo |
|---|---|
| Red interna del servidor | [01-red-interna-deivid.png](evidencias/01-red-interna-deivid.png) |
| Red interna del cliente | [02-red-interna-dns2.png](evidencias/02-red-interna-dns2.png) |
| Reenvío IPv4 y NAT del servidor | [03-reenvio-nat-deivid.png](evidencias/03-reenvio-nat-deivid.png) |
| Archivo de reglas y habilitación al arranque | [04-persistencia-nat-deivid.png](evidencias/04-persistencia-nat-deivid.png) |

He preparado la conexión de las máquinas, he activado el reenvío y el NAT del servidor y he configurado el guardado de las reglas y su carga al arrancar. Este avance todavía no demuestra el funcionamiento completo del escenario: me falta preparar DHCP, comprobar la resolución y el acceso a Internet desde dns2 y verificar la carga de las reglas tras reiniciar.

Mantendré separadas mi actividad anterior ACT2-dns-resolucion/ y mi documentación DNSSEC de dnssec/. Redactaré mis conclusiones finales cuando haya completado y verificado las pruebas DNS y DoH.

## 8. Entrega

Organizaré mi documentación y mis evidencias en GitHub y prepararé el informe en mi GitBook personal, ya que el enunciado solicita la URL de ese informe.

Todavía no he incorporado el enlace de entrega.
