# ADR-0006 · Autenticacion: sesiones clasicas del framework (sin JWT)

**Estado:** aceptado
**Fecha:** 2026-06-28

---

## Contexto y fuerza

**Fuerza principal — tres roles fijos sin modulo de administracion (backlog.json OQ-02 resuelto):** El discovery define exactamente tres actores: Secretaria, Director y Colaborador. No existe un cuarto rol de "Administrador" en el MVP. Los usuarios se pre-cargan mediante datos iniciales (bootstrap) antes del primer uso. No hay flujo de registro libre ni de recuperacion de contrasena en v1 (mvp-canvas: fuera de alcance).

**Fuerza de arquitectura — monolito MVC (ADR-0001):** El servidor genera las paginas HTML y las entrega al navegador. No hay un proceso de frontend separado que necesite llamar a una API REST desde otro origen. El escenario tipico de uso de JWT (cliente SPA que consume API de un servidor diferente) no aplica aqui.

**Fuerza de simplicidad operacional — Riesgo 3 del mvp-canvas:** El sistema debe poder desplegarse en un servidor local sin configuracion avanzada. Las sesiones clasicas son el mecanismo de autenticacion nativo de Django: funcionan sin configuracion adicional, sin claves RSA/HMAC separadas para firmar tokens, y sin logica de refresh/revocacion.

**Fuerza de seguridad:** Las sesiones almacenadas en el servidor (o en la base de datos) pueden invalidarse inmediatamente al cerrar sesion o al deshabilitar un usuario. Los JWT tienen tiempo de vida propio y no pueden revocarse sin infraestructura adicional (lista negra, revocacion store), lo que es un riesgo operacional para una institucion donde un usuario puede salir sin previo aviso.

---

## Decision

Se usa el **sistema de sesiones HTTP nativo del framework (Django sessions)** como mecanismo de autenticacion y mantenimiento del estado de la sesion del usuario.

**Funcionamiento:**

1. El usuario envia sus credenciales (nombre de usuario + contrasena) via formulario POST a la vista de login.
2. El servidor verifica las credenciales contra la tabla de usuarios (hasheadas con PBKDF2 por defecto en Django).
3. Si son correctas, el servidor crea una sesion en la base de datos y envia al navegador una cookie `sessionid` firmada con la clave secreta del servidor.
4. En cada peticion subsiguiente, el navegador envia la cookie; el servidor la verifica, recupera la sesion de la BD y conoce el usuario autenticado.
5. Al cerrar sesion (o al deshabilitar el usuario), la sesion se elimina de la BD y la cookie queda invalida inmediatamente.

**Datos almacenados en sesion:** solo el `usuario_id`. El resto de datos del usuario (nombre, rol, departamento_id) se leen de la base de datos en cada peticion mediante el sistema de autenticacion del framework.

---

## Alternativas consideradas

- **JWT (JSON Web Tokens):** Util cuando el frontend (SPA) y el backend (API REST) son procesos separados en origenes distintos, porque el token viaja en el header de cada peticion sin necesidad de cookie de sesion. En el monolito MVC del ADR-0001, el servidor genera el HTML directamente; no hay peticiones cross-origin. Los JWT introducirian complejidad de firma, distribucion de claves y logica de refresh sin ninguna ventaja operacional. Ademas, los JWT no son revocables sin infraestructura adicional.

- **Autenticacion por token estatico (API key):** Apropiado para integraciones maquina a maquina. No aplica para usuarios humanos que inician sesion con usuario y contrasena.

- **OAuth 2.0 / SAML con proveedor externo (SSO institucional):** El discovery no menciona ningun proveedor de identidad institucional existente (Active Directory, LDAP, etc.). Integrar un proveedor externo que no esta confirmado como disponible introduciria una dependencia que contradice el Riesgo 3 del mvp-canvas. Se puede considerar en v2 si la institucion dispone de directorio corporativo.

---

## Consecuencias

**Ganamos:**
- El sistema de sesiones de Django funciona sin configuracion adicional; las sesiones se almacenan en la misma base de datos del sistema.
- El cierre de sesion invalida la sesion inmediatamente en el servidor, sin ventanas de validez residual.
- Si un usuario pre-cargado debe deshabilitarse (por cambio de personal), basta con deshabilitar su cuenta en la BD; la proxima peticion con su cookie fallara al verificar que el usuario esta activo.
- No hay dependencias externas para la autenticacion: el sistema es completamente autocontenido.

**Aceptamos:**
- Las sesiones se almacenan en la base de datos; en un servidor con muchas sesiones concurrentes, la tabla de sesiones puede crecer. Para el piloto de tres usuarios esto es irrelevante. Si la escala crece, las sesiones pueden moverse a un cache en memoria (Redis) sin cambiar la logica de negocio.
- El sistema no tiene recuperacion de contrasena automatizada en v1 (flujo de "olvide mi contrasena" con email). Si un usuario olvida su contrasena, un administrador debe restablecerla manualmente via el panel admin de Django o via linea de comandos. Esto es aceptable dado que no hay modulo de administracion de usuarios en v1 y los usuarios son un numero fijo y conocido (mvp-canvas: Secretaria, Director, Colaborador).
