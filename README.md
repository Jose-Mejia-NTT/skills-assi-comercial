# skills-assi-comercial

Repositorio de skills de agente para los microservicios **BCV / BACC** (Business Customer Value — Apertura de Cuentas Comerciales) del ecosistema Interbank.

Cada skill en esta carpeta guía a un agente de código para generar, mantener o diagnosticar componentes Java/Spring Boot siguiendo las convenciones, librerías internas y arquitectura de los servicios BACC (Java 21, Spring Boot 3.x, hexagonal, Azure Service Bus, SQL Server + Cosmos DB, OpenFeign, Key Vault).

## Convenciones del repositorio

- Cada skill vive en su propia carpeta bajo `skills/`.
- Cada skill debe contener:
  - `SKILL.md` — instrucciones, triggers, workflow, ejemplos, criterios de evaluación y troubleshooting.
  - `README.md` — documentación orientada al usuario final.
  - `evals/evals.json` — casos de prueba con prompts y expectativas.
  - `references/*.md` — documentos de referencia con patrones, snippets y guías específicas.
- Los skills deben responder en el idioma del usuario.
- El procesamiento interno, código generado y razonamiento estructural se hacen en inglés.
- Nunca se deben hardcodear secretos (connection strings, tokens, webhooks, SAS keys); siempre vienen de Azure Key Vault o variables de entorno.

## Estructura de un skill

```text
skills/<skill-name>/
├── SKILL.md                  # Instrucciones para el agente
├── README.md                 # Documentación para el usuario
├── evals/
│   ├── evals.json            # Casos de prueba
│   └── fixtures/             # Archivos de soporte para evaluaciones
└── references/
    ├── <topic-1>.md          # Patrones y snippets
    ├── <topic-2>.md
    └── ...
```

## Skills disponibles

### Pipeline HU → DHU → código

#### `bcv-hu-context-analyzer`

Investiga una historia de usuario (HU/HAB) de negocio sobre uno o más repositorios BACC usando
graphify CLI y produce un archivo de contexto técnico de bajo costo.

**Cubre:**

- Análisis multi-repo o single-repo.
- Contexto técnico `.context/hu-<code>.md`.
- Gestión de gaps (business / technical / implementation).
- Optimización de tokens (graphs preconstruidos, service-map, gotchas).

**Triggers:** "analyze this HU", "generate technical context", "hu-context", "/hu-analyze", "pre-dhu analysis"

**Archivos clave:** `SKILL.md`, `README.md`, `references/` (workflow, template, gap-handling, limits, optimizations, example)

---

#### `bcv-dhu-writer`

Consume el contexto técnico de `bcv-hu-context-analyzer` y escribe la HU técnica (DHU) final en
Markdown, siguiendo el template `ibk-hu-technical-refinement`.

**Cubre:**

- Template extendido (plan de implementación, mapa técnico, DoR/DoD, checklist pre-merge).
- Modos `single-dhu` y `split-dhu`.
- Auto-validación de calidad y bloqueo si hay gaps críticos.
- Bloque de dudas pendientes con respuestas sugeridas.

**Triggers:** "write DHU", "generate technical HU", "dhu-writer", "/hu-write"

**Archivos clave:** `SKILL.md`, `README.md`, `references/` (output-template, output-template-extended, workflow, language-policy)

---

#### `bcv-hu-implementer`

Consume una DHU refinada y aplica los cambios de implementación en feature branches de los
repositorios backend afectados. Nunca commitea ni pushea automáticamente.

**Cubre:**

- Modos `dry-run` (default) y `apply`.
- Descubrimiento de skills disponibles y mapeo de tareas → skill.
- Generación de código, tests y migraciones.
- Reporte único de implementación.

**Triggers:** "implement this HU", "generate code for HU", "/hu-implement", "apply DHU"

**Archivos clave:** `SKILL.md`, `README.md`, `references/` (workflow, output-template, skill-references, limits)

---

### Arquitectura

#### `bcv-hexagonal-architecture`

Genera un slice vertical completo de arquitectura hexagonal/clean para un nuevo caso de uso.

**Cubre:**

- `core/port/in`, `core/usecase`, `core/port/out`.
- Controllers, DTOs (records) y mappers en `input`.
- Persistence adapters en `output`.
- Registro de beans en `app/config/UseCaseConfig`.
- Reglas de dependencia entre capas.

**Triggers:** "add use case", "new endpoint", "nueva funcionalidad", "agregar caso de uso", "quiero agregar..."

**Archivos clave:** `SKILL.md`, `references/` (hexagonal-rules, project-alignment-checklist, controller-api-conventions, error-handling-patterns)

---

#### `bcv-clean-architecture`

Audita, revisa y refactoriza servicios BCV hacia arquitectura limpia/hexagonal, y migra servicios
legacy de estructura plana.

**Cubre:**

- Clasificación de arquitectura (hexagonal / layered / flat).
- Detección de violaciones de reglas de dependencia.
- Plan de migración flat → hexagonal (p. ej. `service-point-service`).
- Extracción de ports, movimiento de adapters y split de módulos.

**Triggers:** "auditar arquitectura", "migrar a hexagonal", "core depende de output", "dependency rule violation"

**Archivos clave:** `SKILL.md`, `README.md`, `evals/evals.json`, `references/` (dependency-rules, migration-checklist)

---

### Integraciones y contratos

#### `bcv-azure-service-bus`

Responde preguntas y genera código para Azure Service Bus usando `ads-spring-boot-starter-messaging`.

**Cubre:**

- Variantes BACC: topic, messages/queue y Managed Identity (`messagesCross`).
- Publishers y subscribers (`MessagePublisherRegistry` / `MessageSubscriberRegistry`).
- Configuración de topics, queues, labels, subscriptions y retry.
- Troubleshooting de mensajes perdidos, DLQ, label mismatch y `maxConcurrentCalls`.

**Triggers:** "Azure Service Bus", "ads-spring-boot-starter-messaging", "ADS messaging", "DLQ", "maxConcurrentCalls", "label mismatch"

**Archivos clave:** `SKILL.md`, `evals/evals.json`, `references/bacc-ads-messaging.md`

---

#### `bcv-openfeign`

Crea, revisa y diagnostica clientes HTTP OpenFeign.

**Cubre:**

- Interfaces `@FeignClient` con `base-url` y paths configurables (`interbank.ads.httpclient.feign.*`).
- Records de request/response.
- `RequestInterceptor` para propagar `Authorization`/`X-Trace-Id`/`traceparent`.
- `ErrorDecoder` tipado con saneamiento de PII.
- Wiring `@EnableFeignClients` en `-app`.

**Triggers:** "crear cliente Feign", "nuevo @FeignClient", "Feign error decoder", "propagar headers", "mockear Feign client"

**Archivos clave:** `SKILL.md`, `README.md`, `evals/evals.json`, `references/` (feign-client-pattern, feign-error-decoder)

---

#### `bcv-openapi-design`

Diseña, completa y estandariza contratos REST OpenAPI para servicios BACC.

**Cubre:**

- Contrato desde HU/DHU o desde controllers/DTOs existentes.
- Completar `rest-api.yaml`/`openapi.yaml` vacíos o parciales.
- Estandarización de recursos, métodos, status codes, DTOs y errores.
- Alineación con Spring Boot + springdoc.

**Triggers:** "complete rest-api.yaml", "design OpenAPI", "document controllers", "diseñar API REST", "standardize error responses"

**Archivos clave:** `SKILL.md`, `evals/evals.json`, `references/rest-contract-guidelines.md`

---

### Persistencia

#### `bcv-spring-data-jpa-sql-server`

Genera y mantiene componentes de persistencia con Spring Data JPA + SQL Server.

**Cubre:**

- Entidades JPA con convenciones de schema (`BcvBacc`, `dbo`).
- Repositorios Spring Data (derivados, JPQL).
- Datasource con HikariCP y dialectos SQL Server.
- JPA auditing (`@CreatedBy`, `@CreatedDate`, `@LastModifiedBy`, `@LastModifiedDate`, `@Version`).
- SQL Server Always Encrypted y colocación hexagonal.

**Triggers:** "JPA BCV", "SQL Server entity", "Spring Data repository", "HikariCP", "Always Encrypted", "schema BcvBacc"

**Archivos clave:** `SKILL.md`, `README.md`, `evals/evals.json`, `references/` (pom-templates, configuration, always-encrypted, auditing)

---

#### `bcv-cosmos-db`

Diseña, implementa y optimiza el uso de Azure Cosmos DB (tracking/expedientes, RU/s, particionamiento).

**Cubre:**

- Spring Data Cosmos reactive o Cosmos SDK directo.
- Partition key, RU/s, TTL, 429 y retries.
- Modelado documental y consultas por status/fecha.
- Mocks-first cuando Cosmos no está disponible.

**Triggers:** "cosmos db", "spring data cosmos", "partition key", "RU/s", "tracking", "status history"

**Archivos clave:** `SKILL.md`, `README.md`, `evals/evals.json`, `references/` (cosmos-sdk-and-stacks, partitioning-queries-and-rus, tracking-models-and-mocks)

---

### Plataforma y calidad

#### `bcv-java-spring-boot`

Guía de ingeniería Spring Boot BCV: parent/BOM, módulos, configuración y secretos.

**Cubre:**

- Parent/BOM (`ads-spring-boot-dependencies` / `bcv-commons-pomparent`).
- Multi-módulo Maven (`-core`, `-input`, `-output`, `-app`).
- `bootstrap.yml` vs `application.yml`.
- Config Server + Key Vault (`interbank.ads.security` / `interbank.ads.secrets`).
- Datasource `interbank.ads.persistence-sql` y troubleshooting de arranque.

**Triggers:** "Spring Boot BCV", "ADS BOM", "multi-module", "bootstrap.yml", "interbank.ads.security", "no arranca en local"

**Archivos clave:** `SKILL.md`, `evals/evals.json`, `references/` (bom-and-modules, profiles-and-config, config-server-keyvault, startup-troubleshooting, bacc-spring-boot)

---

#### `bcv-commons-observability`

Guía el uso de `bcv-commons-observability` para trazas, métricas, alertas de Teams y data masking.

**Cubre:**

- `@ObservableController`, `@ObservableService`, `@ObservableOperation`.
- Alertas de Teams vía webhook en Key Vault.
- Data masking para PII (DNI, RUC, account numbers).
- Propagación de trazas (síncrono y WebFlux).

**Triggers:** "observabilidad BCV", "@ObservableService", "Teams alert", "data masking", "trace id", "OpenTelemetry"

**Archivos clave:** `SKILL.md`, `evals/evals.json`, `references/` (annotations, teams-alerts, data-masking, webflux-traces)

---

#### `bcv-testing`

Genera y revisa tests unitarios e de integración con JUnit 5 + Mockito + AssertJ.

**Cubre:**

- Tests de use cases, adapters, publishers y subscribers.
- `@ExtendWith(MockitoExtension.class)`, `@Mock`, `@InjectMocks`.
- `ArgumentCaptor<Message>` para side effects.
- `@DataJpaTest` para repositorios y JaCoCo.

**Triggers:** "generar tests", "unit test BCV", "test adapter", "test publisher", "@DataJpaTest", "coverage JaCoCo"

**Archivos clave:** `SKILL.md`, `README.md`, `evals/evals.json`, `references/` (unit-test-patterns, integration-test-patterns)

---

## Cómo agregar un nuevo skill

1. Crear carpeta `skills/<skill-name>/`.
2. Escribir `SKILL.md` con:
   - Frontmatter: `name`, `description`, `metadata`.
   - Reglas de idioma.
   - Objetivo y scope.
   - Inputs y outputs esperados.
   - Workflow (SDD + BMAD si aplica).
   - Patrones obligatorios.
   - Preguntas de clarificación.
   - Criterios de evaluación.
   - Troubleshooting Notes.
   - Tabla de Reference Documents.
3. Escribir `README.md` orientado al usuario final.
4. Crear `evals/evals.json` con al menos 2-3 casos de prueba.
5. Crear `references/*.md` con snippets reutilizables.
6. Validar JSON: `python3 -m json.tool evals/evals.json`.
7. Revisar que no haya secretos hardcodeados.

## Principios transversales

- **SDD (Spec-Driven Development):** especificación primero, código después.
- **BMAD v6:** fases de análisis, solución e implementación.
- **Library-first:** preferir librerías internas (`bcv-commons-*`, `ads-spring-boot-*`) sobre código propio.
- **Security-first:** secretos en Azure Key Vault, nunca en el repo.
- **Test-first:** cada skill debe generar o exigir tests.
- **Observability:** usar `@ObservableController` / `@ObservableService` / `@ObservableOperation` en componentes expuestos.

## Contribuidores

- NTT Data

## Notas

- Los skills asumen proyectos Java 17+ (Java 21 para BACC, Java 17 para `service-point-service`) con Spring Boot 3.x.
- La arquitectura base es hexagonal/clean para los servicios BACC (excepto `service-point-service`, que es legacy de estructura plana).
- El pipeline de trabajo sugerido es: `bcv-hu-context-analyzer` → `bcv-dhu-writer` → `bcv-hu-implementer`, apoyado por los skills técnicos según la tarea.
- Para dudas sobre el uso de un skill, revisar su `README.md` interno.
