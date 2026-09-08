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
