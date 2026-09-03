# bcv-cosmos-db

Skill BCV para diseñar, implementar, revisar y optimizar el uso de Azure Cosmos DB en servicios Java/Spring Boot.

## Cuándo usarlo

Usa este skill cuando el usuario mencione:

- Cosmos DB, Cosmos SDK o Spring Data Cosmos reactive.
- Persistencia de tracking, workflow, status history o correlation id.
- Partition key, RU/s, TTL, contenedores o consultas cross-partition.
- Integración Cosmos en servicios BCV, con o sin wrapper interno.

## Qué hace

- Produce o valida una especificación mínima de Cosmos.
- Recomienda modelo documental, particionamiento y consultas.
- Propone implementación para Spring Data Cosmos reactive o Cosmos SDK directo.
- Sugiere adapters fake o in-memory cuando Cosmos no está disponible.
- Entrega checklist de validación con foco en RU/s, 429, retries y timeouts.

## Entradas esperadas

- Historia de usuario o requerimiento.
- Contexto técnico del servicio.
- Restricciones de volumen, retención, latencia o multi-tenant.
- Código existente, si el usuario ya tiene una implementación.

## Salida esperada

- Spec: Cosmos Tracking.
- Diseño de partición y consultas.
- Snippets de configuración o código listos para adaptar.
- Validación y troubleshooting.

## Principios

- SDD primero, luego diseño, luego implementación.
- BMAD: Understand, Design, Build, Validate.
- Mocks first si Cosmos no está disponible.
- No asumir partition key ni TTL sin confirmación.
- No hardcodear secretos ni connection strings.
