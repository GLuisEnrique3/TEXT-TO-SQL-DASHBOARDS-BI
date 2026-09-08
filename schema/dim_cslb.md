---
name: schema-dim_cslb
description: Ficha de la tabla claro_bi.dim_cslb (dimensión Carrier-State-Line of Business)
---

# `claro_bi.dim_cslb`

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `o`

## Descripción

Dimensión que combina Carrier (aseguradora) + Estado + Línea de Negocio. Se usa para
clasificar cada póliza de `BOB_TD` por línea de negocio (ACA, Medicare, Life, ...).

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK, referenciada desde `BOB_TD.Carrier_State_line_of_Business__c` |
| `Line_Of_Business` | Línea de negocio (`ACA`, `Medicare`, `Life`, `Supplementary`, ...) — ver `context/glossary.md` |
| `Carrier` | Nombre de la aseguradora. Usado para desglosar métricas por carrier (ver `queries/*/*-by-carrier.md`) |
| `State` | Estado (geográfico) de la combinación carrier/LOB. Usado para desglosar métricas por estado (ver `queries/*/*-by-state.md`) |

## Gotchas / notas

- Una fila de `dim_cslb` representa una combinación única de Carrier + State + Line of
  Business — al hacer `GROUP BY o.Carrier` u `o.State` sin fijar también la LOB, ten en
  cuenta que un mismo carrier/estado puede aparecer repartido en varias LOBs.
