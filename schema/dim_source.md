---
name: schema-dim_source
description: Ficha de la tabla claro_bi.dim_source (dimensión de fuente/origen del negocio)
---

# `claro_bi.dim_source`

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `s` (consultas ACA) / `src` (consultas Medicare, Life, Supplementary)

## Descripción

Dimensión de fuente/origen del negocio. Se une a `BOB_TD.Source__c` **por valor**
(`src.Source__c = b.Source__c`).

## Columnas clave

| Columna | Descripción |
|---|---|
| `Source__c` | Fuente/origen del negocio (ej. canal de venta, campaña). Usado para desglosar métricas por fuente (ver `queries/*/*-by-source.md`) |

## Gotchas / notas

- El join es por valor de texto, no por Id — mismo patrón que `dim_policy_status` y
  `dim_policy_type`.
- Documentar aquí los valores válidos de `Source__c` cuando se confirmen con el equipo
  (ver `context/glossary.md`).
