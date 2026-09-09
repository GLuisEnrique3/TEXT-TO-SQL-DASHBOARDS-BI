---
name: schema-payment_commission
description: Ficha de la tabla claro_bi.Payment_Commission (pagos/comisiones por póliza y mes)
---

# `claro_bi.Payment_Commission`

**Tipo:** tabla
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `p`

## Descripción

Pagos de comisiones — una fila por pago/tipo de pago asociado a una póliza en un mes de
comisión determinado (`Commission_Month__c`). Una misma póliza puede tener varias filas
(distintos meses, distintos `Payment_Type__c`).

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK |
| `Policy_Number__c` | Número de póliza. FK (por valor) hacia `BOB_TD.Policy_Number__c` / `BOB_H_TD.Policy_Number__c` |
| `Carrier_State_line_of_Business__c` | FK hacia `dim_cslb.Id` — mismo patrón que `BOB_TD` |
| `Contact__c` | FK hacia `dim_contact.Id` — agente asociado a la póliza |
| `Contact_pay` | FK (por valor) hacia `dim_contact.Id` — **no confundir con `Contact__c`**: es el contacto que efectivamente recibe el pago, puede diferir del agente de la póliza (ej. pago a través de una agencia/tercero). Usado en `queries/agents/agents-with-activation-commission.md` |
| `Commission_Month__c` | Mes al que se **atribuye** el pago (mes contable/de origen). **Contiene filas centinela con valor `2000-01-01`** — ver gotcha abajo. Es la columna que usan todas las consultas verificadas de `queries/payment/` para agrupar por mes |
| `Pay_on_Date__c` | Fecha en que el pago **se desembolsó realmente** — no confundir con `Commission_Month__c`. Ver gotcha abajo sobre la diferencia entre ambas |
| `Main_Payment__c` | BOOLEAN. `TRUE` = pago principal de la póliza en ese mes. Usado para filtrar `Paid Members`/`Paid Policies` (ver `context/business-rules.md` regla 7) |
| `Members_Paid__c` | Miembros pagados en ese registro de pago |
| `Payment_Type__c` | Tipo de pago (ej. `'Commission'`, `'Override'`, `'Bonus'`, ...) |
| `Payment_Status__c` | STRING. Valores confirmados en BigQuery (2026-09-09): `'Paid'` (6.4M), `'Paid by carrier'` (793K), `'On-Hold'` (409K), `'Internal Offset'` (243K), `'Not paid'` (241K), `'Pending'` (30K) |
| `Net_Payment__c` | Monto neto pagado |
| `Pre_Approved__c` | STRING. Valores confirmados en BigQuery (2026-09-09): `'Yes'` (6.6M filas), `'No'` (661K), `'Pb'` (810K — significado sin confirmar, no asumir qué representa). Usado como segmentador en el dashboard (ver `context/filters/bob-history-dashboard-filters.md`, página Overview) |

## Joins típicos

| Join hacia | Condición | Para qué |
|---|---|---|
| `claro_bi.dim_cslb` (alias `o`) | `o.Id = p.Carrier_State_line_of_Business__c` | Resolver línea de negocio, carrier, estado |
| `claro_bi.dim_contact` (alias `c`) | `c.Id = p.Contact__c` | Llegar al agente y, vía `AccountId`, a la agencia/cuenta |
| `claro_bi.BOB_TD` / `BOB_H_TD` (alias `b`) | `b.Policy_Number__c = p.Policy_Number__c` | Traer atributos de la póliza que no están en `Payment_Commission` (ej. `Annual_Premium__c` — ver gotcha de "AP de pólizas pagadas") |

## Gotchas / notas

- **No tiene columna `Annual_Premium__c` propia** — para calcular "AP de pólizas pagadas" hay
  que traerla desde `BOB_TD`/`BOB_H_TD` por `Policy_Number__c`, tomando la prima del
  snapshot más reciente con valor distinto de cero (ver
  `queries/payment/ap-of-paid-policies-life-supplementary.md` y regla 8 de `context/business-rules.md`) — no
  sumar la prima repetida por cada fila de pago, se duplicaría.
- **Filas centinela `Commission_Month__c = '2000-01-01'`**: excluir siempre (ver regla 7 de
  `context/business-rules.md`).
- **`Commission_Month__c` vs `Pay_on_Date__c`**: nunca coinciden (confirmado 2026-09-09, 0 de
  8.1M filas iguales). `Pay_on_Date__c` suele quedar 30-90 días **después** de
  `Commission_Month__c` (el pago se desembolsa uno o más meses después del mes al que se
  atribuye), con outliers extremos (diferencia observada de hasta 1,917 días, y algunos casos
  negativos) — investigar esos casos puntuales antes de confiar en `Pay_on_Date__c` para algo
  crítico. Usa `Commission_Month__c` para "¿cuánto se generó/atribuyó ese mes?" (lo que usan
  todas las consultas verificadas hoy) y `Pay_on_Date__c` solo si la pregunta es
  específicamente sobre flujo de caja real (cuándo se pagó de verdad).
- Una póliza puede tener varias filas por `Payment_Type__c` (Commission, Override, Bonus,
  ...) en el mismo `Commission_Month__c` — al contar/sumar, decidir si se necesita filtrar
  por `Payment_Type__c` según la pregunta de negocio (ver
  `queries/payment/paid-members-history-comission-aca.md`, que sí filtra
  `Payment_Type__c = 'Commission'`, vs. `Paid Members`/`Paid Policies` del dashboard, que no
  filtran por tipo — solo por `Main_Payment__c = TRUE`).
