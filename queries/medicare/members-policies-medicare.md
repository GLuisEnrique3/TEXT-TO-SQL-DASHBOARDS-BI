---
name: members-policies-medicare
description: Total de miembros y total de pólizas de la línea de negocio Medicare, año en curso
domain: medicare
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, medicare, bob]
---

# Members / Policies Numbers — Medicare

## Cuándo usar esta consulta

Responde: "¿cuántos miembros y cuántas pólizas de Medicare tenemos, con fecha de vigencia
dentro del año en curso y hasta hoy?"

Misma estructura que `queries/life/annual-premium-members-life.md`, pero para Medicare y
con `COUNT` de pólizas en vez de suma de prima. Para otra LOB, cambiar
`o.Line_Of_Business IN ('Medicare')`.

## SQL

```sql
SELECT
  SUM(b.Members__c) as total_members,
  COUNT(b.Policy_Number__c) as total_policies
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE
    b.Status__c NOT IN ('NR', 'Terminated')
    AND b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
    AND a.Name_Agencies IS NOT NULL
    AND a.Name_Agencies != 'Agency Test'
    AND o.Line_Of_Business IN ('Medicare')
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
```

## Notas / supuestos

- No filtra `Val_BOB_TD = 1` — **intencional**, a diferencia de `members-policies-aca.md` y
  `agents-with-active-contracts-and-production.md`, que sí lo exigen.
- Filtra por año calendario en curso (`EXTRACT(YEAR FROM CURRENT_DATE)`), no por mes de
  corte tipo `LAST_DAY`.
