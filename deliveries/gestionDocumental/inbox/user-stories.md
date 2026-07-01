# User Stories — Gestión Documental (MVP)

Núcleo de valor: **ciclo de vida documental trazable** — desde el registro con
código único hasta la respuesta vinculada, con bandejas personalizadas y
delegación formal. Son los dolores más frecuentes y compartidos entre las tres
personas del discovery.

---

## Secretaria

### [US-01] Registrar documento externo con código único

- **Como** Secretaria, **quiero** registrar un documento externo con un código
  único generado automáticamente y subir el PDF escaneado, completando los campos
  obligatorios (remitente, institución, asunto, fecha, tipo), **para** garantizar
  que cada oficio que llega quede identificado e irrepetible desde el momento de
  su ingreso.
  - Criterios de aceptación:
    - Dado que llega un oficio en papel, cuando la Secretaria crea el registro, entonces el sistema genera automáticamente un código secuencial inmutable (p. ej. `TRM-2026-0001`).
    - Dado que la Secretaria intenta guardar el registro con algún campo obligatorio vacío, cuando pulsa "Guardar", entonces el sistema bloquea el guardado y muestra un mensaje que indica cuál campo falta.
    - Dado que todos los campos obligatorios están completos, cuando la Secretaria guarda, entonces el registro aparece en la bandeja "Por Despachar" con estado "Ingresado".
  - Fuente: `secretaria.md` (dolores: `registro-sin-trazabilidad`, `metadatos-no-obligados`)

---

### [US-02] Derivar un documento al Director desde la bandeja "Por Despachar"

- **Como** Secretaria, **quiero** ver mis documentos recién registrados en una
  bandeja separada y derivarlos al Director con un clic, **para** no mezclar lo
  que acabo de recibir con el resto del sistema y asegurar que el Director vea el
  cambio de estado sin que yo tenga que avisar por correo o conversación.
  - Criterios de aceptación:
    - Dado que existen documentos con estado "Ingresado", cuando la Secretaria abre el sistema, entonces ve una bandeja "Por Despachar" que solo muestra esos documentos.
    - Dado que la Secretaria selecciona un documento de "Por Despachar", cuando pulsa "Derivar al Director" y confirma (con nota aclaratoria opcional), entonces el estado cambia automáticamente a "Recibido en Dirección" y el documento sale de "Por Despachar".
    - Dado que el estado cambió a "Recibido en Dirección", cuando el Director abre su bandeja, entonces el documento aparece ahí sin que la Secretaria haya enviado ningún correo.
  - Fuente: `secretaria.md` (dolores: `bandeja-mezclada-secretaria`, `derivacion-manual-director`)

---

## Director

### [US-03] Delegar un trámite a un colaborador con sumilla digital

- **Como** Director, **quiero** seleccionar un trámite en mi bandeja, escribir una
  sumilla de instrucción y elegir a qué colaborador asignarlo, **para** que la
  delegación quede registrada formalmente en el sistema y el colaborador reciba
  el aviso directo en su bandeja, sin papelitos ni correos paralelos.
  - Criterios de aceptación:
    - Dado que el Director recibe un documento en su bandeja, cuando pulsa "Delegar", entonces el sistema muestra un campo de sumilla obligatorio (mínimo 10 caracteres) y una lista de colaboradores del departamento.
    - Dado que el Director completa la sumilla y elige un colaborador, cuando confirma la delegación, entonces el estado del trámite cambia a "Asignado" y el documento aparece en la bandeja personal del colaborador seleccionado.
    - Dado que el trámite fue delegado, cuando cualquier usuario autorizado abre la línea de tiempo del trámite, entonces ve el evento de delegación con el texto completo de la sumilla, el nombre del Director, el nombre del colaborador y la fecha/hora.
  - Fuente: `director.md` (dolores: `delegacion-informal`, `ceguera-de-estado`)

---

## Colaborador

### [US-04] Ver únicamente los trámites asignados en mi bandeja personal

- **Como** Colaborador, **quiero** que mi bandeja principal muestre solo los
  trámites que me han asignado directamente a mí, **para** no perder tiempo
  filtrando lo que no me corresponde ni correr el riesgo de atender un trámite
  equivocado.
  - Criterios de aceptación:
    - Dado que el Colaborador inicia sesión, cuando abre su bandeja, entonces solo ve los trámites cuyo campo "asignado a" coincide con su usuario.
    - Dado que existe un documento asignado a otro colaborador o de otra área, cuando el Colaborador navega su bandeja, entonces ese documento no aparece en su vista principal.
    - Dado que le acaban de delegar un nuevo trámite, cuando el Colaborador recarga la bandeja, entonces el trámite nuevo aparece en primer lugar con una etiqueta visual "Nuevo".
  - Fuente: `colaborador.md` (dolor: `bandeja-sin-filtro`)

---

### [US-05] Subir la respuesta y vincularla automáticamente al trámite origen

- **Como** Colaborador, **quiero** subir el PDF de mi respuesta directamente en
  el trámite que estoy atendiendo, **para** que quede vinculado al documento
  origen y el estado cambie automáticamente, sin que yo tenga que avisar por
  separado al Director o a la Secretaria.
  - Criterios de aceptación:
    - Dado que el Colaborador abre un trámite con estado "Asignado", cuando sube un archivo PDF de respuesta, entonces el sistema vincula ese PDF al trámite origen y ambos son accesibles desde la misma vista del trámite.
    - Dado que el PDF de respuesta fue cargado con éxito, cuando el sistema lo procesa, entonces el estado cambia automáticamente a "Respondido - Pendiente de Revisión" sin ninguna acción adicional del Colaborador.
    - Dado que el estado cambió, cuando el Director o la Secretaria abren el trámite, entonces ven el estado actualizado y el enlace al PDF de respuesta.
  - Fuente: `colaborador.md` (dolores: `respuesta-sin-trazabilidad`, `estado-no-automatico`)

---

### [US-06] Consultar la línea de tiempo completa de un trámite

- **Como** Colaborador o Director, **quiero** ver el historial cronológico de
  cada evento del trámite (creación, derivación, delegación, respuesta) con el
  actor responsable y la fecha/hora, **para** entender dónde está un trámite y
  quién hizo qué sin tener que preguntar a nadie.
  - Criterios de aceptación:
    - Dado que cualquier usuario autorizado abre un trámite, cuando accede a la sección "Historial", entonces ve una línea de tiempo con todos los eventos en orden cronológico: creación, derivación, delegación (con sumilla), respuesta y cualquier reasignación futura.
    - Dado que existe un evento de delegación en la línea de tiempo, cuando el usuario lo expande, entonces ve el texto completo de la sumilla del Director.
    - Dado que el trámite aún no tiene respuesta, cuando el usuario ve la línea de tiempo, entonces el estado actual aparece destacado visualmente como el último evento pendiente.
  - Fuente: `colaborador.md` (dolor: `sin-linea-de-tiempo`) · `director.md` (dolor: `ceguera-de-estado`)

---

### [US-07] Acceder solo a los documentos de mi departamento

- **Como** usuario del sistema (Secretaria, Director o Colaborador), **quiero**
  que el sistema me muestre únicamente los documentos de mi departamento, **para**
  que no pueda ver —ni accidentalmente abrir— documentos de otras áreas y los
  datos sensibles de cada área queden protegidos.
  - Criterios de aceptación:
    - Dado que un usuario está autenticado, cuando intenta acceder a un documento de otro departamento (por URL directa o búsqueda), entonces el sistema devuelve un mensaje de "Sin permiso" y no muestra ningún contenido del documento.
    - Dado que el Colaborador realiza una búsqueda por código de trámite, cuando los resultados se muestran, entonces solo aparecen documentos de su departamento.
    - Dado que el administrador crea un nuevo usuario, cuando configura su perfil, entonces debe asociarlo a al menos un departamento antes de que el usuario pueda iniciar sesión.
  - Fuente: `colaborador.md` (dolor: `bandeja-sin-filtro`) · requisito R-16

---

## Fuera del MVP — priorizadas para v2

| Historia | Requisito base | Motivo de exclusión |
|---|---|---|
| Firma digital p12 en línea | R-06 | Integración técnica compleja con certificado p12; el Director puede firmar en papel en v1 sin bloquear el flujo digital |
| Dashboard de indicadores por funcionario | R-07 | La línea de tiempo (US-06) da visibilidad básica suficiente; el panel completo es v2 |
| Plantillas con membrete institucional | R-04 | Word con archivos anteriores funciona; las plantillas mejoran eficiencia pero no la trazabilidad |
| Cierre formal de expediente con validación | R-05 | Sin flujo activo primero, el proceso de cierre no aporta valor real |
| Reasignación masiva por ausencia | R-09 | Caso de borde de baja frecuencia; US-03 cubre la delegación unitaria |
| Búsqueda avanzada por palabra clave y fecha | R-12 | La bandeja personal (US-04) es suficiente en v1; la búsqueda avanzada es optimización |
| Alertas en tiempo real | R-14 | La etiqueta "Nuevo" en la bandeja (US-04) cubre la necesidad mínima |
| Expediente vinculado en árbol jerárquico | R-15 | Alta complejidad de UX; la línea de tiempo (US-06) es el registro suficiente en v1 |
| Rendimiento garantizado a 50 000 documentos | R-17 | Escala que la institución no alcanzará en el horizonte del MVP |
