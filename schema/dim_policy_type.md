---
name: schema-dim_policy_type
description: Ficha de la tabla claro_bi.dim_policy_type (dimensión de tipos de póliza)
---

# `claro_bi.dim_policy_type`

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `t` (consultas ACA) / `pt` (consultas Medicare, Life, Supplementary)

## Descripción

Dimensión de tipos de póliza. Se une a `BOB_TD.Policy_Type__c` **por valor**
(`pt.Policy_Type__c = b.Policy_Type__c`).

## Columnas clave

| Columna | Descripción |
|---|---|
| `Policy_Type__c` | Tipo de póliza. Usado para desglosar métricas por tipo de póliza (ver `queries/*/*-by-policy-type.md`) |

## Gotchas / notas

- El join es por valor de texto, no por Id — mismo patrón que `dim_policy_status`.
- Documentar aquí los valores válidos de `Policy_Type__c` cuando se confirmen con el
  equipo (ver `context/glossary.md`).
