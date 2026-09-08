---
name: annual-premium-members-supplementary-by-carrier
description: Prima anual y miembros totales de Supplementary (año en curso), desglosado por carrier (o.Carrier)
domain: supplementary
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [premium, members, supplementary, bob, by-carrier]
---

# Annual Premium / Members — Supplementary por Carrier

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que
`annual-premium-members-supplementary-by-agency.md`, pero desglosada por aseguradora
(`o.Carrier`) en vez de por agencia.

## SQL

```sql
SELECT
  o.Carrier,
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
    AND o.Line_Of_Business = 'Supplementary'
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
GROUP BY o.Carrier
```

## Notas / supuestos

- Ver `annual-premium-members-supplementary-by-agency.md` para el detalle de las reglas de
  filtrado aplicadas.
