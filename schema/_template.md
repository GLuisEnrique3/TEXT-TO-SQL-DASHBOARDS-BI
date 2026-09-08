---
name: _template
description: Plantilla para documentar una tabla o vista de BigQuery
---

# `dataset.nombre_tabla`

**Tipo:** tabla / vista
**Dataset:** claroinsurance-dataplatform.<dataset>

## Descripción

Qué representa cada fila de esta tabla en términos de negocio.

## Columnas clave

| Columna | Tipo | Descripción |
|---|---|---|
| `Id` | STRING | |
| `...` | | |

## Joins típicos

| Join hacia | Condición | Cuándo usarlo |
|---|---|---|
| `otro_dataset.otra_tabla` | `t.campo = o.Id` | |

## Gotchas / notas

- Columnas ambiguas, soft-deletes, valores especiales (NULL, 'Test', etc.), duplicados
  esperados, granularidad real de la tabla, etc.
