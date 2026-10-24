# Requisitos de SIGA (Sistema de Gestión Académica)

Proyecto formativo independiente de Tecnología en Análisis y Desarrollo de
Software (ADSO). No es una plataforma oficial del SENA ni se integra a sus
sistemas. Durante el desarrollo y las pruebas se usan únicamente datos ficticios.

## 1. Planteamiento del problema

La información académica de aprendices, fichas, sesiones y asistencia suele
llevarse en hojas de cálculo, formatos en papel o archivos separados. Esto
dificulta consultar el historial, detectar inasistencias a tiempo, evitar
registros duplicados y saber quién modificó un dato.

## 2. Justificación

Una aplicación web con base de datos relacional permite centralizar la
información, controlar quién accede a ella según su rol, conservar la
trazabilidad de los cambios y calcular reportes a partir de datos reales. A la
vez, sirve como evidencia de aprendizaje de buenas prácticas de ingeniería de
software.

## 3. Objetivos

**General.** Construir una aplicación web que permita administrar la
información académica de aprendices, fichas, instructores, sesiones y asistencia
desde una plataforma centralizada.

**Específicos.**

1. Implementar autenticación segura y control de permisos por rol verificado en
   el servidor.
2. Gestionar aprendices, fichas, inscripciones e instructores con integridad de
   datos.
3. Programar sesiones de formación y registrar la asistencia con trazabilidad.
4. Generar reportes de asistencia con filtros y exportación a CSV.
5. Documentar, probar y desplegar el sistema usando servicios con plan gratuito,
   cuando estén disponibles.

## 4. Alcance de la versión 1

**Incluye:** autenticación y usuarios, aprendices, fichas, inscripciones,
asignación de instructores, sesiones, asistencia, reportes con CSV, dashboard
por rol, pruebas y documentación.

**No incluye:** recuperación de contraseña por correo, integración con sistemas
oficiales del SENA, notificaciones, aplicación móvil nativa y carga masiva de
datos.

**Limitaciones:**

- Los planes gratuitos de despliegue pueden suspender la aplicación por
  inactividad o cambiar sus condiciones.
- No se garantiza disponibilidad permanente.
- No se debe cargar información real de aprendices sin autorización y sin medidas
  de protección de datos personales.

## 5. Actores

| Actor | Descripción |
|---|---|
| Administrador | Gestiona usuarios, roles y todos los módulos. |
| Líder académico | Administra aprendices, fichas, sesiones, instructores y reportes. |
| Instructor | Consulta sus fichas asignadas, registra sesiones y toma asistencia. |
| Consulta | Visualiza la información autorizada, sin modificarla. |

Los aprendices no inician sesión: son registros administrados por el personal.

## 6. Requisitos funcionales

### Autenticación y usuarios

- **RF-01.** Iniciar y cerrar sesión.
- **RF-02.** Cambiar la contraseña propia.
- **RF-03.** El Administrador crea, edita, activa y desactiva usuarios.
- **RF-04.** El Administrador restablece la contraseña de un usuario.
- **RF-05.** El servidor valida el rol en cada operación protegida.

### Aprendices

- **RF-06.** Registrar y editar aprendices.
- **RF-07.** Listar aprendices con paginación.
- **RF-08.** Buscar por nombre, documento o ficha.
- **RF-09.** Activar y desactivar aprendices.
- **RF-10.** Consultar el historial de inscripciones y de asistencia.

### Fichas e instructores

- **RF-11.** Crear, consultar, editar y cambiar el estado de fichas.
- **RF-12.** Inscribir aprendices en una ficha y consultar los inscritos.
- **RF-13.** Asignar instructores a fichas y consultarlos.
- **RF-14.** Administrar los datos propios de un instructor, vinculados a su usuario.

### Sesiones

- **RF-15.** Crear sesiones con fecha, hora de inicio y fin, ficha, instructor y tema.
- **RF-16.** Consultar sesiones por ficha, instructor y fecha.
- **RF-17.** Editar y cancelar sesiones.
- **RF-18.** Consultar el historial de sesiones realizadas.

### Asistencia

- **RF-19.** Seleccionar una sesión y ver los aprendices inscritos en su ficha.
- **RF-20.** Registrar asistencia individual con estado: presente, ausente,
  tardanza o excusa justificada.
- **RF-21.** Registrar observaciones.
- **RF-22.** Guardar quién registró y quién modificó cada asistencia.
- **RF-23.** Corregir asistencias según el rol.
- **RF-24.** Mostrar el resumen de asistencia de una sesión.

### Reportes y panel

- **RF-25.** Listado de aprendices por ficha.
- **RF-26.** Historial de asistencia por aprendiz.
- **RF-27.** Asistencia por ficha y periodo, con filtros por fecha, ficha,
  instructor y estado.
- **RF-28.** Porcentaje de asistencia por aprendiz y conteo por estado.
- **RF-29.** Exportar a CSV los reportes autorizados.
- **RF-30.** Dashboard por rol con indicadores calculados desde la base de datos.

## 7. Requisitos no funcionales

- **RNF-01 Seguridad.** Contraseñas con Argon2id (o bcrypt como alternativa), sin
  guardarlas en texto plano. Secretos en variables de entorno.
- **RNF-02 Integridad.** Consultas SQL parametrizadas, restricciones en la base de
  datos y transacciones en operaciones atómicas.
- **RNF-03 Autenticación.** Sesiones en el servidor con cookie `HttpOnly`,
  `SameSite` y `Secure` en producción.
- **RNF-04 Usabilidad.** Interfaz en español, responsive, con accesibilidad básica.
- **RNF-05 Mantenibilidad.** Código separado en rutas, controladores, servicios,
  validadores y middlewares.
- **RNF-06 Calidad.** Pruebas automatizadas de la API, los permisos y los reportes.
- **RNF-07 Costo.** Presupuesto de $0 COP.
- **RNF-08 Privacidad.** Solo datos ficticios hasta contar con autorización.
- **RNF-09 Errores.** Mensajes genéricos al cliente; el detalle queda en los
  registros del servidor.

## 8. Reglas de negocio

### Decisiones aprobadas

- **RN-01 (D1).** Express sirve el frontend y la API desde el mismo origen.
- **RN-02 (D2).** La autenticación usa sesiones guardadas en PostgreSQL, sin JWT.
- **RN-03 (D3).** Aprendices, fichas y usuarios se desactivan; no se borran.
- **RN-04 (D4).** Porcentaje de asistencia:
  `(presentes + tardanzas) / (sesiones realizadas - excusas justificadas) x 100`.
  Las sesiones canceladas no cuentan. Las tardanzas cuentan como asistencia.
  Si el denominador es 0, el porcentaje se muestra como "sin datos".
- **RN-05 (D5).** No hay recuperación por correo; el Administrador restablece
  contraseñas.
- **RN-06 (D6).** El Instructor solo ve y gestiona las fichas que tiene asignadas.

### Reglas propuestas (se pueden ajustar antes de la Etapa 4)

- **RN-07.** El documento de un aprendiz (tipo y número) es único.
- **RN-08.** Un aprendiz no puede estar inscrito dos veces en la misma ficha.
- **RN-09.** Un instructor no puede estar asignado dos veces a la misma ficha.
- **RN-10.** La hora de fin de una sesión es posterior a la hora de inicio.
- **RN-11.** Un instructor no puede tener dos sesiones activas que se solapen en
  la misma fecha. Lo mismo aplica para una ficha.
- **RN-12.** Estados de una sesión: programada, realizada y cancelada. Una sesión
  pasa a realizada cuando se guarda su asistencia. Una sesión cancelada no admite
  asistencia.
- **RN-13.** Cada aprendiz tiene un solo registro de asistencia por sesión.
- **RN-14.** Solo se registra asistencia de aprendices con inscripción activa en
  la ficha de la sesión.
- **RN-15.** Toda modificación de asistencia guarda quién la hizo y cuándo. La
  tabla de auditoría conserva el valor anterior.
- **RN-16.** Una cuenta desactivada no puede iniciar sesión y sus sesiones activas
  se invalidan.

## 9. Permisos por rol (resumen)

Leyenda: C = crear, L = leer, E = editar, D = desactivar, - = sin acceso.

| Módulo | Administrador | Líder académico | Instructor | Consulta |
|---|---|---|---|---|
| Usuarios y roles | C L E D | - | - | - |
| Aprendices | C L E D | C L E D | L (solo sus fichas) | L (autorizado) |
| Fichas | C L E D | C L E D | L (solo asignadas) | L (autorizado) |
| Instructores | C L E D | C L E D | L (propio) | L (autorizado) |
| Sesiones | C L E D | C L E D | C L E (solo sus fichas) | L (autorizado) |
| Asistencia | C L E | C L E | C L E (sus sesiones) | L (autorizado) |
| Reportes y CSV | L | L | L (sus fichas) | L (autorizado) |