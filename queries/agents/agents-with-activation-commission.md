---
name: agents-with-activation-commission
description: Agentes nuevos activados por mes (primera vez con un pago principal de comisión), por mes de activación
domain: agents
tables:
  - claro_bi.Payment_Commission
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [agents, activation, payment, commission]
---

# Agents with Activation (Commission) — "_C"

## Cuándo usar esta consulta

Responde: "¿cuántos agentes nuevos recibieron su primer pago de comisión cada mes?" — replica
la medida `Agents with Activation_C` de la página "Activation" del dashboard
`BOB History.pbix`. A diferencia de `queries/agents/agents-with-activation.md` (que mide la
primera póliza productiva en `Book_of_Business`), esta mide la primera vez que un agente
recibió un **pago principal de comisión** (`Main_Payment__c = TRUE`), usando
`Contact_pay` (no `Contact__c`) como identificador del agente — confirmar con el equipo si
`Contact_pay` y `Contact__c` deberían ser siempre la misma persona o pueden diferir (ej. pago
a un tercero/agencia en nombre del agente).

Igual que en la variante base, el DAX original resta dos universos acumulados por no usar
`DATESBETWEEN`; en SQL se calcula directo la fecha de primera activación por comisión y se
agrupa por mes.

## SQL

```sql
WITH activation_c AS (
  SELECT
    p.Contact_pay,
    p.Carrier_State_line_of_Business__c,
    MIN(p.Commission_Month__c) AS activation_month_raw
  FROM `claroinsurance-dataplatform.claro_bi.Payment_Commission` p
  WHERE p.Commission_Month__c != DATE '2000-01-01'
    AND p.Main_Payment__c = TRUE
  GROUP BY p.Contact_pay, p.Carrier_State_line_of_Business__c
),
agent_first_activation_c AS (
  SELECT
    Contact_pay,
    MIN(activation_month_raw) AS first_activation_month
  FROM activation_c
  GROUP BY Contact_pay
)
SELECT
  first_activation_month AS activation_month,
  COUNT(DISTINCT Contact_pay) AS agents_with_activation_commission
FROM agent_first_activation_c
GROUP BY activation_month
ORDER BY activation_month DESC
```

## Notas / supuestos

- Replica la tabla calculada `Activatión_C` del modelo de Power BI: por cada combinación
  (`Contact_pay`, `Carrier_State_line_of_Business__c`), el `Commission_Month__c` mínimo entre
  las filas con `Main_Payment__c = TRUE`. Luego, para el agente (colapsando todas sus líneas
  de negocio), se toma el mes mínimo de todas sus combinaciones.
- `Commission_Month__c` ya viene truncado al mes (primer día del mes) en el dato fuente — no
  hace falta `DATE_TRUNC` como en la variante base (que usa `fecha_carga`, una fecha diaria).
- Aplica la regla 7 de `context/business-rules.md` (excluir fecha centinela `2000-01-01`).
- No aplica exclusión de `Agency Test` ni filtro de línea de negocio — la medida original no
  los tiene.
