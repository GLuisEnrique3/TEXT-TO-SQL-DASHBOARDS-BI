---
name: schema-vw_agent_contracts
description: Ficha de la vista claro_bi.vw_agent_contracts (contratos de agentes)
---

# `claro_bi.vw_agent_contracts`

**Tipo:** vista
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `vw`

## Descripción

Vista de contratos de agentes: una fila por contrato registrado entre un agente y una
combinación carrier/estado/línea de negocio.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Contracted_Agent__c` | FK hacia `dim_contact.Id` — el agente titular del contrato |
| `Status__c` | Estado del contrato. Se une (por valor) a `dim_policy_status.Status`, pero
  ver gotcha abajo |
| `Carrier_State_line_of_Business__c` | FK hacia `dim_cslb.Id` |

## Gotchas / notas

- Usada en `queries/agents/agents-with-active-contracts.md`, que **a pesar del nombre no
  filtra por `Status__c`** — cuenta todos los contratos, no solo los "activos". Ver notas
  de esa consulta.
