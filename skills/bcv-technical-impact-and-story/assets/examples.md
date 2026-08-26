# Examples

## Example 1: Full mode (HU + business-resolution → impact analysis + story)

### Input

**HU:**

> Como oficial de cumplimiento, quiero que el sistema notifique al cliente cuando
> una transacción sea marcada como sospechosa por el motor de riesgo, para que
> el cliente pueda confirmar o rechazar la operación.

**`business-resolution.yaml`** (simplified candidate list):

```yaml
business_context:
  request_summary: Notificar al cliente cuando una transacción es marcada como sospechosa.
candidates:
  - service: bcv-bacc-notification-service
    role: owner
    confidence: high
  - service: bcv-bacc-fraud-service
    role: orchestrator
    confidence: medium
  - service: bcv-bacc-customer-preferences-service
    role: data_owner
    confidence: medium
resolution_status: REVIEW_REQUIRED
```

### Phase A output (`technical-impact-analysis.yaml`)

```yaml
analysis_scope:
  hu_slug: alerta-sospechosa-notificacion
  request_summary: Notificar al cliente cuando una transacción es marcada como sospechosa.
  source_business_resolution: docs/historial/alerta-sospechosa-notificacion-business-resolution.yaml
  business_resolution_status: REVIEW_REQUIRED
  candidate_services_input:
    - bcv-bacc-notification-service
    - bcv-bacc-fraud-service
    - bcv-bacc-customer-preferences-service
  execution_mode: full

primary_service:
  service: bcv-bacc-notification-service
  role: owner
  confidence: high
  evidence:
    - source: graph-index.md
      statement: Servicio propietario de la capability customer-notification.
      freshness: "2026-08-24"
      type: confirmed

supporting_services:
  - service: bcv-bacc-fraud-service
    role: orchestrator
    confidence: medium
    evidence:
      - source: service-map.md
        statement: Publica el evento de alerta de transacción sospechosa.
        freshness: "2026-08-24"
        type: candidate
  - service: bcv-bacc-customer-preferences-service
    role: data_owner
    confidence: medium
    evidence:
      - source: service-map.md
        statement: Posible propietario del canal preferido del cliente.
        freshness: "2026-08-24"
        type: candidate

impacted_apis:
  confirmed:
    - bcv-bacc-notification-service /notifications/send
  candidate:
    - bcv-bacc-fraud-service /alerts/{id}/escalate

impacted_persistence:
  confirmed: []
  candidate:
    - bcv-bacc-customer-preferences-service.customer_channel_preference

impacted_events:
  confirmed:
    - topic: transaction.suspicious.alert.v1
      producer: bcv-bacc-fraud-service
      consumer: bcv-bacc-notification-service
  candidate:
    - topic: alert.escalated.v1
      producer: bcv-bacc-notification-service
      consumer: bcv-bacc-fraud-service

impacted_files:
  confirmed:
    - path: src/main/java/com/bcv/notification/domain/SuspiciousAlertHandler.java
      change: modify
      reason: Agregar lógica de consumo del evento transaction.suspicious.alert.v1 y envío de notificación.
      service: bcv-bacc-notification-service
    - path: src/main/java/com/bcv/notification/application/SendNotificationUseCase.java
      change: create
      reason: Nuevo caso de uso para orquestar el envío por canal preferido.
      service: bcv-bacc-notification-service
    - path: src/main/resources/db/migration/V20260825__add_notification_channel_columns.sql
      change: create
      reason: Columnas para trazabilidad del envío y escalamiento.
      service: bcv-bacc-notification-service
  candidate:
    - path: src/main/java/com/bcv/customer/domain/CustomerChannelPreference.java
      change: modify
      reason: Posible origen del canal preferido; confirmar ownership en IMP-000.
      service: bcv-bacc-customer-preferences-service
    - path: src/main/java/com/bcv/fraud/application/AlertEscalationPublisher.java
      change: create
      reason: Publicar evento de escalamiento si se confirma mecanismo por evento.
      service: bcv-bacc-fraud-service

evidence_freshness:
  graph_timestamp: "2026-08-24T10:00:00Z"
  code_reference: main@a1b2c3d
  warnings: []

inherited_ambiguities:
  - type: ownership-channel-preference
    detail: No queda claro si el canal preferido es gestionado por notification-service o customer-preferences-service.
    source: business-resolution.yaml
  - type: escalation-mechanism
    detail: La HU menciona escalar al área de fraude pero no define si es un ticket, un evento o una bandeja.
    source: business-resolution.yaml

conflicts:
  - channel_preference_owner: notification-service vs customer-preferences-service
  - escalation_mechanism: not defined in available docs

risks:
  - "RISK: El canal preferido podría no estar en customer-preferences-service."
  - "RISK: El mecanismo de escalamiento no está modelado."

assumptions:
  - "ASSUMED: El motor de riesgo publica transaction.suspicious.alert.v1."
  - "ASSUMED: El canal preferido es accesible por notification-service."

verification_notes:
  open_questions:
    - ¿Dónde se almacena el canal preferido del cliente?
    - ¿Cómo se modela el evento de alerta del motor de riesgo?
    - ¿Qué mecanismo de escalamiento al área de fraude existe actualmente?
  escalation_needed: true

technical_status: REVIEW_REQUIRED
```

### Phase B output (`technical-story-enriched.md`)

```markdown
# Technical Story Enriched

> Source impact analysis: `docs/historial/alerta-sospechosa-notificacion-technical-impact-analysis.yaml`
> Technical status consumed: REVIEW_REQUIRED

## 0. Pending Architecture Review

- Open conflicts:
  - Canal preferido: notification-service vs customer-preferences-service.
  - Mecanismo de escalamiento al área de fraude no definido.
- Inherited ambiguities from business resolution:
  - `ownership-channel-preference`: data_owner del canal preferido no confirmado.
  - `escalation-mechanism`: mecanismo de escalamiento al área de fraude no definido.
- Candidate-only impacts blocking a final decision:
  - Persistencia del canal preferido.
  - Evento alert.escalated.v1.
- Required external validation:
  - Confirmar owner del canal preferido.
  - Definir contrato de escalamiento.
- Discovery tasks (IMP-000) blocking technical implementation:
  - IMP-002, IMP-004, IMP-005, IMP-006

## 1. Functional Context Summary

- HU/HAB summary: Notificar al cliente cuando una transacción es marcada como sospechosa.
- Acceptance criteria:
  - Recibir evento de alerta del motor de riesgo.
  - Enviar notificación por canal preferido.
  - Escalar al área de fraude tras 24 horas sin respuesta.
- In scope: recepción del evento, envío de notificación, escalamiento por inactividad.
- Out of scope: evaluación del motor de riesgo, bloqueo definitivo de cuenta.

## 2. Impacted Services Decision

- Primary service: bcv-bacc-notification-service (owner, high confidence).
- Supporting services:
  - bcv-bacc-fraud-service (orchestrator, candidate).
  - bcv-bacc-customer-preferences-service (data_owner, candidate).

## 3. Technical Impact Matrix

### 3.1 APIs

- Confirmed: bcv-bacc-notification-service /notifications/send
- Candidate: bcv-bacc-fraud-service /alerts/{id}/escalate

### 3.2 Persistence

- Confirmed: none
- Candidate: bcv-bacc-customer-preferences-service.customer_channel_preference

### 3.3 Events and Integrations

- Confirmed: transaction.suspicious.alert.v1 (fraud → notification)
- Candidate: alert.escalated.v1 (notification → fraud)

## 4. Technical Task Plan

### IMP-000: Resolver ambigüedades y conflictos heredados (Discovery)

- **Service:** arquitectura / bcv-bacc-notification-service / bcv-bacc-customer-preferences-service / bcv-bacc-fraud-service
- **Type:** discovery
- **Recommended skill category:** `hu-analysis` (mapear al skill de análisis de HU/impacto disponible)
- **Description:** Investigar y resolver las ambigüedades del business resolution y los conflictos técnicos detectados antes de ejecutar tareas dependientes.
- **Step-by-step implementation:**
  1. Revisar `business-resolution.yaml` y extraer ambigüedades (`ownership-channel-preference`, `escalation-mechanism`).
  2. Consultar `service-map.md` y `graph-index.md` para validar ownership del canal preferido.
  3. Coordinar con arquitectura de canales y área de fraude para definir mecanismo de escalamiento.
  4. Documentar decisiones y actualizar `technical-impact-analysis.yaml`.
  5. Desbloquear tareas IMP-002, IMP-004, IMP-005 e IMP-006.
- **Inherited ambiguities / conflicts:**
  - `ownership-channel-preference`: ¿notification-service o customer-preferences-service es data_owner del canal preferido? → Validar con service-map y dueños de datos → Owner: arquitectura de datos.
  - `escalation-mechanism`: ¿El escalamiento es un evento, ticket o bandeja? → Definir contrato con área de fraude → Owner: arquitectura / fraude.
- **Clarification questions:**
  - ¿Dónde se almacena el canal preferido del cliente actualmente?
  - ¿Cómo se modela el evento de alerta del motor de riesgo?
  - ¿Qué mecanismo de escalamiento al área de fraude existe actualmente?
- **Suggested resolution options:**
  - **Option A:** customer-preferences-service es data_owner del canal preferido; notification-service lo consume vía API. Escalamiento por evento `alert.escalated.v1` hacia fraud-service.
  - **Option B:** notification-service almacena una copia del canal preferido sincronizado por evento; escalamiento por ticket en bandeja de fraude.
  - **Option C (fallback):** mantener ambigüedad como riesgo controlado, documentar supuestos y usar notification-service como orchestrator provisional.
- **Impact if unresolved:**
  - IMP-002 e IMP-005 quedan bloqueadas por el canal preferido.
  - IMP-004 e IMP-005 quedan bloqueadas por el mecanismo de escalamiento.
- **Unblock condition:**
  - Se confirma el data_owner del canal preferido y el mecanismo de escalamiento, o se aceptan como riesgos controlados con escenarios documentados.
- **HU traceability:** Criterio 2 y 3 — envío por canal preferido y escalamiento.
- **Impact traceability:** inherited_ambiguities + conflicts
- **Dependencies:** Ninguna
- **Status:** TODO

### IMP-001: Acordar contrato del evento `transaction.suspicious.alert.v1`

- **Service:** bcv-bacc-fraud-service / bcv-bacc-notification-service
- **Type:** contract
- **Recommended skill category:** `contract-design` (mapear al skill de diseño de contratos/APIs disponible)
- **Description:** Definir el schema, claves y semántica del evento que notifica una transacción marcada como sospechosa.
- **Step-by-step implementation:**
  1. Reunirse con los equipos de fraud-service y notification-service para alinear el contrato.
  2. Definir campos obligatorios: transaction_id, timestamp, risk_level.
  3. Versionar el schema en el schema registry o documento de contratos.
  4. Validar el contrato con payload mínimo y completo.
  5. Actualizar documentación de integración.
- **HU traceability:** Criterio 1 — recibir evento de alerta del motor de riesgo.
- **Impact traceability:** impacted_events.confirmed.transaction.suspicious.alert.v1
- **Dependencies:** Ninguna
- **Unblock condition:**
- **Status:** TODO

### IMP-002: Implementar handler de envío de notificación

- **Service:** bcv-bacc-notification-service
- **Type:** domain
- **Recommended skill category:** `backend-dev` (mapear al skill de implementación backend disponible)
- **Description:** Crear o extender el handler que recibe el evento y envía la notificación por el canal preferido del cliente.
- **Step-by-step implementation:**
  1. Extender `SuspiciousAlertHandler` para consumir `transaction.suspicious.alert.v1`.
  2. Crear `SendNotificationUseCase` para orquestar el envío.
  3. Integrar lectura del canal preferido desde el servicio confirmado en IMP-000.
  4. Invocar `/notifications/send` con parámetros nombre, cuenta, CCI y moneda.
  5. Agregar logs y métricas de éxito/fracaso.
- **HU traceability:** Criterio 2 — enviar notificación por canal preferido.
- **Impact traceability:** impacted_apis.confirmed.bcv-bacc-notification-service /notifications/send
- **Dependencies:** IMP-000, IMP-001
- **Unblock condition:** Resuelta la ambigüedad `ownership-channel-preference` en IMP-000.
- **Status:** BLOCKED

### IMP-003: Confirmar fuente del canal preferido del cliente

- **Service:** bcv-bacc-customer-preferences-service (candidate)
- **Type:** persistence
- **Recommended skill category:** `database-dev` (mapear al skill de base de datos disponible)
- **Description:** Validar si el canal preferido se lee desde customer-preferences-service o si notification-service ya lo posee.
- **HU traceability:** Criterio 2 — enviar notificación por canal preferido.
- **Impact traceability:** impacted_persistence.candidate.customer_channel_preference
- **Dependencies:** Ninguna
- **Unblock condition:**
- **Status:** TODO

### IMP-004: Definir mecanismo de escalamiento por timeout

- **Service:** bcv-bacc-notification-service / bcv-bacc-fraud-service
- **Type:** domain
- **Recommended skill category:** `backend-dev` (mapear al skill de implementación backend disponible)
- **Description:** Modelar la publicación del evento alert.escalated.v1 cuando el cliente no responde en 24 horas.
- **HU traceability:** Criterio 3 — escalar al área de fraude tras 24 horas sin respuesta.
- **Impact traceability:** impacted_events.candidate.alert.escalated.v1
- **Dependencies:** IMP-000, IMP-001
- **Unblock condition:** Resuelta la ambigüedad `escalation-mechanism` en IMP-000.
- **Status:** BLOCKED

### IMP-005: Agregar tracing y métricas

- **Service:** bcv-bacc-notification-service
- **Type:** observability
- **Recommended skill category:** `observability-dev` (mapear al skill de observabilidad/ops disponible)
- **Description:** Instrumentar el envío de notificación y el escalamiento para observabilidad.
- **HU traceability:** Regla de negocio — tiempo máximo de espera de 24 horas.
- **Impact traceability:** impacted_events.confirmed + candidate
- **Dependencies:** IMP-000, IMP-002, IMP-004
- **Unblock condition:** Resueltas las ambigüedades `ownership-channel-preference` y `escalation-mechanism` en IMP-000.
- **Status:** BLOCKED

### IMP-006: Tests de integración

- **Service:** bcv-bacc-notification-service
- **Type:** testing
- **Recommended skill category:** `testing-dev` (mapear al skill de testing disponible)
- **Description:** Tests de integración para envío por correo, SMS y push, más escenario de escalamiento por timeout.
- **HU traceability:** Todos los criterios de aceptación.
- **Impact traceability:** impacted_apis.confirmed + impacted_events.confirmed/candidate
- **Dependencies:** IMP-000, IMP-002, IMP-004
- **Unblock condition:** Resueltas las ambigüedades `ownership-channel-preference` y `escalation-mechanism` en IMP-000.
- **Status:** BLOCKED

## 5. Risks and Assumptions

- ASSUMED: El motor de riesgo publica transaction.suspicious.alert.v1.
- ASSUMED: El canal preferido es accesible por notification-service.
- RISK: El canal preferido podría no estar en customer-preferences-service.
- RISK: El mecanismo de escalamiento no está modelado.

## 6. Validation Checklist

- [ ] Coverage of HU rules and acceptance criteria
- [ ] Non-regression checks defined
- [ ] HU -> HT -> tasks traceability complete
- [ ] Confirmed vs candidate split is explicit
- [ ] Every technical item traces back to the source impact analysis
- [ ] Every task links to at least one HU criterion and one impact item
- [ ] Architecture diagram reflects services, APIs, events and persistence from the impact matrix
- [ ] Open review items identified

## 7. Implementation Architecture Diagram

Also written to: `docs/historial/alerta-sospechosa-notificacion-technical-architecture-diagram.mmd`

```mermaid
graph LR
    subgraph "Dominio de Fraude"
        FRAUD[bcv-bacc-fraud-service]
        FRAUD -->|publishes| E1[transaction.suspicious.alert.v1]
        FRAUD -->|receives escalation| E2[alert.escalated.v1 (candidate)]
    end

    subgraph "Dominio de Notificaciones"
        NOTIF[bcv-bacc-notification-service]
        NOTIF -->|consumes| E1
        NOTIF -->|calls| API1[/notifications/send (confirmed)\]
        NOTIF -->|publishes| E2
    end

    subgraph "Dominio de Cliente"
        PREFS[bcv-bacc-customer-preferences-service]
        DB1[(customer_channel_preference (candidate))]
        PREFS --> DB1
    end

    NOTIF -.->|reads channel (candidate)| PREFS

    style E2 stroke-dasharray: 5 5
    style API1 stroke-width:2px
    style DB1 fill:#f9f,stroke:#333,stroke-dasharray: 5 5
```

## 8. Detailed Action Plan

### IMP-000: Resolver ambigüedades heredadas de business resolution

- **Definition of Ready (DoR):**
  - Equipo de arquitectura disponible para validar ownership del canal preferido.
  - Documentación actualizada de service-map y customer-preferences-service accesible.
- **Definition of Done (DoD):**
  - Se confirma quién es data_owner del canal preferido.
  - Se define el mecanismo de escalamiento al área de fraude (evento/ticket/bandeja).
  - El technical-impact-analysis.yaml se actualiza con evidencia confirmada.
- **Technical acceptance criteria:**
  - El contrato de lectura del canal preferido está acordado.
  - El evento o API de escalamiento tiene owner y contrato definido.
- **Test cases:**
  - Verificar que notification-service puede obtener el canal preferido del owner correcto.
  - Simular timeout de 24h y confirmar que el escalamiento llega al destino acordado.
- **Error scenarios:**
  - Si el owner del canal preferido no responde, mantener el item como REVIEW_REQUIRED.
  - Si el mecanismo de escalamiento no existe, documentar como dependencia externa.
- **External dependencies:**
  - Aprobación de arquitectura de canales y fraude.
- **Deployment considerations:**
  - Ninguna; esta tarea es de clarificación previa al desarrollo.
- **Files affected:**
  - Ninguno; esta tarea es de clarificación.
- **Security / compliance notes:**
  - Confirmar que el canal preferido no expone PII adicional al necesario.

### IMP-001: Acordar contrato del evento `transaction.suspicious.alert.v1`

- **Definition of Ready (DoR):**
  - Equipos de fraud-service y notification-service alineados.
  - Schema registry o contrato de eventos disponible.
- **Definition of Done (DoD):**
  - Schema del evento documentado y versionado.
  - Ambos servicios validan el contrato.
- **Technical acceptance criteria:**
  - El evento incluye identificador de transacción, timestamp y nivel de riesgo.
  - El producer y consumer están acordados.
- **Test cases:**
  - Publicar evento de prueba y verificar que notification-service lo consume.
  - Validar schema con payload mínimo y completo.
- **Error scenarios:**
  - Evento con schema inválido: descartar y loggear error.
  - Consumer caído: evento queda en cola con reintentos configurados.
- **External dependencies:**
  - Schema registry / Kafka / broker de eventos.
- **Deployment considerations:**
  - Desplegar consumer antes que producer para no perder eventos.
  - Agregar alerta de lag del consumer.
- **Files affected:**
  - `src/main/java/com/bcv/notification/domain/SuspiciousAlertHandler.java` (modify) — agregar manejador del evento `transaction.suspicious.alert.v1` y delegar al caso de uso de envío.
  - `src/main/java/com/bcv/fraud/application/AlertPublisher.java` (modify, candidate) — publicar el evento con los campos acordados desde el motor de riesgo.
- **Security / compliance notes:**
  - El evento no debe incluir datos sensibles del cliente en claro.

## 10. Repository File Impact

### Files to create

- `src/main/java/com/bcv/notification/application/SendNotificationUseCase.java` — nuevo caso de uso para envío de notificación (bcv-bacc-notification-service)
- `src/main/resources/db/migration/V20260825__add_notification_channel_columns.sql` — migración para trazabilidad (bcv-bacc-notification-service)
- `src/main/java/com/bcv/fraud/application/AlertEscalationPublisher.java` — publicador de escalamiento (bcv-bacc-fraud-service, candidate)

### Files to modify

- `src/main/java/com/bcv/notification/domain/SuspiciousAlertHandler.java` — agregar método de consumo del evento `transaction.suspicious.alert.v1` e invocar caso de uso de envío (bcv-bacc-notification-service).
- `src/main/java/com/bcv/notification/application/SendNotificationUseCase.java` — orquestar lectura de canal preferido e invocación de `/notifications/send` (bcv-bacc-notification-service).
- `src/main/java/com/bcv/customer/domain/CustomerChannelPreference.java` — confirmar/agregar fuente del canal preferido (bcv-bacc-customer-preferences-service, candidate).

### Domains / entities affected

- `Notification` domain — envío de notificaciones por canal preferido
- `CustomerChannelPreference` entity — origen del canal preferido (candidate)
- `FraudAlert` domain — publicación de alerta y escalamiento

### Migrations / configuration

- Migration: `V20260825__add_notification_channel_columns.sql` — columnas de trazabilidad
- Config: `application.yml` — topic names y retry policies

## 11. Developer Review & Sign-off

- [ ] El plan cubre todos los criterios de aceptación de la HU.
- [ ] Cada tarea es entendible y ejecutable sin inventar requisitos.
- [ ] Las dependencias y estados BLOCKED son correctos.
- [ ] Cada tarea BLOCKED tiene una condición de desbloqueo clara.
- [ ] Se cubren escenarios de error y casos límite.
- [ ] Las decisiones técnicas son razonables y están documentadas.
- [ ] Se identifican archivos del repo a crear/modificar/eliminar.
- [ ] No falta configuración, migración o documentación.

**Reviewer:** _________________  
**Date:** _________________  
**Approved:** [ ] Sí  [ ] No — comentarios:

## 12. Open Questions / Clarifications Needed

| # | Question | Blocking | Proposed fallback if not clarified |
|---|----------|----------|------------------------------------|
| 1 | ¿El canal preferido se lee de customer-preferences-service o se replica en notification-service? | Sí | Se asume customer-preferences-service como data_owner provisional. |
| 2 | ¿El escalamiento al área de fraude es un evento o una bandeja/ticket? | Sí | Se documenta como dependencia externa no resuelta. |

**User confirmation:** _________________  
**Date:** _________________

### Chat response example

```text
Generé el análisis técnico con estado REVIEW_REQUIRED porque faltan 2 aclaraciones.

Pregunta 1 de 2 — ¿El canal preferido se lee de customer-preferences-service o se replica en notification-service?
Esta decisión desbloquea la asignación de data_owner y el contrato de lectura.

Responde con una decisión concreta. Ejemplo:
"customer-preferences-service es data_owner y notification-service consulta GET /customers/{id}/channel-preferences."

Si no está definido:
"No está definido. Debe confirmarlo Arquitectura de Canales."

No basta con "ok". Responde en este chat.
```

After the user answers question 1:

```text
Gracias. Pregunta 2 de 2 — ¿El escalamiento al área de fraude es un evento o una bandeja/ticket?
Esta decisión desbloquea el contrato de escalamiento y el rol de orchestrator.

Responde con una decisión concreta. Ejemplo:
"Es un evento alert.escalated.v1 publicado por notification-service y consumido por fraud-service."

Si no está definido:
"No está definido. Debe confirmarlo el área de Fraude."

No basta con "ok".
```

## Example 2: Story-only mode

### Input

Existing `docs/historial/alerta-sospechosa-notificacion-technical-impact-analysis.yaml`
with `technical_status: REVIEW_REQUIRED`.

### Output

Same `technical-story-enriched.md` as Example 1, derived **only** from the YAML.
No graph queries are run. The story opens with the pending-review block.

## Example 3: BLOCKED request

### Input

> Analiza el impacto técnico de esta HU.

No HU text, no `business-resolution.yaml`, no candidate services.

### Output

```yaml
analysis_scope:
  hu_slug: ""
  request_summary: ""
  candidate_services_input: []
  execution_mode: full

primary_service:
  service: ""
  role: ""
  confidence: ""
  evidence: []

supporting_services: []
impacted_apis:
  confirmed: []
  candidate: []
impacted_persistence:
  confirmed: []
  candidate: []
impacted_events:
  confirmed: []
  candidate: []

evidence_freshness:
  graph_timestamp: ""
  code_reference: ""
  warnings:
    - "Missing HU/HAB text and candidate services."

conflicts: []
risks:
  - "RISK: Cannot perform technical impact analysis without inputs."
assumptions: []

verification_notes:
  open_questions:
    - "¿Puedes compartir el texto de la HU/HAB?"
    - "¿Tienes un business-resolution.yaml o una lista de servicios candidatos?"
  escalation_needed: false

technical_status: BLOCKED
```

No `technical-story-enriched.md` is produced.
