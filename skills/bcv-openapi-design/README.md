# bcv-openapi-design

Crea y mantiene **contratos REST / OpenAPI** para proyectos del ecosistema BCV, incluyendo DTOs, respuestas de error estándar (RFC 9457), y alineación con SpringDoc.

## ¿Qué cubre?

Este skill se enfoca en:

- **Contrato OpenAPI 3.x** — definición completa de endpoints, parámetros, respuestas.
- **DTOs (Data Transfer Objects)** — request/response con validaciones (`@NotNull`, `@Size`, etc.).
- **Respuestas de error** — formato RFC 9457, códigos HTTP estándar BCV.
- **Alineación con SpringDoc** — integración con `springdoc-openapi`, generación automática desde anotaciones.
- **Documentación de seguridad** — autenticación, headers requeridos.
- **Validación de consistencia** — coherencia entre OpenAPI y código Java.

## ¿Cuándo usarlo?

- Diseñar un nuevo endpoint o API completa.
- Completar o validar un contrato OpenAPI existente.
- Estandarizar respuestas de error.
- Documentar DTOs y validaciones.

## ¿Cuándo NO usarlo?

- Para implementación de la lógica de negocio.
- Para generación de tests (use `bcv-testing`).
- Para configuración de infraestructura.

## Skills relacionados

- `bcv-hexagonal-architecture` — para alinear el contrato con los controladores del slice.
- `bcv-testing` — para generar tests basados en el contrato.

## Información requerida

El skill necesita:

1. **Descripción del endpoint o API** — método, recurso, parámetros, respuesta.
2. **Modelo de datos (DTOs)** — si ya existen, o descripción del dominio.
3. **Posibles códigos de error** — excepciones o validaciones esperadas.

## Archivos principales

- `SKILL.md` — instrucciones completas y flujo de trabajo.
- `references/` — templates OpenAPI, DTOs de ejemplo, patrones de error RFC 9457.
