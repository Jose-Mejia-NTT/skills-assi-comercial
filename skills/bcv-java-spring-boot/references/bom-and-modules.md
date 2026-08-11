# BCV BOM and Module Conventions

Use this reference for Maven parent/BOM alignment and module layout decisions.

## Parent and BOM patterns

Typical BCV parent options:

- `ads-spring-boot-dependencies`
- `bcv-commons-starter-parent`

Choose the same parent/BOM already used by the target project. Avoid introducing a second parent style unless migration is explicitly requested.

## Multi-module baseline

Recommended logical split:

- `*-core`: domain and business logic
- `*-input`: entry points (controllers, request mapping)
- `*-output`: driven adapters (persistence, messaging, clients)
- `*-app`: executable wiring and runtime configuration

## Plugin placement rule

- Keep `spring-boot-maven-plugin` executable behavior in `*-app` only.
- For non-executable modules, avoid repackage or set plugin skip according to project pattern.

## New module checklist

1. Add `<module>new-module</module>` in parent `pom.xml`.
2. Create child POM inheriting from existing parent.
3. Add dependencies from or to `*-app` only when really needed.
4. Preserve existing groupId/artifactId naming conventions.
5. Run `mvn -q -DskipTests package` to validate graph consistency.
