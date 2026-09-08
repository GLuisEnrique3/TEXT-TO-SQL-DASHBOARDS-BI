---
name: dashboard-filters
description: Catálogo de segmentadores (slicers) del dashboard, mapeados a columna/tabla y a su cadena de join hacia Book_of_Business
metadata:
  type: reference
---

# Segmentadores del Dashboard

Estos son los filtros/segmentadores que el usuario aplica interactivamente en el dashboard.
No son consultas nuevas: son **parámetros** que se agregan al `WHERE` de cualquier consulta
verificada en `queries/`. Úsalos para traducir peticiones como "esto pero solo para Florida
en Q2" al filtro SQL correcto, sin crear una consulta nueva por cada combinación.

> ⚠️ El dashboard llama a la tabla de hechos `Book_of_Business`. Las consultas verificadas
> de este repo usan `claro_bi.BOB_TD`. Pendiente confirmar con el equipo si son la misma
> tabla (¿`Book_of_Business` es un alias/vista de `BOB_TD`, o son tablas distintas?) antes
> de asumir que todos estos slicers aplican tal cual sobre `BOB_TD`.

## Cadena de relaciones (join hacia la tabla de hechos)

```
dim_cslb.Id                 = Book_of_Business.Carrier_State_line_of_Business__c
dim_company.Id              = dim_cslb.Internal_Company_Object__c
dim_policy_status.Status__c = Book_of_Business.Status__c
dim_contact.Id               = Book_of_Business.Contact__c
dim_account_2.Id             = dim_contact.AccountId
dim_calendar.Date            = Book_of_Business.Effective_Date__c
```

Todas las relaciones son de un solo sentido hacia la tabla de hechos (equivalente a los
`LEFT JOIN` que ya usan las consultas verificadas). `dim_company` y `dim_calendar` son
tablas nuevas que **no aparecen todavía** en ninguna consulta de `queries/` — ver notas al
final.

## Tabla de segmentadores → columna → filtro SQL

| Slicer (campo del dashboard) | Tabla / alias | Join hacia BOB | Filtro SQL equivalente |
|---|---|---|---|
| `dim_cslb[Line of Business]` | `dim_cslb` (`o`) | `o.Id = b.Carrier_State_line_of_Business__c` | `o.Line_Of_Business = '...'` |
| `dim_cslb[State]` | `dim_cslb` (`o`) | ídem | `o.State = '...'` |
| `dim_cslb[Carrier]` | `dim_cslb` (`o`) | ídem | `o.Carrier = '...'` |
| `dim_company[Name]` | `dim_company` | `dim_company.Id = o.Internal_Company_Object__c` (2 saltos vía `dim_cslb`) | requiere unir `dim_company` — ver nota |
| `dim_account_2[Agent_Specialist__c]` | `dim_account_2` (`a`) | `a.Id = c.AccountId` (2 saltos vía `dim_contact`) | `a.Agent_Specialist__c = '...'` |
| `dim_account_2[Name Agencies]` | `dim_account_2` (`a`) | ídem | `a.Name_Agencies = '...'` |
| `dim_contact[Contact_Full_name_Formula__c]` | `dim_contact` (`c`) | `c.Id = b.Contact__c` | `c.Contact_Full_name_Formula__c = '...'` |
| `dim_contact[Agent_Status__c]` | `dim_contact` (`c`) | ídem | `c.Agent_Status__c IN (...)` |
| `dim_contact[GRManager__c]` | `dim_contact` (`c`) | ídem | `c.GRManager__c = '...'` |
| `dim_contact[Flag_Investigation__c]` | `dim_contact` (`c`) | ídem | `c.Flag_Investigation__c = '...'` (solo en página Paid Members) |
| `dim_policy_status[Status_Category]` | `dim_policy_status` (`ps`) | `ps.Status__c = b.Status__c` | `ps.Status_Category = '...'` (Overview, Paid Members) |
| `dim_policy_status[Status]` | `dim_policy_status` (`ps`) | ídem | `ps.Status = '...'` (Producers — ver gotcha abajo) |
| `Book_of_Business[Val.BOB_H]` | `Book_of_Business` (`b`) | columna propia | `b.Val_BOB_H = ...` — ver nota sobre `Val_BOB_TD` |
| `Book_of_Business[Effective_Date__c]` | `Book_of_Business` (`b`) / `dim_calendar` | columna propia, también relacionada a `dim_calendar.Date` | rango de fecha, ej. `b.Effective_Date__c BETWEEN @from AND @to` (reemplaza el patrón fijo `<= LAST_DAY(CURRENT_DATE())` de las consultas base cuando el usuario pide un rango específico) |

## Segmentadores por página

| Página | Segmentadores |
|---|---|
| Overview (11) | Line of Business, State, Carrier, Company Name, Agent_Specialist__c, Name Agencies, Contact_Full_name_Formula__c, Agent_Status__c, GRManager__c, Status_Category, Val.BOB_H, Effective_Date__c |
| Producers (10) | Igual que Overview, pero **`dim_policy_status[Status]` en vez de `Status_Category`** |
| Paid Members (11) | Igual que Overview + `dim_contact[Flag_Investigation__c]` |
| Non-Producers (4) | GRManager__c, Contact_Full_name_Formula__c, Name Agencies, Agent_Specialist__c |

## Gotchas / notas

- **`Status` vs `Status_Category` inconsistente entre páginas**: Overview y Paid Members
  filtran por `dim_policy_status.Status_Category`, pero Producers filtra por
  `dim_policy_status.Status` (la misma columna que ya usan las consultas verificadas de
  `queries/*/​*-by-policy-status.md`). Si vas a replicar un filtro igual entre páginas en
  BigQuery, hay que decidir cuál usar como estándar — **pendiente de decisión del equipo**.
  Hasta que se resuelva, al traducir un filtro de este slicer aclarar cuál de las dos
  columnas se está usando.
- **`Book_of_Business` vs `BOB_TD`**: pendiente confirmar si son la misma tabla. Si son
  distintas, `Val.BOB_H` no es necesariamente lo mismo que `Val_BOB_TD` (ver
  `context/business-rules.md` regla 1) — no asumir equivalencia sin confirmar.
- **`dim_company`**: tabla nueva, no documentada aún en `schema/`. Se llega a ella desde
  `dim_cslb.Internal_Company_Object__c` (2 saltos desde `BOB_TD`). Pendiente crear
  `schema/dim_company.md` con sus columnas cuando se inspeccione con el MCP de BigQuery.
- **`dim_calendar`**: tabla nueva, no documentada aún. Se relaciona con
  `Book_of_Business.Effective_Date__c` — típico patrón de tabla de calendario para
  slicers de rango de fecha. Pendiente documentar en `schema/` si se va a usar en consultas
  verificadas (por ahora, filtrar directamente sobre `b.Effective_Date__c` es suficiente).
- Los slicers de 2 saltos (`dim_company`, `dim_account_2` vía `dim_contact`) no cambian la
  cardinalidad de las consultas existentes — ya usan esos mismos joins encadenados
  (`dim_contact` → `dim_account_2`), solo falta agregar `dim_company` cuando se necesite
  filtrar por esa dimensión.
