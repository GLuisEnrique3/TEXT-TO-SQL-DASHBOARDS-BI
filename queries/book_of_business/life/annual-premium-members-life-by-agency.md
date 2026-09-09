---
name: annual-premium-members-life-by-agency
description: Prima anual y miembros totales de Life (año en curso), desglosado por agencia (a.Name_Agencies)
domain: life
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [premium, members, life, bob, by-agency]
---

# Annual Premium / Members — Life por Agencia

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `annual-premium-members-life.md`, pero
desglosada por agencia (`a.Name_Agencies`) en vez de un total único.

## SQL

```sql
SELECT
  a.Name_Agencies,
  SUM(b.Annual_Premium__c) AS annual_premium,
  SUM(b.Members__c) as total_members
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE
    b.Status__c NOT IN ('NR', 'Terminated')
    AND b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
    AND a.Name_Agencies IS NOT NULL
    AND a.Name_Agencies != 'Agency Test'
    AND o.Line_Of_Business = 'Life'
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
GROUP BY a.Name_Agencies
```

## Notas / supuestos

- Ver `queries/book_of_business/supplementary/` para el equivalente de esta familia de desgloses aplicado a
  la línea Supplementary.
- No filtra `Val_BOB_TD = 1` — mismo comportamiento intencional que la consulta base.
