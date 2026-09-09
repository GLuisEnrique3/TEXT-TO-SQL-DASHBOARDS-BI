---
name: bob-history-dashboard-filters
description: Catálogo de segmentadores (slicers) del dashboard BOB History.pbix, mapeados a columna/tabla y a su cadena de join
metadata:
  type: reference
---

# Segmentadores del Dashboard — BOB History.pbix

> Este archivo documenta específicamente el dashboard **BOB History.pbix**. Para
> Producers.pbix ver `context/filters/producers-dashboard-filters.md`.

Por ahora solo cubre la página **Overview**. Las demás páginas (By Agents, By Carrier, By
State, Activation, Details, análisis, Manto., FB, HB1) quedan pendientes de revisar.

## Particularidad de este dashboard: dos tablas de hechos

A diferencia de Producers.pbix (una sola tabla de hechos, `Book_of_Business`/`BOB_TD`),
BOB History.pbix combina dos:

- **`Book_of_Business`** → mapea a `claro_bi.BOB_H_TD` (histórico de snapshots, ver
  `schema/BOB_H_TD.md`).
- **`Payments`** → mapea a `claro_bi.Payment_Commission`, filtrada para excluir la fecha
  centinela `2000-01-01` (ver `context/business-rules.md` regla 7 y `schema/Payment_Commission.md`).

Algunos segmentadores de Overview filtran una tabla, otros la otra (o ambas, vía las
dimensiones compartidas). Al traducir un filtro de esta página a SQL, primero identifica cuál
de las dos tablas de hechos aplica a la métrica que estás calculando.

## Overview

| Slicer (campo) | Tabla que filtra | Cadena de join |
|---|---|---|
| `dim_cslb[Line of Business]` | `BOB_H_TD` y `Payment_Commission` | `dim_cslb.Id = <fact>.Carrier_State_line_of_Business__c` (ambas tienen esa FK) |
| `dim_cslb[Carrier]` | ídem | ídem |
| `dim_cslb[State]` | ídem | ídem |
| `dim_company[Name]` | `BOB_H_TD` (vía `dim_cslb`) | `dim_company.Id = dim_cslb.Internal_Company_Object__c` → `dim_cslb` → `BOB_H_TD` (3 saltos) |
| `dim_contact[Agent_Specialist__c]` | `BOB_H_TD` / `Payment_Commission` | `dim_contact.Id = <fact>.Contact__c` — columna propia del contacto/agente |
| `dim_account_(2)[Agent_Specialist__c]` | ídem | `dim_account_2.Id = dim_contact.AccountId` → `dim_contact` → `<fact>` (2 saltos) — columna de la agencia, **no confundir con la anterior** (mismo nombre, tabla distinta) |
| `dim_account_(2)[Name Agencies]` | ídem | ídem (2 saltos) |
| `dim_contact[Contact_Full_name_Formula__c]` | `BOB_H_TD` / `Payment_Commission` | `dim_contact.Id = <fact>.Contact__c` |
| `dim_contact[GRManager__c]` | ídem | ídem |
| `dim_policy_status[Status_Category]` | `BOB_H_TD` | `dim_policy_status.Status__c = BOB_H_TD.Status__c` (por valor) |
| `Payments[Pre_Approved__c]` | `Payment_Commission` | columna propia, sin join |
| `Payments[Payment_Status__c]` | `Payment_Commission` | columna propia, sin join |

## Gotchas / notas

- **`Agent_Specialist__c` existe en dos tablas** con el mismo nombre pero significado
  distinto: `dim_contact.Agent_Specialist__c` (especialista del agente) vs.
  `dim_account_2.Agent_Specialist__c` (especialista de la agencia/cuenta). Ver
  `schema/dim_contact.md`.
- Ninguno de los 12 segmentadores filtra por fecha (`Commission_Month__c` / `fecha_carga`) —
  para recortes de tiempo, este dashboard probablemente usa otro control fuera de los
  slicers estándar (pendiente de revisar).
- Todas las relaciones son de un solo sentido hacia la(s) tabla(s) de hechos (equivalente a
  `LEFT JOIN`), igual que en Producers.pbix.
