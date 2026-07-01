# ADR-0003 · Almacenamiento de archivos: sistema de archivos local (directorio media/)

**Estado:** aceptado
**Fecha:** 2026-06-28

---

## Contexto y fuerza

**Fuerza principal — dos tipos de PDF en el ciclo documental (R-01, R-11):**

- R-01: al registrar un documento externo, la Secretaria sube el PDF escaneado del oficio en papel.
- R-11: al responder, el Colaborador sube el PDF de respuesta, que el sistema vincula automaticamente al tramite origen.

Ambos PDFs deben poder subirse, almacenarse y descargarse desde la interfaz web. El tamano tipico de un oficio escaneado es de 100 KB a 2 MB; el volumen esperado en el piloto (un departamento, carga diaria de 3 usuarios) no es significativo.

**Fuerza de infraestructura — Riesgo 3 del mvp-canvas:** "La institucion puede no disponer de infraestructura minima (servidor local o nube) con conectividad estable." El sistema debe funcionar en un servidor local sin dependencias de servicios en la nube.

**Fuerza de seguridad — control de acceso a los archivos (R-16, US-07 AC-1):** Los PDFs no deben ser accesibles mediante URL estatica publica. El acceso debe verificar que el usuario tiene permiso sobre el departamento al que pertenece el tramite, igual que cualquier otra vista del sistema.

---

## Decision

Los archivos PDF (escaneados y de respuesta) se almacenan en el **sistema de archivos del servidor**, en un directorio configurable llamado `media/`. La ruta del archivo se guarda como campo de texto en la tabla `documento` de la base de datos.

**Reglas de la implementacion:**

- El directorio `media/` esta fuera del directorio de archivos estaticos publicos del framework; no es accesible directamente por URL.
- La descarga de un PDF ocurre a traves de una vista del servidor de aplicacion que primero verifica que el usuario autenticado tiene permiso sobre el departamento del tramite (mismo control que aplica el Middleware de acceso del ADR-0004), y solo entonces sirve el archivo con `FileResponse` / `X-Accel-Redirect`.
- La estructura interna del directorio es `media/tramites/<anio>/<codigo_TRM>/<tipo>_<nombre_original>.pdf` para facilitar un respaldo manual si se necesita (OQ-A4).

---

## Alternativas consideradas

- **Almacenamiento de blobs en la base de datos (columna BYTEA/BLOB):** Almacenar PDFs como bytes en la BD mezcla dos naturalezas de dato (estructurado y binario) en el mismo motor. Degrada el rendimiento de las queries de listado y de backup. Aumenta el tamano de la BD de forma rapida y no proporcional a la informacion textual. No se considera adecuado para archivos binarios de tamano variable.

- **Object storage en la nube (AWS S3, MinIO, Cloudflare R2):** Exige conectividad estable con el proveedor externo o desplegar MinIO como servicio adicional en el servidor. Contradice el Riesgo 3 del mvp-canvas. MinIO en el mismo servidor anadiriria un proceso adicional a operar y configurar, sin beneficio real para el volumen del piloto.

- **MinIO local (object storage autoalojado):** Tecnicamente resuelve el riesgo de nube, pero introduce un proceso adicional de servidor que aumenta la complejidad de despliegue. Para un piloto de tres usuarios, el beneficio no justifica el costo operacional. Se puede reconsiderar en v2 si el volumen de archivos crece significativamente.

---

## Consecuencias

**Ganamos:**
- Cero dependencias externas para el almacenamiento: el sistema funciona con solo el SO del servidor.
- El respaldo de archivos es tan simple como copiar el directorio `media/` (incluso con herramientas del SO como `rsync` o copia manual).
- El control de acceso a los PDFs usa exactamente el mismo mecanismo que el resto del sistema (verificacion de departamento en la vista), sin logica adicional.

**Aceptamos:**
- Si el sistema escala a multiples servidores (horizontalmente), el directorio `media/` debe estar en un volumen compartido o migrarse a object storage. Para el piloto de un servidor esto no es un problema.
- La carpeta `media/` debe estar excluida del control de versiones (`.gitignore`) y debe respaldarse por separado de la base de datos, ya que los PDFs no estan en la BD.
- El tamano en disco del servidor debe monitorearse si el volumen de tramites crece. Para el piloto (estimado de decenas a pocos cientos de documentos en 60 dias segun la metrica del mvp-canvas) el espacio no es un problema pratico.
