# ACT10 — Manual de operaciones: FusionDirectory, OpenLDAP y NFS

Profesor: Quim.

## 1. Objetivo y organización del manual

He estructurado este manual procedimental para documentar la instalación, configuración e integración de FusionDirectory sobre OpenLDAP y su posterior vinculación con recursos NFS restringidos mediante grupos del directorio.

El documento se organiza en tres partes:

- **Avance ejecutado:** operaciones ya realizadas y verificadas.
- **Incidencias y resolución:** errores encontrados, diagnóstico y corrección aplicada.
- **Pendientes y evidencias futuras:** tareas restantes y espacio reservado para nuevas capturas.

## 2. Contexto del entorno

La actividad se desarrolla sobre la infraestructura previa de OpenLDAP, utilizando FusionDirectory como interfaz de administración web del directorio.

| Elemento | Valor utilizado |
|---|---|
| Dominio LDAP | `dc=hoostlock,dc=lan` |
| Base DN | `dc=hoostlock,dc=lan` |
| Rama de usuarios | `ou=usuarios` |
| Rama de grupos POSIX | `ou=grupos` |
| Interfaz web | `http://127.0.0.1:8080/fd/` |
| Usuario administrador | `fd-admin` |
| Grupos existentes | `desarrollo`, `usuarios` |
| Grupo creado | `soporte` |

## 3. Avance ejecutado

### 3.1. Acceso a FusionDirectory

He accedido correctamente a la interfaz web de FusionDirectory con el usuario `fd-admin`.

La pantalla principal confirma el acceso administrativo:

```text
Welcome System Administrator!
Signed in: fd-admin
```

![Acceso como administrador](evidencias/fusiondirectory-acceso-administrador.png)

### 3.2. Revisión inicial del directorio

Antes de modificar la configuración, he comprobado el estado inicial del árbol LDAP y la presencia de las unidades organizativas y grupos existentes.

![Inspección inicial](evidencias/fusiondirectory-inspeccion-inicial.png)

Tras la inicialización, he verificado que el directorio quedaba accesible para la administración web.

![Inspección tras inicialización](evidencias/fusiondirectory-inspeccion-tras-inicializacion.png)

### 3.3. Configuración de la rama de grupos POSIX

He accedido a **Configuration → Plugins** y he configurado el parámetro:

```text
POSIX groups RDN = ou=grupos
```

Con este ajuste, FusionDirectory localiza los grupos POSIX en:

```text
ou=grupos,dc=hoostlock,dc=lan
```

![Configuración de la rama de usuarios y grupos](evidencias/fusiondirectory-configuracion-rama-usuarios.png)

### 3.4. Verificación de grupos existentes

Después de guardar la configuración, he accedido a **Groups and roles** y he comprobado que FusionDirectory muestra correctamente los grupos POSIX existentes:

- `desarrollo`
- `usuarios`

Esto confirma que la ruta `ou=grupos` está correctamente asociada a la administración de grupos POSIX.

### 3.5. Creación del grupo `soporte`

He creado un nuevo grupo POSIX con los siguientes valores:

| Campo | Valor |
|---|---|
| Base | `ou=grupos,dc=hoostlock,dc=lan` |
| Name | `soporte` |
| Force GID | Desactivado |
| GID | Asignación automática |
| Group members | Sin miembros iniciales |
| System trust | `disabled` |

El grupo ha quedado visible en la lista junto a `desarrollo` y `usuarios`.

![Grupo soporte creado](evidencias/fusiondirectory-grupo-soporte-creado.png)

## 4. Incidencias y resolución

### 4.1. Error al guardar la configuración de plugins

**Síntoma**

Al intentar guardar la configuración desde la pestaña `Plugins`, FusionDirectory devolvía errores LDAP de sintaxis inválida.

Los mensajes incluían clases como:

```text
fdAutofsPluginConf
fdSystemsPluginConf
fdWebservicePluginConf
fdMailPluginConf
fdSudoPluginConf
```

**Diagnóstico**

El servidor OpenLDAP no reconocía las clases de configuración correspondientes a varios plugins de FusionDirectory. La imagen utilizada incluía los plugins, pero no todos sus esquemas `*-fd-conf.schema` habían sido cargados en `cn=config`.

**Causa**

Faltaban esquemas de configuración de FusionDirectory en el servidor LDAP. Los esquemas del núcleo habían sido cargados, pero no los asociados a plugins como AutoFS, Systems, Webservice, Mail y Sudo.

**Corrección aplicada**

He extraído los esquemas `*-fd-conf.schema` desde el contenedor de FusionDirectory, los he convertido a formato LDIF mediante `slaptest` y los he cargado en la configuración dinámica de OpenLDAP con:

```bash
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f <esquema>.ldif
```

En lugar de añadirlos uno por uno, he preparado un procedimiento que detecta y carga todos los esquemas pendientes, evitando duplicados.

**Resultado**

Tras cargar los esquemas pendientes, el guardado de la configuración de FusionDirectory se completó correctamente.

### 4.2. Página principal bloqueada tras guardar

**Síntoma**

Después de guardar la configuración, al pulsar `Main` la página permanecía cargando y no llegaba a mostrar el panel principal.

**Diagnóstico**

He comprobado que el problema no afectaba a toda la aplicación accediendo directamente a la sección de grupos:

```text
http://127.0.0.1:8080/fd/main.php?plug=38&reset=1
```

La página de grupos cargaba correctamente.

**Corrección aplicada**

He recargado la interfaz con una URL limpia:

```text
http://127.0.0.1:8080/fd/main.php?reset=1
```

**Resultado**

La interfaz ha vuelto a cargar con normalidad y he podido continuar con la administración de grupos.

## 5. Evidencias actuales

Las capturas disponibles se encuentran en [`evidencias/`](evidencias/).

| Captura | Contenido | Enlace |
|---|---|---|
| Acceso inicial | Acceso inicial a la interfaz web | [Ver captura](evidencias/fusiondirectory-01-acceso-web.png) |
| Acceso administrador | Sesión iniciada como `fd-admin` | [Ver captura](evidencias/fusiondirectory-acceso-administrador.png) |
| Requisitos | Comprobación de requisitos previos | [Ver captura](evidencias/fusiondirectory-comprobacion-requisitos.png) |
| Rama de usuarios | Configuración de la rama de usuarios y grupos | [Ver captura](evidencias/fusiondirectory-configuracion-rama-usuarios.png) |
| Grupo soporte | Grupo `soporte` creado y visible | [Ver captura](evidencias/fusiondirectory-grupo-soporte-creado.png) |
| Asistente | Inicio del asistente de configuración | [Ver captura](evidencias/fusiondirectory-inicio-asistente.png) |
| Inspección inicial | Inspección inicial del directorio | [Ver captura](evidencias/fusiondirectory-inspeccion-inicial.png) |
| Inspección final | Estado del directorio tras la inicialización | [Ver captura](evidencias/fusiondirectory-inspeccion-tras-inicializacion.png) |

## 6. Evidencias pendientes

Este apartado queda reservado para las capturas y salidas que se añadirán en las siguientes fases de la actividad.

### 6.1. NFS

- Instalación y configuración del servidor NFS.
- Creación del recurso compartido.
- Asignación de permisos POSIX al grupo autorizado.
- Configuración del cliente.
- Montaje del recurso NFS.

### 6.2. Verificación de acceso

- Reconocimiento de usuarios y grupos mediante `getent`.
- Acceso permitido con un usuario miembro del grupo autorizado.
- Acceso denegado con un usuario ajeno al grupo.
- Comprobación de escritura dentro del recurso compartido.

### 6.3. Comprobaciones finales

- Estado de los servicios OpenLDAP, NSS/SSSD y NFS.
- Configuración final del recurso compartido.
- Resumen de pruebas de segregación de acceso.

Las nuevas imágenes se añadirán en `evidencias/` con nombres descriptivos y se insertarán junto a la explicación correspondiente.

## 7. Próximos pasos

1. Documentar el despliegue del servidor NFS.
2. Crear y configurar el recurso compartido.
3. Asignar permisos mediante un grupo LDAP.
4. Configurar el cliente y montar el recurso.
5. Realizar las pruebas de acceso permitido y denegado.
6. Completar la sección de evidencias y conclusiones.

---

[← Volver al panel de actividades ASIXII](../README.md#actividades) · [Actividades de Quim](https://github.com/deivid-cardona/Base_de_datos-Quim#actividades)
