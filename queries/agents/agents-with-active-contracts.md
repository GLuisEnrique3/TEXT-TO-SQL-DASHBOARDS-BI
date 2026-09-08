---
name: agents-with-active-contracts
description: Conteo de agentes distintos con al menos un contrato registrado en vw_agent_contracts
domain: agents
tables:
  - claro_bi.vw_agent_contracts
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_policy_status
  - claro_bi.dim_cslb
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [agents, contracts]
---

# Agents with Active Contracts

## Cuándo usar esta consulta

Responde: "¿cuántos agentes distintos tienen contratos registrados?", excluyendo la cuenta
de prueba `Agency Test`.

## SQL

```sql
SELECT COUNT(DISTINCT vw.Contracted_Agent__c)
FROM `claroinsurance-dataplatform.claro_bi.vw_agent_contracts` vw
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = vw.Contracted_Agent__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_policy_status` p ON vw.Status__c = p.Status__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = vw.Carrier_State_line_of_Business__c
WHERE a.Name_Agencies != 'Agency Test'  AND a.Name_Agencies IS NOT NULL
```

## Notas / supuestos

- A pesar del nombre ("Active Contracts"), el `WHERE` **no filtra por `vw.Status__c` ni por
  `p.Status__c`** — cuenta todos los contratos en `vw_agent_contracts`, sin excluir
  terminados/inactivos. Los joins a `dim_policy_status` y `dim_cslb` están presentes pero no
  se usan para filtrar en esta versión; confirmar con el equipo si falta un filtro de
  estado antes de usar esta consulta como fuente de "agentes con contrato **activo**".
- Distinta de `active-agents-td.md` (cuenta por `Agent_Status__c` en `dim_contact`, sin
  pasar por contratos) y de `agents-with-active-contracts-and-production.md` (exige
  producción real en `BOB_TD`).
