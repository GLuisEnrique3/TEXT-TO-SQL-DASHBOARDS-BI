---
name: schema-dim_account_2
description: Ficha de la tabla claro_bi.dim_account_2 (dimensión de cuentas/agencias)
---

# `claro_bi.dim_account_2`

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `a`

## Descripción

Dimensión de cuentas (agencias). Se llega a ella desde `contact.AccountId`.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK |
| `Name_Agencies` | Nombre de la agencia. Contiene una cuenta de prueba `'Agency Test'` que
  debe excluirse siempre de reportes de negocio real (ver `context/business-rules.md`). |
| `Subagencia_Name` | Nombre de la subagencia. Usado para desglosar métricas por subagencia (ver `queries/agents/agents-with-active-contracts-and-production-by-subagencies.md`) |
| `Agent_Specialist__c` | Especialista/agente asignado a la cuenta. Usado como segmentador en el dashboard (ver `context/dashboard-filters.md`) |

## Gotchas / notas

- Siempre excluir `Name_Agencies = 'Agency Test'` y `Name_Agencies IS NULL` en reportes de
  negocio real.
