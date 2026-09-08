---
name: active-agents-td
description: Conteo de agentes activos (por estado de agente), excluyendo cuentas de prueba
domain: agents
tables:
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [agents, active, contact]
---

# Active Agents TD

## Cuándo usar esta consulta

Responde: "¿cuántos agentes distintos están activos (o en un estado asimilable a activo)
actualmente?", excluyendo la cuenta de prueba `Agency Test`.

## SQL

```sql
SELECT COUNT (DISTINCT c.Id)
FROM `claroinsurance-dataplatform.claro_bi.dim_contact` c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE c.Agent_Status__c IN ('Active','Awating Partial Release','Awaiting Complete Release','Pending Documents','Dependent Agent','Suspended by carrier')
AND a.Name_Agencies != 'Agency Test'
AND a.Name_Agencies IS NOT NULL
```

## Notas / supuestos

- `Agent_Status__c` incluye varios estados considerados "activo" en sentido amplio (no solo
  `'Active'` literal) — ver `context/glossary.md` para la definición de cada uno cuando se
  documente. Ojo: `'Awating Partial Release'` está tal cual en el dato fuente (con typo).
- No usa `dim_cslb` ni línea de negocio — este conteo es agnóstico de LOB.
- Distinto de `agents-with-active-contracts.md` (que cuenta agentes con contrato vigente en
  `vw_agent_contracts`) y de `agents-with-active-contracts-and-production.md` (que además
  exige producción en `BOB_TD`). Confirmar con el equipo cuál métrica usar según el caso.
