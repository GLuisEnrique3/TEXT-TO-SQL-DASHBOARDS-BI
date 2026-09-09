---
name: reconciliations-type-lob
description: Cantidad de reconciliaciones Won/Lost por mes (Date_to_Claim__c), para las 4 líneas de negocio principales
domain: payment
tables:
  - claro_bi.reconcilations
  - claro_bi.dim_reconciliation
  - claro_bi.dim_cslb
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [reconciliations, payment, aca, life, medicare, supplementary]
---

# Reconciliations — Won/Lost por línea de negocio

## Cuándo usar esta consulta

Responde: "¿cuántas reconciliaciones se ganaron o perdieron cada mes, en ACA/Life/Medicare/
Supplementary?" — cuenta filas de `reconcilations` con resultado `'Won'` o `'Lost'`, agrupadas
por `Date_to_Claim__c`, filtradas a las 4 líneas de negocio principales (excluye
`'ACA - ICHRA'`, `'Discount Medical Plans'` y `'No Assigned'`, que también existen en
`dim_cslb.Line_Of_Business` pero no están incluidas aquí a propósito).

## SQL

```sql
SELECT
  r1.Date_to_Claim__c,
  COUNT(*) AS reconciliations
FROM `claroinsurance-dataplatform.claro_bi.reconcilations` r1
JOIN `claroinsurance-dataplatform.claro_bi.dim_reconciliation` r2
  ON r1.Reconciliations = r2.Type_reconciliations
JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o
  ON r1.CarrierStateLineOfBusiness__c = o.Id
WHERE r2.Type_reconciliations IN ('Won','Lost')
  AND o.Line_Of_Business IN ('ACA','Life','Medicare','Supplementary')
GROUP BY r1.Date_to_Claim__c
ORDER BY r1.Date_to_Claim__c DESC
```

## Notas / supuestos

- **`Date_to_Claim__c` es tipo TIMESTAMP pero en la práctica ya viene truncado a mes** (todos
  los valores observados caen en el día 1 del mes, a las 00:00:00 — 471 valores distintos de
  timestamp = 471 valores distintos de fecha). Por eso `GROUP BY` directo sobre la columna
  funciona igual que si se truncara con `DATE_TRUNC`; no hace falta agregar ese cast.
- El filtro `o.Line_Of_Business IN ('ACA','Life','Medicare','Supplementary')` **sí excluye
  datos reales**: `dim_cslb` también tiene los valores `'ACA - ICHRA'` (26 filas),
  `'Discount Medical Plans'` (1 fila) y `'No Assigned'` (1 fila). Si se necesita incluir
  `ACA - ICHRA` dentro de "ACA", agrégalo explícitamente al `IN`.
- **Gotcha de calidad de datos, no filtrada aquí por ser marginal**: `Date_to_Claim__c` tiene
  6 filas (de 239,052) con fechas placeholder típicas de sistemas legados —
  `1899-12-30` (5 filas, el "cero" de fechas de Excel) y `2039-04-01` (1 fila, posible "sin
  fecha definida"). No se excluyen por ser un volumen insignificante, pero si aparecen picos
  raros en la serie de tiempo, revisar si crecieron.
