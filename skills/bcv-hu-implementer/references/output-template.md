> This template defines the exact public output format for the implementation report.
> The skill generates a SINGLE file (no separate draft). It follows the structure below and adds a `Skill` column so the developer knows which skill to use per task.

```markdown
# Implementation Report — HU-<code>

> **Mode:** `dry-run` (por defecto; no se creó ninguna rama ni se modificó ningún archivo)
> **DHU source:** [hu-technical-refinement/HU-<code>-refined-...md](HU-<code>-refined-...md)
> **Context source:** [.context/hu-<code>.md](../.context/hu-<code>.md)
> **Generated:** {YYYY-MM-DD}

## Pre-implementation validation

| Check | Resultado |
|---|---|
| Blocking gaps | ✅ / ❌ |
| Estado DHU | ✅ / ❌ |
| Criterios de aceptación | ✅ / ❌ |
| Mapa técnico | ✅ / ❌ |
| DoR | ✅ / ❌ |

Validación superada → se procede en modo `dry-run`.

---

## Summary

| Metric | Value |
|---|---|
| Repositorios afectados | {N} |
| Ramas propuestas | {N} |
| Archivos a modificar (estimado) | {N} |
| Archivos a crear (estimado) | {N} |
| Tests a crear/actualizar | {N} |
| Migraciones a crear | {N} |
| Linter | N/A (sin cambios aplicados) |
| Tests | N/A (sin cambios aplicados) |

⚠️ **Hallazgo de la investigación de código** (si aplica): {texto}

---

## Repositories

### {service-name}

**Branch (propuesta):** `feature/HU-<code>`
**Status:** ready for apply

**Skills recomendados:** `{bcv-openapi-design}`, `{bcv-spring-data-jpa-sql-server}` (descubiertos en este workspace/usuario)

> **Cómo se codificó cada tarea:** el código de cada tarea se generó **leyendo y aplicando** el skill referenciado (su `SKILL.md` + `references/`). El skill es la autoridad de cómo se escribió el código. Ejemplo: `bcv-openapi-design` → records con `@Schema`, errores RFC 9457, contrato en OpenAPI; `bcv-spring-data-jpa-sql-server` → `@Entity`/`@Column`, repositorio Spring Data, migración Flyway `V{ts}__`.

**Files modified:**

| File | Change | Skill (cómo codificar) |
|---|---|---|
| `.../{File}.java` | {brief description} | `{skill-name}` — {una línea de la convención que aplica} |
| `.../{Mapper}.java` | {brief description} | `{skill-name}` — {una línea de la convención que aplica} |

**Files created:**

| File | Purpose | Skill (cómo codificar) |
|---|---|---|
| `.../{NewClass}.java` | {brief description} | `{skill-name}` — {una línea de la convención que aplica} |
| `.../{NewTest}.java` | {brief description} | `{skill-name}` — {una línea de la convención que aplica} |
| `.../db/migration/V...__.sql` | {brief description} | `{skill-name}` — {una línea de la convención que aplica} |

**Diff summary (optional excerpts):**

```diff
{short diff excerpt}
```

---

### {service-name-2}

**Branch:** no requerida — **sin cambios de código**.
{nota de regresión}

---

## Manual tasks remaining

- [ ] {task 1}
- [ ] {task 2}

## Next steps

1. Confirmar este plan de cambios (o pedir ajustes).
2. Ejecutar el skill en modo **`apply`** (`bcv-hu-implementer --apply`) para crear ramas y aplicar cambios.
3. Tras `apply`, se ejecuta `./mvnw verify` por repositorio; si falla, se detiene y reporta.
4. Revisar el diff, commitear y crear PRs manualmente (el skill nunca commitea/pushea).
```

---

## Skill column rules

- The `Skill` column lists the recommended skill **discovered on the user's machine / workspace** (via glob `find . -type f -name 'SKILL.md'` plus `.github/copilot-instructions.md`).
- If no discovered skill matches the task, write `not available in Copilot Chat`.
- One primary skill per row; if a task spans concerns, list the primary first.
- The skill set is discovered once at the start (workspace + user level) and reused for all rows.
