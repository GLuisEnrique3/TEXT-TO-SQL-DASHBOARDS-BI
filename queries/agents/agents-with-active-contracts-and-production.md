---
name: agents-with-active-contracts-and-production
description: Conteo de agentes distintos con producción vigente y válida en el Book of Business
domain: agents
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [agents, contracts, production, bob]
---

# Agents with Active Contracts and Production

## Cuándo usar esta consulta

Responde: "¿cuántos agentes distintos tienen producción real y vigente?" (a diferencia de
`agents-with-active-contracts.md`, que solo mira si existe un contrato registrado, esta
consulta exige que el agente tenga al menos una póliza válida y vigente en `BOB_TD`).

## SQL

```sql
SELECT COUNT(DISTINCT b.Contact__c)
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o ON o.Id = b.Carrier_State_line_of_Business__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
AND b.Val_BOB_TD = 1
AND b.Status__c NOT IN ('NR','Terminated')
AND a.Name_Agencies != 'Agency Test'
AND a.Name_Agencies IS NOT NULL
AND b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
```

## Notas / supuestos

- Aplica las mismas reglas estándar de `context/business-rules.md` que
  `members-policies-aca.md` (`Val_BOB_TD = 1`, exclusión de `NR`/`Terminated`, exclusión de
  `Agency Test`, vigencia hasta fin de mes), pero sin filtrar por línea de negocio — cuenta
  agentes con producción en cualquier LOB.
- El join a `dim_cslb` (alias `o`) no se usa en `WHERE` ni `SELECT`; queda disponible para
  desglosar por línea de negocio si se necesita.
