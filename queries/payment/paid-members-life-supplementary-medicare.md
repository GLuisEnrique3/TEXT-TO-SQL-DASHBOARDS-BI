---
name: paid-members-life-supplementary-medicare
description: Total de miembros pagados (Paid Members) de Life, Supplementary y Medicare, por mes de comisión (Commission Month) — pagos principales de Payment_Commission filtrados por línea de negocio
domain: payment
tables:
  - claro_bi.Payment_Commission
  - claro_bi.dim_cslb
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [members, payment, life, supplementary, medicare, paid]
---

# Paid Members — Life + Supplementary + Medicare

## Cuándo usar esta consulta

Responde: "¿cómo ha evolucionado mes a mes la cantidad de miembros pagados de Life,
Supplementary y Medicare?" — replica la medida `Paid Members` del dashboard de Power BI
(`BOB History.pbix`), filtrada a Life **+** Supplementary **+** Medicare combinadas (no
separadas), y desglosada por `Commission_Month__c`. Cuenta `Members_Paid__c` solo de las
filas marcadas como pago principal (`Main_Payment__c = TRUE`), sin filtrar por
`Payment_Type__c` (a diferencia de
`queries/payment/paid-members-history-comission-aca.md`, que sí filtra por
`Payment_Type__c = 'Commission'` porque responde una pregunta más específica). Para separar
una línea de negocio de las demás, cambia
`o.Line_Of_Business IN ('Life','Supplementary','Medicare')` por el valor único deseado, o
agrega `o.Line_Of_Business` al `SELECT`/`GROUP BY` para desglosar las tres en la misma
consulta.

## SQL

```sql
SELECT
  p.Commission_Month__c,
  COALESCE(SUM(p.Members_Paid__c), 0) AS paid_members
FROM `claroinsurance-dataplatform.claro_bi.Payment_Commission` p
JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o
  ON o.Id = p.Carrier_State_line_of_Business__c
WHERE p.Main_Payment__c = TRUE
  AND p.Commission_Month__c != DATE '2000-01-01'
  AND o.Line_Of_Business IN ('Life','Supplementary','Medicare')
GROUP BY p.Commission_Month__c
ORDER BY p.Commission_Month__c DESC
```

## Notas / supuestos

- Aplica la regla 7 de `context/business-rules.md` (excluir fecha centinela `2000-01-01`).
- No aplica exclusión de `Agency Test` — la medida original del dashboard (`Paid Members`) no
  se une a `dim_account_2` ni filtra agencia; si se necesita esa exclusión, agregar el join a
  `dim_contact` → `dim_account_2` y el filtro estándar (regla 3).
- **Life, Supplementary y Medicare van combinadas** en esta versión — si el usuario pide solo
  una de las tres, no asumas que puedes filtrar el resultado combinado; vuelve a ejecutar con
  `o.Line_Of_Business = 'Life'` (o `'Supplementary'`/`'Medicare'`) para esa línea específica.
- `queries/payment/ap-of-paid-policies-life-supplementary.md` (AP de pólizas pagadas) todavía
  solo cubre Life + Supplementary — no incluye Medicare todavía; avisar si se necesita
  ampliarla igual que esta consulta.
