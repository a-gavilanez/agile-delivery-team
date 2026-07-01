# ADR-0005 · Registro de eventos (linea de tiempo): tabla append-only con metadata JSON

**Estado:** aceptado
**Fecha:** 2026-06-28

---

## Contexto y fuerza

**Fuerza principal — US-06 (tres criterios de aceptacion sobre la linea de tiempo):**

- US-06 AC-1: la seccion "Historial" muestra todos los eventos en orden cronologico (creacion, derivacion, delegacion con sumilla, carga de respuesta y cualquier otro evento registrado).
- US-06 AC-2: el usuario puede expandir el evento de delegacion y ver el texto completo de la sumilla del Director, el nombre del colaborador asignado y la fecha/hora.
- US-06 AC-3: el estado actual aparece destacado visualmente como el ultimo evento pendiente cuando el tramite no tiene respuesta.

**Fuerza secundaria — R-13:** "Cada tramite muestra una linea de tiempo con el historial completo: creacion, sumilla, asignacion, respuesta y cualquier reasignacion, con actor responsable y fecha/hora de cada evento."

**Fuerza de integracion — US-03 AC-3:** el evento de delegacion debe preservar el texto completo de la sumilla del Director. La sumilla es texto libre (minimo 10 caracteres, sin limite maximo definido en el inbox); debe almacenarse como dato estructurado, no solo como referencia al campo del formulario.

**Fuerza de auditoria:** La linea de tiempo es el mecanismo de auditoria del sistema; es la respuesta a "quien hizo que y cuando" (mvp-canvas: "El Director puede auditar el estado de todos los tramites sin preguntar individualmente"). Los registros de eventos no deben poder editarse ni borrarse.

---

## Decision

Se implementa una **tabla `evento_tramite` en la base de datos relacional, de naturaleza append-only**:

**Esquema de la tabla:**

| Columna | Tipo | Descripcion |
|---|---|---|
| `id` | INTEGER PRIMARY KEY | Identificador autoincremental |
| `tramite_id` | FK → documento.id | Tramite al que pertenece el evento |
| `tipo_evento` | ENUM o VARCHAR | REGISTRO, DERIVACION, DELEGACION, RESPUESTA_CARGADA |
| `actor_id` | FK → usuario.id | Usuario que realizo la accion |
| `timestamp` | DATETIME | Momento exacto del evento (UTC) |
| `metadata` | JSON / TEXT | Datos adicionales del evento (ver abajo) |

**Contenido del campo `metadata` segun tipo de evento:**

- `REGISTRO`: `{"codigo_trm": "TRM-2026-0001", "remitente": "...", "asunto": "..."}`
- `DERIVACION`: `{"nota_aclaratoria": "..."}`  (puede ser null si la nota es opcional)
- `DELEGACION`: `{"sumilla": "<texto completo>", "colaborador_nombre": "...", "colaborador_id": N}`
- `RESPUESTA_CARGADA`: `{"nombre_archivo": "...", "ruta_relativa": "media/tramites/..."}`

**Garantia append-only:** No se definen metodos de UPDATE ni DELETE sobre esta tabla en la capa de servicio. Las migraciones futuras no deben agregar columnas que permitan modificar registros existentes. La unica operacion permitida es INSERT.

**Relacion con ADR-0002:** La tabla `evento_tramite` es complementaria al campo `estado` de la tabla `documento`. El campo `estado` es el source of truth del estado actual; la tabla `evento_tramite` es el registro historico de como se llego a ese estado. Ambas se actualizan en la misma transaccion de base de datos.

---

## Alternativas consideradas

- **Event sourcing completo (el estado se deriva reproduciendo los eventos):** El estado actual del tramite se calcula aplicando los eventos en orden. Util cuando el estado es complejo, cuando hay proyecciones multiples o cuando se necesita "viajar en el tiempo". Para cuatro estados lineales y una sola proyeccion (el estado actual) es una complejidad innecesaria. Ademas, requeriria una vista materializada o cache para que las queries de bandeja sean eficientes.

- **Log en archivo de texto:** Los eventos se escriben en un archivo de log del servidor (por ejemplo, usando el sistema de logging del framework). No es queryable directamente: mostrar la linea de tiempo de US-06 requeriria parsear el archivo de log, lo cual es fragil y lento. No cumple US-06 AC-2 (expandir el evento de delegacion para ver la sumilla) de forma estructurada.

- **Campo JSON de historial en la tabla `documento`:** El historial de eventos se almacena como un array JSON en una columna de la tabla `documento`. Facil de leer para un tramite, pero dificil de consultar entre tramites, no garantiza la inmutabilidad (el JSON puede sobreescribirse) y crece indefinidamente en la misma fila. No es una buena practica para datos de auditoria.

---

## Consecuencias

**Ganamos:**
- La linea de tiempo de US-06 es una query `SELECT * FROM evento_tramite WHERE tramite_id = X ORDER BY timestamp ASC` — simple, eficiente e indexable.
- El campo `metadata` JSON permite almacenar el texto de la sumilla (US-03 AC-3, US-06 AC-2) sin necesitar una tabla separada por tipo de evento.
- La naturaleza append-only garantiza que el historial es inmutable: ningun actor puede modificar o eliminar eventos pasados, lo que da valor de auditoria real al sistema.
- El esquema es extensible: si en v2 se agrega el evento `REASIGNACION` (R-09, fuera de alcance en v1), basta con agregar el valor al enum de tipo y definir el contenido del JSON de metadata.

**Aceptamos:**
- La columna `metadata` de tipo JSON requiere que el codigo de presentacion sepa como parsear cada tipo de evento para mostrarlo en la linea de tiempo. Esto debe encapsularse en la capa de presentacion (template o clase de vista) para evitar logica dispersa.
- Si SQLite es el motor del piloto (OQ-A1), el soporte de JSON es funcional pero mas limitado que en PostgreSQL. Las operaciones de metadata se hacen principalmente en Python (deserializacion del JSON al leer), no con funciones JSON del motor de BD, lo que es compatible con ambos motores.
- La consistencia entre el INSERT en `evento_tramite` y el UPDATE en `documento.estado` debe garantizarse en una transaccion atomica (relacionado con la consecuencia aceptada en ADR-0002).
