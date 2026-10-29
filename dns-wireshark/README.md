# Actividad 8 — Seguridad en DNS: análisis con Wireshark y DNS sobre HTTPS

## Índice

1. [Objetivo](#1-objetivo)
2. [Escenario y máquinas reutilizadas](#2-escenario-y-máquinas-reutilizadas)
3. [Preparación y comprobaciones iniciales](#3-preparación-y-comprobaciones-iniciales)
4. [Consultas DNS y análisis con Wireshark](#4-consultas-dns-y-análisis-con-wireshark)
5. [Diagrama del recorrido DNS](#5-diagrama-del-recorrido-dns)
6. [DNS sobre HTTPS](#6-dns-sobre-https)
7. [Evidencias y conclusiones](#7-evidencias-y-conclusiones)
8. [Entrega](#8-entrega)

## 1. Objetivo

Analizar la seguridad del servicio DNS mediante capturas de Wireshark y comparar las consultas DNS convencionales con las realizadas mediante DNS sobre HTTPS (DoH).

Esta actividad corresponde a ACT8 en el índice del repositorio, aunque el documento del enunciado se denomina ACT2-DNS-Wireshark.

## 2. Escenario y máquinas reutilizadas

Reutilizaré las máquinas Deivid y dns2 de la práctica anterior de DNSSEC, adaptando sus funciones a este enunciado.

| Máquina | Función anterior | Función acordada para esta actividad | IP de referencia anterior |
|---|---|---|---|
| Deivid | Servidor DNS principal con BIND9 | Servidor DNS con reenviador, DHCP y puerta de enlace | 192.168.6.100/24 |
| dns2 | Servidor DNS secundario con BIND9 | Cliente con interfaz gráfica y navegador para las pruebas DNS y DoH | 192.168.6.101 |

Las direcciones anteriores son referencias del laboratorio DNSSEC, no una confirmación de la configuración actual ni de la dirección final del cliente. El cliente deberá recibir la configuración del servicio DHCP que preparemos.

### Topología prevista

```text
Internet
   |
Adaptador NAT o puente
   |
Deivid: servidor DNS, DHCP y puerta de enlace
   |
Red interna de VirtualBox
   |
dns2: cliente de pruebas
```

El servidor tendrá dos adaptadores: uno en la red interna y otro en NAT o puente. El cliente estará conectado a la misma red interna y utilizará el servidor para salir a Internet, sin una salida NAT propia que evite este recorrido.

En la última práctica documentada, el adaptador 2 de ambas máquinas estaba en una red solo-anfitrión para permitir las consultas desde Windows. Por tanto, antes de cambiar nada comprobaré su estado actual y después lo adaptaré a la red interna exigida.

No doy por configurados DHCP, el reenviador DNS ni el encaminamiento: su estado está pendiente de comprobación.

## 3. Preparación y comprobaciones iniciales

### Orden de trabajo

1. Comprobar las interfaces, direcciones, rutas, resolución DNS y servicios de ambas máquinas.
2. Guardar el estado del laboratorio DNSSEC antes de modificarlo; queda pendiente decidir y realizar el respaldo o las instantáneas.
3. Adaptar los adaptadores de VirtualBox a la topología requerida.
4. Revisar y preparar DHCP, DNS con reenviador y la salida a Internet a través de Deivid.
5. Adaptar dns2 a su función de cliente, comprobando el DNS que utiliza y evitando que su configuración anterior como secundario interfiera en las pruebas.
6. Verificar la conectividad, la asignación DHCP y la resolución desde el cliente.
7. Realizar las capturas DNS convencionales y las pruebas DoH.
8. Incorporar las evidencias verificadas, el diagrama y las conclusiones.

### Comandos de comprobación propuestos

Ejecutar en cada máquina y conservar las salidas identificando el equipo:

```bash
hostname
ip -4 -br addr
ip -4 route
cat /etc/resolv.conf
systemctl is-active named
systemctl is-active isc-dhcp-server
systemctl is-active kea-dhcp4-server
sysctl net.ipv4.ip_forward
```

Estos comandos consultan el estado; no modifican la configuración. Que un servicio no exista o aparezca inactivo se registrará como parte del diagnóstico inicial.

### Estado actual del trabajo

| Elemento | Estado |
|---|---|
| Máquinas y funciones | Acordadas: Deivid servidor y dns2 cliente |
| Apartado del repositorio | Confirmado: dns-wireshark/ |
| Estado actual de interfaces, IP y rutas | Pendiente de comprobar |
| Respaldo del laboratorio anterior | Pendiente |
| Adaptación a red interna | Pendiente |
| DHCP, DNS con reenviador y encaminamiento | Pendientes de verificar y preparar |
| Navegador y Wireshark del cliente | Pendientes de comprobar |
| Capturas y resultados | Pendientes |

## 4. Consultas DNS y análisis con Wireshark

### Requisitos

- Realizar peticiones a tres dominios distintos de Internet poco habituales para reducir la posibilidad de respuestas previamente almacenadas en caché.
- Utilizar ping, nslookup, otros comandos que generen tráfico DNS o el navegador.
- Comprobar y controlar la caché antes de las pruebas cuando sea necesario.
- Capturar el tráfico con Wireshark.
- Explicar para cada petición el número de paquetes observado, los protocolos, las direcciones, los puertos utilizados y si los datos están cifrados.
- Incluir las capturas y los resultados reales, sin dar por supuesto un número fijo de paquetes.

### Resultados

Dominios, consultas, capturas y análisis pendientes de realizar y documentar.

## 5. Diagrama del recorrido DNS

El esquema del apartado 2 representa la topología prevista. El gráfico definitivo mostrará el recorrido de una consulta desde dns2 hasta Deivid, el reenvío hacia Internet y la respuesta al cliente, ajustándose al tráfico observado.

Pendiente de incorporar tras verificar el escenario.

## 6. DNS sobre HTTPS

### Requisitos

- Configurar el navegador de dns2 para utilizar DNS sobre HTTPS.
- Identificar el navegador y mostrar la ruta y la opción de configuración activada.
- Repetir desde el navegador las peticiones a los mismos dominios utilizados antes.
- Explicar qué se ha hecho para observar las peticiones en Wireshark y qué información muestra la captura.
- Distinguir el tráfico cifrado visible de una eventual inspección del contenido descifrado, si se realiza.
- Documentar las diferencias observadas respecto a las consultas DNS convencionales.

### Resultados

Configuración, procedimiento de captura y comparación pendientes de realizar y documentar.

## 7. Evidencias y conclusiones

Las nuevas capturas se organizarán en evidencias/ y se incorporarán junto a su explicación cuando se realicen y verifiquen las pruebas. No se reutilizarán capturas de otras actividades como si fueran resultados de esta práctica.

La actividad anterior ACT2-dns-resolucion/ y la documentación DNSSEC de dnssec/ se mantendrán separadas.

Conclusiones pendientes: se redactarán a partir de los resultados reales, sin presentar la planificación como trabajo completado.

## 8. Entrega

El enunciado solicita la URL del informe documentado en el GitBook personal. GitHub se utilizará para organizar la documentación y las evidencias de esta actividad.

Enlace de entrega pendiente.
