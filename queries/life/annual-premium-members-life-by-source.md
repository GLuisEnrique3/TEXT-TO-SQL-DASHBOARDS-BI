---
name: annual-premium-members-life-by-source
description: Prima anual y miembros totales de Life (año en curso), desglosado por fuente/origen (src.Source__c)
domain: life
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_source
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [premium, members, life, bob, by-source]
---

# Annual Premium / Members — Life por Source

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `annual-premium-members-life-by-agency.md`,
pero desglosada por fuente/origen del negocio (`src.Source__c`, vía `dim_source`) en vez
de por agencia.

## SQL

```sql
SELECT
  src.Source__c,
  SUM(b.Annual_Premium__c) AS annual_premium,
  SUM(b.Members__c) as total_members
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_source` src ON src.Source__c = b.Source__c
WHERE
    b.Status__c NOT IN ('NR', 'Terminated')
    AND b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
    AND a.Name_Agencies IS NOT NULL
    AND a.Name_Agencies != 'Agency Test'
    AND o.Line_Of_Business = 'Life'
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
GROUP BY src.Source__c
```

## Notas / supuestos

- Ver `annual-premium-members-life-by-agency.md` para el detalle de la separación
  Life/Supplementary y de las reglas de filtrado aplicadas.
