---
name: members-policies-medicare-by-source
description: Miembros y pólizas de Medicare (año en curso), desglosado por fuente/origen (src.Source__c)
domain: medicare
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_source
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, medicare, bob, by-source]
---

# Members / Policies Numbers — Medicare por Source

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-medicare.md`, pero
desglosada por fuente/origen del negocio (`src.Source__c`, vía `dim_source`) en vez de un
total único.

## SQL

```sql
SELECT
  src.Source__c,
  SUM(b.Members__c) as total_members,
  COUNT(b.Policy_Number__c) as total_policies
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
    AND o.Line_Of_Business IN ('Medicare')
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
GROUP BY src.Source__c
```

## Notas / supuestos

- Ver `members-policies-medicare.md` para el detalle de cada regla de filtrado aplicada.
