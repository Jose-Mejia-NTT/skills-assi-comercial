# Adaptación de Skills IA para el Ecosistema BCV/BACC

## Presentación Ejecutiva

> **Proyecto:** Adaptación de skills de agente (IA) para los microservicios backend de Apertura de Cuentas Comerciales
> **Ecosistema:** BCV/BACC — Interbank
> **Fecha:** Septiembre 2026
> **Estado:** Semana 3 en curso · Inicio 13 ago 2026 · Fin 09 oct 2026

---

## 01 · Contexto del Proyecto

---

### 1.1 Contexto de la Iniciativa

#### ¿Qué es este proyecto?

El ecosistema **BCV/BACC** (Business Customer Value — Apertura de Cuentas Comerciales) de Interbank opera sobre **7 microservicios backend** Java/Spring Boot, fuertemente integrados (Azure Service Bus, OpenFeign, SQL Server + Cosmos DB, Key Vault).

El proyecto consiste en **construir y adaptar un conjunto de skills de agente** (instrucciones especializadas para asistentes de código basados en IA, como GitHub Copilot) que permiten a un agente de IA **generar, mantener y diagnosticar** estos microservicios, convirtiendo una **Historia de Usuario (HU)** de negocio en una **Historia Técnica (DHU)** detallada y, opcionalmente, en cambios de código.

#### Problema a resolver

> Analizar e implementar Historias de Usuario con un LLM que lee repositorios completos es **caro y lento** (~50.000–90.000 tokens por HU), y **no sigue un estándar corporativo**. Esto dificulta escalar el uso de asistentes de IA en el desarrollo del ecosistema BACC.

#### Objetivo Principal del Proyecto

> 🎯 **Construir skills de agente que codifiquen el conocimiento del dominio BACC** y que implementen el pipeline **HU → DHU → código**, usando **graphify** como motor de análisis local de código para **reducir el consumo de tokens en ~85 %**, alineados al template corporativo `ibk-hu-technical-refinement`.

---

### 1.2 Estado Actual → Estado Objetivo

| Dimensión                   |              Estado Actual               |                       Estado Objetivo                       |
| --------------------------- | :--------------------------------------: | :---------------------------------------------------------: |
| **Análisis de HUs**         | LLM lee repos completos (~70k tokens/HU) |      graphify local + contexto mínimo (~10k tokens/HU)      |
| **Especificación**          |          Informal, sin estándar          |       DHU con template `ibk-hu-technical-refinement`        |
| **Implementación**          |       Manual, sin guía de dominio        | Asistida por skills especializados + ramas `feature/HU-...` |
| **Conocimiento de dominio** |          Implícito en el equipo          |            Codificado en 13 skills reutilizables            |

---

### 1.3 Valor para el Banco (cuantificado)

- **~85 % menos tokens** por HU analizada (de ~70.000 a ~10.000).
- **13 skills de dominio** que capturan convenciones BACC (hexagonal, ASB, OpenFeign, JPA, Cosmos, observabilidad, testing).
- **Pipeline reproducible** HU → DHU → código, con validación y gates propios (DoR/DoD).
- **Onboarding acelerado** del equipo de desarrollo en el uso de asistentes de IA.
- **Trazabilidad corporativa** vía DHU estándar y contexto versionado (`service-map.md`, `gotchas.md`).

---

### 1.4 Metodología: SDD + BMAD

El proyecto sigue **Spec-Driven Development (SDD)** y **BMAD**:

- **SDD** — la especificación (DHU) es el artefacto central y va **antes** del código.
- **BMAD** — `Understand → Design → Build → Validate` para la construcción de cada cambio.

| Fase SDD                  | Acción del flujo BCV                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| Especificar (entrada)     | HU funcional de negocio                                                                     |
| Research-Driven Context   | `bcv-hu-context-analyzer` investiga el código real con graphify → `.context/hu-<codigo>.md` |
| Especificar formalmente   | `bcv-dhu-writer` escribe la DHU (criterios de aceptación, endpoints, mapa técnico)          |
| Implementar desde la spec | `bcv-hu-implementer` aplica la DHU en ramas feature (dry-run / apply)                       |
| Verificar / feedback      | Linter + tests, revisión humana y gotchas que vuelven al contexto                           |

---

### 1.5 ¿Por qué no Spec-Kit ni OpenSpec?

Se evaluaron los frameworks SDD genéricos [Spec-Kit](https://github.com/github/spec-kit) y [OpenSpec](https://github.com/openspec) y se **descartaron a favor del flujo custom BCV** por:

1. **Alineación corporativa** — el DHU debe seguir el template `ibk-hu-technical-refinement` (estándar Interbank/BCV).
2. **Integración con graphify** — investigación local de código con ~0 tokens.
3. **Eficiencia de tokens** — contexto mínimo (`service-map.md`, `gotchas.md`) y consultas quirúrgicas.
4. **Multi-servicio** — BACC son 7 repos independientes con un workspace compartido.
5. **Control del pipeline** — gates propios (gaps, DoR/DoD, ramas `feature/HU-...` sin commit automático).

| Aspecto                 | Flujo BCV (custom)                          | Spec-Kit / OpenSpec                  |
| ----------------------- | ------------------------------------------- | ------------------------------------ |
| Formato de spec         | DHU (`ibk-hu-technical-refinement`)         | `spec.md` propio / `specs/` + deltas |
| Investigación de código | graphify (grafo local, ~0 tokens)           | LLM lee archivos                     |
| Contexto multi-servicio | workspace + `service-map.md` + `gotchas.md` | repo-local                           |
| Alineación corporativa  | Alta (BCV/BACC)                             | Baja (genérico)                      |

---

### 1.6 Ahorro de Tokens

| Escenario                 | Tokens estimados por HU |
| ------------------------- | ----------------------- |
| **Con skills + graphify** | **~8.000 – 12.000**     |
| Con skills, sin graphify  | ~25.000 – 45.000        |
| Sin skills ni graphify    | ~50.000 – 90.000        |

**Ahorro vs LLM puro: ~85 % menos.** Para la HU de ejemplo "Agregar oficina registral en canal BCW": ~10.000 tokens con skills + graphify, frente a ~33.000 sin graphify y ~70.000 sin skills ni graphify.

---

### 1.7 Stack Tecnológico

| Capa           | Tecnología                                              |
| -------------- | ------------------------------------------------------- |
| Lenguaje       | Java 21 (Java 17 en `service-point-service`)            |
| Framework      | Spring Boot 3.x (`ads-spring-boot-dependencies`)        |
| Arquitectura   | Hexagonal / Ports & Adapters (6 de 7 servicios)         |
| Mensajería     | Azure Service Bus (`ads-spring-boot-starter-messaging`) |
| Persistencia   | SQL Server (JPA/Hibernate), Azure Cosmos DB             |
| Clientes HTTP  | OpenFeign                                               |
| Seguridad      | JWT Bearer + Azure Key Vault + Spring Cloud Config      |
| Observabilidad | `bcv-commons-observability`, Spring Actuator            |

---

## 02 · Alcance del Proyecto

---

### 2.1 Resumen de Alcance

> **7 microservicios · 13 skills de agente · ~85 % menos tokens · 8 semanas (13 ago → 09 oct 2026)**

---

### 2.2 Incluido — 7 microservicios backend BACC

| Servicio                             | Arquitectura                     |
| ------------------------------------ | -------------------------------- |
| `party-lifecycle-management-service` | Hexagonal (orquestador central)  |
| `channel-activity-service`           | Hexagonal (procesador stateless) |
| `compliance-service`                 | Hexagonal                        |
| `current-account-service`            | Hexagonal                        |
| `customer-service`                   | Hexagonal                        |
| `account-opening-reporting-service`  | Hexagonal (SQL + Cosmos)         |
| `service-point-service`              | Layered (legacy)                 |

---

### 2.3 Incluido — 13 skills de agente

| Grupo                   | Skills                                                                                                                                                                                                                                        |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pipeline (3)**        | `bcv-hu-context-analyzer`, `bcv-dhu-writer`, `bcv-hu-implementer`                                                                                                                                                                             |
| **Implementación (10)** | `bcv-hexagonal-architecture`, `bcv-clean-architecture`, `bcv-java-spring-boot`, `bcv-openapi-design`, `bcv-openfeign`, `bcv-azure-service-bus`, `bcv-spring-data-jpa-sql-server`, `bcv-cosmos-db`, `bcv-commons-observability`, `bcv-testing` |

Incluye además el **flujo completo HU → DHU → implementación** con graphify y contexto mínimo versionado (`service-map.md`, `architecture-conventions.md`, `cross-service-patterns.md`, `gotchas.md` por servicio).

---

### 2.4 Fuera de Alcance / Candidatos a Etapa 2

| Ítem                                                 | Motivo                                              |
| ---------------------------------------------------- | --------------------------------------------------- |
| Frontend, pantallas y lógica de presentación         | El flujo analiza microservicios backend únicamente  |
| Migración de `service-point-service` a hexagonal     | Deuda técnica detectada; candidata a trabajo futuro |
| Completar `rest-api.yaml` vacíos en varios servicios | Deuda de documentación de contratos                 |
| Activar quality gates (SpotBugs/PMD en `skip=true`)  | Mejora de calidad independiente                     |
| Otros ecosistemas no BACC                            | Fuera del dominio del proyecto                      |

---

## 03 · Equipo de Trabajo

---

### 3.1 Equipo IA (NTT Data)

| Nombre                           | Rol           | Disponibilidad       | Responsabilidad principal                                                                                                                                                             |
| -------------------------------- | ------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Jose Luis Mejia Rojas**        | Arquitecto IA | 13 ago – 10 sep 2026 | Arquitectura del flujo HU → DHU, decisiones de contexto y adopción de graphify, diseño del workspace, validación de DHUs e interlocución con el cliente.                              |
| **Oscar Fabian Castro Severino** | Ingeniero IA  | 13 ago – 09 oct 2026 | Implementación y refinamiento de skills, grafos graphify, ejecución del pipeline (análisis, DHU, implementación), pruebas y reportes. A partir del 10 sep: **soporte y afinaciones**. |

---

### 3.2 Equipo de Desarrollo del Banco (colaboración)

El equipo dev del banco es la contraparte que valida en la práctica: dispone del entorno en su propia PC y ejecuta **iteraciones con feedback** sobre los skills.

| Nombre           | Rol           |
| ---------------- | ------------- |
| Sergio Manuel    | Líder técnico |
| Lionel Gonzales  | Desarrollador |
| Fernando Camargo | Desarrollador |
| Jose Peramas     | Desarrollador |

---

### 3.3 Responsable por Entregable

| Entregable                                           | Responsable                                         |
| ---------------------------------------------------- | --------------------------------------------------- |
| Arquitectura del flujo + workspace y convenciones    | Jose Luis (Arquitecto IA)                           |
| Contexto versionado (`service-map.md`, convenciones) | Jose Luis                                           |
| Skills del pipeline (3)                              | Jose Luis + Oscar                                   |
| Skills de implementación (10)                        | Oscar                                               |
| Grafos graphify (7 repos)                            | Oscar                                               |
| Análisis de HUs y generación de DHU                  | Oscar (ejecución) + Jose Luis (validación)          |
| Implementación y reporte                             | Oscar                                               |
| `gotchas.md` por servicio                            | Jose Luis y Oscvar (con soporte IA)                 |
| Validación y feedback                                | Equipo dev banco (Sergio Manuel como líder técnico) |

---

### 3.4 Riesgo de Dotación

- **10 sep 2026** — Jose Luis finaliza su participación (mitad del proyecto). Su trabajo queda **culminado y traspasado** en esta fecha; a partir de aquí **Oscar** queda como único recurso, en rol de **soporte y afinaciones** finales.

---

## 04 · Roadmap del Proyecto

---

### 4.1 Metodología y Estimación

**Metodología:** iterativa (SDD + BMAD), con ciclos semanales. Equipo de 2 personas durante ~8 semanas.

#### Resumen de Esfuerzo por Bloque (estimación inicial)

| Bloque de trabajo                          | Semanas | Esfuerzo |
| ------------------------------------------ | ------- | :------: |
| Consolidación de repos + entorno           | S1      |   10 %   |
| Skills del pipeline + graphify             | S2      |   20 %   |
| Skills de implementación + iteración       | S3–S6   |   40 %   |
| Validación con equipo banco + correcciones | S7      |   15 %   |
| Documentación, onboarding y cierre         | S8      |   10 %   |
| Soporte y afinaciones (Oscar)              | Cierre  |   5 %    |

#### Distribución del Esfuerzo por Tipo de Actividad

| Tipo de actividad                             | Esfuerzo aprox |
| --------------------------------------------- | :------------: |
| Creación y refinamiento de skills             |      45 %      |
| Investigación y análisis de código (graphify) |      20 %      |
| Validación y feedback                         |      15 %      |
| Documentación y onboarding                    |      15 %      |
| Configuración y setup                         |      5 %       |

> 💡 **El 65 % del esfuerzo es trabajo técnico** (skills + análisis de código). El 35 % restante es validación, documentación y setup.

---

### 4.2 Roadmap por Semanas

| Semana     | Fechas          | Estado        | Hito                                                                | Detalle / Entregable                                                                                                                                                                                                                                                                                                   |
| ---------- | --------------- | ------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1**      | 13 – 19 ago     | ✅ Completada | Consolidación de repositorios + entorno                             | Se consolidó toda la información de los repos a trabajar (los 7 microservicios BACC) y se configuró el entorno.                                                                                                                                                                                                        |
| **2**      | 20 – 26 ago     | ✅ Completada | Optimización de tokens → graphify y creación de skills del pipeline | Se adoptó **graphify** como mecanismo de ahorro de tokens, con el flujo de workspace documentado en `docs/hu-dhu-workflow-guide.md`. Se crearon los skills del pipeline: **`bcv-hu-context-analyzer`**, **`bcv-dhu-writer`** y **`bcv-hu-implementer`**. Se acordó con el cliente la entrega de una **HU de negocio**. |
| **3**      | 27 ago – 02 sep | 🔄 En curso   | Primer ejercicio HU → DHU → código                                  | Se completó el primer ejercicio end-to-end (ejecutado el **01 sep** con **Fernando Camargo**): se recibió la **HU de negocio**, se ejecutó `bcv-hu-context-analyzer` con graphify, se generó la **HU técnica (DHU)**, se implementó en código y se envió al ambiente de **desarrollo**. Validación/prueba en curso.                                                                    |
| **4**      | 03 – 09 sep     | ⬜ Propuesto  | Consolidación + iteración de skills                                | Consolidar el primer ejercicio validado en desarrollo, incorporar el feedback de **Fernando Camargo** y continuar la iteración de skills con código real. Preparar el traspaso de Jose Luis (10 sep).                                                                                                                                                           |
| **5**      | 10 – 16 sep     | ⬜ Propuesto  | Iteración + DHU                                                     | Seguir refinando skills según feedback. Generar la DHU técnica (`bcv-dhu-writer`) y resolver dudas pendientes con el cliente.                                                                                                                                                                                          |
| **6**      | 17 – 23 sep     | ⬜ Propuesto  | Iteración + implementación                                          | Continuar la mejora de skills. Ejecutar `bcv-hu-implementer` (dry-run → apply): ramas `feature/HU-...` y reporte de implementación.                                                                                                                                                                                    |
| **7**      | 24 – 30 sep     | ⬜ Propuesto  | Validación con equipo dev del banco                                 | El **equipo dev del banco prueba en su propio entorno (PC)** y devuelve feedback para corregir los skills. Ciclo de corrección: ajustar, regenerar y revalidar.                                                                                                                                                        |
| **8**      | 01 – 07 oct     | ⬜ Propuesto  | Estabilización y cierre                                             | Estabilización final de los skills con el feedback consolidado, documentación/onboarding, demo con el cliente y lecciones aprendidas.                                                                                                                                                                                  |
| **Cierre** | 08 – 09 oct     | ⬜ Propuesto  | Cierre de entregables                                               | Cierre final y traspaso a cargo de **Oscar** (Jose Luis finalizó el 10 sep).                                                                                                                                                                                                                                           |

---

### 4.3 Hitos Clave

| Hito                                       | Fecha ref.  | Descripción                                                          |
| ------------------------------------------ | :---------: | -------------------------------------------------------------------- |
| 🚀 **Kick-off**                            | 13 ago 2026 | Inicio oficial del proyecto                                          |
| 🧭 **Graphify + skills del pipeline**      | 26 ago 2026 | Decisión de ahorro de tokens + creación de los 3 skills del pipeline |
| 💻 **Primer ejercicio HU → DHU → código**  | 01 sep 2026 | Primer flujo completo en desarrollo (ejecutado con Fernando Camargo)    |
| 🏁 **Culminación Arquitecto IA**           | 10 sep 2026 | Traspaso de Jose Luis; Oscar queda como soporte/afinaciones          |
| 🧪 **Validación con equipo dev del banco** | 30 sep 2026 | Feedback incorporado a los skills                                    |
| 🎯 **Cierre del proyecto**                 | 09 oct 2026 | Skills estabilizados + documentación y onboarding                    |

---

### 4.4 Dependencias y Condiciones de Inicio

| Dependencia                                |   Responsable    |      Estado       | Impacto si no se resuelve                  |
| ------------------------------------------ | :--------------: | :---------------: | ------------------------------------------ |
| Entrega de la HU de negocio por el cliente |    Interbank     | ✅ Completada (S3) | Sin HU no se valida el pipeline end-to-end |
| Entorno de trabajo del equipo dev (PC)     | Equipo dev banco |   ✅ Disponible   | Bloquea la validación con feedback         |
| Grafos graphify de los 7 repos             |    Equipo IA     | ✅ Generados (S1) | Sin grafos, el análisis consume tokens     |
| Acceso a repositorios BACC                 |    Interbank     |        ✅         | Bloquea análisis e implementación          |
| Continuidad tras el 10 sep                 |    Equipo IA     |    Planificado    | Oscar asume soporte y afinaciones          |

---

### 4.5 Riesgos

| Severidad | Riesgo                                                              | Mitigación                                                                                |
| --------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Alto      | Salida de Jose Luis (Arquitecto IA) el 10 sep, a mitad del proyecto | Traspaso planificado; trabajo culminado antes del 10 sep; Oscar cubre soporte/afinaciones |
| Medio     | Dependencia de una HU real del cliente para validar el pipeline     | Acuerdo de entrega de HU ya conversado con el cliente                                     |
| Medio     | Feedback del equipo dev del banco fuera de tiempo                   | El equipo dev ya dispone del entorno en su PC; canal de iteración definido                |
| Medio     | `service-point-service` legacy (Java 17, estructura plana)          | Skill `bcv-clean-architecture` para migrar; no bloquea el resto                           |
| Bajo      | Grafos graphify desactualizados                                     | Regeneración en CI / `check-update`                                                       |

---

### 4.6 Problemas y Riesgos Técnicos Identificados

| Severidad | Problema                                                                             |
| --------- | ------------------------------------------------------------------------------------ |
| Medio     | Arquitectura heterogénea: `service-point-service` es el único sin patrón hexagonal   |
| Medio     | `rest-api.yaml` vacíos en varios servicios                                           |
| Bajo      | Quality gates (SpotBugs/PMD) en `skip=true` en varios repos                          |
| Bajo      | Feature flags implementados como propiedades Spring simples, sin plataforma dedicada |

---

## 05 · Criterios de Éxito (KPIs)

| Criterio                        | Indicador                                                |
| ------------------------------- | -------------------------------------------------------- |
| ✅ 13 skills operativos         | Pipeline + implementación validados sobre código real    |
| ✅ Ahorro de tokens demostrado  | ~85 % vs LLM puro (~8.000–12.000 tokens/HU)              |
| ✅ Pipeline validado end-to-end | HU de negocio → DHU → código en ambiente de desarrollo   |
| ✅ Estándar corporativo         | DHU generado con template `ibk-hu-technical-refinement`  |
| ✅ Feedback incorporado         | Correcciones del equipo dev aplicadas en las iteraciones |
| ✅ Onboarding entregado         | Documentación + equipo dev autónomo con los skills       |

---

## 06 · Próximos Pasos Inmediatos

Los pasos 1–3 ya se ejecutaron en la **semana 3** (01 sep, con Fernando Camargo), quedando el flujo HU → DHU → código enviado a desarrollo:

1. ✅ **Recibir la HU de negocio** del cliente.
2. ✅ **Ejecutar `bcv-hu-context-analyzer`** con graphify → `.context/hu-<codigo>.md`.
3. ✅ **Iterar los skills** con código real (validación con Fernando Camargo).
4. ⬜ **Consolidar el feedback** de desarrollo e incorporar las correcciones (semana 4).
5. ⬜ **Preparar el traspaso** de Jose Luis (10 sep): cerrar arquitectura, validaciones y acuerdos con el cliente.

---

## Referencias

- `docs/hu-dhu-workflow-guide.md` — flujo HU → DHU → código, metodología y estimación de tokens.
- `docs/analisis-minucioso-ecosistema-bcv.md` — análisis detallado de los 7 microservicios.
- `README.md` — catálogo completo de skills del repositorio.
