---
name: agents-with-no-production
description: Conteo de agentes activos que no tienen ninguna póliza en ACA, Medicare, Life o Supplementary
domain: agents
tables:
  - claro_bi.dim_contact
  - claro_bi.dim_account_2
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
verified_by: Enrique Guerra
verified_date: 2026-09-08
tags: [agents, no-production, contact]
---

# Agents with No Production

## Cuándo usar esta consulta

Responde: "¿cuántos agentes activos (por `Agent_Status__c`) no tienen ninguna póliza
registrada en `BOB_TD` en ninguna de las líneas de negocio principales (ACA, Medicare,
Life, Supplementary)?" Es el complemento de `active-agents-td.md`: mismo universo base de
agentes activos, pero exigiendo que **no exista** producción asociada.

## SQL

```sql
SELECT COUNT(DISTINCT c.Id) AS agents_non_producers
FROM `claroinsurance-dataplatform.claro_bi.dim_contact` c
LEFT JOIN `claroinsurance-dataplatform.claro_bi.dim_account_2` a ON a.Id = c.AccountId
WHERE c.Agent_Status__c IN ('Active','Awating Partial Release','Awaiting Complete Release','Pending Documents','Dependent Agent','Suspended by carrier')
  AND a.Name_Agencies != 'Agency Test'
  AND a.Name_Agencies IS NOT NULL
  AND NOT EXISTS (
    SELECT 1
    FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
    JOIN `claroinsurance-dataplatform.claro_bi.dim_cslb` cslb
      ON b.Carrier_State_line_of_Business__c = cslb.Id
    WHERE b.Contact__c = c.Id
      AND cslb.Line_Of_Business IN ('ACA','Medicare','Life','Supplementary')
  )
```

## Notas / supuestos

- La condición de "activo" (`Agent_Status__c IN (...)`) es idéntica a `active-agents-td.md`
  — mismo listado de estados, incluido el typo de origen `'Awating Partial Release'`.
- El `NOT EXISTS` **no filtra** `b.Val_BOB_TD = 1` ni `b.Status__c` dentro de la subconsulta
  — cualquier fila de `BOB_TD` en esas 4 LOBs (incluidas terminadas o inválidas para
  reporting) cuenta como "producción" y excluye al agente. Confirmar con el equipo si debe
  alinearse con las reglas 1 y 2 de `context/business-rules.md` antes de usarla como fuente
  oficial de "agentes sin producción real".
- Lista de LOBs fija (`'ACA','Medicare','Life','Supplementary'`) — actualizar si se agregan
  nuevas líneas de negocio al glosario.
