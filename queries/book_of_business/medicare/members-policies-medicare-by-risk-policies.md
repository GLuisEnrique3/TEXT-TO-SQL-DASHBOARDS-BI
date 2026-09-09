---
name: members-policies-medicare-by-risk-policies
description: Miembros y pólizas de Medicare en estados de riesgo (Binder Payment, Late Payment), por estado de póliza
domain: medicare
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_policy_status
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, medicare, bob, risk, by-status]
---

# Members / Policies Numbers — Medicare Pólizas en Riesgo

## Cuándo usar esta consulta

Responde: "¿cuántos miembros y pólizas de Medicare están en un estado de riesgo (pago
pendiente o atrasado), desglosado por ese estado?" A diferencia de
`members-policies-medicare.md`, **no** excluye `NR`/`Terminated` — en su lugar filtra
explícitamente a solo los estados de riesgo. Equivalente a
`queries/book_of_business/aca/members-policies-aca-by-risk-policies.md` pero para Medicare.

## SQL

```sql
SELECT
  ps.Status,
  SUM(b.Members__c) as total_members,
  COUNT(b.Policy_Number__c) as total_policies
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_policy_status` ps ON ps.Status = b.Status__c
WHERE
    b.Status__c IN ('Binder Payment','Late Payment')
    AND b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
    AND a.Name_Agencies IS NOT NULL
    AND a.Name_Agencies != 'Agency Test'
    AND o.Line_Of_Business IN ('Medicare')
    AND EXTRACT(YEAR FROM b.Effective_Date__c) = EXTRACT(YEAR FROM CURRENT_DATE)
    AND b.Effective_Date__c <= CURRENT_DATE
GROUP BY ps.Status
```

## Notas / supuestos

- **Importante:** `b.Status__c IN ('Binder Payment','Late Payment')` reemplaza al filtro
  `NOT IN ('NR','Terminated')` de la consulta base — universo distinto (pólizas "en
  riesgo"), no un desglose de `members-policies-medicare.md`.
