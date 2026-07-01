# ADR-0002 · Maquina de estados del documento: campo enum + tabla de eventos append-only

**Estado:** aceptado
**Fecha:** 2026-06-28

---

## Contexto y fuerza

**Fuerza principal — cambios de estado automaticos (US-02 AC-2, US-03 AC-2, US-05 AC-2):**

- US-02 AC-2: cuando la Secretaria pulsa "Derivar al Director", el estado pasa automaticamente de `Ingresado` a `Recibido en Direccion`.
- US-03 AC-2: cuando el Director confirma la delegacion, el estado pasa automaticamente a `Asignado`.
- US-05 AC-2: cuando el Colaborador sube el PDF de respuesta con exito, el estado pasa automaticamente a `Respondido - Pendiente de Revision`.

Los tres criterios de aceptacion exigen que la transicion ocurra sin intervencion adicional del usuario. Esto requiere logica de negocio que controle cuales transiciones son validas desde cada estado.

**Fuerza secundaria — linea de tiempo auditable (US-06, R-13):**

US-06 AC-1 exige ver todos los eventos del tramite en orden cronologico con actor y timestamp. US-06 AC-2 exige ver el texto completo de la sumilla en el evento de delegacion. Esto requiere que cada transicion de estado genere un registro persistente con metadata adicional.

**Fuerza de alcance — cuatro estados lineales (mvp-canvas, backlog.json OQ-03 resuelto):**

El ciclo documental del MVP tiene exactamente cuatro estados con una cadena lineal de transiciones: `Ingresado → Recibido en Direccion → Asignado → Respondido - Pendiente de Revision`. El estado `Respondido - Pendiente de Revision` es el estado final en v1 (cierre formal R-05 fuera de alcance segun mvp-canvas).

---

## Decision

Se implementa la maquina de estados con **dos mecanismos complementarios**:

1. **Campo `estado` (enum) en la tabla `documento`:** es el source of truth del estado actual del tramite. Permite filtrar bandejas y mostrar el estado actual con una sola columna indexada. Los cuatro valores posibles son: `INGRESADO`, `RECIBIDO_DIRECCION`, `ASIGNADO`, `RESPONDIDO_PENDIENTE`.

2. **Logica de transicion en la capa de servicio (no en vistas):** una funcion/metodo de servicio valida que la transicion solicitada esta permitida desde el estado actual antes de ejecutarla. Las transiciones validas son exactamente las cuatro del ciclo (no existen bifurcaciones en v1). Si se solicita una transicion invalida, el servicio lanza una excepcion de negocio y la base de datos no se modifica.

3. **Tabla `evento_tramite` append-only:** cada vez que la capa de servicio ejecuta una transicion valida, inserta una fila en `evento_tramite` con: `tramite_id`, `tipo_evento` (REGISTRO, DERIVACION, DELEGACION, RESPUESTA_CARGADA), `actor_id`, `timestamp`, `metadata` (JSON — contiene la sumilla en el evento DELEGACION y la ruta del archivo en RESPUESTA_CARGADA). Esta tabla nunca se actualiza ni se borra.

La tabla `evento_tramite` **no** es el source of truth del estado — el campo `estado` lo es. La tabla sirve exclusivamente para la linea de tiempo (US-06).

---

## Alternativas consideradas

- **Event sourcing completo (estado derivado de eventos):** El estado actual del tramite se recalcula reproduciendo todos los eventos. Util cuando el estado es complejo o puede recalcularse de multiples formas. Innecesario aqui: cuatro estados lineales sin bifurcaciones no justifican la complejidad de un projector de eventos. Ademas, filtrar bandejas por estado requeriria una vista materializada adicional.

- **Solo campo enum sin tabla de eventos:** Simple para el estado actual, pero no cumple US-06 (linea de tiempo con actor y timestamp por cada evento). La historia US-03 AC-3 exige el texto de la sumilla disponible en la linea de tiempo, que no puede almacenarse en el campo enum.

- **Tabla de transiciones generica con origen/destino configurable:** Util cuando los estados y transiciones son configurables por el usuario final (workflow engine). El ciclo documental del MVP es fijo y conocido; no hay razon para hacerlo configurable en v1 (aumentaria la complejidad sin valor para el piloto).

---

## Consecuencias

**Ganamos:**
- El estado actual del tramite es una columna indexable: las queries de bandeja (US-02 AC-1, US-04 AC-1) son eficientes sin joins adicionales.
- La validacion de transiciones en la capa de servicio garantiza que ningun estado invalido puede llegar a la base de datos, independientemente de como se invoque el servicio (vista web, script de bootstrap, test).
- La tabla `evento_tramite` cumple los tres criterios de aceptacion de US-06 con una sola query de SELECT ordenada por timestamp.
- El modelo es extensible: si en v2 se agrega el estado `Archivado` (R-05), basta con anadir el valor al enum y una nueva transicion en la capa de servicio.

**Aceptamos:**
- La consistencia entre el campo `estado` y la tabla `evento_tramite` debe garantizarse en una sola transaccion de base de datos (actualizar estado + insertar evento en un mismo bloque atomico). Si se separan, puede existir un estado sin evento o un evento sin cambio de estado.
- Si en v2 aparecen bifurcaciones de estado (por ejemplo, el Director puede rechazar una respuesta y devolverla a `Asignado`), la logica de transicion debera revisarse, aunque el modelo sigue siendo valido.
