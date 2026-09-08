# Contexto de negocio: Claro Insurance BI (BigQuery)

Este repositorio es la fuente de verdad de contexto de negocio para consultas SQL contra
BigQuery (`claroinsurance-dataplatform`). Se usa junto al MCP de BigQuery
(`mcp__claude_ai_Google_Cloud_BigQuery__*`). Este archivo son tus instrucciones de trabajo.

## Cómo trabajar en este proyecto

1. **Antes de escribir una consulta nueva**, busca en `queries/` si ya existe una consulta
   verificada para esa métrica o dominio (Members, Policies, Retention, Commissions, etc.).
   Si existe, **reutilízala como base** — no reinventes la lógica de negocio. Las reglas de
   filtrado (`WHERE`) en las consultas verificadas encapsulan decisiones de negocio que no
   son obvias desde el esquema (ver `context/business-rules.md`).
2. **Si necesitas una tabla que no conoces**, revisa `schema/` primero. Si no está documentada,
   usa `get_table_info` / `get_dataset_info` del MCP de BigQuery para inspeccionarla, y
   considera añadir un archivo nuevo en `schema/` con lo que aprendiste (ver plantilla).
3. **Antes de ejecutar SQL contra BigQuery**, usa `execute_sql_readonly` salvo que el usuario
   pida explícitamente escribir/modificar datos.
4. **Si construyes una consulta nueva que el usuario confirma como correcta**, ofrece
   guardarla en `queries/<dominio>/` siguiendo la plantilla en `queries/_template/query.md`,
   para que quede disponible como referencia verificada en el futuro.
5. Los nombres de líneas de negocio (ACA, Medicare, Life, etc.) y estados de póliza están
   documentados en `context/glossary.md`. Úsalos para interpretar peticiones ambiguas del
   usuario (p. ej. "miembros activos" → ver regla de `Status__c` en `business-rules.md`).
6. **Si el usuario pide filtrar/segmentar una consulta** (por estado, carrier, agencia,
   agente, rango de fecha, etc. — como los segmentadores del dashboard), consulta
   `context/dashboard-filters.md` para saber a qué columna/tabla corresponde cada filtro y
   cómo se une hacia `BOB_TD`, en vez de adivinar el nombre de columna.

## Cómo responder al usuario

- Usa todo el contexto técnico de este repo (`schema/`, `queries/`, `context/`) **para
  construir y ejecutar** la consulta correcta — pero **no lo expongas en la respuesta**.
- Responde en lenguaje de negocio: la cifra o el hallazgo, y si aporta, una frase breve de
  contexto (ej. "miembros activos de ACA vigentes a fin de mes"). No menciones nombres de
  tabla, columnas, alias, joins, ni rutas de archivo (`BOB_TD`, `dim_cslb`,
  `queries/aca/...`, etc.) en la respuesta por defecto.
- Muestra el detalle técnico (SQL usado, tablas, columnas, archivo de `queries/` de
  referencia) únicamente si el usuario lo pide explícitamente — por ejemplo "¿cómo lo
  calculaste?", "muéstrame el SQL", "¿qué tabla usaste?".

## Estructura del repositorio

- `context/` — glosario de negocio, reglas de filtrado estándar, líneas de negocio,
  catálogo de segmentadores del dashboard (`dashboard-filters.md`).
- `schema/` — una ficha por tabla/vista relevante: columnas clave, joins típicos, gotchas.
- `queries/<dominio>/` — consultas SQL **verificadas** por humanos, con metadata y notas.
  `<dominio>` = línea de negocio o área (aca, medicare, life, commissions, retention...).
- `queries/_template/query.md` — plantilla para añadir una nueva consulta verificada.

## Reglas de oro

- Nunca inventes nombres de columnas o tablas: verifica con el MCP de BigQuery
  (`list_dataset_ids`, `list_table_ids`, `get_table_info`) si no están documentadas aquí.
- Respeta siempre los filtros de exclusión estándar documentados en
  `context/business-rules.md` (p. ej. excluir `Agency Test`, excluir `Status__c` en
  `('NR','Terminated')`) salvo que el usuario pida explícitamente lo contrario.
- Si una consulta verificada queda obsoleta (columna renombrada, regla de negocio cambiada),
  actualiza el archivo en `queries/` en lugar de crear una versión paralela.
