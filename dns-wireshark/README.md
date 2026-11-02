# Actividad 8 — Seguridad en DNS: Wireshark y DNS sobre HTTPS

## 1. Objetivo

He organizado esta práctica para documentar la preparación de mi laboratorio DNS, las evidencias de conectividad y consultas con Wireshark y la configuración de DNS sobre HTTPS en Firefox. He utilizado como referencia el enunciado ACT2-DNS-Wireshark y he mantenido esta actividad separada de mis prácticas de resolución DNS y DNSSEC.

## 2. Mi escenario de trabajo

He reutilizado las máquinas Deivid y dns2 del laboratorio DNSSEC y he creado una instantánea de cada una antes de adaptar su configuración.

| Máquina | Función en mi laboratorio |
|---|---|
| Deivid | Servidor BIND9 con reenviadores, servidor DHCP y puerta de enlace de la red interna |
| dns2 | Cliente con interfaz gráfica, Wireshark y Firefox |

He preparado el siguiente esquema de conexión:

```text
Internet
   |
NAT de VirtualBox
   |
Deivid
   |
Red interna: internet
   |
dns2
```

He utilizado `internet` como nombre de la red interna de VirtualBox. En Deivid he identificado `enp0s3` como interfaz exterior, con dirección `10.0.2.15` y puerta de enlace `10.0.2.2`, y `enp0s8` como interfaz interna, con dirección `192.168.6.100/24`.

## 3. Configuración de la red virtual

### 3.1. Servidor Deivid

He mantenido el adaptador 1 en NAT y he configurado el adaptador 2 en la red interna `internet` para separar la conexión exterior de la conexión con el cliente.

![01 — Red interna de Deivid](evidencias/01-red-interna-deivid.png)

### 3.2. Cliente dns2

He conectado el adaptador 2 de dns2 a la misma red interna y he deshabilitado su adaptador 1 para evitar una salida NAT independiente.

![02 — Red interna de dns2](evidencias/02-red-interna-dns2.png)

## 4. Configuración del servidor

### 4.1. DNS con BIND9

He conservado los reenviadores `1.1.1.1` y `1.0.0.1`, el modo `forward only`, la recursión y el permiso de consulta para `localhost` y `192.168.6.0/24`.

### 4.2. Reenvío IPv4 y NAT

He activado el reenvío IPv4 y he guardado el ajuste en `/etc/sysctl.d/99-act8-forward.conf`. He comprobado que su valor es `1`.

He creado la tabla `act8_nat` y una cadena `postrouting` con prioridad `srcnat`. He configurado una regla `masquerade` para el tráfico de `192.168.6.0/24` que sale por `enp0s3`.

![03 — Reenvío IPv4 y regla NAT](evidencias/03-reenvio-nat-deivid.png)

### 4.3. Configuración de carga al arranque

He guardado una copia del archivo anterior, he almacenado las reglas en `/etc/nftables.conf`, he validado el archivo sin errores y he habilitado `nftables` para su carga al arrancar. En esta comprobación he obtenido el estado `enabled`.

![04 — Archivo de reglas y habilitación de nftables](evidencias/04-persistencia-nat-deivid.png)

### 4.4. Servicio DHCP

He instalado `kea-dhcp4-server`, he conservado una copia de la configuración original en `/etc/kea/kea-dhcp4.conf.antes-act8` y he preparado `/etc/kea/kea-dhcp4.conf` con estos parámetros:

| Parámetro | Valor que he configurado |
|---|---|
| Interfaz | `enp0s8` |
| Subred | `192.168.6.0/24` |
| Identificador de subred | `1` |
| Rango de asignación | `192.168.6.150–192.168.6.180` |
| Opción de puerta de enlace | `192.168.6.100` |
| Opción de servidor DNS | `192.168.6.100` |
| Duración de la concesión | `3600` segundos |
| Almacenamiento | `memfile` con persistencia activada |

He dejado fuera del rango la dirección del servidor y la antigua dirección de dns2, `192.168.6.101`.

Durante la validación inicial como `root` he recibido `Unable to open file`. He repetido la validación como `_kea` y he obtenido el reconocimiento de la subred y la interfaz sin errores de configuración. Después he reiniciado el servicio y he comprobado su estado `active (running)`.

![05 — Estado del servicio DHCP](evidencias/05-servicio-dhcp-deivid.png)

## 5. Evidencias del cliente y de Wireshark

### 5.1. Conectividad de dns2

He incorporado la captura de conectividad del cliente como evidencia 06.

![06 — Conectividad de dns2](evidencias/06-conectividad-dns2.png)

### 5.2. Consultas DNS

He incorporado la captura de consultas DNS en Wireshark como evidencia 07.

![07 — Consultas DNS en Wireshark](evidencias/07-consultas-dns-wireshark.png)

## 6. DNS sobre HTTPS en Firefox

He abierto la configuración de DNS sobre HTTPS desde `about:preferences#dnsOverHttps` en Firefox. He seleccionado `Personalizado`, he mantenido `Cloudflare (predeterminado)` como proveedor y he marcado `Avisarme siempre si el DNS seguro no está disponible`.

He comprobado que Firefox muestra el aviso verde: «DNS sobre HTTPS está utilizando el proveedor Cloudflare».

![08 — DNS sobre HTTPS con Cloudflare en Firefox](evidencias/08-configuracion-doh-firefox.png)

## 7. Conclusiones

He configurado la red virtual, el reenvío IPv4 y el enmascaramiento del servidor. He guardado las reglas de red, he habilitado su carga al arranque y he comprobado que Kea DHCPv4 se encuentra en ejecución.

He reunido las evidencias del cliente y de las consultas DNS en sus apartados correspondientes. En Firefox he comprobado el estado de DNS sobre HTTPS con Cloudflare mediante el aviso mostrado por el navegador.

He organizado las ocho capturas en orden de ejecución y he integrado cada imagen una sola vez junto a su explicación. He conservado los comandos visibles en las capturas sin duplicarlos como transcripciones en el informe.
