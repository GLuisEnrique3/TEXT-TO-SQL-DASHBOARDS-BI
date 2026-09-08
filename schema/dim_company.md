---
name: schema-dim_company
description: Ficha de la tabla claro_bi.dim_company (dimensión de compañía interna), usada en el dashboard pero aún no en consultas verificadas
---

# `claro_bi.dim_company` (pendiente de confirmar dataset)

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi (asumido — confirmar con `get_table_info`)
**Alias habitual:** sin uso todavía en `queries/`

## Descripción

Dimensión de "compañía interna" (`Internal_Company_Object__c`). Se llega a ella a 2 saltos
desde `BOB_TD`, vía `dim_cslb`. Aparece como segmentador ("Company Name") en el dashboard,
pero **todavía no se usa en ninguna consulta verificada** de `queries/`.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK, referenciada desde `dim_cslb.Internal_Company_Object__c` |
| `Name` | Nombre de la compañía. Usado como segmentador en el dashboard (ver `context/dashboard-filters.md`) |

## Joins típicos

| Join hacia | Condición | Para qué |
|---|---|---|
| `claro_bi.dim_cslb` | `dim_company.Id = o.Internal_Company_Object__c` | Llegar desde `BOB_TD` (vía `dim_cslb`) a la compañía interna asociada al carrier/LOB |

## Gotchas / notas

- Ficha creada a partir del mapeo de segmentadores del dashboard, no de una inspección
  directa de la tabla en BigQuery — verificar dataset, tipos de columna y si tiene más
  columnas relevantes con `get_table_info` antes de usarla en una consulta verificada.
- Si se añade un desglose "by-company" a las familias de `queries/*/`, seguir el mismo
  patrón de las demás dimensiones (alias corto, join por Id, nota de a qué otras consultas
  complementa).
