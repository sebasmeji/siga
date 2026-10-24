# Historias de usuario de SIGA

Cada historia indica el actor, la necesidad, los criterios de aceptación y los
requisitos (RF) y reglas (RN) que cubre. Los códigos remiten a
`docs/requisitos.md`.

Decisión adicional **D7**: el rol Consulta puede leer aprendices, fichas,
sesiones, asistencia y reportes de todas las fichas. No ve usuarios, auditoría ni
los datos de contacto del aprendiz (correo y teléfono), y no puede modificar nada.

## Autenticación y usuarios

### HU-01. Iniciar sesión
**Como** usuario registrado, **quiero** iniciar sesión con mi correo y contraseña,
**para** acceder a las funciones de mi rol.
**Cubre:** RF-01, RN-02, RN-16.

Criterios de aceptación:
- Con credenciales correctas y cuenta activa, se crea una sesión y se redirige al
  panel principal.
- Con credenciales incorrectas se muestra un mensaje genérico ("Correo o
  contraseña incorrectos"), sin indicar cuál de los dos falló.
- Una cuenta desactivada no puede iniciar sesión.
- Tras varios intentos fallidos seguidos, el servidor limita temporalmente los
  nuevos intentos.
- La contraseña nunca se muestra ni se registra en los logs.

### HU-02. Cerrar sesión
**Como** usuario autenticado, **quiero** cerrar sesión, **para** proteger mi cuenta
en un equipo compartido.
**Cubre:** RF-01.

Criterios de aceptación:
- Al cerrar sesión, el servidor invalida la sesión y se regresa a la pantalla de
  inicio de sesión.
- Después de cerrar sesión, volver atrás en el navegador no permite ver datos
  protegidos: cualquier solicitud a la API responde 401.
- Si la sesión expira, la interfaz informa al usuario y lo envía al inicio de
  sesión.

### HU-03. Cambiar mi contraseña
**Como** usuario autenticado, **quiero** cambiar mi contraseña, **para** mantener
mi cuenta segura.
**Cubre:** RF-02.

Criterios de aceptación:
- Se exige la contraseña actual y una nueva contraseña con confirmación.
- La nueva contraseña cumple una longitud mínima (propuesta: 10 caracteres) y es
  distinta de la actual.
- Si la contraseña actual es incorrecta, el cambio se rechaza.
- La contraseña se guarda solo como hash.

### HU-04. Gestionar usuarios
**Como** Administrador, **quiero** crear, editar, activar y desactivar usuarios y
asignarles un rol, **para** controlar quién usa el sistema.
**Cubre:** RF-03, RN-03, RN-16.

Criterios de aceptación:
- El correo de cada usuario es único; un duplicado se rechaza con un mensaje claro.
- Solo el Administrador accede a este módulo; cualquier otro rol recibe 403.
- Al desactivar un usuario, no puede iniciar sesión y sus sesiones activas se
  invalidan.
- Un usuario no puede desactivar su propia cuenta de Administrador si es el único
  Administrador activo.
- No se borran usuarios; solo se desactivan.

### HU-05. Restablecer la contraseña de un usuario
**Como** Administrador, **quiero** restablecer la contraseña de otro usuario,
**para** que recupere el acceso sin usar correo.
**Cubre:** RF-04, RN-05.

Criterios de aceptación:
- El Administrador define una contraseña temporal que cumple la política mínima.
- La contraseña temporal nunca se muestra después de guardada ni se registra en los
  logs.
- Las sesiones activas del usuario afectado se invalidan.
- Se deja un registro de auditoría de quién hizo el cambio y sobre qué usuario.

## Aprendices

### HU-06. Registrar un aprendiz
**Como** Líder académico, **quiero** registrar aprendices, **para** tenerlos
disponibles para inscripciones y asistencia.
**Cubre:** RF-06, RN-07.

Criterios de aceptación:
- Los campos obligatorios son tipo de documento, número de documento, nombres y
  apellidos.
- La combinación tipo y número de documento es única; un duplicado se rechaza con
  un mensaje claro.
- Los campos opcionales (correo y teléfono) se validan en su formato si se
  diligencian.
- El servidor ignora cualquier campo que no pertenezca al formulario.
- Tras guardar, el aprendiz aparece en el listado.

### HU-07. Listar y buscar aprendices
**Como** Líder académico o Instructor, **quiero** buscar aprendices por nombre,
documento o ficha, **para** encontrarlos rápidamente.
**Cubre:** RF-07, RF-08, D6.

Criterios de aceptación:
- El listado está paginado y muestra el total de resultados.
- La búsqueda por texto aplica a nombres, apellidos y documento.
- El filtro por ficha muestra los aprendices inscritos en ella.
- El Instructor solo ve aprendices de las fichas que tiene asignadas.
- Si no hay resultados, se muestra un mensaje de estado vacío.
- Mientras carga, se muestra un indicador de carga.

### HU-08. Editar y desactivar un aprendiz
**Como** Líder académico, **quiero** corregir los datos de un aprendiz o
desactivarlo, **para** mantener la información al día.
**Cubre:** RF-06, RF-09, RN-03.

Criterios de aceptación:
- Se pueden editar todos los datos, respetando la unicidad del documento.
- Desactivar un aprendiz exige una confirmación explícita.
- Un aprendiz desactivado no aparece en las listas de toma de asistencia nuevas,
  pero su historial se conserva.
- No se eliminan registros físicamente.

### HU-09. Consultar el historial de un aprendiz
**Como** Líder académico o Instructor, **quiero** ver las inscripciones y la
asistencia de un aprendiz, **para** hacer seguimiento a su proceso.
**Cubre:** RF-10, RF-26.

Criterios de aceptación:
- Se muestran las fichas en las que está o estuvo inscrito, con su estado.
- Se muestra su asistencia por sesión, con estado y observación.
- Se muestra su porcentaje de asistencia calculado con la fórmula de RN-04.
- El Instructor solo ve el historial correspondiente a sus fichas.

## Fichas e instructores

### HU-10. Gestionar fichas
**Como** Líder académico, **quiero** crear y editar fichas, **para** organizar los
grupos de formación.
**Cubre:** RF-11, RN-03.

Criterios de aceptación:
- Los datos obligatorios son código, programa, jornada, fecha de inicio y estado.
- El código de ficha es único.
- La fecha de fin, si existe, no es anterior a la de inicio.
- Una ficha se desactiva o cambia de estado, pero no se borra.

### HU-11. Inscribir aprendices en una ficha
**Como** Líder académico, **quiero** inscribir aprendices en una ficha, **para**
que aparezcan en la asistencia de sus sesiones.
**Cubre:** RF-12, RN-08, RN-14.

Criterios de aceptación:
- No se permite inscribir dos veces al mismo aprendiz en la misma ficha.
- La pantalla de la ficha lista los aprendices inscritos y su estado de inscripción.
- Se puede retirar a un aprendiz cambiando el estado de su inscripción, sin perder
  su historial de asistencia.
- La inscripción de varios aprendices a la vez es atómica: si una falla, no se
  guarda ninguna.

### HU-12. Asignar instructores a una ficha
**Como** Líder académico, **quiero** asignar instructores a una ficha, **para**
que puedan programar sesiones y tomar asistencia.
**Cubre:** RF-13, RF-14, RN-06, RN-09.

Criterios de aceptación:
- Solo se pueden asignar usuarios con rol Instructor.
- No se puede asignar dos veces el mismo instructor a la misma ficha.
- Un instructor solo ve las fichas que tiene asignadas.
- Se puede consultar los instructores asignados a cada ficha.

## Sesiones

### HU-13. Programar una sesión
**Como** Instructor o Líder académico, **quiero** programar una sesión, **para**
planificar la formación y luego tomar asistencia.
**Cubre:** RF-15, RN-10, RN-11.

Criterios de aceptación:
- Los datos obligatorios son ficha, instructor, fecha, hora de inicio, hora de fin
  y tema.
- La hora de fin es posterior a la de inicio.
- No se permite una sesión que se solape con otra activa del mismo instructor o de
  la misma ficha en esa fecha; el mensaje indica cuál es el conflicto.
- El Instructor solo programa sesiones en sus fichas asignadas y a su nombre.
- La sesión nueva queda en estado "programada".

### HU-14. Consultar sesiones
**Como** Instructor, Líder académico o Consulta, **quiero** consultar sesiones por
ficha, instructor y fecha, **para** ver la programación y el historial.
**Cubre:** RF-16, RF-18.

Criterios de aceptación:
- Se puede filtrar por ficha, instructor, rango de fechas y estado.
- El listado está paginado y ordenado por fecha.
- El historial muestra las sesiones realizadas.
- El Instructor solo ve sesiones de sus fichas.

### HU-15. Editar o cancelar una sesión
**Como** Instructor o Líder académico, **quiero** editar o cancelar una sesión,
**para** reflejar cambios en la programación.
**Cubre:** RF-17, RN-12.

Criterios de aceptación:
- Una sesión programada se puede editar respetando las reglas de horario de HU-13.
- Cancelar exige confirmación y se puede registrar un motivo.
- Una sesión cancelada no admite asistencia y no cuenta en los porcentajes.
- Una sesión con asistencia registrada no se puede cancelar sin antes resolver ese
  registro (regla propuesta; se confirma en la Etapa 4).
- No se borran sesiones.

## Asistencia

### HU-16. Tomar asistencia
**Como** Instructor, **quiero** registrar la asistencia de los aprendices de una
sesión, **para** llevar el control de la formación.
**Cubre:** RF-19, RF-20, RF-21, RF-22, RN-12, RN-13, RN-14.

Criterios de aceptación:
- Al elegir una sesión, se muestran los aprendices con inscripción activa en su
  ficha.
- Cada aprendiz puede quedar como presente, ausente, tardanza o excusa
  justificada, con una observación opcional.
- Una sesión cancelada no permite tomar asistencia.
- Solo el Instructor asignado a esa ficha, el Líder académico o el Administrador
  pueden registrarla.
- No se duplica la asistencia de un aprendiz en la misma sesión.
- Guardar toda la lista es atómico: o se guardan todos los registros o ninguno.
- Se guarda quién registró cada asistencia y cuándo.
- Al guardar, la sesión pasa a "realizada".

### HU-17. Corregir una asistencia
**Como** Instructor o Líder académico, **quiero** corregir un registro de
asistencia, **para** arreglar errores.
**Cubre:** RF-23, RN-15.

Criterios de aceptación:
- Se puede cambiar el estado y la observación de un registro existente.
- Se guarda quién modificó y cuándo, y el valor anterior queda en la auditoría.
- Un usuario con rol Consulta no puede modificar asistencia (403).
- Un Instructor solo corrige asistencias de sus propias fichas.

### HU-18. Ver el resumen de asistencia de una sesión
**Como** Instructor o Líder académico, **quiero** ver cuántos aprendices
estuvieron presentes, ausentes, con tardanza o con excusa en una sesión, **para**
evaluar la asistencia del grupo.
**Cubre:** RF-24.

Criterios de aceptación:
- Se muestran los conteos por estado y el total de aprendices.
- Los conteos se calculan a partir de los registros guardados, no de valores fijos.
- Si la sesión aún no tiene asistencia, se muestra un estado vacío.

## Reportes y panel

### HU-19. Consultar reportes de asistencia
**Como** Líder académico, Instructor o Consulta, **quiero** reportes de asistencia
con filtros, **para** analizar la situación de fichas y aprendices.
**Cubre:** RF-25, RF-26, RF-27, RF-28, RN-04.

Criterios de aceptación:
- Se puede filtrar por fecha, ficha, instructor y estado.
- El porcentaje de asistencia sigue la fórmula de RN-04: las tardanzas cuentan
  como asistencia, las excusas justificadas salen del denominador y las sesiones
  canceladas no cuentan.
- Si el denominador es 0, se muestra "sin datos" y no un porcentaje.
- Cada rol ve solo lo que le corresponde (el Instructor, solo sus fichas).
- Los resultados se calculan desde la base de datos.

### HU-20. Exportar un reporte a CSV
**Como** usuario autorizado, **quiero** exportar un reporte a CSV, **para**
analizarlo o archivarlo fuera del sistema.
**Cubre:** RF-29.

Criterios de aceptación:
- El archivo contiene exactamente los datos filtrados y autorizados para ese
  usuario.
- El archivo abre correctamente en Excel con tildes y eñes (codificación UTF-8 con
  marca BOM).
- Los valores que comienzan con `=`, `+`, `-` o `@` se neutralizan para evitar
  inyección de fórmulas.
- Si no hay datos, el sistema avisa en lugar de descargar un archivo vacío.

### HU-21. Ver el panel principal
**Como** usuario autenticado, **quiero** ver un panel con indicadores de mi rol,
**para** tener una visión rápida del estado del sistema.
**Cubre:** RF-30.

Criterios de aceptación:
- Se muestran aprendices activos, fichas registradas, sesiones programadas, últimas
  sesiones realizadas y un resumen de asistencia, según lo que el rol puede ver.
- Todas las cifras se calculan desde la base de datos.
- Se muestran accesos rápidos solo a los módulos autorizados.
- El Instructor ve solo las cifras de sus fichas.

## Seguridad transversal

### HU-22. Aplicar los permisos en el servidor
**Como** responsable del sistema, **quiero** que cada operación valide el rol en el
servidor, **para** que ocultar botones no sea la única protección.
**Cubre:** RF-05, RN-06.

Criterios de aceptación:
- Una solicitud sin sesión recibe 401.
- Una solicitud con sesión pero sin permiso recibe 403, aunque se haga fuera de la
  interfaz (por ejemplo, con Postman).
- Un Instructor no puede leer ni modificar datos de fichas que no tiene asignadas.
- Las operaciones que modifican datos exigen la cabecera de protección CSRF.
- Los errores no revelan detalles internos ni consultas SQL.