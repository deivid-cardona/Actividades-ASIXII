# ACT10 — Manual de operaciones: FusionDirectory, OpenLDAP y NFS

Profesor: Quim.

## 1. Objetivo y organización del manual

He estructurado este manual procedimental a partir del enunciado para organizar la instalación, configuración e integración de FusionDirectory sobre OpenLDAP y su vinculación con carpetas compartidas NFS restringidas por grupos del directorio.

He separado la contextualización del caso de uso, los prerrequisitos, los tres módulos técnicos y la entrega. Esta estructura inicial no presenta instalaciones, comandos ni pruebas como ejecutados.

He situado el informe en Actividades-ASIXII para mantener una única versión de la documentación y facilitar su consulta desde el apartado de actividades de Quim.

## 2. Contextualización del caso de uso

### Modalidad A — Proyecto de Sistemas

Adaptaré el procedimiento a la arquitectura del proyecto: dominio, Base DN, unidades organizativas, direcciones IP, grupos departamentales y puntos de montaje NFS. Integraré el playbook como Manual de Operaciones de Identidad y Almacenamiento en la memoria oficial del proyecto en GitHub.

### Modalidad B — Caso personal o alternativa DUAL

Para esta modalidad utilizaré el caso solicitado en el enunciado:

| Elemento | Parámetro exigido para esta modalidad |
|---|---|
| Base DN | `dc=sistemapropio,dc=local` |
| Unidad organizativa de usuarios | `ou=usuarios` |
| Grupo POSIX | `desarrollo` |
| Usuarios de prueba | Al menos dos; incluiré un caso autorizado y otro ajeno al grupo para la segregación |
| Recurso NFS | `/srv/nfs/desarrollo` |
| Permisos de la carpeta | `770` |
| Entrega | Manual genérico y reutilizable en PDF |

Aplicaré solo los parámetros de la modalidad elegida al procedimiento ejecutado, sin mezclar los dos escenarios.

## 3. Prerrequisitos y escenario

Tomaré como referencia mi [práctica previa de OpenLDAP](../ACT5-openldap/README.md). Antes de la integración comprobaré:

- El servicio `slapd`, la estructura base del dominio y la rama principal de administración.
- La integración NSS mediante `nslcd` o `sssd`, tanto en el servidor como en los clientes.
- El reconocimiento de los usuarios y grupos del directorio mediante `getent passwd` y `getent group`.
- La existencia de usuarios y grupos de prueba para comprobar el acceso permitido y el denegado.

Documentaré la distribución y versión de cada equipo, sus funciones, direcciones IP, Base DN, unidades organizativas, grupos y rutas realmente utilizados. No daré por existente la integración NSS/SSSD únicamente por haber instalado OpenLDAP en la actividad anterior.

## 4. Módulo A — Gestión e instalación de FusionDirectory

### 4.1. Dependencias y repositorio

Documentaré la instalación de Apache, PHP y sus extensiones, incluidas `php-ldap` y `php-mbstring`, junto con la adición del repositorio oficial de FusionDirectory adecuado al sistema utilizado.

### 4.2. Gestión de esquemas

Documentaré la carga de los esquemas del núcleo, incluidos `core-fd.schema` y `template-fd.schema`, y de los plugins utilizados mediante `fusiondirectory-schema-manager`. Acompañaré los comandos ejecutados con sus salidas de comprobación.

### 4.3. Asistente web de configuración

Explicaré las seis etapas del asistente de `/fusiondirectory/setup` tal como aparezcan en la versión instalada. Incluiré la creación del token de seguridad, la verificación de conexión LDAP, la definición de las ramas `ou=usuarios` y `ou=grupos` y el guardado de `/etc/fusiondirectory/fusiondirectory.conf`.

### 4.4. Administración visual

Describiré el alta de usuarios con perfil POSIX y de grupos desde la interfaz web. Relacionaré los usuarios creados con los grupos que utilizaré para las pruebas de acceso NFS.

## 5. Módulo B — Despliegue e integración de NFS

### 5.1. Instalación y configuración del servidor

Documentaré la instalación de `nfs-kernel-server` y la configuración utilizada para publicar el recurso del caso de uso.

### 5.2. Directorio compartido y permisos

Documentaré la creación de la carpeta compartida, la asignación del grupo LDAP y los permisos POSIX. Para la modalidad B utilizaré `/srv/nfs/desarrollo` con permisos `770`.

### 5.3. Acceso desde el cliente

Registraré la configuración de acceso y el punto de montaje utilizado en el cliente, junto con las comprobaciones de reconocimiento de usuarios y grupos necesarias para la prueba del recurso.

## 6. Módulo C — Verificación y troubleshooting

### 6.1. Comprobaciones en el servidor y los clientes

Incorporaré los comandos realmente ejecutados, las salidas de verificación y la interpretación de los resultados de OpenLDAP, NSS/SSSD, FusionDirectory y NFS.

### 6.2. Segregación de acceso

Documentaré dos pruebas diferenciadas sobre el mismo recurso:

1. Acceso y escritura con un usuario perteneciente al grupo LDAP autorizado.
2. Intento de acceso con un usuario ajeno al grupo y comprobación de su bloqueo.

Identificaré el usuario, los grupos efectivos, el recurso, la acción y el resultado en cada prueba.

### 6.3. Incidencias y resolución

Para las incidencias que encuentre registraré el síntoma, el diagnóstico, la causa identificada, la corrección aplicada y la comprobación posterior. No presentaré errores hipotéticos como incidencias ocurridas.

## 7. Criterio de documentación y evidencias

Redactaré el procedimiento en primera persona y acompañaré cada operación de su explicación, comandos ejecutados, ficheros de configuración comentados y capturas o salidas de verificación. Situaré cada imagen junto a la explicación correspondiente, sin duplicar la galería.

Guardaré las capturas en `evidencias/` con nombres numerados y descriptivos. No publicaré contraseñas, tokens de instalación ni credenciales presentes en ficheros de configuración.

## 8. Entrega y conclusiones

Prepararé la entrega conforme a la modalidad elegida: integración en la memoria oficial del proyecto en GitHub para la modalidad A, o manual reutilizable en PDF para la modalidad B. Redactaré las conclusiones a partir de la instalación, integración y pruebas documentadas.

---

[← Volver al panel de actividades ASIXII](../README.md#actividades) · [Actividades de Quim](https://github.com/deivid-cardona/Base_de_datos-Quim#actividades)
