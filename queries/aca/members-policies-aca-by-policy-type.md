---
name: members-policies-aca-by-policy-type
description: Miembros y pólizas activas de ACA, desglosado por tipo de póliza (t.Policy_Type__c)
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_policy_type
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob, by-policy-type]
---

# Members / Policy Numbers — ACA por Tipo de Póliza

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-aca.md`, pero desglosada
por tipo de póliza (`t.Policy_Type__c`, vía `dim_policy_type`) en vez de un total único.

## SQL

```sql
SELECT
  t.Policy_Type__c,
  SUM(Members__C) as Members,
  COUNT(b.Policy_Number__c) as Policy_Numbers
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_policy_type` t ON t.Policy_Type__c = b.Policy_Type__c
WHERE
      b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
      AND b.Val_BOB_TD = 1
      AND b.Status__c NOT IN ('NR','Terminated')
      AND a.Name_Agencies != 'Agency Test'
      AND a.Name_Agencies IS NOT NULL
      AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
      AND o.Line_Of_Business = 'ACA'
GROUP BY t.Policy_Type__c
```

## Notas / supuestos

- Ver `members-policies-aca.md` para el detalle de cada regla de filtrado aplicada.
- `dim_policy_type` se une por `t.Policy_Type__c = b.Policy_Type__c` (join por valor, no
  por Id) — mismo patrón que `dim_policy_status` en la consulta por estado de póliza.
