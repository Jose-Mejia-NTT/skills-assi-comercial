# Examples

## Example 1: HU about alert notification

**Input (HU):**

> Como oficial de cumplimiento, quiero que el sistema notifique al cliente cuando
> una transacción sea marcada como sospechosa por el motor de riesgo, para que
> el cliente pueda confirmar o rechazar la operación antes de que se aplique
> cualquier bloqueo definitivo.
>
> Criterios de aceptación:
> - El sistema recibe el evento de alerta generado por el motor de riesgo.
> - Se envía una notificación al cliente por el canal preferido (correo, SMS o push).
> - Si el cliente no responde en 24 horas, la alerta se escala al área de fraude.
> - No se bloquea la cuenta como resultado directo de esta notificación.

**Expected output (`business-resolution.yaml`):**

```yaml
business_context:
  request_summary: >
    Notificar al cliente cuando una transacción es marcada como sospechosa
    por el motor de riesgo, permitiéndole confirmar o rechazar la operación
    antes de un posible bloqueo.
  systems:
    - motor de riesgo
    - canal de notificaciones
    - sistema de fraude
  entities:
    - transacción
    - alerta de riesgo
    - cliente
    - canal preferido
  actions:
    - marcar transacción como sospechosa
    - notificar al cliente
    - confirmar operación
    - rechazar operación
    - escalar alerta
  events:
    - alerta de transacción sospechosa generada
    - notificación enviada al cliente
    - alerta escalada por falta de respuesta
  rules:
    - El canal preferido se elige entre correo, SMS o push.
    - El tiempo máximo de espera por respuesta del cliente es 24 horas.
    - Esta notificación no produce bloqueo definitivo de la cuenta.
  acceptance_criteria:
    - Recibir evento de alerta del motor de riesgo.
    - Enviar notificación por canal preferido.
    - Escalar al área de fraude tras 24 horas sin respuesta.
    - No bloquear cuenta como consecuencia directa.
  scope:
    in:
      - recepción del evento de alerta
      - envío de notificación
      - escalamiento por inactividad
    out:
      - bloqueo definitivo de cuenta
      - evaluación del motor de riesgo

capabilities:
  - id: customer-notification
    normalized_name: Customer Notification
    evidence:
      - source: bcv-bacc-capability-catalog.yaml
        statement: Capacidad de enviar notificaciones a clientes por múltiples canales.
        date: "2026-08-24"
  - id: fraud-case-management
    normalized_name: Fraud Case Management
    evidence:
      - source: bcv-bacc-capability-catalog.yaml
        statement: Capacidad de gestionar alertas de fraude y su escalamiento.
        date: "2026-08-24"

candidates:
  - service: bcv-bacc-notification-service
    role: owner
    confidence: high
    evidence:
      - source: bcv-bacc-capability-catalog.yaml
        statement: El catálogo asigna customer-notification a bcv-bacc-notification-service.
        date: "2026-08-24"
  - service: bcv-bacc-fraud-service
    role: orchestrator
    confidence: medium
    evidence:
      - source: bcv-bacc-capability-catalog.yaml
        statement: El motor de riesgo genera la alerta y el área de fraude recibe el escalamiento.
        date: "2026-08-24"
  - service: bcv-bacc-customer-preferences-service
    role: data_owner
    confidence: medium
    evidence:
      - source: service-map.md
        statement: Posible propietario del canal preferido del cliente.
        date: "2026-08-24"

ambiguities:
  - type: ownership-channel-preference
    detail: No queda claro si el canal preferido es gestionado por notification-service o customer-preferences-service.
    impact: Puede afectar la asignación de data_owner y requiere confirmación técnica.
  - type: escalation-mechanism
    detail: La HU menciona escalar al área de fraude pero no define si es un ticket, un evento o una bandeja.
    impact: Impacta en el rol de orchestrator y en el contrato de handoff.

resolution_status: REVIEW_REQUIRED

handoff:
  to_skill: bcv-technical-impact-and-story
  output_artifact: docs/historial/alerta-sospechosa-notificacion-business-resolution.yaml
  business_resolution_status: REVIEW_REQUIRED
  original_request_summary: >
    Notificar al cliente cuando una transacción es marcada como sospechosa
    por el motor de riesgo.
  acceptance_criteria:
    - Recibir evento de alerta del motor de riesgo.
    - Enviar notificación por canal preferido.
    - Escalar al área de fraude tras 24 horas sin respuesta.
    - No bloquear cuenta como consecuencia directa.
  candidate_verification_questions:
    - ¿Dónde se almacena el canal preferido del cliente?
    - ¿Cómo se modela el evento de alerta del motor de riesgo?
    - ¿Qué mecanismo de escalamiento al área de fraude existe actualmente?
  technical_verification_questions:
    - Confirmar si notification-service expone o consume /notifications/send.
    - Verificar si fraud-service publica transaction.suspicious.alert.v1.
    - Validar si customer-preferences-service posee la tabla customer_channel_preference.
  assumptions:
    - ASSUMED: El canal preferido ya está disponible antes de enviar la notificación.
    - ASSUMED: El motor de riesgo publica un evento que puede ser consumido.
  risks:
    - RISK: Si el canal preferido cambia con frecuencia, la confianza en data_owner podría disminuir.
    - RISK: Si el escalamiento no está modelado, el rol de orchestrator podría cambiar.
```

## Example 2: BLOCKED request

**Input:**

> Necesito analizar una HU sobre cuentas.

**Expected output:**

```yaml
resolution_status: BLOCKED
ambiguities:
  - type: insufficient-input
    detail: No se proporcionó el texto de la HU ni criterios de aceptación.
    impact: No es posible extraer entidades, acciones, eventos ni capacidades.
handoff:
  candidate_verification_questions:
    - ¿Puedes compartir el texto completo de la HU o HAB?
    - ¿Cuáles son los criterios de aceptación o reglas de negocio involucradas?
    - ¿Hay algún catálogo de capacidades o mapa de servicios que deba cargar?
```
