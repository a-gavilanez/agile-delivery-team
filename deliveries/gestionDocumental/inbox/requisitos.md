# Requisitos candidatos — Gestión Documental

---

## Flujo de recepción y registro (Secretaria)

- **[R-01]** El sistema genera un código secuencial único e inmutable al registrar un documento externo, permite subir el PDF escaneado y exige completar los campos: remitente, institución de origen, asunto, fecha de recepción y tipo de documento.
  - Tipo: funcional
  - Origen: `secretaria.md` · Secretaria

- **[R-02]** El sistema bloquea el guardado del registro si alguno de los campos obligatorios definidos en R-01 está vacío, mostrando un mensaje de error descriptivo.
  - Tipo: funcional
  - Origen: `secretaria.md` · Secretaria

- **[R-03]** Los documentos recién registrados se acumulan en una bandeja diferenciada "Por Despachar", separada de la bandeja principal; al derivar el documento al Director el estado cambia automáticamente a "Recibido en Dirección" y se habilita un campo para nota aclaratoria opcional.
  - Tipo: funcional
  - Origen: `secretaria.md` · Secretaria

- **[R-04]** El sistema provee un catálogo de plantillas oficiales precargadas (membrete, logotipo institucional, pie de página) con autocompletado de fecha actual, nombre del jefe y cargo; permite editar el cuerpo en línea y descargar el documento en formato editable.
  - Tipo: funcional
  - Origen: `secretaria.md` · Secretaria

- **[R-05]** El sistema permite archivar y cerrar un expediente solo si existe al menos un documento de respuesta o resolución vinculado; al confirmar el cierre solicita la serie documental virtual de destino y cambia el estado a "Archivado", retirando el expediente de la bandeja diaria activa.
  - Tipo: funcional
  - Origen: `secretaria.md` · Secretaria

---

## Firma, control y delegación (Director)

- **[R-06]** El Director dispone de una bandeja de firma digital que lista únicamente los documentos pendientes de su aprobación; puede abrirlos en el mismo sistema para revisión y estampar la firma p12 ingresando su clave, sin necesidad de imprimir.
  - Tipo: funcional
  - Origen: `director.md` · Director

- **[R-07]** El sistema presenta al Director un dashboard con indicadores de documentos ingresados, en proceso, pendientes y archivados; resalta en rojo los trámites que han superado el plazo de respuesta; ofrece filtros por rango de fechas y por funcionario.
  - Tipo: funcional
  - Origen: `director.md` · Director

- **[R-08]** Desde el sistema, el Director puede seleccionar un documento, elegir un colaborador de su equipo, escribir una sumilla digital obligatoria (instrucción textual) y delegar; el colaborador recibe una notificación directa en su bandeja.
  - Tipo: funcional
  - Origen: `director.md` · Director

- **[R-09]** El Director puede seleccionar a un funcionario ausente, ver todos sus trámites activos y reasignarlos —de forma masiva o individual— a otro miembro disponible del mismo departamento; el sistema registra quién realizó la reasignación, la fecha y el motivo.
  - Tipo: funcional
  - Origen: `director.md` · Director

---

## Bandeja personal, respuesta y trazabilidad (Colaborador)

- **[R-10]** La bandeja de trabajo del colaborador muestra únicamente los trámites asignados directamente a ese usuario; los documentos de otras áreas no son visibles en su vista principal.
  - Tipo: funcional
  - Origen: `colaborador.md` · Colaborador

- **[R-11]** Al subir el PDF de respuesta, el sistema lo vincula automáticamente al documento origen y cambia el estado del trámite a "Respondido - Pendiente de Revisión" sin intervención adicional del usuario.
  - Tipo: funcional
  - Origen: `colaborador.md` · Colaborador

- **[R-12]** El sistema ofrece búsqueda avanzada de trámites por palabra clave (coincidencia exacta y parcial en el asunto), rango de fechas y número de trámite; los resultados solo incluyen documentos de las áreas a las que el usuario tiene permiso de acceso.
  - Tipo: funcional
  - Origen: `colaborador.md` · Colaborador

- **[R-13]** Cada trámite muestra una línea de tiempo con el historial completo: creación, sumilla, asignación, respuesta y cualquier reasignación, con actor responsable y fecha/hora de cada evento.
  - Tipo: funcional
  - Origen: `colaborador.md` · Colaborador

- **[R-14]** El sistema emite alertas en tiempo real dentro de la interfaz (ícono con contador) cuando se delega un nuevo trámite al colaborador o cuando un trámite está próximo a vencer; las alertas se clasifican por prioridad (alta en rojo para vencimientos, media para nuevas asignaciones) y al hacer clic navegan directamente al documento.
  - Tipo: funcional
  - Origen: `colaborador.md` · Colaborador

- **[R-15]** El colaborador puede vincular documentos existentes (por código) a un trámite madre mediante la opción "Vincular Documento Existente"; el sistema presenta los documentos vinculados en árbol jerárquico o lista de expediente.
  - Tipo: funcional
  - Origen: `colaborador.md` · Colaborador

---

## No funcionales

- **[R-16]** El control de acceso restringe la visualización de documentos estrictamente a las áreas departamentales sobre las que el usuario tiene permiso explícito; documentos de otros departamentos son inaccesibles aunque el usuario conozca su código.
  - Tipo: no funcional — seguridad
  - Origen: `colaborador.md` · Colaborador

- **[R-17]** La búsqueda avanzada (R-12) devuelve resultados en menos de 3 segundos para colecciones de hasta 50 000 documentos.
  - Tipo: no funcional — rendimiento
  - Origen: `colaborador.md` · Colaborador (inferido del dolor de búsqueda "lentísima")
