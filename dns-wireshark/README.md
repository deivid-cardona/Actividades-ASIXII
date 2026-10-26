

## Índice

1. [Objetivo](#1-objetivo)
2. [Escenario requerido](#2-escenario-requerido)
3. [Consultas DNS y análisis con Wireshark](#3-consultas-dns-y-análisis-con-wireshark)
4. [Diagrama del recorrido DNS](#4-diagrama-del-recorrido-dns)
5. [DNS sobre HTTPS](#5-dns-sobre-https)
6. [Evidencias y conclusiones](#6-evidencias-y-conclusiones)
7. [Entrega](#7-entrega)

## 1. Objetivo

Analizar e implementar la seguridad en el servicio DNS, observando el tráfico con Wireshark y comparando las consultas DNS convencionales con las realizadas mediante DNS sobre HTTPS.

## 2. Escenario requerido

El enunciado requiere dos máquinas virtuales:

- Cliente conectado a una red interna.
- Servidor con dos adaptadores: uno conectado a la red interna y otro en modo puente o NAT para la salida a Internet.

El servidor debe reenviar el tráfico de la red interna hacia Internet y disponer de un servicio DHCP y un servicio DNS con reenviador.

La configuración concreta, las direcciones IP y las comprobaciones se incorporarán cuando estén verificadas.

## 3. Consultas DNS y análisis con Wireshark

### Requisitos

- Realizar peticiones a tres dominios distintos de Internet poco habituales para evitar respuestas de consultas anteriores en caché.
- Utilizar ping, nslookup, otros comandos que generen tráfico DNS o el navegador.
- Capturar el tráfico con Wireshark.
- Explicar para cada petición el número de paquetes observado, los protocolos, los puertos utilizados y si los datos están cifrados.
- Incluir las capturas y los resultados reales de las pruebas.

### Resultados

Pendientes de documentar.

## 4. Diagrama del recorrido DNS

Incluir un gráfico que represente el recorrido de una petición desde el cliente hasta recibir la respuesta, ajustado a la configuración utilizada.

Pendiente de incorporar.

## 5. DNS sobre HTTPS

### Requisitos

- Configurar el navegador de la máquina cliente para utilizar DNS sobre HTTPS.
- Identificar el navegador y mostrar dónde se activa la configuración.
- Repetir las consultas anteriores.
- Explicar qué se ha hecho para observar las peticiones en Wireshark y qué muestra la captura.
- Documentar las diferencias observadas respecto a las consultas DNS convencionales.

### Resultados

Pendientes de documentar.

## 6. Evidencias y conclusiones

Las capturas, los datos de las pruebas y las conclusiones se añadirán conforme se realicen y verifiquen. No se reutilizan las evidencias de la actividad Resolución DNS.

## 7. Entrega

El enunciado solicita la URL del informe documentado en el GitBook personal.

Enlace de entrega pendiente.
