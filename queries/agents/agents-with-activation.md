---
name: agents-with-activation
description: Agentes nuevos activados por mes (primera vez con una póliza en Book_of_Business con estado productivo), por mes de activación
domain: agents
tables:
  - claro_bi.BOB_H_TD
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [agents, activation, history]
---

# Agents with Activation

## Cuándo usar esta consulta

Responde: "¿cuántos agentes nuevos se activaron cada mes?" — replica la medida
`Agents with Activation` de la página "Activation" del dashboard `BOB History.pbix`. Un
agente se considera "activado" en el mes de la fecha de carga (`fecha_carga`) más antigua en
la que tuvo una póliza con estado productivo (no `NR`/`Terminated`), sin importar la línea de
negocio. Es la variante basada en `Book_of_Business` (producción) — para la variante basada
en pagos/comisión ver `queries/agents/agents-with-activation-commission.md`.

El DAX original calcula esto restando dos "universos acumulados" (`Agents Universo` -
`Agents Universo -1`) porque el modelo no usa `DATESBETWEEN`. En SQL no hace falta ese rodeo:
se calcula directo la fecha de primera activación de cada agente y se agrupa por mes — el
resultado es equivalente.

## SQL

```sql
WITH activation AS (
  SELECT
    b.Contact__c,
    b.Carrier_State_line_of_Business__c,
    MIN(b.fecha_carga) AS activation_date
  FROM `claroinsurance-dataplatform.claro_bi.BOB_H_TD` b
  WHERE b.fecha_carga != DATE '2000-01-01'
    AND b.Status__c NOT IN ('NR','Terminated')
  GROUP BY b.Contact__c, b.Carrier_State_line_of_Business__c
),
agent_first_activation AS (
  SELECT
    Contact__c,
    MIN(activation_date) AS first_activation_date
  FROM activation
  GROUP BY Contact__c
)
SELECT
  DATE_TRUNC(first_activation_date, MONTH) AS activation_month,
  COUNT(DISTINCT Contact__c) AS agents_with_activation
FROM agent_first_activation
GROUP BY activation_month
ORDER BY activation_month DESC
```

## Notas / supuestos

- Replica la tabla calculada `Activatión` del modelo de Power BI: por cada combinación
  (`Contact__c`, `Carrier_State_line_of_Business__c`), la fecha de carga mínima entre las
  filas con `Status__c` fuera de `('NR','Terminated')`. Luego, para el agente (colapsando
  todas sus líneas de negocio), se toma la fecha mínima de todas sus combinaciones — esa es
  su "fecha de activación" real.
- Aplica la regla 7 de `context/business-rules.md` (excluir fecha centinela `2000-01-01`).
- Usa `BOB_H_TD` (histórico), no `BOB_TD` — necesita ver todas las fechas de carga pasadas
  para encontrar la primera.
- No aplica exclusión de `Agency Test` ni filtro de línea de negocio — la medida original no
  los tiene. Si se necesita filtrar por LOB, únete a `dim_cslb` por
  `Carrier_State_line_of_Business__c` y agrega el filtro antes del `GROUP BY Contact__c` del
  CTE `agent_first_activation` (ojo: cambiaría la semántica a "primera activación en esa LOB
  específica", no la primera activación general del agente).
