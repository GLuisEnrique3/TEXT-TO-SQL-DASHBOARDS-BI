---
name: ap-of-paid-policies-life-supplementary
description: Prima anual (AP) de las pólizas de Life y Supplementary que tienen al menos un pago principal con miembros pagados, por mes de comisión (Commission Month)
domain: payment
tables:
  - claro_bi.Payment_Commission
  - claro_bi.dim_cslb
  - claro_bi.BOB_H_TD
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [ap, premium, payment, life, supplementary, paid]
---

# AP of Paid Policies — Life + Supplementary

## Cuándo usar esta consulta

Responde: "¿cómo ha evolucionado mes a mes la prima anual (AP) de las pólizas de Life y
Supplementary que sí tuvieron pago?" — replica la medida `AP of Paid Policies` del dashboard
de Power BI (`BOB History.pbix`), filtrada a Life **y** Supplementary combinadas (no
separadas), y desglosada por `Commission_Month__c` (igual que
`queries/payment/paid-members-life-supplementary.md`). Para separar Life de Supplementary, cambia
`o.Line_Of_Business IN ('Life','Supplementary')` por el valor único deseado, o agrega
`o.Line_Of_Business` al `SELECT`/`GROUP BY` para desglosar ambas en la misma consulta.

⚠️ **Ojo con la unidad de conteo**: el DAX original
(`SUM(Payments[Annual_Premium_c])`, filtrado solo por `[Paid Members]>0` a nivel de póliza,
**sin** filtrar `Main_Payment__c` en el `SUM` externo) suma la prima **por cada fila de pago**
de la póliza en ese mes, no una vez por póliza. Como `Annual_Premium_c` es una columna
calculada (`RELATED('AP BOB'[Annual Premium])`) que repite el mismo valor de prima en cada
fila, una póliza con varios `Payment_Type__c` en el mismo mes (ej. Commission + Bonus) cuenta
su prima **una vez por cada fila**.

## SQL

```sql
WITH paid_policies AS (
  SELECT p.Policy_Number__c, p.Commission_Month__c
  FROM `claroinsurance-dataplatform.claro_bi.Payment_Commission` p
  JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o
    ON o.Id = p.Carrier_State_line_of_Business__c
  WHERE p.Commission_Month__c != DATE '2000-01-01'
    AND o.Line_Of_Business IN ('Life','Supplementary')
  GROUP BY p.Policy_Number__c, p.Commission_Month__c
  HAVING SUM(CASE WHEN p.Main_Payment__c = TRUE THEN p.Members_Paid__c ELSE 0 END) > 0
),
policy_ap AS (
  SELECT
    b.Policy_Number__c,
    ARRAY_AGG(b.Annual_Premium__c ORDER BY b.fecha_carga DESC LIMIT 1)[OFFSET(0)] AS Annual_Premium__c
  FROM `claroinsurance-dataplatform.claro_bi.BOB_H_TD` b
  WHERE b.fecha_carga != DATE '2000-01-01'
    AND b.Annual_Premium__c != 0
  GROUP BY b.Policy_Number__c
)
SELECT
  p.Commission_Month__c,
  COALESCE(SUM(policy_ap.Annual_Premium__c), 0) AS ap_of_paid_policies
FROM `claroinsurance-dataplatform.claro_bi.Payment_Commission` p
JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o
  ON o.Id = p.Carrier_State_line_of_Business__c
JOIN paid_policies pp
  ON pp.Policy_Number__c = p.Policy_Number__c
  AND pp.Commission_Month__c = p.Commission_Month__c
JOIN policy_ap
  ON policy_ap.Policy_Number__c = p.Policy_Number__c
WHERE p.Commission_Month__c != DATE '2000-01-01'
  AND o.Line_Of_Business IN ('Life','Supplementary')
GROUP BY p.Commission_Month__c
ORDER BY p.Commission_Month__c DESC
```

## Notas / supuestos

- **Sí duplica la prima por fila de pago, a propósito** — es el comportamiento real de la
  medida `AP of Paid Policies` del dashboard, no un error. Si en algún momento se necesita la
  versión "correcta" sin duplicar (prima una sola vez por póliza), quitar el `JOIN` directo a
  `Payment_Commission`/`dim_cslb` en el `SELECT` final y unir `paid_policies` → `policy_ap`
  directo.
- `policy_ap` replica exactamente la tabla calculada `AP BOB` del modelo de Power BI: la
  prima anual del snapshot (`fecha_carga`) más reciente **con valor distinto de cero** por
  póliza, desde `BOB_H_TD` (no `BOB_TD`). No se recalcula por mes — es la prima más reciente
  conocida de la póliza en general, igual que `AP BOB` (no depende del mes reportado).
- `paid_policies` agrupa por `Policy_Number__c` **y** `Commission_Month__c` para que "pago
  principal con miembros pagados" (`Main_Payment__c = TRUE`) se evalúe mes a mes — mismo
  criterio que `queries/payment/paid-members-life-supplementary.md`.
- Aplica la regla 7 de `context/business-rules.md` (excluir fecha centinela `2000-01-01`) en
  ambas tablas.
- No aplica exclusión de `Agency Test` (igual que la medida original del dashboard, que no se
  une a `dim_account_2`).
- **Life y Supplementary van combinadas** en esta versión — si el usuario pide solo una de las
  dos, no asumas que puedes filtrar el resultado combinado; vuelve a ejecutar con
  `o.Line_Of_Business = 'Life'` (o `'Supplementary'`) para esa línea específica.
