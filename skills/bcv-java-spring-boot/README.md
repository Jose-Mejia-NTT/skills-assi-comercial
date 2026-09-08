# bcv-java-spring-boot

Configura y mantiene proyectos **Java 21 + Spring Boot 3.x** del ecosistema BCV, incluyendo ADS BOM, estructura multi-módulo, perfiles, y integración con Azure Key Vault.

## ¿Qué cubre?

Este skill se enfoca en:

- **ADS BOM (Bill of Materials)** — dependencias centralizadas y versionadas para el ecosistema BCV.
- **Estructura multi-módulo Maven** — módulos padre, librerías, módulos ejecutables (`-app`).
- **Profiles y bootstrapping** — `bootstrap.yml`, `application.yml`, perfiles (dev, test, prod).
- **Spring Cloud Config + Azure Key Vault** — inyección segura de secrets.
- **Convenciones de paquetes BCV** — estructura de carpetas, módulos, main class.
- **Executable JAR setup** — `spring-boot-maven-plugin`, única aplicación ejecutable por proyecto.
- **Troubleshooting de startup** — errores comunes, classpath, configuración.

## ¿Cuándo usarlo?

- Crear un nuevo proyecto Spring Boot BCV desde cero.
- Agregar o configurar un módulo nuevo en un proyecto existente.
- Resolver problemas de startup o configuración.
- Actualizar dependencias o profiles.

## ¿Cuándo NO usarlo?

- Para diseño de bases de datos (use `bcv-spring-data-jpa-sql-server` o `bcv-cosmos-db`).
- Para arquitectura de slices (use `bcv-hexagonal-architecture`).
- Para configuración de CI/CD o infraestructura.

## Skills relacionados

- `bcv-hexagonal-architecture` — para la estructura de slices dentro del proyecto.
- `bcv-spring-data-jpa-sql-server` — para configurar persistencia JPA.
- `bcv-azure-service-bus` — para configurar mensajería ASB.

## Información requerida

El skill necesita:

1. **Acción** — nuevo proyecto, nuevo módulo, o troubleshooting.
2. **Nombre del servicio BCV** (`bcv-bacc-...`, `bcv-h2h-...`, etc.).
3. **Stack específico** — persistencia, mensajería, etc.

## Archivos principales

- `SKILL.md` — instrucciones completas y flujo de trabajo.
- `references/` — templates de pom.xml, profiles, Key Vault integration, convencionesde módulos.
