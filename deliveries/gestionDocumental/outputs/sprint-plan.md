# Sprint 1 — Al cierre del sprint, cualquier funcionario traza un tramite desde su ingreso hasta la carga de respuesta en una sola linea de tiempo auditada, sin coordinacion fuera del sistema

**Capacidad:** 20 pts
**Comprometido:** 19 pts
**Fecha inicio:** 2026-06-28
**Duracion:** 2 semanas

---

## Historias comprometidas

| Historia | Titulo | Epica | Pts | Prioridad | DoR |
|----------|--------|-------|-----|-----------|-----|
| US-01 | Registrar documento externo con codigo unico | E-01 | 5 | 1 | OK |
| US-02 | Derivar documentos al Director desde la bandeja Por Despachar | E-01 | 3 | 2 | OK |
| US-03 | Delegar un tramite a un colaborador con sumilla digital | E-02 | 3 | 3 | OK |
| US-04 | Ver unicamente los tramites asignados en la bandeja personal | E-02 | 2 | 4 | OK |
| US-05 | Subir la respuesta y vincularla automaticamente al tramite origen | E-03 | 3 | 5 | OK |
| US-06 | Consultar la linea de tiempo completa de un tramite | E-03 | 3 | 6 | OK |

---

## Sprint Goal — justificacion

Las seis historias comprometidas completan el ciclo documental end-to-end en un solo sprint:

- **US-01 y US-02** (E-01) dan entrada al ciclo: la Secretaria registra el oficio con codigo irrepetible y lo deriva al Director sin correo ni aviso informal. Sin estas dos piezas no existe ningun tramite sobre el que operar.
- **US-03 y US-04** (E-02) trasladan la responsabilidad con trazabilidad: el Director delega con sumilla escrita y auditada, y el Colaborador trabaja solo su propia cola. Cierran la coordinacion informal de delegacion.
- **US-05 y US-06** (E-03) completan el ciclo y activan la metrica de exito: el Colaborador carga la respuesta vinculada (segunda transicion automatica de estado) y cualquier actor autorizado consulta la linea de tiempo completa sin preguntar a nadie.

Juntas, estas historias convierten el outcome del MVP — "los funcionarios cierran tramites dentro del sistema sin coordinacion fuera" — en algo demostrable al final del sprint con un tramite real de punta a punta.

---

## Historias en el backlog (no entran en Sprint 1)

| Historia | Titulo | Pts | Motivo de exclusion |
|----------|--------|-----|---------------------|
| US-07 | Acceder solo a los documentos del propio departamento | 5 | Capacidad agotada: incluirla elevaria el total a 24 pts (>20 pts). Cumple DoR y puede entrar en Sprint 2; el piloto arranca con un departamento unico (OQ-01 resuelto), lo que permite diferirla sin bloquear el despliegue inicial. |

---

## Impedimentos detectados

**IMP-01 — OQ-04 sin resolucion: mecanismo de medicion de la metrica de exito**

La metrica de exito del MVP (>=70% de tramites con >=2 transiciones automaticas en 60 dias) carece de mecanismo de medicion operacional en v1, porque el dashboard de indicadores (R-07) esta fuera de alcance segun mvp-canvas. OQ-04 en el backlog no tiene resolucion documentada.

Riesgo: al finalizar el sprint se podra demostrar que el ciclo funciona, pero no se podra confirmar automaticamente que se alcanzo la metrica sobre el volumen total de tramites. Alguien del equipo (PO o Director como actor piloto) debe definir antes del final del sprint como se contabilizaran las transiciones manualmente o por consulta directa a la base de datos durante los 60 dias del piloto.

**Accion requerida antes del fin del Sprint 1:** el PO debe resolver OQ-04 y documentar la resolucion en backlog.json, o bien proponer una historia tecnica de medicion para Sprint 2.
