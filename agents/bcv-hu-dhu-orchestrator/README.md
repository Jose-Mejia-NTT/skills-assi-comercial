# bcv-hu-dhu-orchestrator

Orquestador human-in-the-loop para el pipeline BCV **HU → DHU → implementación** en GitHub Copilot.

## Qué es

No es un skill que escriba código ni haga análisis técnico. Es un **agente conductor** que guía al usuario por las fases del pipeline en el orden correcto y valida gates entre ellas.

Como GitHub Copilot no soporta agentes autónomos que invoquen otros skills, este orquestador funciona como un **playbook interactivo**: le indica al usuario qué skill ejecutar, qué inputs dar y qué gate validar a continuación.

## Arquitectura

1 agente + 3 skills + 1 prompt por fase:

| Fase | Skill (capacidad especializada) | Prompt | Artefacto | Gate |
|---|---|---|---|---|
| Fase 1 — Contexto técnico | `bcv-hu-context-analyzer` | Prompt Fase 1 | `.context/hu-<code>.md` | Gate 0 |
| Fase 2 — Historia técnica (DHU) | `bcv-dhu-writer` | Prompt Fase 2 | `hu-technical-refinement/HU-<identifier>-refined-<timestamp>.md` | Gate 1 |
| Fase 3 — Implementación | `bcv-hu-implementer` | Prompt Fase 3 | `hu-technical-refinement/HU-<identifier>-implementation-report-<timestamp>.md` | Gate 2 |

El agente **produce prompts; no ejecuta las skills directamente**. El usuario corre cada skill con su prompt de fase.

## Fases del pipeline

1. **Fase 1 — Contexto técnico:** `bcv-hu-context-analyzer` → `.context/hu-<code>.md`
2. **Fase 2 — Historia técnica (DHU):** `bcv-dhu-writer` → `hu-technical-refinement/HU-<identifier>-refined-<timestamp>.md`
3. **Fase 3 — Implementación:** `bcv-hu-implementer` → `hu-technical-refinement/HU-<identifier>-implementation-report-<timestamp>.md` (+ ramas feature en modo `apply`)

## Archivo de estado

El orquestador mantiene:

```text
docs/historial/<hu-slug>-pipeline-state.yaml
```

Este archivo registra la fase actual, el estado de los gates y los bloqueadores para poder reanudar la sesión.

## Gates

| Gate | Fase | Validaciones |
|---|---|---|
| Gate 0 | Contexto técnico | Contexto existe, servicios clasificados, punto de inyección, gaps con tipo/blocking/sugerencia. |
| Gate 1 | DHU | Sin gaps bloqueantes (o `EN ELABORACIÓN`), ≥3 CAs, endpoints con errores, mapa técnico no vacío, DoR. |
| Gate 2 | Implementación | Reporte único con pre-validación superada y columna `Skill` por tarea. |

## Uso

1. Proporciona el texto de la HU funcional y la ruta del workspace.
2. El orquestador inicializa el archivo de estado y te da el **Prompt Fase 1**.
3. Después de cada fase, confirma la ruta del artefacto (o el orquestador la lee directamente).
4. El orquestador valida el gate y se detiene con bloqueadores/gaps o continúa a la siguiente fase.

## Modos

- **Mode A — Fast path:** los artefactos ya existen → valida gates y produce el resumen final.
- **Mode B — Por fases:** faltan artefactos → presenta **un prompt por fase** (contexto → DHU → implementación), validando el gate antes de pasar a la siguiente. Nunca corre todo el pipeline de una vez.
- **Mode C — Clarification:** gaps bloqueantes en la DHU → presenta una duda a la vez y espera respuestas.

## Política de idioma

- El agente detecta el idioma del usuario al inicio y responde **siempre en el idioma del usuario**.
- Si no puede detectarlo con confianza, usa inglés por defecto.
- El procesamiento interno y la estructura técnica pueden hacerse en inglés, sin que se filtren al output visible.
- **Código fuente**, nombres de clases/métodos/variables/paquetes, campos OpenAPI/JSON y construcciones HTTP van en inglés.
- Los **commits de Git** van en inglés salvo que el usuario pida lo contrario.
- Los nombres de skills (ej: `bcv-hu-context-analyzer`) y las rutas de archivo se mantienen tal cual.

## Archivos

| Archivo | Propósito | ¿Lo lee Copilot? |
|---|---|---|
| `AGENT.md` | Instrucciones principales de orquestación. **Autocontenido** (la plantilla YAML está inline). | ✅ Sí |
| `README.md` | Documentación para humanos. | ❌ No |

> **Importante:** GitHub Copilot Chat solo lee el `AGENT.md`. Todo lo que el agente necesite (plantillas, gates, prompts) debe estar **inline** dentro de `AGENT.md`. El `README.md` es documentación para personas, no para Copilot.
