# ACT4 — Investigación: seguridad en el servicio DHCP

**Módulo:** 037 — Seguridad en servicios  
**Ciclo formativo:** ASIX  
**Alumno:** Deivid Cardona  
**Formato de entrega:** URL al informe documentado en GitBook personal.

## Objetivo

En esta actividad voy a realizar un análisis teórico del protocolo DHCPv4 y de los riesgos de seguridad asociados a su uso. El objetivo es comprender cómo funciona el servicio, qué información utiliza para identificar a los clientes, qué vulnerabilidades presenta y qué medidas pueden aplicarse para proteger una red.

Esta actividad es principalmente de **investigación y análisis**. La simulación práctica del ataque se desarrolla en la actividad ACT5.

## Contenido

1. [Introducción](01-introduccion.md)
2. [Fundamentos de DHCP](02-fundamentos-dhcp.md)
3. [Proceso DORA](03-proceso-dora.md)
4. [Anatomía del paquete DHCP](04-anatomia-paquete.md)
5. [Autenticación y RFC](05-autenticacion-rfc.md)
6. [DHCP Starvation teórico](06-starvation-teorico.md)
7. [Consecuencias y APIPA](07-consecuencias-apipa.md)
8. [Hardening de infraestructura](08-hardening-infraestructura.md)
9. [Hardening en Linux](09-hardening-linux.md)
10. [Políticas de lease](10-lease-policies.md)
11. [Modo promiscuo](11-modo-promiscuo.md)
12. [Aspectos legales](12-aspectos-legales.md)
13. [Conclusiones](13-conclusiones.md)

## Estructura

```text
ACT4-investiga-dhcp/
├── README.md
├── 01-introduccion.md
├── 02-fundamentos-dhcp.md
├── 03-proceso-dora.md
├── 04-anatomia-paquete.md
├── 05-autenticacion-rfc.md
├── 06-starvation-teorico.md
├── 07-consecuencias-apipa.md
├── 08-hardening-infraestructura.md
├── 09-hardening-linux.md
├── 10-lease-policies.md
├── 11-modo-promiscuo.md
├── 12-aspectos-legales.md
├── 13-conclusiones.md
├── evidencias/
│   └── README.md
└── gitbook/
    └── README.md
```

## Aviso ético

El análisis teórico y las posibles pruebas deben realizarse únicamente en entornos propios y controlados. No deben ejecutarse contra redes domésticas, educativas, corporativas ni redes de terceros.