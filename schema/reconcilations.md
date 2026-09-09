---
name: schema-reconcilations
description: Ficha de la tabla claro_bi.reconcilations (reconciliaciones de comisiones/pólizas con carriers)
---

# `claro_bi.reconcilations`

**Tipo:** tabla
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `r1`

> Nota: el nombre de la tabla en BigQuery es `reconcilations` (sin la primera "i" de
> "reconciliations") — es el nombre real, no un typo de este repo. Reprodúcelo tal cual.

## Descripción

Reconciliaciones de pólizas/comisiones contra carriers — una fila por caso de reconciliación.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | PK |
| `Reconciliations` | Tipo/resultado de la reconciliación. FK hacia `dim_reconciliation.Type_reconciliations` |
| `contact__c` | FK hacia `dim_contact.Id` |
| `Date_to_Claim__c` | Fecha del reclamo. FK hacia `dim_calendar.Date` |
| `CarrierStateLineOfBusiness__c` | FK hacia `dim_cslb.Id` — mismo patrón que `BOB_TD.Carrier_State_line_of_Business__c`, úsalo para resolver línea de negocio vía `dim_cslb` (no el nombre similar `Carrier_State_line_of_Business__c` de otras tablas) |
| `Lines_of_Business__c` | Línea de negocio **denormalizada** (texto directo, sin join). Alternativa más corta a unir con `dim_cslb`, pero no confirmado si siempre coincide — preferir el join a `dim_cslb` por consistencia con el resto del repo salvo que se confirme equivalencia |
| `Status__c` | Estado de la reconciliación |
| `Disposition__c` | Disposición/resolución del caso |
| `Members_Owed__c` | Miembros adeudados |
| `Policy_ID__c` | Número de póliza asociado |
| `Agency__c` | Agencia (texto, no confirmado si es FK a `dim_account_2`) |

## Joins típicos

| Join hacia | Condición | Para qué |
|---|---|---|
| `claro_bi.dim_reconciliation` (alias `r2`) | `r2.Type_reconciliations = r1.Reconciliations` | Resolver el tipo/resultado de la reconciliación (ej. `'Won'`) |
| `claro_bi.dim_cslb` (alias `o`) | `o.Id = r1.CarrierStateLineOfBusiness__c` | Resolver línea de negocio, carrier, estado |
| `claro_bi.dim_contact` (alias `c`) | `c.Id = r1.contact__c` | Llegar al agente y, vía `AccountId`, a la agencia/cuenta |
| `claro_bi.dim_calendar` | `dim_calendar.Date = r1.Date_to_Claim__c` | Análisis por fecha de reclamo |

## Gotchas / notas

- El join hacia `dim_reconciliation` es por **valor de texto** (`Reconciliations` = nombre de
  columna Y de tabla a la vez, cuidado al leer el SQL), no por Id.
- Aún sin confirmar con el equipo si `Lines_of_Business__c` (denormalizada) coincide siempre
  con `dim_cslb.Line_Of_Business` vía `CarrierStateLineOfBusiness__c` — no asumir
  equivalencia sin verificar.
- **`Date_to_Claim__c`** es TIMESTAMP pero en la práctica ya viene truncado a mes (día 1,
  00:00:00) — no hace falta `DATE_TRUNC` para agrupar por mes. Tiene una cantidad marginal
  (6 de 239,052 filas, confirmado 2026-09-09) de fechas placeholder tipo `1899-12-30` (cero
  de Excel) y `2039-04-01` — insignificante para la mayoría de análisis, pero puede explicar
  un pico raro si aparece en una serie de tiempo.
