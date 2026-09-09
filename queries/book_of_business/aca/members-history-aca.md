---
name: members-history-aca
description: Tendencia mensual de miembros ACA vigentes, a partir de los snapshots históricos de BOB_H_TD (una fila por fecha de carga)
domain: aca
tables:
  - claro_bi.BOB_H_TD
  - claro_bi.dim_cslb
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
verified_by: Enrique Guerra
verified_date: 2026-09-09
tags: [members, history, trend, aca]
---

# Members History ACA

## Cuándo usar esta consulta

Responde: "¿cómo ha evolucionado mes a mes la cantidad de miembros de ACA?" — una fila por
fecha de carga (`fecha_carga`), con el total de miembros vigentes a esa fecha. A diferencia
de las consultas de `queries/book_of_business/aca/members-policies-aca*.md` (que responden un corte puntual a
hoy sobre `BOB_TD`), esta usa `BOB_H_TD`, que guarda un snapshot histórico por cada carga, y
construye una serie de tiempo real.

## SQL

```sql
SELECT
  b.fecha_carga,
  SUM(b.Members__c) AS members
FROM `claroinsurance-dataplatform.claro_bi.BOB_H_TD` b
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` o
  ON b.Carrier_State_line_of_Business__c = o.Id
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_contact` c
  ON c.Id = b.Contact__c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a
  ON a.Id = c.AccountId
WHERE
  b.Effective_Date__c IS NOT NULL
  AND b.Effective_Date__c <= LAST_DAY(b.fecha_carga)
  AND a.Name_Agencies != 'Agency Test'
  AND a.Name_Agencies IS NOT NULL
  AND o.Line_Of_Business IN ('ACA')
  AND EXTRACT(YEAR FROM b.fecha_carga) IN (2025, 2026)
GROUP BY
  b.fecha_carga
ORDER BY
  b.fecha_carga DESC
```

## Notas / supuestos

- **Vigencia relativa a cada snapshot, no a hoy**: usa
  `b.Effective_Date__c <= LAST_DAY(b.fecha_carga)` en vez del patrón estándar
  `LAST_DAY(CURRENT_DATE())` de `context/business-rules.md` regla 4 — es la adaptación
  correcta para una serie histórica: para cada fecha de carga, cuenta miembros vigentes *a
  esa fecha*, no a la fecha de hoy.
- **No excluye `Status__c IN ('NR','Terminated')`** (regla 2) **ni filtra `RecordTypeId`**
  (regla 6) — a diferencia del resto de consultas de `queries/book_of_business/aca/`. Confirmado
  intencionalmente con el equipo (2026-09-09): esta métrica busca el histórico bruto de
  miembros por snapshot, no solo el negocio activo/productivo. No agregar estos filtros sin
  volver a confirmar con el equipo.
- Sí excluye `Agency Test` (regla 3) y filtra `Line_Of_Business = 'ACA'` (regla 5), igual que
  el resto de consultas del dominio.
- **`EXTRACT(YEAR FROM b.fecha_carga) IN (2025, 2026)`** es un filtro de alcance ajustable —
  amplíalo o quítalo si se necesita el histórico completo; se mantiene por defecto para
  limitar el volumen de `BOB_H_TD` (9.3M filas) a los años relevantes.
- `BOB_H_TD` es una tabla de snapshots (una fila por póliza por fecha de carga), no debe
  confundirse con `BOB_TD` (snapshot único, "hoy") — ver `context/semantic-model/semantic-model.md`. No
  tiene columna `Val_BOB_TD`, por eso esa regla (1) tampoco aplica aquí.
