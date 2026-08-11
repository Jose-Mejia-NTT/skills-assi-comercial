# Modelos de tracking y estrategia mocks-first para Cosmos DB

## Propósito

Este documento ayuda al skill `bcv-cosmos-db` a decidir:

- cómo modelar documentos de tracking/workflow
- cuándo usar event log, snapshot o híbrido
- cómo aplicar **mocks-first** cuando Cosmos no está disponible
- qué artefactos producir para implementación y review

---

## Regla BCV para este skill

El objetivo principal no es persistencia documental genérica, sino **tracking operacional y de workflow**.

Eso implica que el skill debe favorecer modelos alineados a:

- identificadores de tracking
- estados de workflow
- fechas de registro/cambio
- identificadores de negocio relacionados
- auditoría mínima
- troubleshooting y trazabilidad

---

## Preguntas obligatorias de modelado

Antes de proponer el documento final, el skill debe aclarar:

1. ¿Se necesita historial completo de cada cambio?
2. ¿Importa más consultar el **estado actual** o la **traza histórica**?
3. ¿Se consulta por expediente, correlación, cliente u otro identificador?
4. ¿Se necesita ver el tracking ordenado por fecha?
5. ¿Hay retención limitada o auditoría prolongada?
6. ¿El tracking será consumido solo por el servicio o también por soporte/operaciones?

---

## Opción 1 — Event log

## Descripción

Cada cambio relevante del workflow se guarda como un evento independiente.

### Cuándo conviene

- cuando se requiere trazabilidad detallada
- cuando auditoría y reconstrucción histórica son importantes
- cuando el proceso tiene múltiples transiciones que deben preservarse

### Ejemplo conceptual de campos

- `id`
- `trackingId`
- `correlationId`
- `businessId`
- `workflowStatus`
- `eventType`
- `registeredAt`
- `payload`
- `createdBy`
- `schemaVersion`

### Ventajas

- historial completo
- alta auditabilidad
- más flexibilidad para reconstruir secuencias

### Riesgos

- más lecturas para conocer el estado actual
- mayor costo si se consulta constantemente “último estado”
- más complejidad para agregación

---

## Opción 2 — Snapshot actualizable

## Descripción

Un documento representa el estado actual del tracking y se actualiza con cada cambio.

### Cuándo conviene

- cuando el acceso principal es al estado actual
- cuando importa la velocidad de lectura
- cuando el historial fino no es obligatorio

### Ejemplo conceptual de campos

- `id`
- `trackingId`
- `correlationId`
- `businessId`
- `currentStatus`
- `registeredAt`
- `lastUpdatedAt`
- `attributes`
- `schemaVersion`

### Ventajas

- lectura simple y rápida
- menor esfuerzo para consultas del estado vigente
- menor cantidad de documentos

### Riesgos

- pérdida de historial detallado si no se complementa
- menor auditabilidad
- riesgo de sobreescritura si no se controla concurrencia lógica

---

## Opción 3 — Híbrido

## Descripción

Se conserva:

- un documento snapshot con el estado actual
- y eventos separados para la historia relevante

### Cuándo conviene

- cuando el negocio necesita estado actual y trazabilidad
- cuando soporte u operaciones necesitan investigar secuencias
- cuando el tracking tiene valor operativo y forense

### Ventajas

- balance entre lectura rápida e historial
- mejor soporte a troubleshooting
- más alineado con escenarios de tracking complejos

### Riesgos

- mayor complejidad de consistencia
- más decisiones de diseño
- potencial duplicación controlada de datos

### Recomendación general del skill

Para tracking BCV, el skill puede considerar el modelo **híbrido** como candidato frecuente, pero debe justificarlo según el caso.

---

## Auditoría mínima recomendada

Si no existe otro estándar explícito, el skill debe sugerir campos conceptuales como:

- `createdAt`
- `updatedAt`
- `createdBy`
- `updatedBy`
- `schemaVersion`
- `sourceSystem`
- `traceId` o `correlationId`

No debe imponer nombres exactos si el servicio ya tiene convenciones propias.

---

## Versionado del documento

El skill debe recomendar versionado cuando:

- el documento puede evolucionar
- hay más de un consumidor
- existe riesgo de cambios de estructura entre releases

### Sugerencia conceptual

- incluir `schemaVersion`
- evitar cambios destructivos sin compatibilidad
- documentar campos obligatorios y opcionales

---

## Estrategia mocks-first

Si Cosmos no está disponible o no hay acceso a credenciales/entorno:

1. definir una interfaz/puerto de persistencia
2. crear un adapter fake o in-memory
3. validar contratos funcionales
4. recién después preparar la integración real

Esto sigue el lineamiento general:

> dependencias externas = mock primero

---

## Qué debe producir el skill en modo mock

Cuando el usuario pide implementación pero no hay Cosmos disponible, el skill debe generar:

- interfaz del repositorio/adapter
- implementación fake/in-memory
- fixtures de documentos de tracking
- pruebas de contrato simples
- guía de reemplazo por adapter real

---

## Reglas para el fake/in-memory

El fake debe:

- imitar los casos principales del adapter real
- soportar búsquedas por identificador de tracking
- soportar filtros por estado si eso es parte del caso
- conservar orden temporal si la lógica lo requiere
- no inventar comportamiento no soportado por el contrato real

---

## Qué validar antes de pasar de mock a Cosmos real

- modelo documental confirmado
- consultas prioritarias confirmadas
- decisión de partition key documentada
- decisión de TTL documentada
- política de retries/timeout definida
- manejo de errores 429 y disponibilidad contemplado

---

## Ejemplo de decisión que debería producir el skill

### Caso

“Necesito guardar tracking de un flujo H2H y consultar por correlación, estado y fecha.”

### Respuesta esperada del skill

- identificar si el acceso principal es por correlación o por expediente
- decidir si conviene snapshot, eventos o híbrido
- advertir costo potencial de consultas por estado global
- si no hay Cosmos disponible, generar adapter fake primero
- solo después proponer adapter real con Spring Data Cosmos o SDK directo
