# Adaptación de Skills IA para el Ecosistema BCV/BACC

## Presentación Ejecutiva — Resumen

> **Proyecto:** Adaptación de skills de agente (IA) para los microservicios backend de Apertura de Cuentas Comerciales
> **Ecosistema:** BCV/BACC — Interbank
> **Fecha:** Septiembre 2026
> **Estado:** Sprint 2 en curso · Inicio 13 ago 2026 · Fin 09 oct 2026

---

## 1. Alcance del Proyecto

> **7 microservicios backend · pipeline de 3 skills de agente IA · ~85 % menos tokens por HU · 8 semanas (13 ago → 09 oct 2026)**

El ecosistema **BCV/BACC** de Interbank opera sobre 7 microservicios backend Java/Spring Boot. El proyecto construye un **pipeline de 3 skills de agente** que convierte una Historia de Usuario (HU) de negocio en una Historia Técnica (DHU) y, opcionalmente, en código — usando **graphify** para reducir el consumo de tokens en ~85 %.

---

## 2. Metodología: SDD + BMAD

El proyecto sigue **Spec-Driven Development (SDD)** y **BMAD**:

- **SDD** — framework de desarrollo guiado por especificaciones: antes de escribir código se produce y aprueba una spec (la **DHU**) con criterios de aceptación y mapa técnico. `bcv-hu-implementer` no genera código si la DHU tiene gaps sin resolver.
- **BMAD** — `Understand → Design → Build → Validate`, el ciclo interno que **cada skill** sigue al construir algo. El pipeline en sí corre en **sprints iterativos de 2 semanas**, con **graphify** como motor de research local (código real, ~0 tokens de LLM) en vez de que el LLM lea repositorios completos.

| Fase SDD                  | Acción del flujo BCV                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| Intención de entrada      | HU funcional de negocio                                                                     |
| Research-Driven Context   | `bcv-hu-context-analyzer` investiga el código real con graphify → `.context/hu-<codigo>.md` |
| Especificar formalmente   | `bcv-dhu-writer` escribe la DHU (criterios de aceptación, endpoints, mapa técnico)          |
| Implementar desde la spec | `bcv-hu-implementer` aplica la DHU en ramas feature (dry-run / apply)                       |
| Verificar / feedback      | Linter + tests, revisión humana y gotchas que vuelven al contexto                           |

---

## 3. Skills que se Identificaron (flujo principal)

Se identificó y construyó un **pipeline de 3 skills principales**, uno por cada paso del flujo SDD:

| Paso del flujo  | Skill                      | Qué hace                                                   |
| --------------- | --------------------------- | ----------------------------------------------------------- |
| Research        | `bcv-hu-context-analyzer`  | Investiga el código real con graphify → contexto técnico    |
| Especificación   | `bcv-dhu-writer`            | Escribe la DHU (criterios de aceptación, endpoints, mapa técnico) |
| Implementación   | `bcv-hu-implementer`        | Aplica la DHU en ramas feature (dry-run → apply)            |

**Flujo:** HU funcional → `bcv-hu-context-analyzer` (graphify) → contexto técnico → `bcv-dhu-writer` → DHU → `bcv-hu-implementer` → código.

**Primera entrega: un plan de tareas validado**

> `bcv-hu-context-analyzer` (research con graphify) y `bcv-dhu-writer` (especificación) ejecutan la primera parte del pipeline y entregan un **plan de tareas (la DHU) validado**. Desde ahí, `bcv-hu-implementer` aplica el plan y genera el código, iterando con el feedback del equipo dev.

---

## 4. Resultados (hitos medidos)

Los dos primeros ejercicios con el equipo dev del banco cuantificaron el ahorro frente a la forma tradicional:

| Usuario          | HU / EVT                                                             | Complejidad | Tradicional (h) | Copilot (h) | Skill + Graphify (h) | Ahorro vs Tradicional |
| ---------------- | ---------------------------------------------------------------------- | ----------- | :-------------: | :---------: | :-------------------: | :--------------------: |
| Fernando Camargo  | Identificación de canal OneAPP en trama Postmortem                     | Baja        |        8         |      4      |          2            |         **75 %**        |
| Lionel Gonzales   | [EVT] Implementación de arquitectura cluster-to-cluster – Paquete 1     | Media       |        48        |     40      |         10            |         **79 %**        |

> 💡 En el caso de mayor complejidad (Lionel Gonzales), pasar de **48 h a 10 h** equivale a un ahorro de **~79 %** (~4 días de implementación).

---

## 5. Roadmap

4 sprints de 2 semanas, del **13 ago** al **09 oct 2026**:

| Sprint       | Período         | Estado         | Resumen                                                                 |
| ------------ | --------------- | -------------- | ------------------------------------------------------------------------ |
| **Sprint 1** | 13 – 28 ago     | ✅ Completado   | Consolidación de repos + adopción de graphify + creación del pipeline (3 skills) + primer ejercicio con equipo dev. |
| **Sprint 2** | 31 ago – 11 sep | 🔄 En curso     | Validación EVT con equipo dev + consolidación de feedback + skills adicionales (`bcv-hexagonal-architecture`, `bcv-openapi-design`, `bcv-spring-data-jpa-sql-server`). |
| **Sprint 3** | 14 – 25 sep     | ⬜ Propuesto    | Iteración y ajustes de skills definidos con HUs/EVTs + métricas obtenidas de ejecución. |
| **Sprint 4** | 28 sep – 09 oct | ⬜ Propuesto    | Iteración y ajustes (continúa) + elaboración de informe final y recomendaciones. |

---

## 6. Próximos Pasos

1. **Skills adicionales: `bcv-hexagonal-architecture` y `bcv-openapi-design`.** Complementan el pipeline: `hexagonal-architecture` define el patrón de todo el código, y `openapi-design` define lo que se expone y valida la HU contra su contrato. `bcv-spring-data-jpa-sql-server` (persistencia SQL Server, usada por casi toda HU) se suma como tercer complemento.
2. **Iterar los skills en las próximas Historias de Usuario** reales del banco, ampliando su cobertura con casos de uso adicionales.
3. **Oscar queda a cargo** del proyecto tras el traspaso del Arquitecto IA (11 sep): soporte continuo y afinación de los skills.
4. **Refinar** los skills con más código real y el feedback del equipo dev del banco.

---

## Referencias

- `docs/presentacion-ejecutiva-skills-ia-bcv.md` — versión completa y detallada de esta presentación.
- `docs/hu-dhu-workflow-guide.md` — flujo HU → DHU → código, metodología y estimación de tokens.
