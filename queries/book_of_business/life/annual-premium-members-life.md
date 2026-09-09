---
name: annual-premium-members-life
description: Prima anual total y miembros totales de la línea de negocio Life, año en curso
domain: life
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [premium, members, life, bob]
---

# Annual Premium / Members — Life

## Cuándo usar esta consulta

Responde: "¿cuál es la prima anual total y el total de miembros de la línea Life, para
pólizas con fecha de vigencia dentro del año en curso y hasta hoy?"

Para otra LOB, cambiar `o.Line_Of_Business IN ('Life')` (ver `context/glossary.md`). Ver
`queries/book_of_business/supplementary/` para el equivalente de Supplementary (LOB separada, no combinada
en esta consulta). Nota: filtra por **año calendario en curso** (`EXTRACT(YEAR ...)`), a
diferencia de la consulta de ACA que usa `LAST_DAY(CURRENT_DATE())` sin restringir año.

## SQL

```sql
SELECT
  SUM(b.Annual_Premium__c) AS annual_premium,
  SUM(b.Members__c) as total_members,
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE
    b.Status__c NOT IN ('NR', 'Terminated')
    AND b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
    AND a.Name_Agencies IS NOT NULL
    AND a.Name_Agencies != 'Agency Test'
    AND o.Line_Of_Business IN ('Life')
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
```

## Notas / supuestos

- No filtra `Val_BOB_TD = 1` — **intencional**, a diferencia de `members-policies-aca.md` y
  `agents-with-active-contracts-and-production.md`, que sí lo exigen.
- Filtra por año calendario en curso, no por mes de corte — ajustar `EXTRACT(YEAR FROM
  CURRENT_DATE)` si se pide otro año.
