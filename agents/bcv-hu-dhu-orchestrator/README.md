# bcv-hu-dhu-orchestrator

Orquestador human-in-the-loop para el pipeline BCV **HU → DHU → implementación** en GitHub Copilot.

## Qué es

No es un skill que escriba código ni haga análisis técnico. Es un **agente conductor** que guía al usuario por los tres skills BCV en el orden correcto y valida gates entre ellos.

Como GitHub Copilot no soporta agentes autónomos que invoquen otros skills, este orquestador funciona como un **playbook interactivo**: le indica al usuario qué skill ejecutar, qué inputs dar y qué gate validar a continuación.

## Pasos del pipeline

1. `bcv-hu-context-analyzer` → `.context/hu-<code>.md`
2. `bcv-dhu-writer` → `hu-technical-refinement/HU-<identifier>-refined-<timestamp>.md`
3. `bcv-hu-implementer` → `hu-technical-refinement/HU-<identifier>-implementation-report-<timestamp>.md` (+ ramas feature en modo `apply`)

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
2. El orquestador inicializa el archivo de estado y te indica ejecutar el Paso 1.
3. Después de cada paso, confirma la ruta del artefacto (o el orquestador la lee directamente).
4. El orquestador valida el gate y se detiene con bloqueadores/gaps o continúa al siguiente paso.

## Modos

- **Mode A — Fast path:** los artefactos ya existen → valida gates y produce el resumen final.
- **Mode B — One-shot:** faltan artefactos → genera un único prompt compuesto para correr todo el pipeline.
- **Mode C — Clarification:** gaps bloqueantes en la DHU → presenta una duda a la vez y espera respuestas.

## Política de idioma

- El agente detecta el idioma del usuario al inicio y responde **siempre en el idioma del usuario**.
- Si no puede detectarlo con confianza, usa inglés por defecto.
- El procesamiento interno y la estructura técnica pueden hacerse en inglés, sin que se filtren al output visible.
- **Código fuente**, nombres de clases/métodos/variables/paquetes, campos OpenAPI/JSON y construcciones HTTP van en inglés.
- Los **commits de Git** van en inglés salvo que el usuario pida lo contrario.
- Los nombres de skills (ej: `bcv-hu-context-analyzer`) y las rutas de archivo se mantienen tal cual.

## Archivos

| Archivo | Propósito |
|---|---|
| `AGENT.md` | Instrucciones principales de orquestación. |
| `assets/pipeline-state.template.yaml` | Plantilla del archivo de estado. |
