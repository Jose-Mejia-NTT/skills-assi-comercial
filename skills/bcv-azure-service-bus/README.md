# bcv-azure-service-bus

Genera y mantiene componentes de mensajería **Azure Service Bus (ASB)** para proyectos Java del ecosistema BCV usando la librería `ads-spring-boot-starter-messaging`.

## ¿Qué cubre?

Este skill se enfoca en:

- **Publicadores (Publishers)** de ASB con `MessagePublisherRegistry`.
- **Suscriptores (Subscribers)** de ASB con `MessageSubscriberRegistry`.
- **Topics, colas y subscripciones** de Azure Service Bus.
- **Configuración de conexión** (connection strings, Managed Identity).
- **Manejo de reintentos, Dead Letter Queues (DLQ)** y errores.
- **Convenciones BACC** (`topic.publishers`, `messages.publishers`, `queue.subscribers`, etc.).
- **Integración con arquitectura hexagonal** (adaptadores `in.broker` y `out.broker`).

## ¿Cuándo usarlo?

- Agregar un nuevo publicador o suscriptor de mensajes.
- Configurar topics, colas o subscripciones en ASB.
- Investigar pérdida de mensajes o problemas de reintentos.
- Cambiar la estrategia de mensajería de un servicio BACC.

## ¿Cuándo NO usarlo?

- Para seleccionar la tecnología de mensajería (use skills de arquitectura).
- Para tutoriales genéricos de Azure SDK (fuera del contexto BACC).
- Para cambios puramente de infraestructura sin código Java.

## Skills relacionados

- `bcv-hexagonal-architecture` — para la colocación de los adaptadores en el slice.
- `bcv-java-spring-boot` — para la configuración del proyecto y ADS BOM.

## Información requerida

El skill necesita:

1. **Nombre del servicio BACC** (`bcv-bacc-account-opening-reporting-service`, etc.).
2. **Nombre del tema/cola** y su variant (topic + queue, messages/queue, o Managed Identity).
3. **Comportamiento esperado** — publicar evento, procesar mensaje, etc.

## Archivos principales

- `SKILL.md` — instrucciones completas y flujo de trabajo.
- `references/` — patrones, configuraciones y ejemplos por variant.
