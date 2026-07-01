# ADR-0001 · Estilo de aplicacion: monolito MVC con renderizado en servidor

**Estado:** aceptado
**Fecha:** 2026-06-28

---

## Contexto y fuerza

**Fuerza principal — Riesgo 3 del mvp-canvas:** "La institucion puede no disponer de infraestructura minima (servidor local o nube) con conectividad estable." La institucion debe poder desplegar el sistema en un servidor local sin personal de DevOps especializado. Cualquier arquitectura que exija configurar contenedores separados, balanceadores, o pipelines de CI/CD introduce friccion de despliegue incompatible con este riesgo.

**Fuerza secundaria — Dominio simple y carga baja:** El MVP contempla tres actores, cuatro estados de tramite y un solo departamento en el piloto (backlog.json OQ-01 resuelto). No hay integraciones externas, ni procesamiento asincrono complejo, ni multi-tenencia en v1. El dominio no justifica la complejidad de una arquitectura distribuida.

**Fuerza de producto — modulo admin fuera de alcance (backlog.json OQ-02 resuelto):** Los usuarios se pre-cargan mediante datos iniciales (bootstrap). Django, el framework MVC propuesto, incluye un panel de administracion integrado que cubre este bootstrap sin codigo adicional.

---

## Decision

Se construye el sistema como un **monolito MVC con renderizado de HTML en el servidor**, usando **Django (Python)** como implementacion de referencia.

- El servidor genera las paginas HTML completas y las entrega al navegador.
- No existe un proceso de frontend separado (sin build de JavaScript, sin servidor de desarrollo de SPA).
- La base de datos, la logica de negocio y las vistas residen en el mismo proceso del servidor de aplicacion.
- El despliegue minimo viable es: `python manage.py runserver` en el servidor local, con SQLite como base de datos (cero dependencias externas adicionales).

**Razon de Django sobre otras opciones MVC:**
- ORM con migraciones, autenticacion, manejo de archivos y panel admin integrados — reduce el codigo a escribir.
- SQLite viene con Python (zero config para el piloto); migracion a PostgreSQL es cambiar una linea de configuracion.
- Amplia documentacion y comunidad; reduce el riesgo de bloqueo si cambia el desarrollador.

---

## Alternativas consideradas

- **SPA (React/Vue) + API REST:** Requiere configurar CORS, dos procesos separados (frontend build + backend API), y desplegar artefactos de frontend. Introduce complejidad operacional que contradice el Riesgo 3. El ciclo de estados documental no necesita interactividad en tiempo real ni una UX de aplicacion movil.

- **Laravel (PHP):** Comparable a Django en simplicidad MVC. PHP esta preinstalado en muchos servidores compartidos, lo que es una ventaja. Sin embargo, el equipo de discovery no menciona PHP y Django ofrece el mismo nivel de simplicidad con SQLite integrado. Se puede sustituir sin cambiar la decision arquitectonica (monolito MVC sigue siendo la decision).

- **Microservicios:** Incompatible con el Riesgo 3 y con la carga de 3 usuarios. No se considera para v1.

---

## Consecuencias

**Ganamos:**
- Despliegue en una sola maquina con un solo comando; sin necesidad de orquestracion ni contenedores.
- Toda la logica de negocio en un unico lugar; facil de depurar y modificar.
- El panel admin de Django cubre el bootstrap de usuarios (OQ-02 resuelto) sin codigo adicional.
- Sin complejidad de CORS, autenticacion distribuida ni sincronizacion de estado entre cliente y servidor.

**Aceptamos:**
- En v2, si se requieren alertas en tiempo real (R-14, fuera de alcance en v1 segun mvp-canvas), habra que anadir WebSockets o SSE al monolito o extraerlo a un componente separado.
- Si la carga crece a multiples departamentos concurrentes, el monolito podra necesitar ser ejecutado detras de un servidor WSGI (Gunicorn) con varios workers — cambio operacional, no arquitectonico.
- Renderizado en servidor implica recargas de pagina completas; la experiencia de usuario es menos fluida que una SPA, pero suficiente para el caso de uso institucional de escritorio.
