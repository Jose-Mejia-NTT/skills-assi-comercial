# bcv-commons-observability

Genera y mantiene componentes de **observabilidad** (trazas, métricas, alertas en Teams) para proyectos Java del ecosistema BCV usando las anotaciones de `bcv-commons-observability`.

## ¿Qué cubre?

Este skill se enfoca en:

- **Anotaciones observables**: `@ObservableController`, `@ObservableService`, `@ObservableOperation`.
- **Trazas distribuidas** (OpenTelemetry, trace IDs, correlación de eventos).
- **Métricas** y exposición a Application Insights.
- **Alertas en Teams** (webhooks, configuración, enmascaramiento de datos).
- **Data masking (enmascaramiento de datos sensibles)** — PII, contraseñas, tokens.
- **Configuración de endpoint de webhook** (`commons.exception.teams.webhook.endpoint`).
- **Integración con la arquitectura hexagonal** — colocación de observabilidad en controladores, servicios y adaptadores.

## ¿Cuándo usarlo?

- Agregar observabilidad a un nuevo controlador, servicio o caso de uso.
- Configurar alertas en Teams para excepciones críticas.
- Enmascarar datos sensibles en logs y trazas.
- Investigar problemas de falta de trazas o alertas no recibidas.

## ¿Cuándo NO usarlo?

- Para monitoreo a nivel de infraestructura (AKS, Azure Monitor genérico).
- Para tutoriales genéricos de OpenTelemetry.
- Para configuración de base de datos o services internos.

## Skills relacionados

- `bcv-hexagonal-architecture` — para identificar controladores y servicios a instrumentar.
- `bcv-java-spring-boot` — para configurar el proyecto y propiedades de observabilidad.

## Información requerida

El skill necesita:

1. **Componente a instrumentar** — controlador, servicio, o adaptador específico.
2. **Objetivo** — mejorar trazas, agregar alertas, enmascarar datos específicos.
3. **Webhook de Teams (si aplica)** — se obtiene de Azure Key Vault.

## Archivos principales

- `SKILL.md` — instrucciones completas y flujo de trabajo.
- `references/` — anotaciones, patrones de enmascaramiento, configuración de Teams.
