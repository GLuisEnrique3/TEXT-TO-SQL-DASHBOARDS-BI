---
name: schema-dim_policy_status
description: Ficha de la tabla claro_bi.dim_policy_status (dimensión de estados de póliza)
---

# `claro_bi.dim_policy_status`

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `p` (en `agents-with-active-contracts.md`) o `ps` (en las consultas
`*-by-policy-status.md` y `*-by-risk-policies.md`)

## Descripción

Dimensión de estados de póliza. Se une a `BOB_TD.Status__c` **por valor**, no por Id
(`ps.Status = b.Status__c`), lo que sugiere que `Status` en esta tabla es el mismo texto
que aparece en `BOB_TD.Status__c`.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Status` | Valor de estado de póliza (ej. `NR`, `Terminated`, `Binder Payment`, `Late Payment`, ...). Ver `context/glossary.md` para el detalle de cada valor |
| `Status_Category` | Agrupación/categoría de más alto nivel sobre `Status` (ej. varios `Status` distintos podrían caer en la misma categoría). Usado como segmentador en las páginas Overview y Paid Members del dashboard |

## Gotchas / notas

- **`Status` vs `Status_Category`**: el dashboard usa `Status_Category` como segmentador en
  Overview/Paid Members, pero `Status` en Producers — y todas las consultas verificadas de
  `queries/*/*-by-policy-status.md` usan `Status`. Ver `context/dashboard-filters.md` para
  el detalle; pendiente decidir cuál es el estándar para reportes nuevos.

- El join es por valor de texto (`ps.Status = b.Status__c`), no por una clave subrogada —
  confirmar con el equipo si existe un Id preferido para evitar problemas de
  mayúsculas/espacios en el texto.
- En `agents-with-active-contracts.md` se une contra `vw.Status__c` (de
  `vw_agent_contracts`) en vez de `BOB_TD.Status__c`, pero el resultado del join **no se
  usa** para filtrar en esa consulta (ver notas de ese archivo).
