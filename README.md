# skills-assi-comercial

Repositorio de skills de agente para proyectos **BCV** del ecosistema Interbank.

Cada skill en esta carpeta guía a un agente de código para generar, mantener o diagnosticar componentes Java/Spring Boot siguiendo las convenciones, librerías internas y arquitectura de los proyectos BCV.

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

### `bcv-hexagonal-architecture`

Genera un slice completo de arquitectura hexagonal/clean para un nuevo caso de uso en servicios BCV H2H.

**Cubre:**

- Generación de `core/port/in`, `core/usecase`, `core/port/out`.
- Generación de controllers, DTOs, mappers y application services en `input`.
- Generación de persistence adapters en `output`.
- Registro de beans en `app/config/UseCaseConfig`.
- Reglas de dependencia entre capas.

**Triggers:** "add use case", "new endpoint", "nueva funcionalidad", "agregar caso de uso", "quiero agregar..."

**Archivos clave:**

- `SKILL.md`
- `README.md`
- `evals/evals.json`

---

### `bcv-azure-service-bus`

Responde preguntas y genera código para Azure Service Bus en proyectos BCV.

**Cubre:**

- Publishers y subscribers para cada variante de proyecto.
- `bcv-commons-pubsub`, `bcv-commons-topic`, ADS messaging y `commons.audit`.
- Configuración de topics, labels, subscriptions, connection strings y retry.
- Troubleshooting de mensajes perdidos, DLQ, label mismatches y `maxConcurrentCalls`.

**Triggers:** "Azure Service Bus", "bcv-commons-pubsub", "bcv-commons-topic", "ADS messaging", "DLQ", "maxConcurrentCalls", "label mismatch"

**Archivos clave:**

- `SKILL.md`
- `README.md`
- `evals/evals.json`
- `references/pubsub-publisher.md`
- `references/commons-topic-pubsub.md`
- `references/ads-messaging-pubsub.md`
- `references/commons-audit.md`

---

### `bcv-commons-observability`

Guía el uso de `bcv-commons-observability` para trazas, métricas, alertas de Teams y enmascaramiento de datos.

**Cubre:**

- `@ObservableController`, `@ObservableService`, `@ObservableOperation`.
- Configuración de alertas de Teams vía webhook en Azure Key Vault.
- Data masking para PII (DNI, RUC, account numbers, etc.).
- Propagación de trazas en código sincrónico y WebFlux.
- Troubleshooting de trazas faltantes y alertas no enviadas.

**Triggers:** "observabilidad BCV", "@ObservableService", "Teams alert", "data masking", "trace id", "OpenTelemetry", "enmascarar DNI"

**Archivos clave:**

- `SKILL.md`
- `README.md`
- `evals/evals.json`
- `references/annotations.md`
- `references/teams-alerts.md`
- `references/data-masking.md`
- `references/webflux-traces.md`

---

### `bcv-spring-data-jpa-sql-server`

Genera y mantiene componentes de persistencia con Spring Data JPA + SQL Server para proyectos BCV.

**Cubre:**

- Entidades JPA con convenciones de schema (`Disbursement`, `H2H`, etc.).
- Repositorios Spring Data con queries derivados y JPQL.
- Configuración de datasource con HikariCP y dialectos SQL Server.
- JPA auditing (`@CreatedBy`, `@CreatedDate`, `@LastModifiedBy`, `@LastModifiedDate`, `@Version`).
- SQL Server Always Encrypted.
- Colocación en arquitectura hexagonal/clean.

**Triggers:** "JPA BCV", "SQL Server entity", "Spring Data repository", "HikariCP", "Always Encrypted", "schema Disbursement", "BaseEntity", "JPA auditing"

**Archivos clave:**

- `SKILL.md`
- `README.md`
- `evals/evals.json`
- `evals/fixtures/pom-example.xml`
- `references/pom-templates.md`
- `references/configuration.md`
- `references/always-encrypted.md`
- `references/auditing.md`

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
- **Library-first:** preferir librerías internas (`bcv-commons-*`) sobre código propio.
- **Security-first:** secretos en Azure Key Vault, nunca en el repo.
- **Test-first:** cada skill debe generar o exigir tests.
- **Observability:** usar `@ObservableController` / `@ObservableService` / `@ObservableOperation` en componentes expuestos.

## Contribuidores

- NTT Data

## Notas

- Los skills asumen proyectos Java 17+ con Spring Boot 3.x.
- La arquitectura base es hexagonal/clean para servicios H2H y layered para `bcv-disb-business-service`.
- Para dudas sobre el uso de un skill, revisar su `README.md` interno.
