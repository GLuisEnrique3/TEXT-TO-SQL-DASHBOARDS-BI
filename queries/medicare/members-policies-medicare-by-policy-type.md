---
name: members-policies-medicare-by-policy-type
description: Miembros y pólizas de Medicare (año en curso), desglosado por tipo de póliza (pt.Policy_Type__c)
domain: medicare
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_policy_type
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, medicare, bob, by-policy-type]
---

# Members / Policies Numbers — Medicare por Tipo de Póliza

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-medicare.md`, pero
desglosada por tipo de póliza (`pt.Policy_Type__c`, vía `dim_policy_type`) en vez de un
total único.

## SQL

```sql
SELECT
  pt.Policy_Type__c,
  SUM(b.Members__c) as total_members,
  COUNT(b.Policy_Number__c) as total_policies
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_policy_type` pt ON pt.Policy_Type__c = b.Policy_Type__c
WHERE
    b.Status__c NOT IN ('NR', 'Terminated')
    AND b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
    AND a.Name_Agencies IS NOT NULL
    AND a.Name_Agencies != 'Agency Test'
    AND o.Line_Of_Business IN ('Medicare')
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
GROUP BY pt.Policy_Type__c
```

## Notas / supuestos

- Ver `members-policies-medicare.md` para el detalle de cada regla de filtrado aplicada.
