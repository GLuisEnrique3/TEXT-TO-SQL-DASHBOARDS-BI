---
name: members-policies-aca-by-source
description: Miembros y pólizas activas de ACA, desglosado por fuente/origen (s.Source__c)
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_source
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob, by-source]
---

# Members / Policy Numbers — ACA por Source

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-aca.md`, pero desglosada
por fuente/origen del negocio (`s.Source__c`, vía `dim_source`) en vez de un total único.

## SQL

```sql
SELECT
  s.Source__c,
  SUM(Members__C) as Members,
  COUNT(b.Policy_Number__c) as Policy_Numbers
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_source` s ON s.Source__c = b.Source__c
WHERE
      b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
      AND b.Val_BOB_TD = 1
      AND b.Status__c NOT IN ('NR','Terminated')
      AND a.Name_Agencies != 'Agency Test'
      AND a.Name_Agencies IS NOT NULL
      AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
      AND o.Line_Of_Business = 'ACA'
GROUP BY s.Source__c
```

## Notas / supuestos

- Ver `members-policies-aca.md` para el detalle de cada regla de filtrado aplicada.
- Definir en `context/glossary.md` (pendiente) qué valores toma `Source__c` y qué
  representa cada uno (ej. canal de venta, campaña, etc.).
