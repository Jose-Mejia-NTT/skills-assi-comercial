# bcv-copilot-plugin

Plugin importable de GitHub Copilot para el ecosistema BCV (Interbank). Agrupa skills, agentes personalizados e instrucciones para el flujo HU → DHU → implementación.

## Estructura

```text
bcv-copilot-plugin/
├── .github/
│   ├── copilot-instructions.md          ← system prompt (instrucciones a nivel repo)
│   ├── instructions/                    ← instrucciones por ruta (opcional)
│   └── agents/
│       └── bcv-hu-dhu-orchestrator.agent.md   ← agente personalizado (frontmatter + prompt)
├── skills/                              ← skills reutilizables (13)
│   ├── bcv-hu-context-analyzer/SKILL.md
│   ├── bcv-dhu-writer/SKILL.md
│   ├── bcv-hu-implementer/SKILL.md
│   ├── bcv-hexagonal-architecture/SKILL.md
│   ├── bcv-clean-architecture/SKILL.md
│   ├── bcv-java-spring-boot/SKILL.md
│   ├── bcv-openfeign/SKILL.md
│   ├── bcv-openapi-design/SKILL.md
│   ├── bcv-azure-service-bus/SKILL.md
│   ├── bcv-spring-data-jpa-sql-server/SKILL.md
│   ├── bcv-cosmos-db/SKILL.md
│   ├── bcv-commons-observability/SKILL.md
│   └── bcv-testing/SKILL.md
├── AGENTS.md                            ← instrucciones para agentes IA
└── README.md                            ← este archivo
```

## Mapeo de conceptos

| Concepto que pediste | Ubicación real en Copilot                               |
| -------------------- | ------------------------------------------------------- |
| System prompt        | `.github/copilot-instructions.md`                       |
| Skills               | `skills/` (cada uno con `SKILL.md`)                     |
| Instrucciones        | `AGENTS.md` + `.github/instructions/*.instructions.md`  |
| Agente               | `.github/agents/*.agent.md` (frontmatter YAML + prompt) |

## Cómo importarlo

### Como carpeta en tu workspace

1. Copia (o clona) la carpeta `bcv-copilot-plugin/` dentro de tu proyecto/workspace BCV.
2. GitHub Copilot detectará automáticamente:
   - `.github/copilot-instructions.md` (instrucciones globales).
   - `.github/agents/*.agent.md` (agentes personalizados).
   - `skills/` (skills reutilizables).
   - `AGENTS.md` (instrucciones para agentes).

### Agentes personalizados

En VS Code / JetBrains, los agentes aparecen en el dropdown de agentes de Copilot Chat (`.github/agents/`). El agente `bcv-hu-dhu-orchestrator` orquesta el flujo por fases.

## Cómo el agente usa otros skills

1. El agente (`bcv-hu-dhu-orchestrator`) descubre los skills en `skills/`.
2. Para cada tarea, referencia el skill adecuado por nombre.
3. El skill lee su `SKILL.md` + `references/` y aplica sus convenciones.

El `bcv-hu-implementer`, al generar código, **lee y aplica** los skills de generación (`bcv-hexagonal-architecture`, `bcv-spring-data-jpa-sql-server`, etc.), no solo los lista.

## Skills incluidos (13)

| Skill                            | Propósito                              |
| -------------------------------- | -------------------------------------- |
| `bcv-hu-context-analyzer`        | HU funcional → contexto técnico        |
| `bcv-dhu-writer`                 | contexto → DHU técnica                 |
| `bcv-hu-implementer`             | DHU → código (dry-run/apply)           |
| `bcv-hexagonal-architecture`     | slices hexagonales                     |
| `bcv-clean-architecture`         | auditar/refactorizar a clean/hexagonal |
| `bcv-java-spring-boot`           | setup Spring Boot                      |
| `bcv-openfeign`                  | clientes HTTP OpenFeign                |
| `bcv-openapi-design`             | contratos REST/OpenAPI                 |
| `bcv-azure-service-bus`          | mensajería ASB                         |
| `bcv-spring-data-jpa-sql-server` | persistencia JPA + SQL Server          |
| `bcv-cosmos-db`                  | persistencia Cosmos DB                 |
| `bcv-commons-observability`      | observabilidad                         |
| `bcv-testing`                    | tests unitarios/integración            |
