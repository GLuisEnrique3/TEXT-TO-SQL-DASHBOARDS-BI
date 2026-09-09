---
name: business-rules
description: Reglas de filtrado estándar que deben aplicarse en la mayoría de reportes de negocio
---

# Reglas de filtrado estándar

Estas reglas encapsulan decisiones de negocio no evidentes desde el esquema. Aplícalas por
defecto en cualquier consulta de "negocio activo" salvo que el usuario pida explícitamente
lo contrario (por ejemplo, si pide "incluye pólizas terminadas para análisis de churn").

## 1. Registros válidos del Book of Business

```sql
b.Val_BOB_TD = 1
```
Filtra el snapshot de `BOB_TD` a únicamente los registros vigentes/válidos para reporting.

## 2. Excluir estados no productivos

```sql
b.Status__c NOT IN ('NR','Terminated')
```
`NR` (no registrada) y `Terminated` no cuentan como negocio activo.

## 3. Excluir cuentas de prueba

```sql
a.Name_Agencies != 'Agency Test'
AND a.Name_Agencies IS NOT NULL
```
`dim_account_2` puede contener la cuenta de pruebas internas `Agency Test`; siempre excluirla
de reportes de negocio real. También excluir `NULL` (agencia no resuelta / dato incompleto).

## 4. Vigencia a la fecha de corte (mes actual)

```sql
b.Effective_Date__c <= LAST_DAY(CURRENT_DATE())
```
Patrón estándar para "pólizas vigentes hasta fin de mes actual". Ajustar `LAST_DAY(...)` si
el usuario pide un mes/fecha de corte distinto.

## 5. Filtrar por línea de negocio

```sql
o.Line_Of_Business = 'ACA'   -- o Medicare, Life, etc.
```
Se resuelve vía join a `dim_cslb` (alias `o`) por `o.Id = b.Carrier_State_line_of_Business__c`.
Ver `schema/dim_cslb.md` y `context/glossary.md` para valores válidos.

## 6. RecordType de póliza

```sql
b.RecordTypeId IN ('0121G000000bpwqQAA', '0121G000000bpwvQAA')
```
Estos IDs de RecordType corresponden a los tipos de póliza incluidos en reportes de BOB
estándar (confirmar con el equipo qué representa cada uno antes de reutilizar en otro
contexto — documentar aquí cuando se confirme).

## 7. Excluir filas centinela con fecha `2000-01-01`

```sql
-- En BOB_H_TD:
b.fecha_carga != DATE '2000-01-01'
-- En Payment_Commission:
p.Commission_Month__c != DATE '2000-01-01'
```
Tanto `BOB_H_TD` como `Payment_Commission` contienen filas con esta fecha centinela/placeholder
(confirmado inspeccionando el Power Query de `BOB History.pbix`, tabla `Payments` y
`Book_of_Business`, que las excluyen con un filtro `FiltroFecha2000 = 0`). Excluir siempre en
consultas nuevas sobre estas dos tablas — no se ha confirmado qué representan esas filas, pero
el dashboard nunca las incluye en reportes.

## 8. AP de pólizas pagadas (ojo con duplicar la prima por fila de pago)

`Payment_Commission` no tiene columna de prima anual propia — para "AP de pólizas pagadas" hay
que traerla desde `BOB_TD`/`BOB_H_TD` por `Policy_Number__c`, tomando **la prima del snapshot
más reciente con valor distinto de cero** (replica la tabla calculada `AP BOB` de
`BOB History.pbix`).

**Decide con cuidado si deduplicar por póliza o no**, según qué se está replicando:
- La medida `AP of Paid Policies` del dashboard **sí duplica la prima por cada fila** de
  `Payment_Commission` que tenga esa póliza en ese mes (no filtra `Main_Payment__c` en su
  `SUM` externo) — si quieres que tu SQL coincida con el número del dashboard, une la prima a
  nivel de fila de pago (sin deduplicar), no a nivel de póliza. Ver
  `queries/payment/ap-of-paid-policies-life-supplementary.md`.
- Si en cambio quieres una cifra de negocio "correcta" sin ese artefacto de duplicación
  (prima real de las pólizas pagadas, una vez cada una), deduplica por `Policy_Number__c`
  antes de sumar.

```sql
SELECT
  b.Policy_Number__c,
  ARRAY_AGG(b.Annual_Premium__c ORDER BY b.fecha_carga DESC LIMIT 1)[OFFSET(0)] AS Annual_Premium__c
FROM `claro_bi.BOB_H_TD` b
WHERE b.fecha_carga != DATE '2000-01-01'
  AND b.Annual_Premium__c != 0
GROUP BY b.Policy_Number__c
```
**No** hagas `SUM(Annual_Premium__c)` uniendo `Payment_Commission` directo a `BOB_TD`/`BOB_H_TD`
por `Policy_Number__c` sin deduplicar — si la póliza tiene varias filas de pago (varios meses o
tipos de pago), la prima se multiplicaría. Ver `queries/payment/ap-of-paid-policies-life-supplementary.md`.
