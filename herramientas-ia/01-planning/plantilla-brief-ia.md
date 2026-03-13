## Technical Brief


### 1. Título de la tarea

Módulo en página web para gestión de reservas de estudio de pilates 

---

### 2. Contexto

Actualmente las reservas se ejecutan a través del canal de whatsapp. Laboratorio de pilates es un estudio de pilates que ofrece clases a sus usuarios de lunes a sábado. Las clases deben ser reservadas con anticipación por parte de los usuarios para poder asistir a la clases. El proceso hoy es manual, lo que genera varios problemas:

- El proceso es lento y esto genera una mala experiencia de usuario
- El proceso es muy operativo y no permite escalabilidad del negocio
- El proceso es muy manual y genera altos costos de operación

El objetivo es tener una página web con un módulo de reservas que le permita a los usuarios que tienen un plan activo ingresar y reservar clases para asistir posteriormente a la clase. 

---

### 3. Requerimientos funcionales

**Definiciones**

- **Plan activo (vigente)**: plan del usuario cuya vigencia (fecha inicio–fin) incluye la fecha actual; si el plan vence, no puede reservar hasta renovar.
- **Clases disponibles**: número de clases del plan que el usuario aún no ha consumido (ni por reserva confirmada ni por uso); si tiene 8 clases/mes y ya reservó 3, tiene 5 disponibles hasta que use o venzan.

**Funcionamiento**

- El usuario puede reservar únicamente si tiene un plan activo (ver definiciones).
- El usuario puede reservar únicamente si tiene clases disponibles por usar (ver definiciones).
- El usuario puede reservar si hay cupos disponibles en la clase. 
- Solamente hay 5 cupos disponibles para cada clase. 
- Las clases son de 1 hora de duración. 
- Las clases empiezan a las 6am, 7am, 8am, 9am, 10am, 3pm, 4pm, 5pm, 6pm, 7pm de lunes a viernes. Y los sábados las clases empiezan a 7am, 8am, 9am, 10am, 11am. 
- En caso que no haya cupo disponible, el usuario puede entrar a la lista de espera y la prioridad se determina por orden de entrada a la lista de espera. 
- El usuario puede cancelar la reserva siempre y cuando sea con 3 horas de anticipación. En el caso de la clase de 6am debe ser con 10 horas de anticipación. Si el usuario intenta cancelar en este horario, la clase será descontada igualmente de su plan. 

### 4. Requerimientos técnicos

**Lenguaje**

- Backend: Python (p. ej. FastAPI o Django).
- Frontend del módulo: según stack del sitio (p. ej. React, Vue o plantillas del backend).
- Base de datos: a definir (PostgreSQL/SQLite según entorno).

**Arquitectura**

- Separación entre capa de presentación (página/módulo web) y lógica de negocio (reservas, planes, lista de espera).
- Servicio o módulo de reservas que pueda extenderse con nuevas estrategias (horarios, reglas de cancelación) sin modificar el núcleo.
- API REST o endpoints claros para: consultar clases disponibles, reservar, cancelar, listar reservas del usuario, lista de espera.

**Modelo de datos**

Entidades mínimas sugeridas:

- **Usuario**: id, email/identificador, relación con plan(es).
- **Plan**: id, usuario, vigencia (fecha inicio/fin), cantidad de clases incluidas (o ilimitado).
- **Clase**: id, día de la semana, hora de inicio, duración (1h), instructor, cupo máximo (5).
- **Instructor**: id, nombre.
- **Reserva**: id, usuario, clase, fecha de la clase, estado (activa, cancelada, usada), fecha de creación.
- **Lista de espera**: id, usuario, clase, fecha de la clase, posición/orden, fecha de entrada.

Reglas: un usuario no puede tener más de una reserva activa para la misma clase (mismo día y hora). Las “clases disponibles” del plan son el saldo de clases que el usuario aún no ha consumido (por reserva + asistencia o por reserva no cancelada a tiempo).

**Comportamiento esperado**

1. Usuario se autentica en la página (login/sesión).
2. Sistema valida plan activo y clases disponibles; si no aplica, no permite reservar y muestra mensaje claro.
3. Usuario ve calendario o listado de clases (por día/hora) con cupos disponibles o “lista de espera”.
4. Usuario elige clase y confirma reserva; sistema verifica cupo, descuenta una “clase disponible” del plan y crea la reserva (o la entrada en lista de espera si no hay cupo).
5. Al cancelar: sistema comprueba ventana (3h o 10h para 6am); si está dentro, cancela y libera cupo (y opcionalmente asigna al primero de la lista de espera); si no, notifica al usuario que la clase se considerará consumida, si el usuario confirma, entonces se efectúa. 
6. Todas las operaciones respetan timezone definido (ej. America/Bogota) para las reglas de “3 horas / 10 horas de anticipación”.



### 5. Constraints (Restricciones)

- **Timezone**: Todas las reglas de cancelación (3h / 10h de anticipación) se calculan en una zona horaria definida (ej. America/Bogota). Definir y documentar en el código.
- **Una reserva por slot**: Un usuario no puede tener dos reservas activas para la misma clase (mismo día y misma hora de inicio).
- **Lista de espera**: Al liberar un cupo por cancelación, el sistema debe poder asignar la plaza al primero en la lista de espera (y notificar si está definido en alcance).
- **Fuente de planes**: Definir si “plan activo” y “clases disponibles” vienen de BD del mismo sistema o de integración externa (API/CRM); el módulo debe tener una interfaz clara para obtener estos datos.

**Casos borde a manejar**

- El usuario no tiene plan activo → no puede reservar; mensaje claro.
- El usuario tiene clases pendientes por usar pero ya las tiene todas reservadas → no puede reservar más hasta cancelar o usar una.
- El usuario intenta cancelar una reserva pasada → no permitir; mensaje claro.
- El usuario intenta cancelar fuera de la ventana (menos de 3h o menos de 10h para clase 6am) → la clase se considera consumida; no se libera cupo.
- No hay cupo en la clase → el usuario puede solo entrar a lista de espera (no reservar).
- El usuario intenta reservar dos veces la misma clase (mismo día/hora) → rechazar.
- Clase de 6am: ventana de cancelación 10h (ej. cancelar antes de 20:00 del día anterior).


---

### 6. Definition of Done (DoD)

El trabajo se considera terminado cuando:

- El código pasa **flake8**
- La cobertura de tests es **> 90%**
- Existen tests unitarios usando **pytest**
- Los tests incluyen:
  - reservas cuando el plan está activo
  - reservas cuando el plan está vencido
  - reservas cuando no tiene más clases disponibles
  - reservas cuando no hay cupo (solo lista de espera) y cuando hay cupo
  - lista de espera: orden de prioridad y (si aplica) asignación al cancelar un cupo
  - cancelación dentro de la ventana (3h) y fuera de la ventana (clase descontada)
  - cancelación clase 6am: dentro de 10h y fuera de 10h
  - no permitir doble reserva en el mismo slot
  - no permitir cancelar una reserva pasada

- Todas las interfaces públicas tienen **docstrings**
- El servicio puede extenderse agregando nuevas estrategias sin modificar el servicio principal
