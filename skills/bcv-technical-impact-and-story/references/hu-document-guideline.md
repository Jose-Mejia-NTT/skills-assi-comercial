# Lineamiento Base — Documentos de Historias de Usuario (HU)

> **Versión:** 2.0.0  
> **Fecha:** 16 de abril de 2026  
> **Base:** Cambio de lineamiento IBK — reducción de secciones obligatorias (18 → 7)  
> **Versión anterior:** 1.0.0 (26 de marzo de 2026)  
> **Objetivo:** Establecer el estándar mínimo y recomendado que todo documento de HU del proyecto debe respetar.

---

## 1. Estructura del Documento — Secciones Obligatorias

Las siguientes **7 secciones** son el **núcleo mínimo** que todo documento de HU debe incluir, sin excepción.

| # | Sección | Propósito / Contenido esperado |
|---|---------|-------------------------------|
| 1 | **Título del documento** | Para identificar la funcionalidad. Ej: `"HU — Plin TC"`, `"DHU — CDA con campaña"`, `"Doc. Funcional — Seguro de viaje"` |
| 2 | **Descripción breve** | Contexto mínimo para entender el flujo (2-5 líneas, sin tecnicismos excesivos) |
| 3 | **¿Cuál es la necesidad?** | Ayuda a identificar escenarios. Problema o vacío puntual que esta iniciativa resuelve |
| 4 | **HUs — "Yo como / Quiero / Para"** | Base de los escenarios. Narrativa de historia de usuario por cada funcionalidad (ver §3) |
| 5 | **Criterios de Aceptación (CAs)** | **Insumo MÁS importante para QA.** Reglas de negocio / técnicas numeradas por HU (ver §4) |
| 6 | **Diagrama de flujo TO BE** | Permite derivar: caminos felices, alternos y excepciones. Imagen embebida o enlace |
| 7 | **Referencias** | Links a: Figma, Example Mapping, Arquitectura, Confluence, otras HUs relacionadas |

---

## 2. Secciones Recomendadas

### 2.1 Secciones condicionales (nuevas, según tipo de iniciativa)

Incluir según el tipo de documento y la complejidad de la iniciativa:

| Sección | Cuándo incluirla |
|---------|-----------------|
| **Diagrama de integración** (arquitectura técnica) | Si hay integraciones con APIs, servicios externos o arquitectura distribuida |
| **Tramas (RT / Post Mortem)** como HU específica | Si hay transacciones financieras o se requiere trazabilidad para monitoreo/fraude |
| **Objetivos cuantitativos** (KPIs / metas de negocio) | Si el equipo de producto tiene OKRs o cifras de impacto asociadas a la iniciativa |
| **ANEXOS** | Si hay casuísticas extensas, tablas de parámetros o reglas edge-case |
| **Contenido / Índice de HUs** | Si el documento tiene más de 6 HUs |

### 2.2 Secciones anteriormente obligatorias (ahora opcionales/recomendadas)

> Estas secciones eran obligatorias en el lineamiento v1.x (18 secciones). A partir del v2.0 son opcionales pero **altamente recomendadas** cuando el equipo QA las necesite para test planning.

| Sección | Valor para QA | Cuándo incluirla |
|---------|---------------|-----------------|
| **Alcance** | Delimita el scope del test plan | Siempre que sea posible |
| **Canales** | **Crítico** — determina pruebas de compatibilidad y accesibilidad | Recomendado siempre |
| **Aplicaciones Involucradas** | Determina pruebas de integración | Recomendado siempre |
| **Público Objetivo** | Informa el tipo de actor para test cases | Recomendado |
| **¿Cuáles son los beneficios?** | Contexto de valor para pruebas de negocio | Cuando aplique |
| **Dependencias** | Detecta prerequisitos de ejecución | Cuando haya dependencias |
| **Fuera del Alcance** | Evita scope creep en el test plan | Recomendado siempre |
| **Diagrama de flujo ASIS** | Comparativa de estados | Si hay proceso previo |
| **Stakeholders Involucrados** | Orienta la asignación de ejecuciones | Cuando sea relevante |
| **Control de Versiones** | Trazabilidad del documento | Recomendado para documentos formales |
| **Glosario de Términos** | Desambiguación de términos del dominio | Dominios especializados |

> **Nota para QA:** Si el PO no incluye Canales o Aplicaciones Involucradas, solicite esta información durante el refinement o derívela de la Descripción breve, los CAs y el TO BE Diagram.

---

## 3. Formato de la Historia de Usuario Individual

### 3.1 Narrativa obligatoria

Cada HU dentro del documento DEBE tener la narrativa en este formato exacto:

```
Yo como [rol del actor]
Quiero [acción o capacidad deseada]
Para [beneficio o valor que obtiene]
```

### 3.2 Perspectivas válidas del "Yo como"

El rol declarado en la narrativa define el tipo de documento y debe ser coherente en todo el documento:

| Perspectiva | Cuándo usarla | Ejemplos |
|-------------|--------------|---------|
| **Cliente / Usuario final** | Documentos CX / APP | `"cliente Interbank con acceso al APP"` |
| **Plataforma Digital (sistema)** | Documentos de backend / APIs / integraciones | `"Plataforma digital"` |
| **RF (Representante de Fuerza de ventas)** | Documentos de canal Tiendas | `"RF"` |
| **Analista de Riesgos** | HUs de back-office o aprobación | `"Analista de Riesgos"` |
| **[Rol del sistema]** | HUs que son acciones de sistema sin actor humano | `"el sistema de monitoreo"` |

> **Regla:** La perspectiva debe ser **consistente dentro del mismo documento**. Un documento no debe mezclar perspectiva de cliente con perspectiva de sistema en el mismo flujo principal, salvo que sean HUs de back-office explícitamente diferenciadas.

### 3.3 Campos mínimos por HU individual

| Campo | Obligatorio | Notas |
|-------|-------------|-------|
| Narrativa (Yo como / Quiero / Para) | ✅ | Siempre explícita en el texto |
| Criterios de Aceptación | ✅ | Mínimo 3 por HU |
| Indicador de flujo secuencial | ⚠️ Cuando aplica | Si las HUs son pasos de un proceso: numerarlas como Paso 1, Paso 2… |
| Alcance local | ⚠️ Cuando aplica | Si la HU tiene un alcance distinto al del documento, declararlo internamente |
| Dependencias locales | ⚠️ Cuando aplica | Si la HU depende de otra HU específica del mismo documento |

---

## 4. Criterios de Aceptación (CAs)

### 4.1 Formato de identificador

Los CAs se numeran con el formato:

```
CA XX
```

Donde `XX` es un número de dos dígitos con cero a la izquierda: `CA 01`, `CA 02`, `CA 03`…

> **Nota:** El separador es **espacio** (no guion). Este es el formato predominante en los documentos reales del proyecto (3 de 4 documentos). Se admite `CA-XX` solo si el equipo tiene una convención previa establecida y la mantiene consistentemente en todo el documento.

### 4.2 Numeración de CAs

| Opción | Descripción | Cuándo usarla |
|--------|-------------|--------------|
| **Continua por documento** | Los CAs se numeran sin reiniciar entre HUs: `CA 01` → `CA 51` | Documentos de cliente / APP donde el flujo es lineal y los CAs son acumulables |
| **Independiente por HU** | Cada HU inicia su propia numeración desde `CA 01` | Documentos backend con HUs técnicamente independientes |

> **Regla de oro:** Definir el esquema al inicio del documento y mantenerlo sin mezclar. No usar ambos esquemas en el mismo documento.

### 4.3 Contenido mínimo por CA

Cada criterio debe:

- Describir **un único comportamiento verificable** (no combinar varios)
- Usar **datos concretos** cuando sea posible ("código HTTP 200", "mensaje: 'Registro exitoso'")
- Ser **observable y medible**: que un QA pueda ejecutar un test y determinar si pasa o falla
- No usar lenguaje ambiguo: "rápido", "seguro", "correcto" sin una métrica asociada

### 4.4 Cobertura mínima por HU

| Tipo de CA | Cantidad mínima |
|------------|-----------------|
| Flujo exitoso (Happy Path) | 1 |
| Validación / Error | 1 por regla de negocio clave |
| Caso borde o excepcional | 1 cuando aplique |
| CAs de seguridad / autenticación | Obligatorios si hay OTP, NMA, roles o permisos |

### 4.5 CAs cancelados

Si un CA definido previamente deja de aplicar, se debe marcar explícitamente como `[CANCELADO]` en lugar de eliminarlo, para mantener la trazabilidad del historial del documento.

---

## 5. Control de Versiones — Estados Estándar

Para unificar la columna "Estado" en la tabla de Control de Versiones, los únicos valores aceptados son:

| Estado | Cuándo usarlo |
|--------|--------------|
| `EN ELABORACIÓN` | El documento está siendo redactado, aún no revisado |
| `EN REVISIÓN` | El documento está bajo revisión de stakeholders |
| `APROBADO` | El documento fue aprobado y está listo para desarrollo |
| `FINALIZADO` | El sprint/entrega asociado fue completado |
| `OBSOLETO` | La iniciativa fue descartada o reemplazada |

> **Regla:** No usar términos libres ("TERMINADO", "COMPLETADO", "En curso", "Finalizado"). Usar exclusivamente los estados de la tabla anterior.

---

## 6. Nomenclatura del Tipo de Documento

El título del documento debe incluir un prefijo que identifique el tipo:

| Prefijo | Descripción | Ejemplo |
|---------|-------------|---------|
| `HU` | Historia de Usuario estándar | `HU — Plineo favorito sin OTP` |
| `DHU` | Detailed History — HU con alto nivel de detalle de negocio | `DHU — CDA con campaña` |
| `Doc. Funcional` | Documento funcional orientado a UX/CX | `Doc. Funcional — Seguro de viaje (CX)` |

---

## 7. Reglas de Referencias

Toda sección de **Referencias** debe incluir al menos uno de los siguientes:

| Tipo de referencia | Herramienta | Obligatorio |
|--------------------|-------------|-------------|
| Diseño / Wireframes | Figma | ⚠️ Si hay pantallas o flujos UX |
| Mapeo de reglas de negocio | FigJam / Example Mapping | ⚠️ Si hay reglas complejas |
| Arquitectura técnica | Confluence / Miro / draw.io | ⚠️ Si hay integraciones |
| HUs relacionadas | Jira links | ⚠️ Si hay dependencias |

---

## 8. Reglas de Calidad para Diagramas

| Diagrama | Regla |
|----------|-------|
| **ASIS** | Obligatorio. Si el proceso no existía antes, indicar explícitamente "No aplica — funcionalidad nueva" |
| **TO BE** | Obligatorio. Debe reflejar el estado final tras implementar todas las HUs del documento |
| Formato | Imagen embebida en Confluence, o enlace a FigJam/Miro con acceso compartido |
| Placeholder | No dejar el diagrama vacío — si está en progreso, indicar `[EN ELABORACIÓN]` |

---

## 9. Resumen Visual — Checklist de Entregable

Antes de marcar un documento de HU como listo para sprint, verificar:

```
CABECERA DEL DOCUMENTO
□ Título con prefijo de tipo (HU / DHU / Doc. Funcional)
□ Alcance definido
□ Descripción breve (2-5 líneas)
□ Tabla de Stakeholders con Nombre / Función / Equipo
□ Público Objetivo declarado
□ Canales declarados
□ Aplicaciones Involucradas listadas
□ Necesidad redactada
□ Beneficios redactados
□ Dependencias listadas (o "Sin dependencias" explícito)
□ Fuera del Alcance con mínimo 1 ítem
□ Referencias con al menos 1 link activo

CONTROL Y FLUJOS
□ Control de Versiones con estado del vocabulario estándar
□ Diagrama ASIS presente (o justificación de ausencia)
□ Diagrama TO BE presente (o "[EN ELABORACIÓN]" si está en progreso)
□ Índice/Contenido incluido (si el doc tiene más de 6 HUs)

HUs INDIVIDUALES (verificar por cada HU)
□ Narrativa "Yo como / Quiero / Para" explícita
□ Perspectiva de "Yo como" coherente con el resto del documento
□ Mínimo 3 CAs por HU
□ Identificador de CAs en formato "CA XX"
□ Esquema de numeración definido y aplicado consistentemente
□ Cada CA es verificable de forma objetiva
□ CAs de seguridad presentes si hay autenticación/roles

CIERRE
□ Glosario de Términos con términos clave del dominio
□ Anexos incluidos si hay casuísticas extensas
```

---

## 10. Ejemplo de Estructura Completa

```markdown
# [Tipo] — [Nombre de la funcionalidad]

## Alcance
[Descripción del alcance]

## Descripción breve
[2-5 líneas de contexto]

## Stakeholders Involucrados
| Nombre | Función | Equipo |
|--------|---------|--------|
| ...    | ...     | ...    |

## Público Objetivo
[Descripción del público]

## Canales
[APP / Web / Tiendas / etc.]

## Aplicaciones Involucradas
[Sistema A, Sistema B, Sistema C]

## ¿Cuál es la necesidad?
[Descripción del problema]

## ¿Cuáles son los beneficios?
[Descripción del valor]

## Dependencias
[Ítem 1, Ítem 2 / "Sin dependencias"]

## Fuera del Alcance
- [Exclusión 1]
- [Exclusión 2]

## Referencias
- Figma: [link]
- Example Mapping: [link]

## Control de Versiones
| Versión | Descripción | Fecha | Responsable | Estado |
|---------|-------------|-------|-------------|--------|
| 1.0 | Versión inicial | DD/MM/YYYY | [Nombre] | EN ELABORACIÓN |

## Diagrama de flujo ASIS
[imagen o link]

## Diagrama de flujo TO BE
[imagen o link]

---

## HU 01 — [Título de la HU]

Yo como [rol]
Quiero [acción]
Para [beneficio]

### Criterios de Aceptación

**CA 01** — [Descripción del criterio]
...

**CA 02** — [Descripción del criterio]
...

---

## Glosario de Términos
| Término | Definición |
|---------|-----------|
| [Término] | [Definición] |
```

---

*Este lineamiento es un documento vivo. Actualizar cuando nuevos documentos de HU introduzcan convenciones que deban estandarizarse.*
