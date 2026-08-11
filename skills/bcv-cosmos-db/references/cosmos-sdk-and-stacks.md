# Cosmos stacks soportados en BCV

## Propósito

Este documento describe los dos patrones detectados en BCV para integrar Azure Cosmos DB:

1. **Spring Data Cosmos reactivo** con wrapper interno común
2. **Azure Cosmos Java SDK directo** con configuración/registry propio

El skill `bcv-cosmos-db` debe soportar ambos y elegir según el servicio, la base instalada y la necesidad del cambio.

---

## Stack A — Spring Data Cosmos reactivo

### Cuándo preferirlo

Usar este enfoque cuando el servicio:

- ya está construido sobre Spring Boot reactivo
- ya usa `Spring Data Cosmos`
- ya tiene wrappers/adapters internos basados en abstracciones comunes
- requiere rapidez para CRUD, consultas derivadas o integración idiomática con Spring

### Ventajas

- integración natural con Spring
- repositorios y abstracciones conocidas por equipos Java/Spring
- buen fit para flujos reactivos
- menor boilerplate en escenarios simples

### Riesgos

- esconder detalles importantes de performance si el equipo no revisa particionamiento y queries
- abstraer demasiado la configuración de Cosmos
- inducir a pensar que Cosmos funciona igual que una base relacional o JPA

### Recomendaciones del skill

- validar siempre la `partitionKey` antes de proponer el modelo final
- revisar qué consultas hace realmente el negocio
- no asumir que los repositorios derivados son suficientes para queries de tracking complejas
- encapsular acceso a Cosmos detrás de puertos/adapters si el proyecto usa hexagonal o clean architecture
- usar el wrapper interno `bcv-commons-cosmos` cuando el servicio ya esté alineado a ese patrón

---

## Stack B — Azure Cosmos Java SDK directo

### Cuándo preferirlo

Usar este enfoque cuando el servicio:

- ya usa el SDK oficial directamente
- necesita control fino sobre clientes, contenedores, queries y diagnósticos
- requiere tuning de performance, retries, timeouts o medición de RU/s
- tiene una capa de persistencia propia o un registry/config centralizado

### Ventajas

- mayor control técnico
- mejor visibilidad para troubleshooting
- mayor precisión al diseñar queries, options y diagnósticos
- mejor ajuste para casos donde Cosmos es parte crítica del flujo

### Riesgos

- más boilerplate
- más posibilidad de inconsistencias entre servicios si no existe estándar común
- mayor curva de aprendizaje para el equipo

### Recomendaciones del skill

- estandarizar creación/configuración de clientes
- centralizar retries, timeouts y manejo de 429
- unificar criterios de serialización y versionado documental
- revisar siempre el costo en RU/s de las queries relevantes
- generar adapters claros para que el acceso a Cosmos no quede disperso

---

## Wrapper interno `bcv-commons-cosmos`

El skill debe tratar `bcv-commons-cosmos` como una **abstracción común de persistencia/infraestructura**, sin depender de un repo específico.

### Capacidades esperadas

Puede proveer una o varias de estas capacidades:

- creación/configuración de clientes Cosmos
- helpers para acceso a contenedores
- abstracciones de repositorio
- políticas de retry/timeout
- logging y telemetry
- serialización común
- helpers de queries
- convenciones de auditoría o metadatos

### Regla para el skill

- si el servicio ya usa `bcv-commons-cosmos`, el skill debe respetarlo y construir encima
- si no está disponible en el contexto entregado, el skill puede describir cómo integrarlo sin asumir detalles internos no confirmados
- no inventar APIs concretas de la librería si el usuario no las comparte

---

## Criterio de selección entre stacks

El skill debe evaluar esta tabla antes de proponer implementación:

| Criterio | Spring Data Cosmos reactivo | SDK directo |
|---------|------------------------------|-------------|
| Servicio ya usa Spring Data Cosmos | Sí | No necesariamente |
| Necesidad de bajo boilerplate | Alta | Media |
| Necesidad de control fino técnico | Media | Alta |
| Troubleshooting avanzado | Media | Alta |
| Tuning de RU/s y diagnósticos | Media | Alta |
| Alineación con wrappers internos | Alta | Alta, si el wrapper lo soporta |

---

## Regla final de decisión

Si el stack ya existe en el servicio, **preferir consistencia con la base instalada**.

Solo proponer migración entre stacks si:

- hay evidencia técnica clara
- el usuario lo pide explícitamente
- el beneficio supera el costo de cambio
