# Historias de usuario refinadas — Gestion Documental

> Delivery: `gestionDocumental` | Autor: Developer | Fecha: 2026-06-28
> Todas las historias cumplen INVEST + Definition of Ready. Criterios de aceptacion en Gherkin (Dado/Cuando/Entonces).

---

## [US-01] Registrar documento externo con codigo unico — Epica E-01

**Como** Secretaria, **quiero** registrar un documento externo con un codigo unico generado automaticamente y subir el PDF escaneado, completando los campos obligatorios (remitente, institucion, asunto, fecha, tipo), **para** garantizar que cada oficio que llega quede identificado e irrepetible desde el momento de su ingreso.

**Estimacion:** 5 puntos
**Epica:** E-01 — Ingreso trazable de documentos
**Origen:** `us:US-01`, `req:R-01`, `req:R-02`, `pain:registro-sin-trazabilidad`, `pain:metadatos-no-obligados`
**Dependencias:** ninguna

### Criterios de aceptacion

**AC-1:**
- **Dado** que llega un oficio en papel,
- **Cuando** la Secretaria crea el registro y pulsa Guardar,
- **Entonces** el sistema genera automaticamente un codigo secuencial unico e inmutable con formato TRM-AAAA-NNNN (ej. TRM-2026-0001).

**AC-2:**
- **Dado** que la Secretaria intenta guardar el registro con alguno de los campos obligatorios vacio (remitente, institucion, asunto, fecha de recepcion o tipo),
- **Cuando** pulsa Guardar,
- **Entonces** el sistema bloquea el guardado y muestra un mensaje descriptivo indicando cual campo falta.

**AC-3:**
- **Dado** que todos los campos obligatorios estan completos y el PDF escaneado esta adjunto,
- **Cuando** la Secretaria guarda el registro,
- **Entonces** el documento aparece en la bandeja Por Despachar con estado Ingresado y el codigo generado es visible e inmutable.

---

## [US-02] Derivar documentos al Director desde la bandeja Por Despachar — Epica E-01

**Como** Secretaria, **quiero** ver los documentos recien registrados en una bandeja separada llamada Por Despachar y derivarlos al Director con un clic, **para** no mezclar lo que acabo de recibir con el resto del sistema y que el Director vea el cambio de estado sin que yo tenga que avisar por correo o conversacion.

**Estimacion:** 3 puntos
**Epica:** E-01 — Ingreso trazable de documentos
**Origen:** `us:US-02`, `req:R-03`, `pain:bandeja-mezclada-secretaria`, `pain:derivacion-manual-director`
**Dependencias:** US-01

> Nota de refinamiento: R-03 establece explicitamente que el campo de nota aclaratoria al Director es opcional. Se mantiene opcional en v1. Si el piloto detecta un caso institucional que requiera obligatoriedad, se captura como mejora para v2.

### Criterios de aceptacion

**AC-1:**
- **Dado** que existen documentos con estado Ingresado,
- **Cuando** la Secretaria abre el sistema,
- **Entonces** ve una bandeja Por Despachar que muestra exclusivamente esos documentos y ninguno de otro estado.

**AC-2:**
- **Dado** que la Secretaria selecciona un documento de la bandeja Por Despachar,
- **Cuando** pulsa Derivar al Director y confirma (con nota aclaratoria opcional),
- **Entonces** el estado cambia automaticamente a Recibido en Direccion y el documento desaparece de la bandeja Por Despachar.

**AC-3:**
- **Dado** que el estado del documento cambio a Recibido en Direccion,
- **Cuando** el Director abre su bandeja,
- **Entonces** el documento aparece ahi sin que la Secretaria haya enviado ningun correo ni aviso externo.

---

## [US-03] Delegar un tramite a un colaborador con sumilla digital — Epica E-02

**Como** Director, **quiero** seleccionar un tramite en mi bandeja, escribir una sumilla de instruccion obligatoria y elegir a que colaborador asignarlo, **para** que la delegacion quede registrada formalmente en el sistema con la instruccion completa y el colaborador la reciba directamente en su bandeja sin papelitos ni correos paralelos.

**Estimacion:** 3 puntos
**Epica:** E-02 — Delegacion formal y asignacion
**Origen:** `us:US-03`, `req:R-08`, `pain:delegacion-informal`, `pain:ceguera-de-estado`
**Dependencias:** US-02

### Criterios de aceptacion

**AC-1:**
- **Dado** que el Director tiene un documento en su bandeja con estado Recibido en Direccion,
- **Cuando** pulsa Delegar,
- **Entonces** el sistema muestra un campo de sumilla obligatorio (minimo 10 caracteres) y una lista de colaboradores del departamento.

**AC-2:**
- **Dado** que el Director completa la sumilla y elige un colaborador,
- **Cuando** confirma la delegacion,
- **Entonces** el estado del tramite cambia a Asignado y el documento aparece en la bandeja personal del colaborador seleccionado.

**AC-3:**
- **Dado** que el tramite fue delegado,
- **Cuando** cualquier usuario autorizado abre la linea de tiempo del tramite,
- **Entonces** ve el evento de delegacion con el texto completo de la sumilla, el nombre del Director, el nombre del colaborador asignado y la fecha/hora del evento.

---

## [US-04] Ver unicamente los tramites asignados en la bandeja personal — Epica E-02

**Como** Colaborador, **quiero** que mi bandeja principal muestre solo los tramites asignados directamente a mi usuario, **para** no perder tiempo filtrando lo que no me corresponde ni correr el riesgo de atender un tramite equivocado.

**Estimacion:** 2 puntos
**Epica:** E-02 — Delegacion formal y asignacion
**Origen:** `us:US-04`, `req:R-10`, `pain:bandeja-sin-filtro`
**Dependencias:** US-03

### Criterios de aceptacion

**AC-1:**
- **Dado** que el Colaborador inicia sesion,
- **Cuando** abre su bandeja,
- **Entonces** solo ve los tramites cuyo campo asignado a coincide con su usuario; ningun otro documento es visible en esa vista.

**AC-2:**
- **Dado** que existe un documento asignado a otro colaborador o proveniente de otra area,
- **Cuando** el Colaborador navega su bandeja,
- **Entonces** ese documento no aparece en ninguna parte de su vista principal.

**AC-3:**
- **Dado** que le acaban de delegar un nuevo tramite,
- **Cuando** el Colaborador recarga la bandeja,
- **Entonces** el tramite nuevo aparece en primer lugar con una etiqueta visual Nuevo.

---

## [US-05] Subir la respuesta y vincularla automaticamente al tramite origen — Epica E-03

**Como** Colaborador, **quiero** subir el PDF de mi respuesta directamente en el tramite que estoy atendiendo, **para** que quede vinculado al documento origen y el estado cambie automaticamente, sin que yo tenga que avisar por separado al Director o a la Secretaria.

**Estimacion:** 3 puntos
**Epica:** E-03 — Respuesta vinculada y auditoria del ciclo
**Origen:** `us:US-05`, `req:R-11`, `pain:respuesta-sin-trazabilidad`, `pain:estado-no-automatico`
**Dependencias:** US-04

> Nota de refinamiento: en v1 el estado Respondido - Pendiente de Revision es el estado final del ciclo MVP. El cierre formal (R-05) esta fuera de alcance segun mvp-canvas y se pospone a v2. La linea de tiempo (US-06) registra este estado como ultimo evento conocido, garantizando trazabilidad completa del ciclo activo.

### Criterios de aceptacion

**AC-1:**
- **Dado** que el Colaborador abre un tramite con estado Asignado,
- **Cuando** sube un archivo PDF de respuesta,
- **Entonces** el sistema vincula ese PDF al tramite origen y ambos son accesibles desde la misma vista del tramite.

**AC-2:**
- **Dado** que el PDF de respuesta fue cargado con exito,
- **Cuando** el sistema lo procesa,
- **Entonces** el estado cambia automaticamente a Respondido - Pendiente de Revision sin ninguna accion adicional del Colaborador.

**AC-3:**
- **Dado** que el estado cambio a Respondido - Pendiente de Revision,
- **Cuando** el Director o la Secretaria abren el tramite,
- **Entonces** ven el estado actualizado y el enlace directo al PDF de respuesta.

---

## [US-06] Consultar la linea de tiempo completa de un tramite — Epica E-03

**Como** Colaborador o Director, **quiero** ver el historial cronologico de cada evento del tramite (creacion, derivacion, delegacion, respuesta) con el actor responsable y la fecha/hora de cada accion, **para** entender donde esta el tramite y quien hizo que sin tener que preguntar a nadie.

**Estimacion:** 3 puntos
**Epica:** E-03 — Respuesta vinculada y auditoria del ciclo
**Origen:** `us:US-06`, `req:R-13`, `pain:sin-linea-de-tiempo`, `pain:ceguera-de-estado`
**Dependencias:** US-01, US-03, US-05

### Criterios de aceptacion

**AC-1:**
- **Dado** que cualquier usuario autorizado abre un tramite,
- **Cuando** accede a la seccion Historial,
- **Entonces** ve una linea de tiempo con todos los eventos en orden cronologico: creacion, derivacion, delegacion (con sumilla), carga de respuesta y cualquier otro evento registrado.

**AC-2:**
- **Dado** que existe un evento de delegacion en la linea de tiempo,
- **Cuando** el usuario lo expande,
- **Entonces** ve el texto completo de la sumilla del Director, el nombre del colaborador asignado y la fecha/hora del evento.

**AC-3:**
- **Dado** que el tramite aun no tiene respuesta cargada,
- **Cuando** el usuario ve la linea de tiempo,
- **Entonces** el estado actual aparece destacado visualmente como el ultimo evento pendiente.

---

## [US-07] Acceder solo a los documentos del propio departamento — Epica E-04

**Como** usuario del sistema (Secretaria, Director o Colaborador), **quiero** que el sistema me muestre unicamente los documentos de mi departamento, **para** que no pueda ver ni accidentalmente abrir documentos de otras areas y los datos sensibles de cada departamento queden protegidos.

**Estimacion:** 5 puntos
**Epica:** E-04 — Acceso seguro por departamento
**Origen:** `us:US-07`, `req:R-16`, `pain:bandeja-sin-filtro`
**Dependencias:** US-01

> Nota de refinamiento: el piloto arranca con un solo departamento (mvp-canvas: segmento "del mismo departamento institucional"). No existe rol Administrador en el discovery; en v1 los usuarios se pre-cargan mediante datos iniciales (bootstrap). No se construye modulo de administracion de usuarios en v1.

### Criterios de aceptacion

**AC-1:**
- **Dado** que un usuario esta autenticado,
- **Cuando** intenta acceder a un documento de otro departamento por URL directa o busqueda por codigo,
- **Entonces** el sistema devuelve el mensaje Sin permiso y no muestra ningun contenido del documento.

**AC-2:**
- **Dado** que el Colaborador realiza una busqueda por codigo de tramite,
- **Cuando** los resultados se muestran,
- **Entonces** solo aparecen documentos que pertenecen a su departamento.

**AC-3:**
- **Dado** que un usuario pre-cargado intenta acceder al sistema,
- **Cuando** el sistema valida sus credenciales,
- **Entonces** solo puede acceder si tiene al menos un departamento asignado en los datos de configuracion inicial; en caso contrario el sistema deniega el acceso con un mensaje de usuario sin departamento asignado.

---

## Resumen del sprint

| ID | Titulo | Epica | Puntos | Estado DoR |
|----|--------|-------|--------|------------|
| US-01 | Registrar documento externo con codigo unico | E-01 | 5 | Ready |
| US-02 | Derivar documentos al Director desde la bandeja Por Despachar | E-01 | 3 | Ready |
| US-03 | Delegar un tramite a un colaborador con sumilla digital | E-02 | 3 | Ready |
| US-04 | Ver unicamente los tramites asignados en la bandeja personal | E-02 | 2 | Ready |
| US-05 | Subir la respuesta y vincularla automaticamente al tramite origen | E-03 | 3 | Ready |
| US-06 | Consultar la linea de tiempo completa de un tramite | E-03 | 3 | Ready |
| US-07 | Acceder solo a los documentos del propio departamento | E-04 | 5 | Ready |

**Total:** 24 puntos
