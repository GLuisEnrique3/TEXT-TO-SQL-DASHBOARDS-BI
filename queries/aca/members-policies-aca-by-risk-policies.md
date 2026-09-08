---
name: members-policies-aca-by-risk-policies
description: Miembros y pólizas de ACA en estados de riesgo (Binder Payment, Late Payment), por estado de póliza
domain: aca
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.dim_policy_status
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [members, policies, aca, bob, risk, by-status]
---

# Members / Policy Numbers — ACA Pólizas en Riesgo

## Cuándo usar esta consulta

Responde: "¿cuántos miembros y pólizas de ACA están en un estado de riesgo (pago pendiente
o atrasado), desglosado por ese estado?" A diferencia de `members-policies-aca.md`, **no**
excluye `NR`/`Terminated` — en su lugar filtra explícitamente a solo los estados de riesgo.

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
      AND b.Status__c IN ('Binder Payment','Late Payment')
      AND a.Name_Agencies != 'Agency Test'
      AND a.Name_Agencies IS NOT NULL
      AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
      AND o.Line_Of_Business = 'ACA'
GROUP BY p.Status
```

## Notas / supuestos

- **Importante:** `b.Status__c IN ('Binder Payment','Late Payment')` reemplaza al filtro
  estándar `NOT IN ('NR','Terminated')` de `context/business-rules.md` regla 2 — esta
  consulta es intencionalmente un universo distinto (pólizas "en riesgo"), no un desglose
  de `members-policies-aca.md`.
- Añadir aquí en `context/glossary.md` (pendiente) qué significan exactamente `'Binder
  Payment'` y `'Late Payment'` como estados de riesgo de cobro.
- Para otra LOB, cambiar `o.Line_Of_Business = 'ACA'`.
