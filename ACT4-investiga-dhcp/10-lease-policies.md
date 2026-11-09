# Políticas de lease

## Qué es un lease

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