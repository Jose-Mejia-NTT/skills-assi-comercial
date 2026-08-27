> **Attribution:** This template is copied from and aligned with the output format of `ibk-hu-technical-refinement`.
> `bcv-dhu-writer` uses it as the exact public output format for DHU generation in the BCV ecosystem.

# Output Template - DHU

> This template defines the exact public output format for the skill.
> The result must read as a standalone technical HU and be INVEST-compliant, not as a refinement report.

---

```markdown
## DHU - {service_name}: {technical_responsibility}

> **Transformation date:** {YYYY-MM-DD}
> **Skill used:** bcv-dhu-writer v1.0.0
> **Input format:** {Direct text / Context file: .context/hu-<code>.md}
> **Output file:** `hu-technical-refinement/HU-{identifier}-refined-{YYYYMMDDHHmm}.md`
> **Story Points (Estimated):** {X points}
> **INVEST Status:** I(✅) N(✅) V(✅) E(✅) S(✅) T(✅)

---

### Alcance

{1-3 lines that delimit what this technical HU covers and what it does not cover}

> **INVEST I NOTE (Independent):** This HU is independent. It does not list dependencies on other tickets in the main body.

---

### Descripción breve

{2-5 technical lines describing the behavior of the microservice, API, or component. Focus on WHAT it does, not on WHAT it is implemented with}

---

### ¿Cuál es la necesidad?

{Specific problem that this technical HU solves and why it is necessary for the functional flow}

---

### HU Narrativa

```text
Yo como {Plataforma Digital / Servicio / Sistema}
Quiero {acción técnica observable}
Para que {resultado técnico o dependiente}
```

---

### Criterios de Aceptación

> Usar numeración `CA 01`, `CA 02`, `CA 03`... de forma consistente dentro del documento.
> Cada CA debe describir un comportamiento único, verificable y medible.
> Usar como máximo 8 CAs por HU. Si se requieren más, dividir en dos HUs.
> No incluir lenguaje ambiguo como "rápido", "seguro" o "correcto".

**CA 01 - {main validation}**
```text
Dado: {precondición}
Cuando: {acción}
Entonces: {resultado concreto con códigos, mensajes o estado}
```

**CA 02 — {regla de negocio o integración}**
```text
Dado: {precondición}
Cuando: {acción}
Entonces: {resultado concreto con códigos, mensajes o estado}
```

**CA 03 — {caso de error o borde}**
```text
Dado: {precondición}
Cuando: {acción}
Entonces: {resultado concreto con códigos, mensajes o estado}
```

**CA 04 - {optional, if applicable}**
```text
Dado: {precondición}
Cuando: {acción}
Entonces: {resultado concreto con códigos, mensajes o estado}
```

---

### Diagrama de flujo TO BE

```text
{ASCII flow, sequence, or link to the diagram}
```

---

### Endpoint(s)

> **MANDATORY for HUs that expose a REST API.** If not applicable, indicate "N/A - Internal component".
> Use only endpoints included in the endpoints catalog.

#### Endpoint 1: {operación principal}

**Método HTTP:** `{GET|POST|PUT|PATCH|DELETE}`  
**Path:** `{/api/v1/...}`  
**Descripción:** {Breve descripción de qué hace el endpoint}

**Headers requeridos:**
```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer {token}",
  "X-Request-Id": "{uuid}"
}
```

**Request body schema (si aplica):**
```json
{
  "field1": {
    "type": "string",
    "description": "{description}",
    "required": true,
    "example": "{example}"
  },
  "field2": {
    "type": "integer",
    "description": "{description}",
    "required": false
  }
}
```

**Response body schema (HTTP 200):**
```json
{
  "status": "success",
  "data": {
    "id": "{identifier}",
    "result": "{expected result}",
    "timestamp": "{ISO 8601}"
  },
  "requestId": "{uuid}"
}
```

**Error responses:**

| HTTP Code | Error Code | Message | Example Payload |
|---|---|---|---|
| 400 | `INVALID_REQUEST` | Required or invalid field | `{"status": "error", "error_code": "INVALID_REQUEST", "message": "Field 'field1' is required", "field": "field1"}` |
| 401 | `UNAUTHORIZED` | Missing, expired or invalid token | `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or expired token"}` |
| 403 | `FORBIDDEN` | User has no permission | `{"status": "error", "error_code": "FORBIDDEN", "message": "Insufficient permissions"}` |
| 404 | `NOT_FOUND` | Resource not found | `{"status": "error", "error_code": "NOT_FOUND", "message": "Resource with id {id} does not exist"}` |
| 500 | `INTERNAL_ERROR` | Internal server error | `{"status": "error", "error_code": "INTERNAL_ERROR", "message": "Error processing request", "requestId": "{uuid}"}` |
| 504 | `UPSTREAM_TIMEOUT` | Upstream service timeout | `{"status": "error", "error_code": "UPSTREAM_TIMEOUT", "message": "Service {serviceName} exceeded timeout of {timeoutSec}s", "upstreamService": "{serviceName}", "requestId": "{uuid}"}` |

**Ejemplo de solicitud (cURL):**
```bash
curl -X POST http://localhost:8080/api/v1/{endpoint} \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token}" \
  -d '{
    "field1": "value",
    "field2": 123
  }'
```

**Ejemplo de respuesta exitosa (HTTP 200):**
```json
{
  "status": "success",
  "data": {
    "id": "doc-123456",
    "result": "Validated",
    "timestamp": "2026-05-05T14:30:00Z"
  },
  "requestId": "req-uuid-1234"
}
```

**Ejemplo de respuesta con error (HTTP 400):**
```json
{
  "status": "error",
  "error_code": "INVALID_REQUEST",
  "message": "Field 'documentType' is required",
  "field": "documentType",
  "requestId": "req-uuid-5678"
}
```

---

### Referencias

| Recurso | Enlace |
|---|---|
| HU funcional origen | {link o identificador} |
| OpenAPI 3.0 | {link o identificador} |
| Arquitectura | {link o identificador} |
| Documentación | {link o identificador} |

> **Requerimiento INVEST:** Todas las referencias DEBEN tener URLs funcionales (Jira, Figma, repo, etc.).

---

### Control de Versiones

| Versión | Descripción | Fecha | Responsable | Estado |
|---|---|---|---|---|
| 1.0 | Versión inicial HU técnica | {date} | GitHub Copilot | EN ELABORACIÓN |

---

## APÉNDICE — Contexto Técnico (NO PARTE DE LA HU)

> **IMPORTANTE:** El contenido de este apéndice es contexto para el equipo de implementación.
> NO es parte de la HU técnica en sí (INVEST N compliance).
> El equipo de desarrollo puede elegir una stack diferente siempre que respete el contrato de endpoints y CAs.

### Canales [OPTIONAL]

- {App / Web / API / Tiendas / Backoffice}

### Aplicaciones Involucradas [OPTIONAL]

- {Sistema 1}
- {Sistema 2}
- {Sistema 3}

### Fuera del Alcance [OPTIONAL]

- {Exclusión 1}
- {Exclusión 2}

### Especificación Técnica [OPTIONAL]

| Aspecto | Detalle |
|---|---|
| Lenguaje | {e.g. Java 21} |
| Framework | {e.g. Spring Boot 3.x} |
| Puerto | {e.g. 8080} |
| Contexto | {e.g. /api/v1/documentos} |
| Base de datos | {aplica / no aplica} |
| Autenticación | {requerida / no requerida; si aplica, incluir esquema (Bearer, API Key, etc.)} |
| Timeout (seg) | {e.g. 30} |
| Rate limiting | {aplica / no aplica; si aplica: {requests/minute}} |
| Estructura del proyecto | {resumen corto de paquetes / módulos} |
| Estado | {EN ELABORACIÓN / EN REVISIÓN / APROBADO / FINALIZADO / OBSOLETO} |

> Esta información es contexto de implementación, no forma parte del contrato observable de la HU.
```
