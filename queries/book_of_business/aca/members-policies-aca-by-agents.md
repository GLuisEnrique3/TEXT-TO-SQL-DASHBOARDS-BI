---
name: members-policies-aca-by-agents
description: Miembros y pólizas activas de ACA, desglosado por agente (nombre completo del contacto)
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob, by-agent]
---

# Members / Policy Numbers — ACA por Agente

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-aca.md`, pero desglosada
por agente (`c.Contact_Full_name_Formula__c`, de `dim_contact`) en vez de un total único.
Úsala cuando el usuario pida el desglose por agente ("¿cómo se distribuyen los miembros de
ACA por agente?").

## SQL

```sql
SELECT
  c.Contact_Full_name_Formula__c,
  SUM(Members__C) as Members,
  COUNT(b.Policy_Number__c) as Policy_Numbers
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE
      b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
      AND b.Val_BOB_TD = 1
      AND b.Status__c NOT IN ('NR','Terminated')
      AND a.Name_Agencies != 'Agency Test'
      AND a.Name_Agencies IS NOT NULL
      AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
      AND o.Line_Of_Business = 'ACA'
GROUP BY c.Contact_Full_name_Formula__c
```

## Notas / supuestos

- Ver `members-policies-aca.md` para el detalle de cada regla de filtrado aplicada
  (equivale a las reglas 1, 2, 3, 4, 5 y 6 de `context/business-rules.md`).
- Para desglosar por otra LOB, cambiar `o.Line_Of_Business = 'ACA'`.
- Complementa a `members-policies-aca-by-agency.md` (desglose por agencia) y
  `members-policies-aca-by-state.md`/`members-policies-aca-by-carrier.md` — todas comparten
  la misma base y solo cambian la dimensión del `GROUP BY`.
