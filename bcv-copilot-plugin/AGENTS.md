# AGENTS.md — BCV Copilot Plugin

Instrucciones para agentes IA que trabajan en este repositorio (y en los workspaces BCV donde se importa este plugin).

## Pipeline HU → DHU → implementación

1. `bcv-hu-context-analyzer` → `.context/hu-<code>.md`
2. `bcv-dhu-writer` → `hu-technical-refinement/HU-<identifier>-refined-<timestamp>.md`
3. `bcv-hu-implementer` → `hu-technical-refinement/HU-<identifier>-implementation-report-<timestamp>.md` (+ ramas feature en modo `apply`)

## Reglas obligatorias

1. **Prioridad a `graphify`** para exploración de código (ahorra tokens). Úsalo siempre que haya terminal disponible. Solo si no hay herramienta de terminal, usa exploración directa de archivos (`list_dir`/`read_file`/`grep`). Evita la radiografía LLM de repos completos.
2. Lee el `SKILL.md` de un skill antes de usarlo; aplica sus `references/`.
3. Solo backend: no generes gaps de frontend/UI. Mapea canales a APIs, eventos o DTOs.
4. No leas READMEs completos. Usa `service-map.md`, `architecture-conventions.md`, `cross-service-patterns.md` y `gotchas.md`.
5. No crees `.context/services/<servicio>.md`.
6. Código, clases, métodos, variables y paquetes en inglés.
7. Commits en inglés (conventional commits); antes de commit: linter + tests.
8. Nunca hardcodees secretos (Azure Key Vault o variables de entorno).

## Skills de este plugin

Los skills viven en `skills/`. Un agente puede usar otro skill leyendo su `SKILL.md` y `references/`:

- `bcv-hu-context-analyzer` — análisis de HU → contexto.
- `bcv-dhu-writer` — escritura de DHU.
- `bcv-hu-implementer` — implementación (lee y aplica skills de generación).
- `bcv-hexagonal-architecture` — slices hexagonales.
- `bcv-clean-architecture` — auditar/refactorizar a clean/hexagonal.
- `bcv-java-spring-boot` — setup Spring Boot.
- `bcv-openfeign` — clientes HTTP OpenFeign.
- `bcv-openapi-design` — contratos REST/OpenAPI.
- `bcv-azure-service-bus` — mensajería ASB.
- `bcv-spring-data-jpa-sql-server` — persistencia JPA + SQL Server.
- `bcv-cosmos-db` — persistencia Cosmos DB.
- `bcv-commons-observability` — observabilidad.
- `bcv-testing` — tests unitarios/integración.

## Cómo un agente usa otro skill

1. Descubre los skills disponibles (glob `find . -type f -name 'SKILL.md'` si hay terminal, o la carpeta `skills/` directamente).
2. Para cada tarea, selecciona el skill cuyo scope mejor encaje.
3. Lee su `SKILL.md` + `references/` y aplica sus convenciones al generar código.
4. Si el skill no está disponible, marca la tarea como `not available in Copilot Chat`.
