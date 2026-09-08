---
name: members-policies-aca
description: Total de miembros y número de pólizas activas de la línea de negocio ACA
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob]
---

# Members / Policy Numbers — ACA

## Cuándo usar esta consulta

Responde: "¿cuántos miembros y cuántas pólizas activas de ACA tenemos, vigentes hasta fin
de mes actual, excluyendo cuentas de prueba y pólizas no productivas?"

Es la consulta base para métricas de Book of Business por línea de negocio. Para otra LOB,
cambiar el filtro `o.Line_Of_Business = 'ACA'` (ver `context/glossary.md`). Para otra fecha
de corte, ajustar `LAST_DAY(CURRENT_DATE())`.

## SQL

```sql
SELECT
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
```

## Notas / supuestos

- Aplica las reglas 1, 2, 3, 4, 5 y 6 de `context/business-rules.md` en conjunto.
- `RecordTypeId IN (...)` fija el tipo de póliza incluido en el reporte estándar de BOB —
  no cambiar salvo que se quiera reportar otro universo de pólizas.
- Ver `schema/BOB_TD.md`, `schema/dim_cslb.md`, `schema/dim_account_2.md` para el detalle
  de cada tabla involucrada.
