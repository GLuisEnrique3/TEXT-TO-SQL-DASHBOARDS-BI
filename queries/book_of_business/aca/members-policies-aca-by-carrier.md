---
name: members-policies-aca-by-carrier
description: Miembros y pólizas activas de ACA, desglosado por carrier (o.Carrier)
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob, by-carrier]
---

# Members / Policy Numbers — ACA por Carrier

## Cuándo usar esta consulta

Misma métrica y mismas reglas de filtrado que `members-policies-aca.md`, pero desglosada
por aseguradora (`o.Carrier`, de `dim_cslb`) en vez de un total único. Úsala cuando el
usuario pida el desglose por carrier ("¿cómo se distribuyen los miembros de ACA por
aseguradora?").

## SQL

```sql
SELECT
  o.Carrier,
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
GROUP BY o.Carrier
```

## Notas / supuestos

- Ver `members-policies-aca.md` para el detalle de cada regla de filtrado aplicada
  (equivale a las reglas 1, 2, 3, 4, 5 y 6 de `context/business-rules.md`).
- Para desglosar por otra LOB, cambiar `o.Line_Of_Business = 'ACA'`.
- Se puede combinar con `members-policies-aca-by-state.md` (`GROUP BY o.State, o.Carrier`)
  si el usuario pide el cruce de ambas dimensiones — no está guardado como consulta aparte
  para evitar explosión combinatoria de archivos; constrúyela ad hoc a partir de estas dos.
