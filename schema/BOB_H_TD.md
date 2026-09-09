---
name: schema-bob_h_td
description: Ficha de la tabla claro_bi.BOB_H_TD (Book of Business - histórico de snapshots por fecha de carga)
---

# `claro_bi.BOB_H_TD`

**Tipo:** tabla
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `b`

## Descripción

Histórico de snapshots del Book of Business: a diferencia de `BOB_TD` (un snapshot único,
"hoy"), aquí hay una fila por póliza **por cada fecha de carga** (`fecha_carga`), lo que
permite construir series de tiempo / tendencias (ver `queries/book_of_business/aca/members-history-aca.md`).
9.3M filas (vs. el tamaño mucho menor de `BOB_TD`) — filtrar siempre por `fecha_carga` o año
para no escanear la tabla completa.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Id` | Id de la póliza en ese snapshot |
| `Policy_Number__c` | Número de póliza |
| `Members__c` | Cantidad de miembros cubiertos por la póliza en ese snapshot |
| `Annual_Premium__c` | Prima anual |
| `Status__c` | Estado de la póliza en ese snapshot |
| `Effective_Date__c` | Fecha de vigencia de la póliza |
| `fecha_carga` | **Fecha del snapshot/carga** — la dimensión de tiempo de esta tabla. Una póliza aparece repetida una vez por cada `fecha_carga` en la que existía |
| `FechaCargaBI` | Timestamp de carga a BI (distinto de `fecha_carga`, que es la fecha de negocio del snapshot) |
| `Policy_Type__c` | Tipo de póliza. FK (por valor) hacia `dim_policy_type.Policy_Type__c` |
| `Source__c` | Fuente/origen del negocio. FK (por valor) hacia `dim_source.Source__c` |
| `Carrier_State_line_of_Business__c` | FK hacia `dim_cslb.Id` |
| `Contact__c` | FK hacia `dim_contact.Id` |
| `RecordTypeId` | Tipo de registro de póliza |

## Joins típicos

| Join hacia | Condición | Para qué |
|---|---|---|
| `claro_bi.dim_cslb` (alias `o`) | `o.Id = b.Carrier_State_line_of_Business__c` | Resolver línea de negocio, carrier, estado |
| `claro_bi.dim_contact` (alias `c`) | `c.Id = b.Contact__c` | Llegar al agente y, vía `AccountId`, a la agencia/cuenta |

## Gotchas / notas

- **No tiene columna `Val_BOB_TD`** (ni equivalente) — la regla 1 de `context/business-rules.md`
  (`Val_BOB_TD = 1`) no aplica a esta tabla.
- **No confundir con `BOB_TD`**: mismo dominio de negocio, pero `BOB_TD` es un snapshot único
  ("hoy") y `BOB_H_TD` es histórico (una fila por póliza por `fecha_carga`). Para tendencias
  usa `BOB_H_TD`; para el corte actual usa `BOB_TD`.
- Para simular "vigencia a fin de mes" en una serie histórica, el patrón es
  `Effective_Date__c <= LAST_DAY(fecha_carga)` (no `LAST_DAY(CURRENT_DATE())`) — ver
  `queries/book_of_business/aca/members-history-aca.md`.
- Confirmado con el equipo (2026-09-09) que las métricas de histórico sobre esta tabla **no**
  necesariamente excluyen `Status__c IN ('NR','Terminated')` ni filtran `RecordTypeId` — depende
  de si la métrica busca el histórico bruto o solo negocio activo; confirmar caso por caso.
