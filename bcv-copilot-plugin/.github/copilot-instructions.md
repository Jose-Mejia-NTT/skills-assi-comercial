# Instrucciones para Copilot — Plugin BCV (skills + agentes)

## Qué es este plugin

Este repositorio es un **plugin importable de GitHub Copilot** para el ecosistema BCV (Interbank). Contiene:

- **Skills** en `skills/` — capacidades especializadas (cada una con `SKILL.md`).
- **Agentes** en `.github/agents/` — agentes personalizados que orquestan skills.
- **Instrucciones** en `AGENTS.md` — reglas para agentes IA.

## Reglas generales

1. **Usa los skills de `skills/`** para tareas especializadas. No reimplementes su lógica a mano.
2. **No radiografíes repositorios con LLM.** Usa `graphify` para análisis de código.
3. **Solo backend.** No generes frontend/UI/pantallas. Mapea canales (ej. "canal BCW") a APIs, eventos o DTOs de backend.
4. **Nunca hardcodees secretos** (connection strings, tokens, webhooks, SAS keys). Usa Azure Key Vault o variables de entorno.
5. **Código, clases, métodos, variables y paquetes en inglés.** Commits en inglés (conventional commits).
6. **Respeta el idioma del usuario** en toda respuesta visible. Procesamiento interno en inglés.

## Skills disponibles (13)

| Skill | Cuándo usarlo |
|---|---|
| `bcv-hu-context-analyzer` | Analizar una HU funcional y generar contexto técnico (`.context/hu-<code>.md`). |
| `bcv-dhu-writer` | Escribir la DHU técnica a partir del contexto. |
| `bcv-hu-implementer` | Aplicar la DHU en código (reporte dry-run + ramas feature en apply). |
| `bcv-hexagonal-architecture` | Generar slice hexagonal (controllers, use cases, puertos, mappers). |
| `bcv-clean-architecture` | Auditar/refactorizar un servicio hacia clean/hexagonal (violaciones de dependencia, migración). |
| `bcv-java-spring-boot` | Setup Spring Boot (ADS BOM, módulos, profiles, Key Vault). |
| `bcv-openfeign` | Clientes HTTP OpenFeign (FeignClient, FeignConfig, headers, error handling). |
| `bcv-openapi-design` | Contratos REST/OpenAPI, DTOs, errores RFC 9457. |
| `bcv-azure-service-bus` | Mensajería ASB (publishers, subscribers, topics, DLQ). |
| `bcv-spring-data-jpa-sql-server` | Persistencia JPA + SQL Server (entidades, repos, migraciones). |
| `bcv-cosmos-db` | Persistencia Cosmos DB. |
| `bcv-commons-observability` | Observabilidad (trazas, métricas, alertas, data masking). |
| `bcv-testing` | Tests unitarios/integración (JUnit 5, Mockito, AssertJ, @DataJpaTest, JaCoCo). |

## Cómo usar un skill

Para ejecutar un skill, referencia su nombre (el nombre de la carpeta en `skills/`). Por ejemplo:

- "Usa `bcv-hu-context-analyzer` para analizar esta HU."
- "Genera la DHU con `bcv-dhu-writer`."

Los skills leen su propio `SKILL.md` y sus `references/` para aplicar sus convenciones.

## Agente orquestador

El agente `bcv-hu-dhu-orchestrator` (en `.github/agents/`) orquesta el flujo HU → DHU → implementación por fases. Para el flujo completo, activa ese agente.
