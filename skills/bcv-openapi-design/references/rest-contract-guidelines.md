# REST contract guidelines for BCV OpenAPI design

## Objective

Use this reference when designing or reviewing a REST contract for a BCV backend service.
Its purpose is to keep API contracts predictable, implementation-friendly and aligned with
Spring Boot services that may later use springdoc for runtime documentation.

## Design goals

A good BCV REST contract should be:

- Consumer-friendly: resource names and payloads are easy to understand.
- Stable: avoid unnecessary breaking changes.
- Explicit: document required fields, formats, enums and error responses.
- Implementable: the contract can be mapped cleanly to controllers, DTOs and validation.
- Testable: request and response examples are sufficient to derive mocks and contract tests.

## Contract-first preference

Prefer contract-first design when:

- `rest-api.yaml` is missing or empty.
- Controllers exist but are undocumented or inconsistent.
- Multiple teams depend on the API.
- External integrations require mockable contracts before implementation.

If implementation already exists, extract the contract from the real behavior instead of inventing a new API shape without justification.

## Resource naming rules

### Use nouns for resources

Prefer:
- `/documents`
- `/expedients`
- `/payment-promises`

Avoid:
- `/getDocuments`
- `/createExpedient`
- `/processPaymentPromise`

Use HTTP methods for actions:
- `GET /documents/{documentId}`
- `POST /documents`
- `PUT /documents/{documentId}`
- `PATCH /documents/{documentId}/status`

### Collection and item consistency

Keep collection and item endpoints consistent:

- `GET /documents`
- `POST /documents`
- `GET /documents/{documentId}`
- `PUT /documents/{documentId}`
- `DELETE /documents/{documentId}` only if deletion is a real business capability

### Action-style endpoints

Allow action endpoints only when the operation is not a simple CRUD update and models a business action:

- `POST /expedients/{expedientId}:approve`
- `POST /documents/{documentId}:reject`

If action endpoints are used, document:
- why the action is not modeled as a normal resource update
- request body fields
- resulting state transition
- business error conditions

## HTTP method guidance

- `GET`: read data, no side effects
- `POST`: create resources or trigger a business action
- `PUT`: full replacement when the representation is complete
- `PATCH`: partial update or state transition
- `DELETE`: remove only if the domain allows true deletion

Do not use `GET` for mutating actions.

## Status code guidance

Document at least the main success and error outcomes.

### Typical success responses

- `200 OK`: successful read or update
- `201 Created`: resource created
- `202 Accepted`: async processing started
- `204 No Content`: successful operation without body

### Typical client errors

- `400 Bad Request`: malformed request or validation failure
- `401 Unauthorized`: missing or invalid authentication
- `403 Forbidden`: authenticated but not allowed
- `404 Not Found`: resource does not exist
- `409 Conflict`: duplicate or invalid state transition
- `422 Unprocessable Entity`: semantically invalid request when the team uses this convention

### Typical server/integration errors

- `500 Internal Server Error`: unexpected failure
- `502 Bad Gateway`: upstream dependency failure
- `503 Service Unavailable`: temporary outage or dependency unavailable
- `504 Gateway Timeout`: upstream timeout

Document the actual project convention if it differs.

## Error contract guidance

Prefer one consistent error structure across the API.
A minimal error shape can include:

- `code`: stable machine-readable code
- `message`: human-readable summary
- `details`: optional field-level or contextual information
- `traceId`: correlation or trace identifier when available
- `timestamp`: optional event timestamp

Example:

```yaml
ErrorResponse:
  type: object
  required: [code, message]
  properties:
    code:
      type: string
      example: DOCUMENT_NOT_FOUND
    message:
      type: string
      example: Document 8f2a was not found
    details:
      type: array
      items:
        type: string
    traceId:
      type: string
```

## DTO design rules

### Request DTOs

- Include only fields the client must send.
- Mark `required` explicitly.
- Add `format`, `pattern`, `minLength`, `maxLength`, `minimum`, `maximum` when useful.
- Prefer business names already used in the domain.

### Response DTOs

- Return stable, consumer-oriented fields.
- Avoid leaking internal implementation details.
- Include identifiers and status fields explicitly when clients depend on them.

### Enumerations

Document enums explicitly with meaningful names and example values.
If the enum source is still unstable, call it out as a pending decision.

## Pagination, filtering and sorting

For collection endpoints, define pagination explicitly if relevant:

Query params:
- `page`
- `size`
- `sort`

If business filters exist, document them clearly:
- `status`
- `createdFrom`
- `createdTo`
- `customerId`

Do not add filters the service cannot realistically support.

## Idempotency and async operations

If a create or action endpoint may be retried by clients, document idempotency behavior.

If the operation is asynchronous:
- prefer `202 Accepted`
- describe the follow-up state or tracking mechanism
- explain whether the client receives an operation id, correlation id or status resource

## Versioning guidance

Prefer explicit versioning when the project uses it:

- `/api/v1/documents`
- `/v1/expedients`

Be consistent with the repository convention.
Do not introduce a new versioning scheme if one already exists.

## OpenAPI structure guidance

A production-ready OpenAPI file usually includes:

- `openapi`
- `info`
- `servers` when known
- `tags`
- `paths`
- `components.schemas`
- `components.responses` when shared responses are reused
- `components.parameters` when shared params are reused
- `security` and `components.securitySchemes` when authentication is relevant

## springdoc alignment hints

When the implementation is Spring Boot + springdoc:

- Keep operation summaries short and specific.
- Keep schema names aligned with DTO names when possible.
- Reflect validation constraints that will likely exist in Java annotations.
- Align response codes with controller annotations and exception handlers.

## Review checklist

Use this checklist before finalizing a contract:

- Resource names are nouns and are consistent.
- HTTP methods match semantics.
- Success and error responses are documented.
- DTOs have required fields and useful constraints.
- Versioning matches repository conventions.
- Action endpoints are justified.
- The contract is complete enough to generate mocks and implementation tasks.
