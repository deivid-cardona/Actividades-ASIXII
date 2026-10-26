# Securización de servidores DNS con DNSSEC

- Módulo: Alina.
- Carpeta: dnssec.
- Estado: documentación inicial; implementación y pruebas pendientes de documentar.

## Índice

1. [Introducción](#1-introducción)
2. [Objetivos](#2-objetivos)
   - [Objetivo general](#21-objetivo-general)
   - [Objetivos específicos](#22-objetivos-específicos)
3. [Topología y escenario de partida](#3-topología-y-escenario-de-partida)

## 1. Introducción

Esta actividad tiene como finalidad implementar DNSSEC en una topología formada por un servidor DNS principal —máster—, un servidor secundario —slave— y un cliente desde el que se realizarán las pruebas.

Partiendo de los problemas de seguridad expuestos en el enunciado, se documentará la configuración de los servidores, la implementación de DNSSEC y las comprobaciones necesarias para evaluar su funcionamiento.

El informe recogerá el procedimiento seguido, los cambios de configuración y los resultados obtenidos. Cada captura estará acompañada de una explicación de lo realizado y de lo que demuestra, evitando presentar las imágenes como única documentación.

Las configuraciones y las pruebas se incorporarán a medida que se realicen y verifiquen, sin dar por completado ningún resultado de antemano.

## 2. Objetivos

### 2.1 Objetivo general

Implementar y comprobar DNSSEC en la topología existente, formada por un servidor DNS máster, un servidor slave y un cliente.

### 2.2 Objetivos específicos

1. Documentar la topología y la configuración inicial de los equipos.
2. Implementar DNSSEC y documentar la configuración aplicada en los servidores.
3. Comprobar el funcionamiento del servidor principal y del secundario tras los cambios.
4. Realizar pruebas desde el cliente y explicar los resultados obtenidos.
5. Recoger las consideraciones del enunciado sobre consumo de recursos, rotación de claves y expiración de firmas.
6. Organizar los comandos, configuraciones y capturas, acompañando cada evidencia de su explicación.

## 3. Topología y escenario de partida

Esta actividad reutiliza los dos servidores DNS y el equipo cliente de la práctica anterior de BIND9. El servidor máster y el servidor slave constituyen la base sobre la que se implementará DNSSEC, mientras que el cliente se utilizará para realizar las consultas y comprobar los resultados.

### Equipos utilizados

| Equipo | Función en la actividad |
|---|---|
| Servidor máster | Servidor DNS principal de la práctica anterior de BIND9. |
| Servidor slave | Servidor DNS secundario de la práctica anterior de BIND9. |
| Cliente | Equipo desde el que se realizarán las consultas y las pruebas. |

Antes de aplicar cambios, se verificará la configuración existente y se documentarán los nombres de los equipos, sus direcciones IP y las zonas DNS utilizadas.

No se presupone ninguna dirección IP, nombre de equipo o zona DNS hasta comprobar los datos de la práctica anterior.
