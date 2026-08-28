> **Attribution:** This template extends `output-template.md` with actionable implementation sections.
> Use it when the DHU must be ready for development planning, even if some gaps are still pending.
> Sections blocked by unresolved gaps must be explicitly marked as `PENDIENTE` or `BLOQUEADO`.

# Output Template - DHU Extendido (Opción A)

> This template defines the extended public output format for `bcv-dhu-writer`.
> The result must read as a standalone technical HU and include concrete implementation guidance.
> If a gap blocks a section, the section must still appear with a clear blocker note.

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
| HU funcional origen | {identificador o link si existe} |
| OpenAPI 3.0 | {identificador o link si existe} |
| Arquitectura | {identificador o link si existe} |
| Documentación | {identificador o link si existe} |

> **Nota:** Las referencias son opcionales. Si no se dispone de URLs reales, basta con el identificador (ej: `Jira-12345`, `RFC-042`). No deben bloquear la aprobación de la DHU.

---

### Control de Versiones

| Versión | Descripción | Fecha | Responsable | Estado |
|---|---|---|---|---|
| 1.0 | Versión inicial HU técnica | {date} | GitHub Copilot | EN ELABORACIÓN |

---

## SECCIONES EXTENDIDAS — Plan de Implementación

> Estas secciones hacen la DHU actionable para el equipo de desarrollo.
> Si un gap bloquea la definición, marcar como `PENDIENTE` o `BLOQUEADO` con la razón.

---

### Plan de implementación de tareas

> Desglose por capa y servicio. Una tarea por fila.

| # | Servicio | Capa | Tarea | Estimación | Bloqueada por |
|---|---|---|---|---|---|
| 1 | `{service}` | Input / Controller | {tarea concreta} | {Xh} | {gap o N/A} |
| 2 | `{service}` | Core / Use Case | {tarea concreta} | {Xh} | {gap o N/A} |
| 3 | `{service}` | Output / Repository | {tarea concreta} | {Xh} | {gap o N/A} |
| 4 | `{service}` | Mapper / DTO | {tarea concreta} | {Xh} | {gap o N/A} |

> **Si hay gaps bloqueantes:**
> ```text
> ⚠️ BLOQUEADO: No se puede definir el plan de tareas hasta resolver:
> - [Gap 1]
> - [Gap 2]
> ```

---

### Mapa técnico de implementación

> Mapa completo de los elementos concretos necesarios para desarrollar la DHU.
> Si un gap bloquea algún elemento, marcarlo como `PENDIENTE`.

#### {service-name}

| Elemento | Detalle | Estado |
|---|---|---|
| **Módulo / capa** | `input` / `core` / `output` | ✅ |
| **Archivos exactos** | `src/main/java/.../{Class}.java` | ✅ |
| **Clases y métodos** | `{Class}.{method}()` | ✅ |
| **DTOs / records** | `{RequestRecord}`, `{ResponseRecord}` | ✅ |
| **Entidades / repositorios** | `{Entity}.java`, `{Repository}.java` | ✅ |
| **Endpoints / contratos** | `POST /api/v1/...` | ✅ |
| **Cambios requeridos** | Agregar campo X, validar catálogo, persistir valor. | ✅ |
| **Pruebas asociadas** | Unitarias: `{Class}Test`. Integración: `{Flow}IT`. | ✅ |
| **Dependencias / pendientes** | Ninguna o referencia al gap. | ✅ |

#### {service-name-2}

| Elemento | Detalle | Estado |
|---|---|---|
| **Módulo / capa** | `input` / `core` / `output` | ✅ |
| **Archivos exactos** | `src/main/java/.../{Class}.java` | ✅ |
| **Clases y métodos** | `{Class}.{method}()` | ✅ |
| **DTOs / records** | `{MessageInDto}`, `{MessageOutDto}` | ✅ |
| **Entidades / repositorios** | N/A (stateless) | ✅ |
| **Endpoints / contratos** | ASB topic `{topic-name}` | ✅ |
| **Cambios requeridos** | Incluir campo en payload del evento. | ✅ |
| **Pruebas asociadas** | Unitarias: `{Handler}Test`. | ✅ |
| **Dependencias / pendientes** | Ninguna o referencia al gap. | ✅ |

> **Si hay gaps bloqueantes:**
> ```text
> ⚠️ PENDIENTE: El mapa técnico queda incompleto hasta resolver:
> - [Gap 1]
> - [Gap 2]
> ```

---

### Checklist de validación

> Casos a verificar durante desarrollo y QA.

- [ ] Caso funcional happy path.
- [ ] Campo obligatorio sin valor.
- [ ] Valor fuera del catálogo permitido.
- [ ] Persistencia correcta en base de datos.
- [ ] Visualización en consultas futuras.
- [ ] Auditoría del valor seleccionado.
- [ ] Integración con servicios downstream.
- [ ] Manejo de error del servicio externo (si aplica).

---

### Technical Impact Matrix

> Impacto por servicio y componente.

| Servicio | Componente | Contrato afectado | Base de datos | Riesgo | Responsable |
|---|---|---|---|---|---|
| `{service}` | `{component}` | `{dto/endpoint/queue}` | `{tabla/campo}` | {Alto/Medio/Bajo} | {rol} |
| `{service}` | `{component}` | `{dto/endpoint/queue}` | `{tabla/campo}` | {Alto/Medio/Bajo} | {rol} |

> **Si hay gaps bloqueantes:**
> ```text
> ⚠️ PENDIENTE: La matriz de impacto queda incompleta hasta resolver [gap].
> ```

---

### Definition of Ready (DoR)

> Condiciones que deben cumplirse antes de iniciar implementación.

- [ ] HU funcional aprobada.
- [ ] Endpoint / cola definido.
- [ ] Catálogo / fuente de datos confirmada.
- [ ] Ubicación del campo en dominio/entidad definida.
- [ ] Mecanismo de auditoría definido.
- [ ] Gaps bloqueantes resueltos o aceptados.

> **Si hay gaps abiertos:**
> ```text
> ❌ DoR NO CUMPLIDO: faltan definir [gap 1], [gap 2].
> ```

---

### Definition of Done (DoD)

> Condiciones que deben cumplirse para dar la HU por terminada.

- [ ] Código implementado y revisado.
- [ ] Pruebas unitarias con cobertura mínima definida por el equipo.
- [ ] Pruebas de integración del flujo.
- [ ] Contrato actualizado (OpenAPI, DTO, evento).
- [ ] Migración de base de datos aplicada (si aplica).
- [ ] Documentación técnica actualizada.
- [ ] Validaciones funcionales ejecutadas.
- [ ] Gaps resueltos verificados.

---

### Checklist final pre-merge

> Verificaciones antes de mergear a `main`.

- [ ] Linter sin errores.
- [ ] Tests unitarios e integración pasan.
- [ ] Análisis estático sin issues críticas.
- [ ] Compatibilidad de mensajes validada (ASB/queues).
- [ ] Grafo graphify actualizado si hubo cambios estructurales.
- [ ] DHU actualizada con cambios finales.

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

### Gaps identificados

> Lista de gaps técnicos pendientes. Deben resolverse antes de considerar el DoR completo.

| ID | Gap | Tipo | Estado | Bloquea |
|---|---|---|---|---|
| GAP-01 | {descripción} | {bloqueante / no bloqueante} | {abierto / en análisis / resuelto} | {sección afectada} |
| GAP-02 | {descripción} | {bloqueante / no bloqueante} | {abierto / en análisis / resuelto} | {sección afectada} |

---

## Dudas pendientes — Respuestas sugeridas

> Este bloque se presenta al final del DHU para que el usuario resuelva las dudas en el mismo chat.
> El skill no debe hacer preguntas interactivas durante el análisis.

```text
─────────────────────────────────────────────
DUDAS PENDIENTES — RESPUESTAS SUGERIDAS
─────────────────────────────────────────────

1. [GAP-01] {descripción corta del gap}
   ├─ Opción A (sugerida): {respuesta propuesta}
   ├─ Opción B: {respuesta alternativa}
   └─ Opción C: {otra alternativa}

2. [GAP-02] {descripción corta del gap}
   ├─ Opción A (sugerida): {respuesta propuesta}
   └─ Opción B: {respuesta alternativa}

Responde con el número y la opción, por ejemplo:
"1-A, 2-A"

O indica otra respuesta si ninguna opción aplica.
```

Si no hay gaps pendientes, reemplazar por:

```text
✅ No hay dudas pendientes. Esta DHU está lista para refinamiento final.
```
```
