# Particionamiento, queries y RU/s para tracking en Cosmos DB

## Propósito

Este documento guía las decisiones de diseño para persistencia de tracking/workflow en Azure Cosmos DB dentro de BCV, con énfasis en:

- elección de `partitionKey`
- consultas principales
- ordenamiento por fechas
- control de costo en **RU/s**
- reducción de queries cross-partition

---

## Regla base del skill

El skill **no debe asumir** una `partitionKey` obligatoria ni TTL si el usuario no lo confirma.

En su lugar, debe:

1. preguntar por las consultas principales
2. preguntar por volumen esperado
3. preguntar si hay regla corporativa de particionamiento
4. proponer opciones con trade-offs

---

## Patrones de acceso detectados

A partir del contexto BCV, los accesos más probables son:

1. búsquedas por identificadores de tracking
2. búsquedas por estados de workflow
3. orden por fecha de registro
4. búsquedas por identificadores de negocio relacionados
   - expediente
   - cliente
   - correlación
   - operación

El skill debe validar cuáles de estos son realmente prioritarios.

---

## Checklist para diseñar la partition key

Antes de recomendar una `partitionKey`, responder:

- ¿Cuál es la consulta más frecuente?
- ¿Cuál es la consulta más costosa si cae cross-partition?
- ¿Los datos se consultan por una clave de negocio estable?
- ¿Existe riesgo de hotspot?
- ¿El volumen por partición puede crecer de forma desbalanceada?
- ¿Se requiere orden por fecha dentro de una misma partición?
- ¿Habrá multi-tenant?
- ¿Hay procesos batch o replays masivos?

---

## Candidatos típicos de partition key

### Opción 1 — `/trackingId` o `/correlationId`

### Cuándo puede servir

- cuando el tracking se consulta principalmente por identificador único
- cuando las operaciones son lookups puntuales o actualizaciones sobre un mismo flujo
- cuando se quiere agrupar todos los eventos de un mismo proceso

### Ventajas

- muy buen acceso puntual
- simplifica recuperación de historial del flujo
- reduce cross-partition para lectura por correlación

### Riesgos

- pobre soporte para consultas globales por estado
- puede requerir queries costosas para análisis transversales
- si una correlación concentra demasiados eventos, puede tensionar una sola partición lógica

---

### Opción 2 — `/expedientId` o identificador de negocio principal

### Cuándo puede servir

- cuando el expediente es la clave natural del dominio
- cuando el negocio consulta tracking principalmente por expediente
- cuando múltiples eventos del proceso deben verse juntos por entidad de negocio

### Ventajas

- alinea persistencia con lenguaje del dominio
- útil para auditoría por expediente
- reduce joins mentales entre tracking y negocio

### Riesgos

- no siempre optimiza consultas por estado global
- si algunos expedientes concentran mucho tráfico, puede haber skew

---

### Opción 3 — `/workflowStatus`

### Cuándo puede servir

- rara vez como primera opción
- solo cuando casi todas las consultas están agrupadas por estado y el volumen por estado es estable

### Riesgos importantes

- alto riesgo de particiones muy calientes
- desbalance si ciertos estados concentran la mayoría del tráfico
- mala opción para reconstruir historial por flujo individual

> El skill debe ser conservador y no recomendar esta opción como default.

---

### Opción 4 — clave compuesta lógica derivada

Ejemplos conceptuales:

- `businessId#period`
- `correlationId#month`
- `tenant#businessId`

### Cuándo puede servir

- cuando hay alto volumen
- cuando se necesita controlar distribución
- cuando existe patrón temporal o multi-tenant claro

### Ventajas

- mejor balance potencial
- permite controlar crecimiento por partición lógica
- útil para tracking con gran volumen

### Riesgos

- más complejidad operativa
- mayor carga cognitiva para queries y soporte
- requiere disciplina de diseño y naming

---

## Consultas por estado + fecha

El contexto BCV sugiere este patrón como relevante:

- búsqueda por `workflowStatus`
- orden o filtro por fecha de registro

### Recomendaciones del skill

- modelar un campo temporal consistente, por ejemplo:
  - `registeredAt`
  - `statusChangedAt`
- distinguir entre:
  - fecha de creación del documento
  - fecha del evento de negocio
  - fecha del último cambio
- evitar diseñar queries globales por estado que barran todo el contenedor si el volumen puede crecer

### Señal de alerta

Si el usuario necesita:

- “dame todos los tracking en estado X ordenados por fecha”
- sobre un volumen alto
- y sin una partición alineada a ese acceso,

el skill debe advertir explícitamente sobre costo y escalabilidad.

---

## Estrategia de RU/s

## Qué debe evaluar el skill

Para cualquier propuesta, el skill debe revisar:

- cantidad de lecturas esperadas
- proporción lecturas/escrituras
- tamaño estimado del documento
- frecuencia de actualizaciones
- queries con filtros múltiples
- ordenamientos
- crecimiento histórico del tracking
- necesidad de TTL

---

## Recomendaciones de diseño para reducir RU/s

- preferir **point reads** cuando sea posible
- alinear `partitionKey` con accesos críticos
- evitar queries cross-partition innecesarias
- reducir documentos excesivamente grandes
- evitar almacenar información duplicada que no aporte valor
- separar tracking operacional de analítica, si el caso lo exige
- revisar si conviene almacenar eventos o snapshots según el patrón del negocio

---

## Eventos vs snapshot

El skill debe evaluar cuál modelo conviene más:

| Modelo | Cuándo conviene | Riesgos |
|--------|------------------|---------|
| **Event log** | Cuando se necesita historial completo y auditoría detallada | Más lecturas para reconstruir estado |
| **Snapshot actualizable** | Cuando importa más el estado actual y consultas rápidas | Menor trazabilidad histórica |
| **Híbrido** | Cuando se necesita estado actual + historia acotada | Más complejidad de consistencia |

Para tracking BCV, el skill debe considerar **híbrido** como opción frecuente si el negocio necesita estado actual y trazabilidad.

---

## TTL y retención

Si el usuario no lo confirma, el skill debe preguntar:

- ¿El tracking expira?
- ¿Se debe conservar por días, meses o años?
- ¿Hay requerimientos regulatorios?

### Recomendación

- no activar TTL automáticamente
- explicar que TTL impacta retención y costo
- si el tracking es puramente operativo, TTL puede ser una opción
- si hay auditoría/regulación, validar antes de proponer expiración

---

## Señales para escalar el diseño

El skill debe elevar advertencias cuando detecte:

- volumen no confirmado pero queries globales pesadas
- consultas frecuentes por estado sobre muchas particiones
- necesidad de ordenamientos amplios sin estrategia clara
- mezcla de tracking operacional y reporting analítico
- riesgo de hotspot por una clave demasiado concentrada

---

## Formato recomendado de decisión en respuestas del skill

Cuando proponga una `partitionKey`, usar una tabla como esta:

| Opción | Favorece | Riesgos | Veredicto |
|-------|----------|---------|-----------|
| `/correlationId` | historial por flujo | consultas globales por estado caras | recomendada si el acceso principal es por flujo |
| `/expedientId` | consultas por expediente | skew por entidades calientes | recomendada si el expediente es la clave dominante |
| `/workflowStatus` | filtro directo por estado | hotspots, mala distribución | no recomendada como default |
