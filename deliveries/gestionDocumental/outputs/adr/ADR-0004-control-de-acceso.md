# ADR-0004 · Control de acceso por departamento: RBAC simple con verificacion en capa de servicio

**Estado:** aceptado
**Fecha:** 2026-06-28

---

## Contexto y fuerza

**Fuerza principal — requisito de seguridad R-16:** "El control de acceso restringe la visualizacion de documentos estrictamente a las areas departamentales sobre las que el usuario tiene permiso explicito; documentos de otros departamentos son inaccesibles aunque el usuario conozca su codigo."

**Fuerza de historia — US-07 AC-1:** "Dado que un usuario esta autenticado, cuando intenta acceder a un documento de otro departamento por URL directa o busqueda por codigo, entonces el sistema devuelve el mensaje `Sin permiso` y no muestra ningun contenido del documento." Este criterio exige que el control no se aplique solo en las listas (bandejas), sino tambien en el acceso directo a recursos individuales.

**Fuerza de historia — US-07 AC-2:** Los resultados de busqueda solo incluyen documentos del departamento del usuario.

**Fuerza de configuracion (backlog.json OQ-02 resuelto):** No existe modulo de administracion de usuarios en v1. Los usuarios se pre-cargan con datos iniciales (bootstrap). La asignacion de departamento es un dato estatico de configuracion inicial, no dinamico.

**Fuerza de escala — piloto con un solo departamento (backlog.json OQ-01 resuelto):** El piloto arranca con un unico departamento. La logica de aislamiento entre departamentos es necesaria para garantizar que la arquitectura es correcta desde el inicio, pero no necesita soportar escenarios de multi-tenencia complejos en v1.

---

## Decision

Se implementa un **RBAC (Role-Based Access Control) simple con filtro de departamento aplicado en dos capas**:

**Capa 1 — Modelo de datos:** Cada usuario tiene un campo `departamento_id` (clave foranea a la tabla `departamento`). Cada documento tiene un campo `departamento_id` que se asigna en el momento del registro (US-01) y no puede modificarse. Los tres roles del sistema (Secretaria, Director, Colaborador) se representan como un campo `rol` en la tabla de usuario.

**Capa 2 — Filtro en queries de lista (bandejas):** Todas las queries que devuelven listas de documentos incluyen el filtro `WHERE documento.departamento_id = usuario_actual.departamento_id`. Esto garantiza que las bandejas nunca muestran documentos de otros departamentos.

**Capa 3 — Verificacion en acceso a recurso individual (vista de detalle, descarga de PDF):** Antes de mostrar cualquier documento individual (por URL directa o por codigo), la capa de servicio verifica explicitamente que `documento.departamento_id == usuario_actual.departamento_id`. Si no coincide, devuelve HTTP 403 con el mensaje "Sin permiso" (US-07 AC-1). Esta verificacion ocurre en la capa de servicio, no solo en la vista, para que los tests unitarios puedan validarla independientemente del framework web.

**Implementacion:** Un decorador/mixin reutilizable (`DepartamentoRequeridoMixin`) aplica la verificacion de la capa 3 a todas las vistas de detalle de documento. No se duplica la logica en cada vista.

---

## Alternativas consideradas

- **Row-Level Security (RLS) en PostgreSQL:** El motor de base de datos aplica automaticamente el filtro de departamento a nivel de politica de fila. Elimina el riesgo de olvidar el filtro en alguna query. Sin embargo, requiere PostgreSQL como motor obligatorio (descartando SQLite para el piloto, decision incompatible con OQ-A1), requiere un DBA o conocimiento avanzado de PostgreSQL para configurarlo correctamente, y hace que la logica de seguridad quede oculta en el motor de BD en lugar de ser visible en el codigo del servidor. Para v1 con un departamento y tres usuarios, la complejidad no se justifica.

- **Middleware global que reescribe todas las queries:** Un middleware intercepta todas las queries de ORM y agrega automaticamente el filtro de departamento. Tecnicamente posible, pero opaco y dificil de depurar. Si el middleware falla silenciosamente, el sistema filtra incorrectamente sin error visible. La verificacion explicita en la capa de servicio es mas testeable y auditable.

- **Sin filtro de departamento en v1 (solo aislamiento por piloto):** Dado que el piloto arranca con un solo departamento, tecnicamente no hay datos de otro departamento que proteger. Sin embargo, R-16 es un requisito no funcional de seguridad explicitamente en el backlog (US-07, 5 puntos). Omitirlo significa que cuando se agregue el segundo departamento habra que auditar todo el codigo para agregar los filtros; es mas barato hacerlo bien desde el inicio.

---

## Consecuencias

**Ganamos:**
- El filtro de departamento es visible y testeable en el codigo del servidor; no esta oculto en el motor de BD.
- El decorador `DepartamentoRequeridoMixin` centraliza la logica de verificacion; agregar una nueva vista de detalle no requiere recordar el filtro manualmente.
- El modelo funciona identicamente con SQLite (piloto) y PostgreSQL (produccion).
- US-07 AC-1 (acceso por URL directa) queda cubierto por la capa de servicio, no solo por la UI.

**Aceptamos:**
- La verificacion en la capa de servicio debe aplicarse consistentemente a cada nueva vista que muestre datos de documentos. Si un desarrollador crea una vista nueva sin el decorador, el filtro no se aplica. Esto debe verificarse en code review.
- El campo `departamento_id` en la tabla `documento` debe indexarse para que las queries de bandeja (que siempre filtran por departamento) sean eficientes incluso cuando crezca el volumen de documentos.
