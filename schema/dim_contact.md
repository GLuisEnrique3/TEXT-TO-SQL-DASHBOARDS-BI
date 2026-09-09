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
| `Id` | PK, referenciada desde `BOB_TD.Contact__c`, `BOB_H_TD.Contact__c`, `Payment_Commission.Contact__c`, `reconcilations.contact__c` y `vw_agent_contracts.Contracted_Agent__c` |
| `AccountId` | FK hacia `dim_account_2.Id` (agencia del contacto) |
| `Contact_Full_name_Formula__c` | Nombre completo del contacto/agente. Usado para desglosar métricas por agente (ver `queries/*/*-by-agents.md`) |
| `Agent_Status__c` | Estado del agente. Valores observados: `Active`, `Awating Partial Release` (typo de origen), `Awaiting Complete Release`, `Pending Documents`, `Dependent Agent`, `Suspended by carrier`, entre otros |
| `GRManager__c` | Manager/supervisor del contacto. Usado como segmentador en el dashboard (ver `context/filters/producers-dashboard-filters.md`) |
| `Flag_Investigation_` | Bandera de investigación sobre el contacto/agente. **Ojo: el nombre real no lleva `__c` al final** (es `Flag_Investigation_`, un solo guión bajo final) — corregido, antes esta ficha decía `Flag_Investigation__c` por error. Usado como segmentador en el dashboard |
| `Agent_Specialist__c` | Especialista asignado **al contacto/agente** — no confundir con `dim_account_2.Agent_Specialist__c` (mismo nombre, pero a nivel de agencia/cuenta, un concepto distinto). Usado como segmentador en BOB History.pbix (página Overview) |
| `Active__c` | BOOLEAN — si el contacto/agente está activo |
| `NPN__c` | Número de licencia (National Producer Number) del agente |
| `AE_Full_Name` | Nombre completo del Account Executive asociado |
| `Contracting_Specialist` | Especialista de contratación asignado |
| `MedicareActive__c` | Bandera de actividad específica de Medicare |

Columnas adicionales existentes pero aún no usadas en ninguna consulta/filtro verificado:
`Name`, `CreatedDate`, `OriginCampaign__c`, `Birthdate`, `MailingPostalCode`,
`Language_Preference__c`, `State__c`, `County__c`, `Created_At_Date`, `OtherState`,
`OtherPostalCode`, `Agency_Representative__c`, `ID_Under_Investigation`.

## Gotchas / notas

- `Agent_Status__c` contiene el valor `'Awating Partial Release'` tal cual, con un typo en
  el dato fuente — reproducirlo exactamente así en los filtros (ver `active-agents-td.md` y
  `agents-with-no-production.md`).
- No todo `dim_contact` es necesariamente un agente; en el contexto de `BOB_TD` representa
  al agente asociado a la póliza.
- **`Agent_Specialist__c` existe en dos tablas distintas** (`dim_contact` y `dim_account_2`)
  con el mismo nombre de columna pero significado distinto (especialista del agente vs.
  especialista de la agencia) — al traducir un filtro del dashboard, confirmar de cuál tabla
  viene el slicer antes de asumir cuál es (ver `context/filters/producers-dashboard-filters.md`).
- Esta ficha se corrigió el 2026-09-09 tras inspeccionar el esquema real en BigQuery — tenía
  un nombre de columna incorrecto (`Flag_Investigation__c`) y le faltaban ~19 columnas. Si
  encuentras más columnas usadas que no están aquí, agrégalas.
