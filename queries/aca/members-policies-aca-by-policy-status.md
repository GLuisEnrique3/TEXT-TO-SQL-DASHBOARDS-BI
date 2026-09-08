---
name: members-policies-aca-by-policy-status
description: Miembros y pólizas activas de ACA, desglosado por estado de póliza (p.Status)
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_policy_status
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob, by-status]
---

# Members / Policy Numbers — ACA por Estado de Póliza

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-aca.md`, pero desglosada
por estado de póliza (`p.Status`, vía `dim_policy_status`) en vez de un total único. Como
la base ya excluye `NR` y `Terminated`, el desglose solo mostrará los demás estados
"activos" (ver `context/glossary.md` para la lista de valores de `Status__c`).

## SQL

```sql
SELECT
  p.Status,
  SUM(Members__C) as Members,
  COUNT(b.Policy_Number__c) as Policy_Numbers
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_policy_status` p ON p.Status = b.Status__c
WHERE
      b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
      AND b.Val_BOB_TD = 1
      AND b.Status__c NOT IN ('NR','Terminated')
      AND a.Name_Agencies != 'Agency Test'
      AND a.Name_Agencies IS NOT NULL
      AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
      AND o.Line_Of_Business = 'ACA'
GROUP BY p.Status
```

## Notas / supuestos

- Ver `members-policies-aca.md` para el detalle de cada regla de filtrado aplicada.
- Distinta de `members-policies-aca-by-risk-policies.md`, que en vez de excluir `NR`/
  `Terminated` filtra explícitamente por estados de riesgo (`'Binder Payment'`,
  `'Late Payment'`).
- `dim_policy_status` se une por `p.Status = b.Status__c` (join por valor, no por Id) —
  confirmar con el equipo si existe una clave subrogada preferida.
