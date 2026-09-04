# Guía de Uso: Flujo HU → DHU con Graphify y Skills BCV

## 1. Objetivo de esta guía

Explica al equipo de desarrollo cómo usar los skills `bcv-hu-context-analyzer`, `bcv-dhu-writer` y `bcv-hu-implementer` para convertir Historias de Usuario (HU) del ecosistema BCV en Historias Técnicas (DHU) detalladas y, opcionalmente, aplicar cambios de implementación, usando **graphify** como motor de análisis de código y **minimizando tokens**.

### Alcance

Este flujo analiza **microservicios backend únicamente**. No investiga frontend, pantallas, componentes UI ni lógica de presentación.

Cuando una HU menciona un canal o pantalla (por ejemplo, "canal BCW"), el flujo lo mapea al servicio backend que expone la API, evento o contrato de datos que alimenta ese canal.

---

## 2. ¿Por qué este flujo?

El ecosistema BACC tiene 7 microservicios. Analizar HUs con un LLM leyendo repositorios completos es caro y lento. Este flujo usa **graphify** para indexar código localmente y consultarlo de forma quirúrgica, complementado con **contexto mínimo** escrito por el equipo.

### Pipeline completo

```text
HU funcional
    ↓
bcv-hu-context-analyzer   → .context/hu-<codigo>.md
    ↓
bcv-dhu-writer            → hu-technical-refinement/HU-...-refined-...md
    ↓
bcv-hu-implementer        → cambios en repos afectados
```

### Beneficios

- Menor consumo de tokens que la radiografía con LLM.
- Análisis reproducible basado en código real.
- Contexto reutilizable entre HUs.
- DHUs estandarizados con `ibk-hu-technical-refinement`.

---

## 3. Conceptos clave

### 3.1 Skills

| Skill                     | Responsabilidad                                         | Entrada                   | Salida                                         |
| ------------------------- | ------------------------------------------------------- | ------------------------- | ---------------------------------------------- |
| `bcv-hu-context-analyzer` | Investiga la HU contra los repos usando graphify CLI    | HU de negocio + workspace | `.context/hu-<codigo>.md`                      |
| `bcv-dhu-writer`          | Escribe la especificación técnica final                 | `.context/hu-<codigo>.md` | `hu-technical-refinement/HU-...-refined-...md` |
| `bcv-hu-implementer`      | Aplica cambios de implementación en los repos afectados | DHU + contexto            | Ramas feature con cambios aplicados            |

> **Catálogo completo:** además de estos 3 skills del pipeline, el repo tiene skills de implementación (hexagonal, JPA, ASB, etc.). Ver **[3.5 Skills disponibles en el repo BCV](#35-skills-disponibles-en-el-repo-bcv)** para saber cuándo usar cada uno.

### 3.2 ¿Qué es un gotcha?

Un **gotcha** es una trampa o constraint **no obvia** de un microservicio. Es algo que:

- No se deduce del nombre de una clase o método.
- No se ve fácilmente en el código.
- Afecta decisiones de implementación.
- Sorprendería a un desarrollador que no conoce el historial del servicio.

#### Ejemplos de gotchas

- "`registryOffice` se computa desde `address.province`, no es un catálogo."
- "`verifyIntegrity` solo funciona para expedientes en estado COMPLETED."
- "El servicio convierte automáticamente HTTP 400 a 200 cuando el CU ya existe."

#### ¿Qué NO es un gotcha?

- Documentación general del servicio.
- Lista de endpoints.
- Explicaciones de arquitectura obvias.
- Cualquier cosa que graphify pueda descubrir fácilmente.

### 3.3 Graphify

Herramienta CLI que:

1. Extrae el AST del código fuente.
2. Construye un grafo de nodos (clases, métodos, campos) y relaciones.
3. Permite consultar con `query`, `path`, `explain`.

**Ventaja:** las consultas son locales. No consumen tokens de LLM.

### 3.4 Contexto mínimo

Tres niveles de contexto, todos muy pequeños:

| Nivel               | Archivo                                           | Propósito                                                          | Tamaño       |
| ------------------- | ------------------------------------------------- | ------------------------------------------------------------------ | ------------ |
| Workspace           | `docs/.agent-context/service-map.md`              | Índice de todos los servicios: rol, stack, arquitectura, god nodes | < 100 líneas |
| Workspace           | `docs/.agent-context/architecture-conventions.md` | Patrones de arquitectura por servicio                              | < 80 líneas  |
| Workspace           | `docs/.agent-context/cross-service-patterns.md`   | Convenciones cross-service: naming, comunicación, colas/topics     | < 60 líneas  |
| Servicio (opcional) | `<servicio>/docs/.agent-context/gotchas.md`       | Trampas/constraints no obvias del servicio                         | < 10 líneas  |

**No hay más.** Ni READMEs completos, ni radiografías, ni caches de servicio obligatorios.

### 3.5 Skills disponibles en el repo BCV

Todos los skills viven en `/Users/joseluis/Ntt/milton skills/skills-assi-comercial/skills`. Se dividen en dos grupos: los **del pipeline** (HU → DHU → implementación) y los **de implementación** (capacidades especializadas que se usan dentro de la Fase 3).

#### Skills del pipeline (en orden de uso)

| Skill                     | Cuándo usarlo                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `bcv-hu-context-analyzer` | **Primero.** Analizar la HU funcional contra los repos (con graphify) y generar `.context/hu-<codigo>.md`.           |
| `bcv-dhu-writer`          | **Segundo.** Escribir la DHU técnica a partir del contexto generado.                                                 |
| `bcv-hu-implementer`      | **Tercero.** Aplicar la DHU en código: generar el reporte de implementación (dry-run) y crear ramas feature (apply). |

#### Cuándo usar `bcv-hu-implementer`

- ✅ Después de que `bcv-dhu-writer` produjo una DHU **aprobada** (sin gaps bloqueantes).
- ✅ Para generar el **reporte de implementación** (modo `dry-run`, por defecto).
- ✅ Para **crear ramas feature y aplicar cambios** (modo `--apply`).
- ❌ **NO** para analizar la HU → usa `bcv-hu-context-analyzer`.
- ❌ **NO** para escribir la DHU → usa `bcv-dhu-writer`.
- ❌ **NO** si la DHU tiene gaps bloqueantes o está en `EN ELABORACIÓN`.
- ❌ **NO** para frontend/UI/pantallas.
- ❌ **NO** hace commit ni push automático: siempre revisión humana.

#### Skills de implementación (se usan dentro de la Fase 3)

Estos son los skills que el reporte de implementación referencia en la columna `Skill (cómo codificar)` según la tarea:

| Skill                            | Cuándo usarlo                                                                                                                              |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `bcv-hexagonal-architecture`     | Generar un slice hexagonal completo: controllers, use cases, puertos, mappers, output adapters.                                            |
| `bcv-clean-architecture`         | Auditar/revisar arquitectura, corregir violaciones de dependencias y migrar servicios legacy (p. ej. `service-point-service`) a hexagonal. |
| `bcv-java-spring-boot`           | Setup Spring Boot: ADS BOM, multi-módulo, profiles, Spring Cloud Config + Key Vault, startup.                                              |
| `bcv-openapi-design`             | Contratos REST/OpenAPI: DTOs, requests/responses, errores estandarizados.                                                                  |
| `bcv-openfeign`                  | Clientes HTTP OpenFeign: `@FeignClient`, `FeignConfig`, propagación de headers, `ErrorDecoder`.                                            |
| `bcv-azure-service-bus`          | Mensajería ASB: publishers, subscribers, topics/queues, labels, DLQ, retry.                                                                |
| `bcv-spring-data-jpa-sql-server` | Persistencia JPA + SQL Server: entidades, repositorios, migraciones, Always Encrypted, auditing.                                           |
| `bcv-cosmos-db`                  | Persistencia Cosmos DB: containers, documentos, partition key, RU/s, TTL.                                                                  |
| `bcv-commons-observability`      | Observabilidad: trazas, métricas, alertas de Teams, data masking de PII.                                                                   |
| `bcv-testing`                    | Tests unitarios/integración: JUnit 5, Mockito, AssertJ, `@DataJpaTest`, JaCoCo.                                                            |

> **Relación con el pipeline:** los skills de implementación **no reemplazan** a `bcv-hu-implementer`. `bcv-hu-implementer` orquesta los cambios y referencia estos skills para indicar **cómo** debe escribirse el código de cada tarea.

### 3.6 Metodología: SDD + BMAD

Este flujo **es** Spec-Driven Development (SDD): la especificación (DHU) es el artefacto central y va **antes** del código. No es casualidad — todos los skills declaran en su frontmatter `frameworks: ["Spec-Driven Development", "BMAD"]`, por lo que el pipeline HU → DHU → código es una instancia concreta de SDD aplicada al ecosistema BCV.

#### Mapeo del flujo a SDD

| Fase SDD                  | Acción del flujo BCV                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| Especificar (entrada)     | HU funcional de negocio                                                                     |
| Research-Driven Context   | `bcv-hu-context-analyzer` investiga el código real con graphify → `.context/hu-<codigo>.md` |
| Especificar formalmente   | `bcv-dhu-writer` escribe la DHU con criterios de aceptación, endpoints y mapa técnico       |
| Implementar desde la spec | `bcv-hu-implementer` aplica la DHU en ramas feature (dry-run / apply)                       |
| Verificar / feedback      | Linter + tests, revisión humana y gotchas que vuelven al contexto                           |

#### Principios SDD aplicados

1. **Spec as Lingua Franca** — la DHU es el contrato único que leen negocio (HU) y desarrollo (implementación).
2. **Spec first** — no se escribe código sin una DHU aprobada (`bcv-hu-implementer` rechaza DHUs con gaps bloqueantes).
3. **Research-Driven Context** — el analizador no inventa: consulta el grafo y lee código puntual, no radiografía con LLM.
4. **Executable Specifications** — la DHU incluye criterios de aceptación técnicos testables y DoD que se traducen en tests.
5. **Continuous Refinement** — los gaps se resuelven iterativamente (bloque "dudas pendientes" → regenerar DHU).
6. **Bidirectional Feedback** — lo descubierto en implementación vuelve al contexto (`gotchas.md`, `service-map.md`).
7. **Branching for Exploration** — `bcv-hu-implementer` explora en ramas `feature/HU-...` sin tocar `main`.

#### Relación con BMAD

SDD va acompañado de **BMAD** (_Understand → Design → Build → Validate_), la metodología de fase de build. Por eso cada SKILL.md define un workflow doble:

```text
SDD  → Especificar → Validar → Diseñar → Generar → Verificar
BMAD → Understand → Design → Build → Validate
```

SDD gobierna **qué** se construye (la spec); BMAD gobierna **cómo** se construye cada cambio.

### 3.7 ¿Por qué no Spec-Kit ni OpenSpec?

Existen frameworks SDD genéricos y populares — [Spec-Kit](https://github.com/github/spec-kit) (toolkit de GitHub con `/speckit.*`) y [OpenSpec](https://github.com/openspec) (spec-first con CLI y deltas) — que resuelven el mismo problema "spec antes que código". Se evaluaron y **se descartaron a favor del flujo custom BCV** por estas razones:

1. **Alineación corporativa.** El DHU debe seguir el template `ibk-hu-technical-refinement` (estándar Interbank/BCV). Spec-Kit y OpenSpec imponen su propio formato de spec (`spec.md` / `specs/` con deltas), lo que rompería la trazabilidad y el compliance corporativo.

2. **Integración con graphify.** El diferenciador del flujo es la investigación local de código vía grafo (casi cero tokens). Spec-Kit y OpenSpec no se integran con graphify: dependen de que el LLM lea archivos directamente, con el costo de tokens que este flujo evita.

3. **Eficiencia de tokens.** El flujo BCV mantiene contexto mínimo (`service-map.md`, `gotchas.md`) y consultas quirúrgicas. Los frameworks genéricos generan artifacts más verbosos (constitution, plan, tasks, deltas) que suman tokens en cada HU.

4. **Multi-servicio / contexto distribuido.** BACC son 7 repos independientes con un workspace compartido. Spec-Kit y OpenSpec son repo-local (la spec vive junto al código de un solo repo); no modelan un workspace con `service-map.md` y `gotchas.md` cross-service.

5. **Control del pipeline.** El flujo BCV tiene gates propios (gaps → dudas pendientes, DoR/DoD, auto-validación del DHU, ramas `feature/HU-...` sin commit automático). Adoptar Spec-Kit/OpenSpec implicaría ceder ese control a un lifecycle ajeno.

#### Comparativa

| Aspecto                 | Flujo BCV (custom)                          | Spec-Kit             | OpenSpec                                   |
| ----------------------- | ------------------------------------------- | -------------------- | ------------------------------------------ |
| Formato de spec         | DHU (`ibk-hu-technical-refinement`)         | `spec.md` propio     | `specs/` + deltas (ADDED/MODIFIED/REMOVED) |
| Investigación de código | graphify (grafo local, ~0 tokens)           | LLM lee archivos     | LLM lee archivos                           |
| Contexto multi-servicio | workspace + `service-map.md` + `gotchas.md` | repo-local           | repo-local                                 |
| Alineación corporativa  | Alta (BCV/BACC)                             | Baja (genérico)      | Baja (genérico)                            |
| Curva de adopción       | Alta (interno, requiere onboard)            | Media                | Media                                      |
| Mantenimiento           | Interno (equipo BCV)                        | Comunidad / upstream | Comunidad / upstream                       |
| Cobertura de dominio    | BACC (13 skills especializados)             | Ninguna              | Ninguna                                    |

#### Ventajas del flujo BCV

- Control total del formato, gates y convenciones BACC.
- Ahorro de tokens real (~85% vs LLM puro) gracias a graphify + contexto mínimo.
- Conocimiento de dominio codificado (13 skills: hexagonal, ASB, OpenFeign, JPA, Cosmos, etc.).
- Trazabilidad corporativa (DHU estándar, `catalog-info.yaml`, CODEOWNERS, Backstage).

#### Desventajas del flujo BCV

- Costo de mantenimiento: los skills son internos y el equipo debe mantenerlos.
- Sin comunidad ni mejoras upstream automáticas.
- Onboarding: requiere conocer el flujo y los skills (mitigado por esta guía).

#### Ventajas de Spec-Kit / OpenSpec

- Battle-tested, con comunidad y desarrollo activo.
- Out-of-the-box: constitution, specify, plan, tasks, implement.
- Agnóstico de lenguaje y stack.

#### Desventajas de Spec-Kit / OpenSpec (para BCV)

- No alineados al template `ibk-hu-technical-refinement` ni al dominio BACC.
- Sin graphify → mayor consumo de tokens.
- Repo-local → no modelan el workspace multi-servicio de BACC.
- Impondrían un lifecycle y formato que el equipo no controla.

> **Conclusión:** Spec-Kit y OpenSpec son excelentes SDD genéricos, pero para BACC el costo de desalineación (formato corporativo + graphify + multi-servicio) supera el beneficio de usar un framework listo. Se optó por un SDD **customizado** que mantiene los principios (spec first, research-driven, executable specs) con los artefactos y gates que el ecosistema ya exige.

---

## 4. Filosofía de contexto

> **Menor cantidad de archivos posible. Máximo 100 líneas de contexto escrito en total.**

### ¿Por qué tan poco?

- El contexto escrito es para **clasificar** servicios y detectar **trampas**.
- El código real lo resuelve graphify con consultas baratas.
- Cada línea extra es un token extra en cada HU.

### Reglas

- ❌ No crear `service-baseline.md` extenso por servicio.
- ❌ No crear `.context/services/<servicio>.md` obligatorio.
- ❌ No usar LLM para generar contexto.
- ✅ Un solo `service-map.md` a nivel workspace.
- ✅ Un `architecture-conventions.md` a nivel workspace si hay más de una arquitectura.
- ✅ Un opcional `gotchas.md` por servicio, solo si hay trampas reales.
- ✅ Todo lo demás se resuelve con graphify + lecturas puntuales de código.

---

## 5. Estructura del proyecto

### 5.1 Repositorio del workspace (independiente)

El workspace tiene **su propio repositorio Git**, separado de los microservicios. No es un monorepo. No usa submódulos. Es un repo independiente que solo contiene contexto, análisis de HUs y DHUs.

Ejemplo de repositorio:

```text
bcv-bacc-workspace/                    ← repo Git independiente
│
├── docs/
│   ├── .agent-context/
│   │   ├── service-map.md             ← contexto global
│   │   ├── architecture-conventions.md ← patrones de arquitectura
│   │   └── cross-service-patterns.md  ← convenciones cross-service
│   └── hu-dhu-workflow-guide.md       ← esta guía
│
├── .context/
│   └── hu-<codigo>.md                 ← contexto de HU específica
│
├── hu-technical-refinement/           ← DHUs finales
│   └── HU-<id>-refined-<timestamp>.md
│
├── .gitignore                         ← ignora microservicios y cache
├── .agents.md                         ← instrucciones para agentes
└── .github/copilot-instructions.md    ← instrucciones para Copilot
```

### 5.2 Repositorios de microservicios (independientes)

Cada microservicio es un **repositorio Git propio**. La lista actual de servicios vive en `docs/.agent-context/service-map.md` (fuente de verdad); aquí solo se ilustra el patrón de nombres:

```text
bcv-bacc-<servicio-1>/    ← repo Git independiente
bcv-bacc-<servicio-2>/    ← repo Git independiente
bcv-bacc-<servicio-3>/    ← repo Git independiente
...
```

Cada uno tiene su propio `docs/.agent-context/gotchas.md` versionado en su repo. El workspace comparte `service-map.md`, `architecture-conventions.md` y `cross-service-patterns.md`.

### 5.3 Relación entre workspace y microservicios

**No hay relación de Git entre ellos.** El workspace no incluye los microservicios como submódulos, subtrees ni carpetas versionadas.

Para trabajar localmente, un desarrollador puede clonar todo junto en una misma carpeta padre:

```text
/home/dev/bcv/
├── bcv-bacc-workspace/              ← repo del workspace
├── bcv-bacc-<servicio-1>/
├── bcv-bacc-<servicio-2>/
└── ... (según service-map.md)
```

O puede clonar los microservicios dentro del workspace, pero **ignorados por Git**:

```text
bcv-bacc-workspace/
├── docs/
├── .context/
├── bcv-bacc-<servicio-1>/   ← ignorado por .gitignore
└── bcv-bacc-<servicio-2>/   ← ignorado por .gitignore
```

### 5.4 `.gitignore` del workspace

```gitignore
# Microservicios: son repos independientes, no se versionan aquí.
# Usa un glob para no tener que listar cada servicio nuevo.
bcv-bacc-*/

# graphify cache regenerable
graphify-out/cache/
.graphify_analysis.json
.graphify_labels.json

# IDE / OS
.DS_Store
.idea/
.vscode/
```

> **Regla de oro:** el repo del workspace nunca debe contener código de microservicios. Solo contexto, análisis de HUs y DHUs.

### 5.5 ¿Por qué separarlos?

| Ventaja          | Explicación                                                                                                      |
| ---------------- | ---------------------------------------------------------------------------------------------------------------- |
| Independencia    | Los equipos de cada microservicio no se ven afectados por cambios en el workspace.                               |
| Permisos         | El workspace puede ser público o accesible por arquitectos/analistas; los microservicios mantienen sus permisos. |
| Tamaño           | El workspace es ligero porque no incluye código.                                                                 |
| Historial limpio | El Git del workspace solo tiene commits de contexto y DHUs.                                                      |
| Flexibilidad     | Cada microservicio puede evolucionar a su ritmo sin depender del workspace.                                      |

### 5.6 Crear un nuevo workspace desde el skeleton

Este repositorio incluye un `skeleton/` y un script `init-workspace.sh` para generar workspaces BCV estandarizados.

#### Uso del script

```bash
./init-workspace.sh <nombre-workspace> <ruta-servicios> <prefijo-hu>
```

Ejemplo:

```bash
./init-workspace.sh bcv-bacc-workspace-mi-equipo ../ HU
```

El script:

1. Crea el directorio del nuevo workspace.
2. Copia el contenido de `skeleton/`.
3. Reemplaza placeholders (`{{WORKSPACE_NAME}}`, `{{SERVICES_PATH}}`, `{{HU_PREFIX}}`).
4. Inicializa un repositorio Git y hace un commit inicial.

#### Estructura generada

```text
bcv-bacc-workspace-mi-equipo/
├── docs/
│   ├── .agent-context/
│   │   ├── service-map.md
│   │   ├── architecture-conventions.md
│   │   └── cross-service-patterns.md
│   └── hu-dhu-workflow-guide.md
├── .context/
├── hu-technical-refinement/
├── contexto/
├── .agents.md
├── .github/copilot-instructions.md
├── .gitignore
└── README.md
```

#### Personalización posterior

Después de ejecutar el script:

- Revisa `docs/.agent-context/service-map.md` y actualiza los god nodes reales.
- Revisa `docs/.agent-context/architecture-conventions.md` y ajusta los patrones.
- Revisa `docs/.agent-context/cross-service-patterns.md` y ajusta las convenciones de naming, comunicación y colas/topics.
- Crea `<repo>/docs/.agent-context/gotchas.md` solo si hay constraints no obvias.
- Genera los grafos graphify para cada microservicio.

> **Nota:** el skeleton es la forma recomendada de crear workspaces. Mantiene la estructura, convenciones y archivos de agentes sincronizados con esta guía.

---

## 6. Preparación inicial (one-time setup)

### 6.1 Generar grafos graphify

Desde la raíz del workspace:

```bash
for repo in bcv-bacc-*; do
  echo "Building graph for $repo..."
  graphify "$repo" --code-only
done
```

`--code-only` evita extracción costosa de documentos/imágenes.

**Verificación:**

```bash
for repo in bcv-bacc-*; do
  test -f "$repo/graphify-out/graph.json" && echo "✅ $repo" || echo "❌ $repo"
done
```

### 6.2 Crear `docs/.agent-context/service-map.md`

Un solo archivo con rol, stack, arquitectura y god nodes:

```markdown
# Service Map

| Service | Role | Stack | Architecture | God Nodes |
|---|---|---|---|---|
| party-lifecycle-management-service | Orquesta expediente y cuenta | Java 21, hexagonal, SQL Server, ASB | Hexagonal | CreateBusinessAccountService, SplValidationMapper, ExpedientEntity |
| channel-activity-service | Bridge con SPL/IFX | Java 21, stateless, ASB | Hexagonal | PowersValidationSubscriberHandler, PowersValidationInputMapper |
| service-point-service | Gestión de puntos de servicio | Java 17, layered, SQL Server, ASB | Layered | ServicePointDto |
| account-opening-reporting-service | Reporting a Teradata | Java 21, SQL Server + Cosmos, ASB | Hexagonal | ReportTeradataLEDomain, ReportMapper |
```

**Máximo 100 líneas.** Se versiona en Git.

### 6.3 Crear `docs/.agent-context/architecture-conventions.md`

Opcional, pero recomendado si hay más de una arquitectura en el ecosistema. Describe los patrones por arquitectura:

```markdown
# Architecture Conventions

## Hexagonal Architecture

Applies to: party-lifecycle-management-service, channel-activity-service, ...

Package structure:
- input: controllers, subscribers, records, mappers
- core: domains, ports, use cases
- output: repositories, clients, publishers

## Layered Architecture

Applies to: service-point-service

Package structure:
- controller → service → repository
- facade layer for orchestration
```

**Máximo 80 líneas.** Se versiona en Git.

### 6.4 Crear `docs/.agent-context/cross-service-patterns.md`

Convenciones que aplican a varios servicios (naming, comunicación, colas/topics). Ejemplo:

```markdown
# Cross-Service Patterns — BCV BACC Ecosystem

## Naming conventions
| Layer | Convention | Example |
|---|---|---|
| REST controllers | `*CommandController`, `*QueryController` | `BusinessAccountCommandController` |
| Mappers | MapStruct `*Mapper` con singleton `MAPPER` | `SplValidationMapper.MAPPER` |

## Communication patterns
| Pattern | Technology | Usage |
|---|---|---|
| Síncrono service-to-service | OpenFeign | PLM → customer, compliance |
| Eventos asíncronos | Azure Service Bus | PLM ↔ channel-activity |

## Common queues/topics
| Queue/Topic | Purpose |
|---|---|
| `que-bcv-channel-activity-powers-read-req-01` | SPL validation request |
```

**Máximo 60 líneas.** Se versiona en Git.

### 6.5 Crear `gotchas.md` opcional por servicio

Solo si hay constraints no obvias. Ejemplo para `party-lifecycle-management-service`:

```markdown
# party-lifecycle-management-service gotchas

- registryOffice se computa desde legalEntityDomain.address.province.
- SPL validation fluye siempre a través de channel-activity-service.
- @StandardApi exige respuesta RFC 9457 en command controllers.
```

**Máximo 10 líneas.** No versionar si está vacío o solo tiene información obvia.

---

## 7. Flujo paso a paso

### Paso 1: Tener una HU de negocio

Ejemplo:

```text
Como operador de SPL en el canal BCW
quiero seleccionar la oficina registral desde un listado predefinido
para asegurar el registro correcto y evitar errores manuales en las solicitudes.
```

### Paso 2: Ejecutar `bcv-hu-context-analyzer`

```text
/hu-analyze
Como operador de SPL en el canal BCW
quiero seleccionar la oficina registral desde un listado predefinido
para asegurar el registro correcto y evitar errores manuales en las solicitudes.

Workspace: /Users/joseluis/Downloads/bcv-bacc-account-opening-reporting-service
```

El skill hará:

1. Validar la HU.
2. Leer `docs/.agent-context/service-map.md`.
3. Leer `docs/.agent-context/architecture-conventions.md` si existe.
4. Leer `docs/.agent-context/cross-service-patterns.md` si existe.
5. Leer `<servicio>/docs/.agent-context/gotchas.md` si existe.
6. Ejecutar consultas graphify puntuales (máx. 2 por servicio primario, 1 por secundario).
7. Leer hasta 3 fragmentos de código de 40 líneas si es necesario.
8. Detectar gaps.
9. Generar `.context/hu-<codigo>.md`.

### Paso 3: Revisar el contexto generado

Abrir `.context/hu-<codigo>.md` y validar:

- Servicios clasificados correctamente.
- Punto de inyección identificado.
- Gaps registrados.
- Cambios esperados coherentes.

### Paso 4: Ejecutar `bcv-dhu-writer`

```text
/hu-write .context/hu-agregar-oficina-registral-canal-bcw.md
```

Genera el DHU en `hu-technical-refinement/`.

Antes de entregar el DHU, el skill ejecuta una **auto-validación**:

- Al menos 3 criterios de aceptación técnicos.
- Endpoints con códigos de error documentados.
- Mapa técnico de implementación no vacío.
- DoR completo o justificado.
- Gaps bloqueantes resueltos o DHU marcado como `EN ELABORACIÓN`.

Por defecto usa el **template extendido**, que incluye:

- Criterios de aceptación técnicos.
- Endpoints documentados.
- **Plan de implementación de tareas**.
- **Mapa técnico de implementación**.
- **Checklist de validación**.
- **Technical Impact Matrix**.
- **DoR y DoD**.
- **Checklist final pre-merge**.
- **Dudas pendientes con respuestas sugeridas**.

Si se quiere solo el refinamiento técnico básico sin plan de implementación, usar:

```text
/hu-write .context/hu-agregar-oficina-registral-canal-bcw.md --base
```

Al final del DHU, el skill añade un bloque de **dudas pendientes con respuestas sugeridas**.

### Paso 5: Resolver dudas pendientes

Si el DHU tiene gaps, el skill presenta al final:

```text
─────────────────────────────────────────────
DUDAS PENDIENTES — RESPUESTAS SUGERIDAS
─────────────────────────────────────────────

1. [GAP-01] Origen del catálogo de oficinas registrales
   ├─ Opción A (sugerida): Tabla local en PLM.
   ├─ Opción B: Servicio maestro de catálogos.
   └─ Opción C: Config Server.

2. [GAP-02] Ubicación del campo
   ├─ Opción A (sugerida): BusinessAccountRecord.
   └─ Opción B: LegalEntityRecord.

Responde con el número y opción, ej: "1-A, 2-A"
```

El usuario responde en el mismo chat. El skill actualiza `.context/hu-<codigo>.md` y regenera el DHU.

### Paso 6: Revisar el DHU

Verificar que tenga:

- Título con prefijo `DHU`.
- Alcance, descripción, necesidad, narrativa.
- Criterios de aceptación técnicos con HTTP codes, JSON, field names.
- Endpoints documentados.
- **Plan de implementación de tareas.**
- **Mapa técnico de implementación.**
- **Checklist de validación.**
- **Technical Impact Matrix.**
- **DoR y DoD.**
- **Checklist final pre-merge.**
- Apéndice técnico con servicios y gaps.
- Dudas pendientes con respuestas sugeridas.
- Gaps bloqueantes marcados como `PENDIENTE` o `BLOQUEADO`.

Si se usó el template base, solo revisar las secciones básicas.

### Paso 7: Ejecutar `bcv-hu-implementer` (opcional)

```text
/hu-implement hu-technical-refinement/HU-AGREGAR-OFICINA-REGISTRAL-BCW-refined-202608261830.md --dry-run
```

Por defecto ejecuta en modo **dry-run**, mostrando los cambios propuestos sin aplicarlos.

Para aplicar cambios en ramas feature:

```text
/hu-implement hu-technical-refinement/HU-AGREGAR-OFICINA-REGISTRAL-BCW-refined-202608261830.md --apply
```

El skill:

1. Lee el DHU y el contexto.
2. Descubre los skills disponibles (workspace + usuario) para GitHub Copilot Chat.
3. Genera un **único** reporte de implementación `hu-technical-refinement/HU-<code>-implementation-report-<timestamp>.md` con la columna `Skill` por tarea.
4. Crea una rama `feature/HU-...` en cada repo afectado (solo en `apply`).
5. Aplica los cambios listados en el Mapa técnico de implementación.
6. Ejecuta linter + tests.
7. Muestra un resumen con diff para revisión humana.
8. **No hace commit ni push automáticamente.**

> **Importante:** solo usar `--apply` cuando el DHU no tenga gaps bloqueantes.

---

## 8. Anatomía de una HU en detalle

Esta sección describe qué hace exactamente cada artefacto y skill durante el ciclo de vida de una HU.

### 8.1 HU funcional

Es la entrada. Debe contener:

- Título claro.
- Actor, acción y objetivo (formato estándar).
- Criterios de aceptación (Gherkin preferido).
- Reglas de negocio.
- Alcance funcional.

**No incluye** decisiones técnicas, endpoints ni nombres de campos. Eso se descubre en el análisis.

### 8.2 `bcv-hu-context-analyzer`

Este skill convierte la HU funcional en contexto técnico. Sus tareas:

| Tarea             | Descripción                                                                                   |
| ----------------- | --------------------------------------------------------------------------------------------- |
| Validar           | Comprueba que la HU tenga actor, acción, objetivo y criterios mínimos.                        |
| Clasificar        | Usa `service-map.md` para decidir qué servicios son primary, secondary, omitted o to_confirm. |
| Recordar gotchas  | Lee `gotchas.md` de los servicios afectados.                                                  |
| Investigar        | Ejecuta graphify query/path/explain para encontrar conceptos y puntos de inyección.           |
| Confirmar         | Lee hasta 3 fragmentos de código de 40 líneas si graphify no es suficiente.                   |
| Detectar gaps     | Registra decisiones pendientes o conceptos no encontrados.                                    |
| Escribir contexto | Genera `.context/hu-<codigo>.md` con toda la información técnica descubierta.                 |

**No genera:** código, tests, migraciones ni el DHU final.

### 8.3 `.context/hu-<codigo>.md`

Archivo intermedio que contiene:

- Resumen ejecutivo de la HU.
- Servicios clasificados.
- Punto de inyección principal.
- Cambios esperados por servicio.
- Gaps técnicos identificados.
- Fragmentos de código relevantes.
- Notas de investigación.

Sirve como única fuente de verdad técnica antes de escribir el DHU.

### 8.4 `bcv-dhu-writer`

Este skill toma el contexto técnico y lo convierte en especificación formal. Sus tareas:

| Tarea                   | Descripción                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------- |
| Leer contexto           | Carga `.context/hu-<codigo>.md`.                                                   |
| Decidir estructura      | Determina si la HU requiere un solo DHU o varios.                                  |
| Seleccionar plantilla   | Usa el template extendido por defecto; el base solo si se solicita explícitamente. |
| Rellenar plantilla      | Completa las secciones correspondientes, incluyendo plan de implementación.        |
| Generar CAs técnicos    | Convierte los criterios de negocio en criterios técnicos testables.                |
| Documentar endpoints    | Agrega rutas, métodos HTTP, requests y responses.                                  |
| Marcar gaps             | Secciones bloqueadas por gaps se marcan como `PENDIENTE` o `BLOQUEADO`.            |
| Auto-validar            | Verifica criterios mínimos de calidad antes de entregar el DHU.                    |
| Generar bloque de dudas | Al final del DHU, presenta gaps con respuestas sugeridas.                          |
| Persistir               | Guarda el resultado en `hu-technical-refinement/HU-...-refined-...md`.             |

**Plantillas:**

- **Extendida (default):** incluye plan de implementación, mapa técnico, matriz de impacto, DoR/DoD, checklists.
- **Base (opcional):** refinamiento técnico estándar sin secciones de implementación.

**No hace preguntas interactivas:** las dudas se presentan al final del DHU para que el usuario responda en el mismo chat.

**No implementa:** el DHU es entrada para desarrollo, no código ejecutable.

### 8.5 `hu-technical-refinement/HU-...-refined-...md`

Es el artefacto final. Debe permitir que un desarrollador implemente la HU sin volver a analizar el código. Incluye:

- Título, descripción, necesidad y narrativa.
- Criterios de aceptación técnicos con ejemplos concretos.
- Endpoints, eventos o colas involucradas.
- Cambios esperados por servicio.
- Apéndice técnico con dependencias y gaps.

Por defecto incluye además:

- Plan de implementación de tareas.
- Mapa técnico de implementación.
- Checklist de validación.
- Technical Impact Matrix.
- DoR y DoD.
- Checklist final pre-merge.

Al final, incluye el bloque **Dudas pendientes — Respuestas sugeridas**.

Con el template base, estas secciones extendidas no se incluyen.

### 8.6 `bcv-hu-implementer`

Tercer skill opcional. Toma el DHU aprobado, genera un draft de implementación y aplica cambios en los repositorios afectados.

**Output (un solo archivo):**

El skill genera un único reporte, no un draft separado:

```text
hu-technical-refinement/HU-<code>-implementation-report-<timestamp>.md
```

Este reporte contiene:

- Validación pre-implementación.
- Resumen de métricas.
- Secciones por repositorio con archivos a modificar/crear, cada fila con columna `Skill`.
- Tareas manuales pendientes.
- Next steps.

La columna `Skill (cómo codificar)` indica el skill recomendado **disponible para GitHub Copilot Chat** (workspace + usuario) para esa tarea y la convención que ese skill dicta. El skill referenciado es la autoridad de **cómo debe escribirse el código** (ej: `bcv-openapi-design` → records con `@Schema`, errores RFC 9457). Si no hay skill disponible, dice `not available in Copilot Chat`.

El skill descubre los skills disponibles usando un glob (`find . -type f -name 'SKILL.md'`) tanto en el workspace como en el perfil del usuario (ej: `~/skills`, `~/.config/github-copilot/skills`, y `~/.github/copilot-instructions.md`).

**Pre-validación:**

Antes de aplicar cambios, verifica:

- No hay gaps bloqueantes sin resolver.
- El DHU no está en estado `EN ELABORACIÓN` con gaps abiertos.
- Al menos 3 criterios de aceptación técnicos.
- El Mapa técnico de implementación no está vacío.
- El DoR está cumplido o justificado.

Si falla la pre-validación, se detiene sin modificar archivos.

**Modos:**

- `--dry-run`: muestra cambios propuestos sin aplicarlos (default).
- `--apply`: crea ramas feature y aplica cambios.

**Reglas de seguridad:**

- Nunca modifica `main` directamente.
- Nunca hace commit ni push automático.
- Solo toca archivos del Mapa técnico de implementación.
- Ejecuta linter + tests antes de reportar éxito.
- Genera un único reporte `hu-technical-refinement/HU-<code>-implementation-report-<timestamp>.md` con la columna `Skill`.

### 8.7 Ciclo de feedback

1. El usuario responde las dudas al final del DHU.
2. Se actualiza `.context/hu-<codigo>.md` con las respuestas.
3. Se regenera el DHU con las secciones desbloqueadas.
4. Opcionalmente, se ejecuta `bcv-hu-implementer` para generar el reporte y aplicar cambios.
5. El desarrollador revisa el reporte, invoca los skills recomendados **que tenga instalados** si es necesario, revisa el diff, ajusta y hace commit.
6. Si durante la implementación se descubre un nuevo gotcha, se actualiza `<repo>/docs/.agent-context/gotchas.md`.
7. Si cambia el rol de un servicio, se actualiza `docs/.agent-context/service-map.md`.

---

## 9. Uso de graphify

### Comandos más usados

```bash
# Buscar un concepto
graphify query "registryOffice" --max 5 \
  --graph bcv-bacc-party-lifecycle-management-service/graphify-out/graph.json

# Trazar ruta entre nodos
graphify path "BusinessAccountRecord" "PowersValidationOutDto" --max 5 \
  --graph bcv-bacc-party-lifecycle-management-service/graphify-out/graph.json

# Explicar un nodo
graphify explain "ExpedientDomain" --max 5 \
  --graph bcv-bacc-party-lifecycle-management-service/graphify-out/graph.json
```

### Reglas de oro

- Empezar siempre con `--max 5`.
- Si no hay resultados, declarar el concepto como **nuevo**.
- Máximo 2 consultas por servicio primario.
- Máximo 1 consulta por servicio secundario.
- Confirmar con `grep` + lectura punteada, no archivos completos.

---

## 10. Mantenimiento

### 10.0 Agregar un nuevo microservicio/proyecto al workspace

Cuando un microservicio nuevo entra al workspace, sigue este checklist **en orden**:

1. **`docs/.agent-context/service-map.md`** — agregar la fila del servicio (Role, Stack, Architecture, God Nodes). **Obligatorio.**
2. **`.gitignore`** — agregar el directorio del servicio si se clona dentro del workspace. **Obligatorio.**
3. **Grafos graphify** — `graphify <servicio> --code-only`. **Obligatorio.**
4. **`docs/.agent-context/architecture-conventions.md`** — solo si introduce una arquitectura distinta (ej. layered).
5. **`docs/.agent-context/cross-service-patterns.md`** — solo si introduce nuevas convenciones de naming, comunicación o colas/topics.
6. **`<servicio>/docs/.agent-context/gotchas.md`** — solo si hay constraints no obvias (máx 10 líneas).

> **¿Y esta guía?** No es necesario tocarla. Es documentación genérica; el flujo lee `service-map.md` y los archivos de contexto, no los ejemplos ilustrativos de las secciones 5.2/5.3/5.4. Si quieres que esos ejemplos sigan reflejando el número actual de servicios, puedes actualizarlos, pero **no afecta al funcionamiento**.

### 10.1 Actualizar `service-map.md`

Solo cuando:

- Se agrega un microservicio.
- Un servicio cambia de responsabilidad principal.
- Cambian god nodes importantes.

**No por cada HU.**

### 10.2 Actualizar `gotchas.md`

Solo cuando:

- Se descubre una trampa nueva durante una HU.
- Cambia una constraint arquitectónica.
- Una regla deja de ser válida.

**No por cada cambio de código.**

### 10.3 Actualizar grafos graphify

Comandos:

| Acción | Comando |
| ------ | ------- |
| Verificar si está desactualizado | `graphify check-update <repo>` |
| Actualizar / regenerar el grafo | `graphify <repo> --code-only` |

Actualiza **solo si hubo cambios estructurales**:

| Cambio                                         | Servicio a actualizar             |
| ---------------------------------------------- | --------------------------------- |
| Nuevo controller/subscriber/publisher/use case | El servicio donde se agregó       |
| Nueva entidad/tabla                            | El dueño de la entidad            |
| Nuevo cliente Feign/integración                | El consumidor                     |
| Rename/eliminación de god node                 | El afectado                       |
| Refactor grande de paquetes                    | El afectado                       |
| Merge a `main` estructural                     | Todos los afectados (preferir CI) |

**No actualizar** para bugfixes, nuevos campos en DTOs/entidades existentes, tests nuevos o cambios solo de docs.

### 10.4 Verificar grafo desactualizado

```bash
graphify check-update bcv-bacc-party-lifecycle-management-service
```

### 10.5 Automatización en CI

```yaml
- name: Update graphify graphs
  run: |
    for repo in bcv-bacc-*; do
      graphify "$repo" --code-only
    done
  if: github.ref == 'refs/heads/main'
```

---

## 11. Versionado en Git

### Sí versionar

```bash
git add docs/.agent-context/service-map.md
git add docs/.agent-context/architecture-conventions.md
git add docs/.agent-context/cross-service-patterns.md
git add bcv-bacc-*/docs/.agent-context/gotchas.md
git add .agents.md
git add .github/copilot-instructions.md
git add .context/hu-<codigo>.md
git add hu-technical-refinement/HU-...-refined-...md
git commit -m "docs: update agent context and HU analysis"
```

### No versionar

```gitignore
# graphify cache regenerable
graphify-out/cache/
.graphify_analysis.json
.graphify_labels.json
```

> `graphify-out/graph.json` es opcional. Es grande pero reusable. El equipo decide si versionarlo o generarlo en CI.

---

## 12. Estimación de tokens

### Costo del contexto escrito

| Artefacto                                | Líneas | Tokens aprox. |
| ---------------------------------------- | ------ | ------------- |
| `service-map.md`                         | < 100  | ~200          |
| `architecture-conventions.md`            | < 80   | ~100          |
| `cross-service-patterns.md`              | < 60   | ~80           |
| `gotchas.md` por servicio                | < 10   | ~80           |
| Total por HU (workspace + 2-3 servicios) | ~260   | ~540          |

### Comparación por HU: tres escenarios

| Concepto                  | Con skills + graphify        | Con skills, sin graphify     | Sin skills ni graphify       |
| ------------------------- | ---------------------------- | ---------------------------- | ---------------------------- |
| Lectura de READMEs        | ~200 tokens                  | ~15.000 tokens               | ~20.000 tokens               |
| Lectura de código         | ~1.200 tokens (3 fragmentos) | ~8.000 tokens (6-8 archivos) | ~21.000 tokens               |
| Consultas de exploración  | 0 tokens (graphify local)    | 0 (lecturas directas)        | ~15.000 tokens (iteraciones) |
| Generación contexto + DHU | ~7.300 tokens                | ~9.300 tokens                | ~10.500 tokens               |
| **Total estimado**        | **~8.000 – 12.000**          | **~25.000 – 45.000**         | **~50.000 – 90.000**         |
| **Ahorro vs LLM puro**    | **~85% menos**               | **~50% menos**               | baseline                     |

### Conclusión

Para la HU "Agregar oficina registral en canal BCW":

| Escenario                    | Tokens estimados |
| ---------------------------- | ---------------- |
| **Con skills + graphify**    | **~10.000**      |
| **Con skills, sin graphify** | **~33.000**      |
| **Sin skills ni graphify**   | **~70.000**      |

Graphify no solo ahorra tokens, sino que reduce la incertidumbre: el skill sabe exactamente dónde buscar en el grafo en lugar de leer archivos completos.

---

## 13. Troubleshooting

### "No matching nodes found"

El concepto no existe en el código. Declararlo como nuevo. No buscar sinónimos.

### El skill no encuentra el punto de inyección

- Revisar si la HU afecta un servicio no considerado.
- Verificar que el grafo no esté desactualizado.
- Revisar `gotchas.md` del servicio.

### El DHU tiene muchos gaps

Es normal si la HU introduce conceptos nuevos o depende de decisiones de negocio. Resolver con el equipo funcional antes de implementar.

---

## 14. Ejemplo: Oficina Registral

### HU funcional

```text
Como operador de SPL en el canal BCW
quiero seleccionar la oficina registral desde un listado predefinido
para asegurar el registro correcto y evitar errores manuales en las solicitudes.
```

### Hallazgo clave

`registryOffice` ya existe en el contrato SPL, pero se computa desde `legalEntityDomain.address.province`.

### Servicios clasificados

- **Primary:** `party-lifecycle-management-service`
- **Secondary:** `bcv-bacc-channel-activity-service`
- **To confirm:** `service-point-service`, `account-opening-reporting-service`

### Gaps

1. Origen del catálogo de oficinas registrales.
2. Ubicación del campo (`LegalEntityRecord` vs `BusinessAccountRecord`).
3. Impacto en servicios secundarios.
4. Mecanismo de auditoría.

---

## 15. Checklist

Antes de analizar una HU:

- [ ] Grafos `graphify-out/graph.json` actualizados en servicios relevantes.
- [ ] `docs/.agent-context/service-map.md` existe y está actualizado.
- [ ] `docs/.agent-context/architecture-conventions.md` existe y está actualizado.
- [ ] `docs/.agent-context/cross-service-patterns.md` existe y está actualizado.
- [ ] `gotchas.md` creado solo para servicios con trampas conocidas.

Durante el análisis:

- [ ] HU completa (actor, acción, objetivo, CAs, reglas).
- [ ] Servicios clasificados antes de graphify.
- [ ] Límites de consultas respetados.
- [ ] Gaps registrados.

Después:

- [ ] Contexto generado revisado.
- [ ] DHU generado y validado.
- [ ] Artefactos versionados en Git.

---

## 16. Trabajo en equipo

Este flujo funciona mejor cuando varios desarrolladores colaboran en crear y mantener el contexto. No debe recaer en una sola persona.

### 16.1 Roles sugeridos

| Rol               | Responsabilidad                                                                                                          | Quién                                     |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| **Context owner** | Mantiene `docs/.agent-context/service-map.md`, `architecture-conventions.md` y `cross-service-patterns.md` actualizados. | Un arquitecto o lead por workspace.       |
| **Service owner** | Mantiene `<repo>/docs/.agent-context/gotchas.md` de su microservicio.                                                    | Un desarrollador del equipo del servicio. |
| **HU analyst**    | Ejecuta los skills, revisa contexto y DHU.                                                                               | Cualquier desarrollador asignado a la HU. |
| **Validator**     | Revisa que gaps se resuelvan antes de implementar.                                                                       | Tech lead o arquitecto.                   |
| **Implementer**   | Revisa el diff generado por `bcv-hu-implementer` y hace commit/push manual.                                              | Desarrollador asignado a la HU.           |

### 16.2 Distribución del trabajo inicial

#### Fase 1: Bootstrap (una sola vez)

1. **Arquitecto/lead** genera los grafos graphify para todos los servicios:
   ```bash
   for repo in bcv-bacc-*; do graphify "$repo" --code-only; done
   ```
2. **Arquitecto/lead** crea `docs/.agent-context/service-map.md` con rol, stack y god nodes.
3. **Arquitecto/lead** crea `docs/.agent-context/architecture-conventions.md` y `docs/.agent-context/cross-service-patterns.md`.
4. Cada **service owner** revisa su servicio y crea `gotchas.md` si aplica.
5. Se versiona todo en Git.

#### Fase 2: Operación continua

1. El **HU analyst** recibe una HU y ejecuta `bcv-hu-context-analyzer`.
2. Si descubre un gotcha nuevo, actualiza `<repo>/docs/.agent-context/gotchas.md`.
3. Si encuentra un servicio mal clasificado, avisa al **context owner**.
4. Genera el DHU con `bcv-dhu-writer`.
5. El **validator** revisa gaps y aprueba.

### 16.3 Convenciones de commit

| Cambio                                   | Mensaje de commit                              |
| ---------------------------------------- | ---------------------------------------------- |
| Actualizar `service-map.md`              | `docs: update service map`                     |
| Actualizar `architecture-conventions.md` | `docs: update architecture conventions`        |
| Actualizar `cross-service-patterns.md`   | `docs: update cross-service patterns`          |
| Agregar/actualizar `gotchas.md`          | `docs: add gotchas for <service>`              |
| Contexto de HU                           | `docs: add context for HU-<code>`              |
| DHU final                                | `docs: add technical refinement for HU-<code>` |
| Regenerar grafo                          | `chore: update graphify graph for <service>`   |

### 16.4 Resolución de conflictos

- Si dos desarrolladores editan `service-map.md`, gana la versión más reciente validada por el context owner.
- Si un service owner no puede mantener su `gotchas.md`, el HU analyst lo actualiza temporalmente y lo reporta.
- Los gaps técnicos se discuten en el canal del equipo antes de modificar el contexto.

### 16.5 Ritmo sugerido

| Actividad                             | Frecuencia                                                              |
| ------------------------------------- | ----------------------------------------------------------------------- |
| Revisar `service-map.md`              | Cada sprint o cuando cambia un servicio                                 |
| Revisar `architecture-conventions.md` | Cuando cambia la arquitectura de un servicio                            |
| Revisar `cross-service-patterns.md`   | Cuando cambian convenciones cross-service (naming, comunicación, colas) |
| Actualizar `gotchas.md`               | Cuando surge un gotcha en una HU                                        |
| Regenerar grafos                      | En cada merge a `main` o cuando falla `check-update`                    |
| Revisar DHUs generados                | En cada refinamiento técnico                                            |

### 16.6 Onboarding de nuevos desarrolladores

1. Leer esta guía.
2. Verificar que `graphify-out/graph.json` exista en su máquina.
3. Revisar `service-map.md`, `architecture-conventions.md`, `cross-service-patterns.md` y los `gotchas.md` de los servicios que tocará.
4. Acompañar en el análisis de una HU real antes de hacerlo solo.

---

## 17. Instrucciones para agentes

Para que cualquier agente (GitHub Copilot, Cursor, etc.) sepa cómo trabajar en este workspace, se deben colocar instrucciones en la raíz del workspace.

### 17.1 Archivos recomendados

| Archivo                           | Audiencia         |
| --------------------------------- | ----------------- |
| `.github/copilot-instructions.md` | GitHub Copilot    |
| `.agents.md`                      | Agentes genéricos |

### 17.2 ¿Workspace o microservicio?

**Nivel workspace (obligatorio):**

- Flujo HU → DHU → implementación.
- Reglas de graphify.
- Ubicación del contexto (`service-map.md`, `architecture-conventions.md`, `cross-service-patterns.md`, `gotchas.md`).
- Reglas cross-service.

El flujo HU → DHU es cross-service por naturaleza, por eso las instrucciones generales van en el workspace.

**Nivel microservicio (opcional):**

Solo si un servicio tiene reglas específicas que no aplican a los demás:

```text
bcv-bacc-party-lifecycle-management-service/.agents.md
```

Ejemplo de contenido local:

```markdown
# PLM-specific agent notes

- Always check SplValidationMapper when adding SPL-bound fields.
- Command controllers must use @StandardApi.
- RFC 9457 error envelope is mandatory.
```

**Regla:** si la instrucción aplica a más de un servicio, va al workspace. Si es exclusiva de un servicio, puede ir local.

### 17.3 Contenido mínimo

Los archivos de instrucciones deben incluir:

1. Uso obligatorio de `bcv-hu-context-analyzer`, `bcv-dhu-writer` y `bcv-hu-implementer`.
2. Uso obligatorio de graphify; prohibición de radiografía LLM.
3. Ubicación de `service-map.md`, `architecture-conventions.md`, `cross-service-patterns.md` y `gotchas.md`.
4. Límites de consultas graphify.
5. Convenciones de código y commits.

---

## 18. Referencias

- Skill `bcv-hu-context-analyzer`
- Skill `bcv-dhu-writer`
- Skill `bcv-hu-implementer`
- Plantilla `ibk-hu-technical-refinement`
- Graphify CLI
