# Diccionario de datos de SIGA

Base de datos PostgreSQL. Datos ficticios únicamente. Relaciones y reglas en
`database/modelo-er.md`.

Convenciones:

- **PK:** clave primaria. Todas son `integer GENERATED ALWAYS AS IDENTITY`,
  salvo `roles.id` (`smallint`) y `auditoria.id` (`bigint`).
- **FK:** clave foránea con `ON DELETE RESTRICT`.
- Los nombres de tablas y columnas van en minúsculas, sin tildes ni ñ.
- Las marcas de tiempo usan `timestamptz`; `creado_en` toma `now()` por defecto.

## roles

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | smallint | No | PK |
| nombre | varchar(30) | No | Único. Valores iniciales: `administrador`, `lider_academico`, `instructor`, `consulta` |
| descripcion | varchar(200) | Sí | Texto explicativo |

## usuarios

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| nombres | varchar(100) | No | No vacío |
| apellidos | varchar(100) | No | No vacío |
| correo | varchar(254) | No | Índice único sobre `lower(correo)` |
| password_hash | text | No | Hash Argon2id o bcrypt. Nunca texto plano |
| rol_id | smallint | No | FK a `roles(id)` |
| activo | boolean | No | Por defecto `true`. Si es `false` no puede iniciar sesión (RN-16) |
| debe_cambiar_password | boolean | No | Por defecto `false`. Pasa a `true` tras un restablecimiento (HU-05) |
| ultimo_acceso | timestamptz | Sí | Último inicio de sesión correcto |
| creado_en | timestamptz | No | Por defecto `now()` |
| actualizado_en | timestamptz | No | Por defecto `now()` |

## instructores

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| usuario_id | integer | No | FK a `usuarios(id)`. Único: un usuario es un solo instructor |
| especialidad | varchar(100) | Sí | Área de formación |
| activo | boolean | No | Por defecto `true` |

El servidor verifica que el usuario vinculado tenga el rol `instructor`.

## aprendices

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| tipo_documento | varchar(5) | No | `CHECK` en (`CC`, `TI`, `CE`, `PEP`, `PPT`, `PA`) |
| numero_documento | varchar(20) | No | `CHECK` alfanumérico de 4 a 20 caracteres |
| nombres | varchar(100) | No | No vacío |
| apellidos | varchar(100) | No | No vacío |
| correo | varchar(254) | Sí | Formato validado en el servidor |
| telefono | varchar(20) | Sí | Formato validado en el servidor |
| activo | boolean | No | Por defecto `true` |
| creado_en | timestamptz | No | Por defecto `now()` |
| actualizado_en | timestamptz | No | Por defecto `now()` |

Restricción: `UNIQUE (tipo_documento, numero_documento)`.

## fichas

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| codigo | varchar(20) | No | Único |
| programa | varchar(150) | No | Nombre del programa de formación |
| jornada | varchar(10) | No | `CHECK` en (`diurna`, `nocturna`, `mixta`) |
| fecha_inicio | date | No | |
| fecha_fin | date | Sí | `CHECK (fecha_fin >= fecha_inicio)` |
| estado | varchar(12) | No | `CHECK` en (`activa`, `finalizada`, `inactiva`). Por defecto `activa` |

## inscripciones

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| aprendiz_id | integer | No | FK a `aprendices(id)` |
| ficha_id | integer | No | FK a `fichas(id)` |
| estado | varchar(12) | No | `CHECK` en (`activa`, `retirada`, `finalizada`). Por defecto `activa` |
| fecha_inscripcion | date | No | Por defecto la fecha actual |
| fecha_retiro | date | Sí | `CHECK (fecha_retiro IS NULL OR fecha_retiro >= fecha_inscripcion)` |

Restricción: `UNIQUE (aprendiz_id, ficha_id)`.

## asignaciones_instructor

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| instructor_id | integer | No | FK a `instructores(id)` |
| ficha_id | integer | No | FK a `fichas(id)` |
| fecha_asignacion | date | No | Por defecto la fecha actual |
| activa | boolean | No | Por defecto `true`. Retirar una asignación la pone en `false` y conserva el historial |

Restricción: `UNIQUE (instructor_id, ficha_id)`. Esta restricción también es el
destino de la clave foránea compuesta de `sesiones`.

## sesiones

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| ficha_id | integer | No | Parte de la FK compuesta |
| instructor_id | integer | No | Parte de la FK compuesta |
| fecha | date | No | Fecha local de la sesión |
| hora_inicio | time | No | Hora local |
| hora_fin | time | No | `CHECK (hora_fin > hora_inicio)` |
| tema | varchar(200) | No | No vacío |
| estado | varchar(12) | No | `CHECK` en (`programada`, `realizada`, `cancelada`). Por defecto `programada` |
| motivo_cancelacion | varchar(300) | Sí | Opcional al cancelar |
| creado_por | integer | No | FK a `usuarios(id)` |
| creado_en | timestamptz | No | Por defecto `now()` |
| actualizado_en | timestamptz | No | Por defecto `now()` |

Restricciones:

- `FOREIGN KEY (instructor_id, ficha_id) REFERENCES asignaciones_instructor (instructor_id, ficha_id)`.
- Exclusión por instructor y por ficha para evitar solapes entre sesiones no
  canceladas (a verificar en la Etapa 4).

## asistencias

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | integer | No | PK |
| sesion_id | integer | No | FK a `sesiones(id)` |
| inscripcion_id | integer | No | FK a `inscripciones(id)` |
| estado | varchar(10) | No | `CHECK` en (`presente`, `ausente`, `tardanza`, `excusa`) |
| observacion | varchar(500) | Sí | Texto libre |
| registrado_por | integer | No | FK a `usuarios(id)` |
| registrado_en | timestamptz | No | Por defecto `now()` |
| modificado_por | integer | Sí | FK a `usuarios(id)` |
| modificado_en | timestamptz | Sí | `CHECK ((modificado_por IS NULL) = (modificado_en IS NULL))` |

Restricción: `UNIQUE (sesion_id, inscripcion_id)`.

## auditoria

| Columna | Tipo | Nulo | Descripción y restricciones |
|---|---|---|---|
| id | bigint | No | PK, identidad generada |
| usuario_id | integer | Sí | FK a `usuarios(id)`. Nulo si la acción no tiene usuario (por ejemplo, un intento de inicio de sesión fallido) |
| accion | varchar(50) | No | Por ejemplo `crear`, `editar`, `desactivar`, `restablecer_password` |
| entidad | varchar(50) | No | Nombre de la tabla afectada |
| entidad_id | bigint | Sí | Identificador del registro afectado |
| datos_antes | jsonb | Sí | Estado previo. **Nunca incluye contraseñas ni hashes** |
| datos_despues | jsonb | Sí | Estado posterior. **Nunca incluye contraseñas ni hashes** |
| fecha | timestamptz | No | Por defecto `now()` |

## session (tabla técnica)

La crea la migración inicial y la usa `connect-pg-simple`. El nombre y las
columnas exactas se confirman en la Etapa 5.

| Columna | Tipo | Nulo | Descripción |
|---|---|---|---|
| sid | varchar | No | PK, identificador de sesión |
| sess | json | No | Datos de la sesión (id de usuario y rol, nunca contraseñas) |
| expire | timestamp | No | Vencimiento; con índice |

## schema_migrations (tabla técnica)

| Columna | Tipo | Nulo | Descripción |
|---|---|---|---|
| nombre | varchar(200) | No | PK, nombre del archivo aplicado |
| aplicada_en | timestamptz | No | Por defecto `now()` |

## Valores permitidos

| Campo | Valores |
|---|---|
| aprendices.tipo_documento | CC, TI, CE, PEP, PPT, PA |
| fichas.jornada | diurna, nocturna, mixta |
| fichas.estado | activa, finalizada, inactiva |
| inscripciones.estado | activa, retirada, finalizada |
| sesiones.estado | programada, realizada, cancelada |
| asistencias.estado | presente, ausente, tardanza, excusa |