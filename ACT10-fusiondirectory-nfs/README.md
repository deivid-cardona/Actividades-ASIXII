# ACT10 — Manual de operaciones: FusionDirectory, OpenLDAP y NFS

Profesor: Quim.

## Índice

1. [Objetivo](#1-objetivo)
2. [Entorno](#2-entorno)
3. [Administración con FusionDirectory](#3-administración-con-fusiondirectory)
4. [Despliegue y esquemas de FusionDirectory](#4-despliegue-y-esquemas-de-fusiondirectory)
   - [4.1. Arquitectura](#41-arquitectura)
   - [4.2. Archivos de trabajo](#42-archivos-de-trabajo)
   - [4.3. Conversión a LDIF](#43-conversión-a-ldif)
   - [4.4. Verificación de esquemas cargados](#44-verificación-de-esquemas-cargados)
   - [4.5. Incidencia de plugins](#45-incidencia-de-plugins)
5. [Despliegue y pruebas NFS](#5-despliegue-y-pruebas-nfs)
6. [Inventario de evidencias](#6-inventario-de-evidencias)
7. [Resultados y alcance](#7-resultados-y-alcance)

## 1. Objetivo

Documentar la instalación, configuración e integración de FusionDirectory con OpenLDAP y la publicación de un recurso NFS con permisos asociados al grupo `desarrollo`. El manual diferencia los procedimientos documentados de las comprobaciones visibles en las capturas.

## 2. Entorno

| Elemento | Valor |
|---|---|
| Servidor Linux | `ladapserver` |
| Dominio y base DN | `dc=hoostlock,dc=lan` |
| Rama de usuarios | `ou=usuarios` |
| Rama de grupos POSIX | `ou=grupos` |
| Contenedor web | `fusiondirectory` |
| Interfaz web | `http://127.0.0.1:8080/fd/` |
| Administrador web | `fd-admin` |
| Grupos existentes | `desarrollo`, `usuarios` |
| Grupo creado | `soporte` |
| Directorio de esquemas en el servidor | `/root/fd-schemas` |
| Servidor NFS utilizado | `192.168.111.54` |
| Recurso NFS | `/srv/nfs/desarrollo` |
| Identidad de prueba documentada | `dcardona`, UID `20001`, GID `20000` |

## 3. Administración con FusionDirectory

### 3.1. Acceso administrativo

He accedido a FusionDirectory con `fd-admin`. La pantalla principal muestra:

```text
Welcome System Administrator!
Signed in: fd-admin
```

![Acceso administrativo a FusionDirectory](evidencias/fusiondirectory-acceso-administrador.png)

### 3.2. Inspección del directorio

He revisado el árbol LDAP y las unidades organizativas antes de modificar la configuración.

![Inspección inicial del directorio](evidencias/fusiondirectory-inspeccion-inicial.png)

Tras la inicialización, he comprobado que el directorio queda accesible desde la interfaz web.

![Directorio tras la inicialización](evidencias/fusiondirectory-inspeccion-tras-inicializacion.png)

### 3.3. Rama de grupos POSIX

En Configuration → Plugins he configurado:

```text
POSIX groups RDN = ou=grupos
```

La rama de administración de grupos queda en `ou=grupos,dc=hoostlock,dc=lan`.

![Configuración de las ramas de usuarios y grupos](evidencias/fusiondirectory-configuracion-rama-usuarios.png)

### 3.4. Grupos existentes

En Groups and roles he comprobado que aparecen `desarrollo` y `usuarios`, tras configurar la rama de grupos POSIX.

### 3.5. Creación del grupo soporte

| Campo | Valor |
|---|---|
| Base | `ou=grupos,dc=hoostlock,dc=lan` |
| Name | `soporte` |
| Force GID | Desactivado |
| GID | Asignación automática |
| Group members | Sin miembros iniciales |
| System trust | `disabled` |

El nuevo grupo aparece junto a `desarrollo` y `usuarios`.

![Grupo soporte creado](evidencias/fusiondirectory-grupo-soporte-creado.png)

## 4. Despliegue y esquemas de FusionDirectory

### 4.1. Arquitectura

La interfaz web de FusionDirectory se ejecuta en el contenedor `fusiondirectory`. Las comprobaciones de archivos y del socket LDAP se han realizado desde la terminal del servidor Linux `ladapserver`.

| Componente | Papel documentado |
|---|---|
| `ladapserver` | Nombre del servidor; contiene el directorio de trabajo `/root/fd-schemas` |
| `fusiondirectory` | Contenedor de la interfaz web de administración |
| OpenLDAP | Directorio y configuración dinámica consultada mediante `ldapi:///` en el servidor |
| `fd-schema-source` | Fuente de esquemas citada en el procedimiento inicial; su estado actual no se ha verificado |

`ladapserver` no debe confundirse con un nombre de contenedor: el intento de ejecutar `docker exec` con ese nombre devolvió que no existía. Tampoco se ha encontrado `/root/fd-schemas` dentro del contenedor web.

Para revisar los contenedores desde el servidor se utiliza:

```bash
sudo docker ps -a
```

El borrador inicial indicaba `ldap://host.docker.internal:389` como URI de conexión. Las capturas A10 y A11 no verifican esa URI, por lo que no se presenta aquí como configuración final confirmada.

### 4.2. Archivos de trabajo

He comprobado la identidad del servidor y el contenido del directorio de esquemas con:

```bash
hostname
sudo ls -lah /root/fd-schemas
```

La salida identifica el equipo como `ladapserver` y muestra los archivos utilizados durante la preparación y conversión.

| Tipo | Ejemplos observados |
|---|---|
| Esquemas originales | `core-fd.schema`, `core-fd-conf.schema`, `template-fd.schema`, `autofs-fd-conf.schema`, `systems-fd-conf.schema`, `webservice-fd-conf.schema` |
| Archivos LDIF | `core-fd.ldif`, `core-fd-conf.ldif`, `template-fd.ldif`, `ldapns.ldif`, `autofs-fd-conf.ldif`, `systems-fd-conf.ldif`, `webservice-fd-conf.ldif`, `mail-fd-conf.ldif`, `sudo-fd-conf.ldif` |
| Configuraciones temporales | `slapd.conf.test`, `slapd.conf.plugins`, `slapd.conf.webservice`, `slapd.conf.pending` |
| Directorios de trabajo | `all-fd-schemas`, `pending` y directorios de conversión |

![A10 — Archivos de esquemas y LDIF en el servidor ladapserver](evidencias/A10-esquemas-copiados.png)

Esta evidencia acredita la presencia de los archivos en el servidor. No prueba por sí sola su origen ni su carga en OpenLDAP.

El procedimiento inicial documentaba la extracción desde una fuente de esquemas mediante este patrón, ejecutado desde el anfitrión Docker:

```bash
docker cp "fd-schema-source:/etc/ldap/schema/fusiondirectory/<esquema>.schema" ./<esquema>.schema
```

La captura A10 muestra el estado posterior de los archivos, no la ejecución de esa copia.

### 4.3. Conversión a LDIF

El procedimiento documentado utiliza los archivos `.schema` para preparar LDIF compatibles con la configuración dinámica de OpenLDAP. Los siguientes comandos son plantillas: los valores entre `< >` deben sustituirse por nombres reales y deben respetarse las dependencias entre esquemas. No es necesario repetir la carga para obtener las capturas.

Desde el directorio de trabajo del servidor, con permisos adecuados:

```bash
cd /root/fd-schemas
cp slapd.conf.test slapd.conf.<nombre>
printf '\ninclude /root/fd-schemas/<esquema>.schema\n' >> slapd.conf.<nombre>
TMP=$(mktemp -d /root/fd-schemas/conversion.XXXXXX)
slaptest -f "$PWD/slapd.conf.<nombre>" -F "$TMP"
```

El LDIF preparado para su importación contiene una estructura de este tipo:

```ldif
dn: cn=<esquema>,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: <esquema>
olcAttributeTypes: ...
olcObjectClasses: ...
```

El comando de carga documentado es:

```bash
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f <esquema>.ldif
```

La captura A10 muestra archivos y directorios asociados a esta preparación. La captura A11 verifica el resultado de la carga, no cada paso histórico de conversión.

### 4.4. Verificación de esquemas cargados

Desde la terminal del servidor `ladapserver` he consultado los esquemas existentes en OpenLDAP:

```bash
sudo ldapsearch -Q -Y EXTERNAL -H ldapi:/// \
  -b 'cn=schema,cn=config' -s one -LLL dn
```

La salida muestra los esquemas generales `core`, `cosine`, `nis` e `inetorgperson`, además de los siguientes nueve esquemas:

| Esquema | Entrada observada |
|---|---|
| `core-fd` | `cn={4}core-fd,cn=schema,cn=config` |
| `core-fd-conf` | `cn={5}core-fd-conf,cn=schema,cn=config` |
| `template-fd` | `cn={6}template-fd,cn=schema,cn=config` |
| `ldapns` | `cn={7}ldapns,cn=schema,cn=config` |
| `autofs-fd-conf` | `cn={8}autofs-fd-conf,cn=schema,cn=config` |
| `systems-fd-conf` | `cn={9}systems-fd-conf,cn=schema,cn=config` |
| `webservice-fd-conf` | `cn={10}webservice-fd-conf,cn=schema,cn=config` |
| `mail-fd-conf` | `cn={11}mail-fd-conf,cn=schema,cn=config` |
| `sudo-fd-conf` | `cn={12}sudo-fd-conf,cn=schema,cn=config` |

![A11 — Esquemas cargados en la configuración dinámica de OpenLDAP](evidencias/A11-esquemas-cargados.png)

Esta comprobación confirma que las entradas de esquema existen bajo `cn=schema,cn=config`. Los índices numéricos corresponden a este entorno. El nombre observado es `template-fd`, no `template-fd-conf`.

La configuración de la aplicación citada en la incidencia siguiente, `cn=config,ou=fusiondirectory,dc=hoostlock,dc=lan`, es una entrada distinta de la configuración dinámica del servidor LDAP.

### 4.5. Incidencia de plugins

#### Síntoma documentado

Durante el despliegue, el guardado de Configuration → Plugins devolvía errores de sintaxis, como:

```text
LDAP operation failed!
Object: cn=config,ou=fusiondirectory,dc=hoostlock,dc=lan
Error: Invalid syntax - objectClass: fdAutofsPluginConf
```

En las pruebas iniciales también se documentaron referencias a `fdSystemsPluginConf`, `fdWebservicePluginConf`, `fdMailPluginConf` y `fdSudoPluginConf`.

#### Diagnóstico y corrección documentados

Se identificó la ausencia de esquemas de configuración de plugins. El procedimiento consistió en recopilar los archivos `*-fd-conf.schema`, preparar su conversión con `slaptest`, generar los LDIF correspondientes y cargarlos con `ldapadd` mediante el socket local de OpenLDAP.

El borrador inicial recogía esta copia desde la fuente de esquemas:

```bash
docker cp "fd-schema-source:/etc/ldap/schema/fusiondirectory/." ./all-fd-schemas/
```

La existencia de archivos de conversión no sustituye a la comprobación de los esquemas cargados. La consulta del apartado 4.4 es la evidencia actual de su presencia en OpenLDAP.

#### Resultado y alcance de la evidencia

El procedimiento inicial registra que, después de cargar los esquemas pendientes, se pudo guardar la configuración y administrar los grupos. A11 confirma actualmente la presencia de los cinco esquemas de plugins citados.

No se dispone aquí de una captura del error original ni de una nueva confirmación de guardado. No se recrea el fallo ni se presenta A11 como prueba visual de ese guardado. La configuración de ramas y el grupo creado se documentan en el apartado 3.

## 5. Despliegue y pruebas NFS

### 5.1. Instalación

He instalado `nfs-kernel-server` en el servidor que publica el recurso.

![Instalación del servidor NFS](evidencias/nfs-01-instalacion-servidor.png)

### 5.2. Creación del recurso

```bash
sudo mkdir -p /srv/nfs/desarrollo
```

![Creación del recurso compartido](evidencias/nfs-02-creacion-recurso.png)

### 5.3. Grupo y permisos

He asignado el grupo `desarrollo` y permisos `770` al directorio:

```bash
sudo chown root:desarrollo /srv/nfs/desarrollo
sudo chmod 770 /srv/nfs/desarrollo
```

El propietario y el grupo disponen de permisos sobre el directorio; el resto no tiene permisos mediante estos bits.

![Permisos y grupo del recurso](evidencias/nfs-03-permisos-grupo.png)

### 5.4. Exportación

Configuración documentada en `/etc/exports`:

```text
/srv/nfs/desarrollo *(rw,sync,no_subtree_check)
```

Aplicación y revisión:

```bash
sudo exportfs -ra
sudo exportfs -v
```

![Configuración de exportaciones NFS](evidencias/nfs-04-configuracion-exports.png)

### 5.5. Estado del servicio

```bash
sudo systemctl status nfs-kernel-server --no-pager
```

La comprobación documenta el estado del servicio en el entorno de la actividad.

![Estado del servicio NFS](evidencias/nfs-05-servicio-nfs-activo.png)

### 5.6. Exportaciones desde Windows

Desde el cliente Windows:

```cmd
showmount -e 192.168.111.54
```

La salida documentada muestra `/srv/nfs/desarrollo` disponible para `*`.

![Consulta de exportaciones desde Windows](evidencias/nfs-06-exportaciones.png)

### 5.7. Montaje desde Windows

Desde CMD con los permisos necesarios:

```cmd
mount -o anon \\192.168.111.54\srv\nfs\desarrollo N:
```

El cliente confirma la conexión de la unidad `N:` al recurso.

![Montaje NFS en la unidad N](evidencias/nfs-07-montaje-cliente.png)

### 5.8. Prueba de escritura y lectura

```cmd
echo Prueba de acceso NFS > N:\prueba.txt
dir N:
type N:\prueba.txt
```

La prueba documenta la creación del archivo y la lectura del texto `Prueba de acceso NFS`.

![Creación y lectura del archivo de prueba](evidencias/nfs-08-acceso-permitido.png)

## 6. Inventario de evidencias

Las imágenes se encuentran en [evidencias/](evidencias/).

| Evidencia | Contenido | Enlace |
|---|---|---|
| Acceso web | Pantalla inicial | [Ver](evidencias/fusiondirectory-01-acceso-web.png) |
| Administrador | Sesión como `fd-admin` | [Ver](evidencias/fusiondirectory-acceso-administrador.png) |
| Requisitos | Comprobación previa | [Ver](evidencias/fusiondirectory-comprobacion-requisitos.png) |
| Asistente | Inicio de configuración | [Ver](evidencias/fusiondirectory-inicio-asistente.png) |
| Inspección inicial | Árbol LDAP inicial | [Ver](evidencias/fusiondirectory-inspeccion-inicial.png) |
| Inspección posterior | Árbol tras la inicialización | [Ver](evidencias/fusiondirectory-inspeccion-tras-inicializacion.png) |
| Ramas | Configuración de usuarios y grupos | [Ver](evidencias/fusiondirectory-configuracion-rama-usuarios.png) |
| Grupo soporte | Grupo creado | [Ver](evidencias/fusiondirectory-grupo-soporte-creado.png) |
| Docker | Captura disponible; no utilizada para acreditar servicios LDAP no verificados | [Ver](evidencias/fusiondirectory-09-contenedores.png) |
| A10 | Archivos de esquemas y LDIF en `ladapserver` | [Ver](evidencias/A10-esquemas-copiados.png) |
| A11 | Esquemas cargados en OpenLDAP | [Ver](evidencias/A11-esquemas-cargados.png) |
| NFS 01 | Instalación | [Ver](evidencias/nfs-01-instalacion-servidor.png) |
| NFS 02 | Creación del directorio | [Ver](evidencias/nfs-02-creacion-recurso.png) |
| NFS 03 | Grupo y permisos | [Ver](evidencias/nfs-03-permisos-grupo.png) |
| NFS 04 | Exportación | [Ver](evidencias/nfs-04-configuracion-exports.png) |
| NFS 05 | Estado del servicio | [Ver](evidencias/nfs-05-servicio-nfs-activo.png) |
| NFS 06 | Exportaciones desde Windows | [Ver](evidencias/nfs-06-exportaciones.png) |
| NFS 07 | Montaje en `N:` | [Ver](evidencias/nfs-07-montaje-cliente.png) |
| NFS 08 | Escritura y lectura | [Ver](evidencias/nfs-08-acceso-permitido.png) |

## 7. Resultados y alcance

- FusionDirectory permite administrar las ramas y grupos documentados.
- A10 verifica los archivos de trabajo de esquemas en el servidor `ladapserver`.
- A11 verifica las entradas de esquema cargadas en OpenLDAP, incluidos los cinco esquemas de configuración de plugins citados en la incidencia.
- La secuencia NFS documenta publicación, montaje desde Windows y creación y lectura de `prueba.txt`.

La identidad de prueba registrada en el manual es `dcardona`, con UID `20001` y GID `20000`. La prueba de escritura no acredita por sí sola el mapeo de esa identidad: el montaje mostrado utiliza `-o anon`. La verificación del mapeo y una prueba de acceso denegado a una identidad no autorizada deben documentarse por separado antes de dar por demostrada la segregación completa de acceso.

---

[← Volver al panel de actividades ASIXII](../README.md#actividades) · [Actividades de Quim](https://github.com/deivid-cardona/Base_de_datos-Quim#actividades)
