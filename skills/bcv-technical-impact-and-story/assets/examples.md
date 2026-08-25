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

evidence_freshness:
  graph_timestamp: "2026-08-24T10:00:00Z"
  code_reference: main@a1b2c3d
  warnings: []

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
- Candidate-only impacts blocking a final decision:
  - Persistencia del canal preferido.
  - Evento alert.escalated.v1.
- Required external validation:
  - Confirmar owner del canal preferido.
  - Definir contrato de escalamiento.

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

### 4.1 Contract

- Task: Definir/esperar contrato de transaction.suspicious.alert.v1.
- Traceability: HU criterio 1 → impacted_events confirmed.

### 4.2 Domain/Application

- Task: Implementar handler en notification-service para enviar notificación por canal preferido.
- Traceability: HU criterio 2 → impacted_apis confirmed.

### 4.3 Persistence/Messaging

- Task: Confirmar lectura del canal preferido; agregar topic alert.escalated.v1 si aplica.
- Traceability: HU criterio 3 → impacted_persistence candidate, impacted_events candidate.

### 4.4 Observability/Security

- Task: Tracing del envío de notificación y del escalamiento.
- Traceability: reglas de negocio de tiempo de respuesta.

### 4.5 Testing

- Task: Tests de integración para envío por cada canal y para escalamiento por timeout.
- Traceability: todos los criterios de aceptación.

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
- [ ] Open review items identified
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
