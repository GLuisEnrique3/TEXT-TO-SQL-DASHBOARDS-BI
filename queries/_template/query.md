---
name: nombre-corto-de-la-consulta
description: Qué responde esta consulta, en una frase (para que Claude decida si aplica)
domain: aca            # aca | medicare | life | commissions | retention | ...
tables:
  - claro_bi.BOB_TD
  - claro_bi.dim_cslb
verified_by: nombre-del-analista
verified_date: 2026-01-01
tags: [members, policies]
---

# Título legible de la consulta

## Cuándo usar esta consulta

Descripción en lenguaje de negocio de qué pregunta responde y cuándo es apropiado
reutilizarla o adaptarla (ej. "número de miembros y pólizas activas de ACA a fin de mes").

## SQL

```sql
SELECT
  ...
FROM `claroinsurance-dataplatform.claro_bi.BOB_TD` b
WHERE ...
```

## Notas / supuestos

- Qué reglas de `context/business-rules.md` aplica.
- Qué parámetros son ajustables (fecha de corte, LOB, etc.) y cómo adaptarlos.
- Casos límite conocidos.
