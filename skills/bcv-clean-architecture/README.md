# bcv-clean-architecture

Skill BCV para auditar, revisar y refactorizar servicios Java/Spring Boot hacia arquitectura
limpia/hexagonal, y para migrar servicios legacy de estructura plana.

## Cuándo usarlo

Usa este skill cuando el usuario pida:

- Auditar la arquitectura de un servicio BCV.
- Revisar o corregir violaciones de dependencias entre capas (`core -> output`, `input -> output`).
- Migrar un servicio plano/legacy (ej. `bcv-bacc-service-point-service`) a hexagonal.
- Extraer ports, mover adapters o dividir módulos `-core/-input/-output/-app`.
- Encontrar fugas de infraestructura en el dominio (JPA/Spring en `core`).

## Qué hace

- Clasifica el servicio (hexagonal, layered o flat).
- Genera un reporte de violaciones de reglas de dependencia.
- Produce un plan de migración ordenado por riesgo.
- Aplica refactors mínimos que preservan el comportamiento.
- Genera la estructura de módulos y el cableado de `pom.xml` en migraciones.

## Entradas esperadas

- Ruta del repositorio o workspace.
- Objetivo: `audit`, `refactor` o `migrate`.
- Opcional: `GRAPH_REPORT.md` o un contexto acotado a enfocar.

## Salida esperada

- Clasificación de arquitectura + reporte de violaciones.
- Plan de refactor/migración.
- Diffs de movimiento de clases y fixes de imports.
- Checklist de verificación (build + tests + reglas).

## Principios

- SDD: especificar el refactor antes de aplicarlo.
- BMAD: Understand, Design, Build, Validate.
- Preservar comportamiento: solo estructura e imports, no lógica.
- Reglas de dependencia: `input -> core`, `output -> core`, `app -> all`; prohibido
  `core -> input/output` y `input -> output`.
- Migraciones incrementales: build y tests después de cada paso.

## Relación con otros skills

- `bcv-hexagonal-architecture`: genera un slice nuevo; este skill audita/refactoriza/migra existentes.
- `bcv-openapi-design`: contratos REST.
- `bcv-azure-service-bus`: cambios de mensajería.
