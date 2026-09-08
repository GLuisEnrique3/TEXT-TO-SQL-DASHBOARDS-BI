---
name: schema-bob_td
description: Ficha de la tabla claro_bi.BOB_TD (Book of Business - detalle de pólizas)
---

# `claro_bi.BOB_TD`

**Tipo:** tabla
**Dataset:** claroinsurance-dataplatform.claro_bi
**Alias habitual:** `b`

## Descripción

Detalle transaccional del Book of Business: una fila por póliza (o versión de póliza) con
sus atributos de vigencia, estado y línea de negocio.

## Columnas clave

| Columna | Descripción |
|---|---|
| `Policy_Number__c` | Número de póliza |
| `Members__C` | Cantidad de miembros cubiertos por la póliza |
| `Annual_Premium__c` | Prima anual de la póliza. Usada en las métricas de `queries/life/` y `queries/supplementary/` |
| `Status__c` | Estado de la póliza (`NR`, `Terminated`, otros — ver `context/glossary.md`) |
| `Val_BOB_TD` | 1 = registro válido para reporting (ver `context/business-rules.md` regla 1) |
| `RecordTypeId` | Tipo de registro de póliza (ver `context/business-rules.md` regla 6) |
| `Effective_Date__c` | Fecha de vigencia de la póliza |
| `Policy_Type__c` | Tipo de póliza. FK (por valor) hacia `dim_policy_type.Policy_Type__c` |
| `Source__c` | Fuente/origen del negocio. FK (por valor) hacia `dim_source.Source__c` |
| `Carrier_State_line_of_Business__c` | FK hacia `dim_cslb.Id` |
| `Contact__c` | FK hacia `dim_contact.Id` |

## Joins típicos

| Join hacia | Condición | Para qué |
|---|---|---|
| `claro_bi.dim_cslb` (alias `o`) | `o.Id = b.Carrier_State_line_of_Business__c` | Resolver línea de negocio, carrier, estado |
| `claro_bi.dim_contact` (alias `c`) | `c.Id = b.Contact__c` | Llegar al agente y, vía `AccountId`, a la agencia/cuenta |

## Gotchas / notas

- Siempre filtrar `Val_BOB_TD = 1` para evitar duplicados/snapshots inválidos — salvo en
  Medicare y Life/Supplementary, donde es intencional no filtrarlo (ver esas consultas).
- Excluir `Status__c IN ('NR','Terminated')` para negocio activo.
- **`Book_of_Business` (dashboard) vs `BOB_TD`**: el dashboard referencia la tabla de
  hechos como `Book_of_Business`, con una columna `Val.BOB_H`. Pendiente confirmar si es la
  misma tabla que `BOB_TD` (¿alias/vista, o tabla distinta?) y si `Val_BOB_H` equivale a
  `Val_BOB_TD` antes de asumirlo — ver `context/dashboard-filters.md`.
