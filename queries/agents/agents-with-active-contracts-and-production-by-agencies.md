---
name: agents-with-active-contracts-and-production-by-agencies
description: Conteo de agentes distintos con producción vigente en el BOB, desglosado por agencia
domain: agents
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [agents, contracts, production, bob, by-agency]
---

# Agents with Active Contracts and Production — por Agencia

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que
`agents-with-active-contracts-and-production.md`, pero desglosada por agencia
(`a.Name_Agencies`) en vez de un total único.

## SQL

```sql
SELECT
  a.Name_Agencies,
  COUNT(DISTINCT b.Contact__c)
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
AND b.Val_BOB_TD = 1
AND b.Status__c NOT IN ('NR','Terminated')
AND a.Name_Agencies != 'Agency Test'
AND a.Name_Agencies IS NOT NULL
AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
GROUP BY a.Name_Agencies
```

## Notas / supuestos

- Ver `agents-with-active-contracts-and-production.md` para el detalle de las reglas de
  filtrado aplicadas (equivalen a las reglas 1, 2, 3, 4 y 6 de
  `context/business-rules.md`; no filtra por LOB).
- Complementa a `agents-with-active-contracts-and-production-by-subagencies.md` (mismo
  universo, desglosado por subagencia en vez de agencia).
