# bcv-hexagonal-architecture

Genera y mantiene **slices verticales completos** de arquitectura hexagonal (o ports-and-adapters) para proyectos Java del ecosistema BCV.

## ¿Qué cubre?

Este skill se enfoca en:

- **Slice vertical completo** para un nuevo caso de uso: core (puertos, casos de uso), input (controladores, DTOs, mappers), output (adaptadores).
- **Convenciones de paquetes BCV** — carpetas `core`, `in`, `out`, `app`, estructura modular.
- **Inyección de dependencias** — beans Spring, configuración en `*Config`, wiring de puertos.
- **Integración con arquitectura hexagonal** — aislamiento de dominio, inversión de dependencias.
- **Validación de consistencia arquitectónica** — no hay dependencias hacia atrás (out → core, in → core, in → out).

## ¿Cuándo usarlo?

- Agregar un nuevo caso de uso o endpoint a un servicio.
- Generar un slice vertical completo (controlador → use case → puerto → adaptador).
- Refactorizar código hacia hexagonal.
- Validar que la arquitectura cumple con las reglas BCV.

## ¿Cuándo NO usarlo?

- Para diseño de contratos OpenAPI (use `bcv-openapi-design`).
- Para generación de tests (use `bcv-testing`).
- Para cambios puramente de infraestructura.

## Skills relacionados

- `bcv-openapi-design` — para definir el contrato REST antes del slice.
- `bcv-spring-data-jpa-sql-server` — para adaptadores de persistencia.
- `bcv-azure-service-bus` — para adaptadores de mensajería.
- `bcv-testing` — para agregar tests al slice después de generado.

## Información requerida

El skill necesita:

1. **Descripción del caso de uso** — en lenguaje natural o criterios de aceptación.
2. **Tipo de caso de uso** — API REST, consumidor de mensaje, consulta, etc.
3. **Persistencia o integración requerida** — JPA, Cosmos, ASB, OpenFeign, etc.

## Archivos principales

- `SKILL.md` — instrucciones completas y flujo de trabajo.
- `references/` — checklist de alineación, ejemplos de slices, patrones de configuración.
