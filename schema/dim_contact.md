---
name: schema-dim_contact
description: Ficha de la tabla claro_bi.dim_contact (dimensión de contactos/agentes)
---

# `claro_bi.dim_contact`

**Tipo:** tabla de dimensión
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `c`

## Descripción

Dimensión de contactos. Se usa tanto para llegar de una póliza (`BOB_TD.Contact__c`) a su
agencia (vía `AccountId`), como para representar agentes directamente (ver
`queries/agents/`).

> Nota: la consulta original de ejemplo (`members-policies-aca.md`) usaba
> `salesforce_claro.contact`; todas las consultas verificadas actuales usan
> `claro_bi.dim_contact` como fuente estándar de contacto/agente.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK, referenciada desde `BOB_TD.Contact__c` y `vw_agent_contracts.Contracted_Agent__c` |
| `AccountId` | FK hacia `dim_account_2.Id` (agencia del contacto) |
| `Contact_Full_name_Formula__c` | Nombre completo del contacto/agente. Usado para desglosar métricas por agente (ver `queries/*/*-by-agents.md`) |
| `Agent_Status__c` | Estado del agente. Valores observados: `Active`, `Awating Partial Release` (typo de origen), `Awaiting Complete Release`, `Pending Documents`, `Dependent Agent`, `Suspended by carrier`, entre otros |
| `GRManager__c` | Manager/supervisor del contacto. Usado como segmentador en el dashboard (ver `context/dashboard-filters.md`) |
| `Flag_Investigation__c` | Bandera de investigación sobre el contacto/agente. Usado como segmentador solo en la página "Paid Members" del dashboard |

## Gotchas / notas

- `Agent_Status__c` contiene el valor `'Awating Partial Release'` tal cual, con un typo en
  el dato fuente — reproducirlo exactamente así en los filtros (ver `active-agents-td.md` y
  `agents-with-no-production.md`).
- No todo `dim_contact` es necesariamente un agente; en el contexto de `BOB_TD` representa
  al agente asociado a la póliza.
