---
name: agents-with-active-contracts-and-production-by-subagencies
description: Conteo de agentes distintos con producción vigente en el BOB, desglosado por subagencia
domain: agents
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [agents, contracts, production, bob, by-subagency]
---

# Agents with Active Contracts and Production — por Subagencia

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que
`agents-with-active-contracts-and-production.md`, pero desglosada por subagencia
(`a.Subagencia_Name`) en vez de un total único.

## SQL

```sql
SELECT
  a.Subagencia_Name,
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
GROUP BY a.Subagencia_Name
```

## Notas / supuestos

- Ver `agents-with-active-contracts-and-production.md` para el detalle de las reglas de
  filtrado aplicadas (equivalen a las reglas 1, 2, 3, 4 y 6 de
  `context/business-rules.md`; no filtra por LOB).
- `GROUP BY Subagencia_Name` en el original no calificaba la columna con el alias `a.` —
  se normalizó a `a.Subagencia_Name` en el `GROUP BY` para que sea inequívoco si en el
  futuro se agregan más tablas con una columna del mismo nombre; el comportamiento es
  idéntico.
- Complementa a `agents-with-active-contracts-and-production-by-agencies.md` (mismo
  universo, desglosado por agencia en vez de subagencia).
