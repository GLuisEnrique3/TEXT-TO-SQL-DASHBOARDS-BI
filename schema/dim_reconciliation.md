---
name: schema-dim_reconciliation
description: Ficha de la tabla claro_bi.dim_reconciliation (dimensión de tipo/resultado de reconciliación)
---

# `claro_bi.dim_reconciliation`

**Tipo:** tabla de dimensión (catálogo pequeño, 3 filas)
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `r2`

## Descripción

Catálogo del resultado/tipo de una reconciliación (ej. `Won`). Se une a
`reconcilations.Reconciliations` **por valor** (`r2.Type_reconciliations = r1.Reconciliations`).

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK (INTEGER) |
| `Type_reconciliations` | Valor del resultado de reconciliación (ej. `'Won'`). Solo 3 valores distintos — documentar aquí el catálogo completo cuando se confirme con el equipo |

## Gotchas / notas

- El join hacia `reconcilations` es por **valor de texto**, no por Id, a pesar de que esta
  tabla sí tiene un `Id` propio.
- Catálogo muy pequeño (3 filas) — considerar listar los 3 valores exactos aquí una vez
  confirmados con `SELECT DISTINCT Type_reconciliations FROM dim_reconciliation`.
