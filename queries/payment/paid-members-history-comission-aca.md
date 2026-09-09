---
name: paid-members-history-comission-aca
description: Miembros pagados de comisión (Payment_Type__c = 'Commission') de ACA, por mes de comisión
domain: payment
tables:
  - claro_bi.Payment_Commission
  - claro_bi.dim_cslb
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [members, payment, commission, aca, history]
---

# Paid Members (Commission) — History ACA

## Cuándo usar esta consulta

Responde: "¿cómo ha evolucionado mes a mes los miembros pagados **específicamente por
comisión** (`Payment_Type__c = 'Commission'`) en ACA?" — a diferencia de
`queries/payment/paid-members-life-supplementary-medicare.md` (que cuenta todo pago principal
sin filtrar tipo de pago), esta consulta se limita al tipo de pago "Commission".

## SQL

```sql
SELECT
  p.Commission_Month__c,
  SUM(p.Members_Paid__c) AS members_paid
FROM `claroinsurance-dataplatform.claro_bi.Payment_Commission` p
JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o
  ON o.Id = p.Carrier_State_line_of_Business__c
WHERE p.Main_Payment__c = TRUE
  AND p.Payment_Type__c = 'Commission'
  AND p.Commission_Month__c != DATE '2000-01-01'
  AND o.Line_Of_Business = 'ACA'
  AND EXTRACT(YEAR FROM p.Commission_Month__c) IN (2025, 2026)
GROUP BY p.Commission_Month__c
ORDER BY p.Commission_Month__c DESC
```

## Notas / supuestos

- Aplica la regla 7 de `context/business-rules.md` (excluir fecha centinela `2000-01-01`) —
  antes solo quedaba excluida implícitamente por el filtro de año; ahora es explícita para
  que siga funcionando aunque se quite o cambie ese filtro.
- **`EXTRACT(YEAR ...) IN (2025, 2026)`** es un filtro de alcance ajustable — amplíalo o
  quítalo si se necesita el histórico completo.
- La versión original de este archivo traía `LEFT JOIN` adicionales a `dim_contact`,
  `dim_account_2` y `BOB_TD` sin usar ninguna de sus columnas. Se verificaron contra BigQuery
  (mismo resultado con y sin esos joins, para junio 2026) y se quitaron por ser peso muerto —
  si en el futuro se necesita desglosar por agente/agencia, vuelve a agregarlos con sus
  columnas correspondientes en el `SELECT`/`GROUP BY`.
- No aplica exclusión de `Agency Test` — no hay join a `dim_account_2` en esta versión
  simplificada (ver punto anterior).
