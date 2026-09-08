# Claro Insurance BI — Contexto para Claude Project

Repositorio de contexto de negocio y consultas SQL verificadas para el equipo de BI de
Claro Insurance, pensado para conectarse a un **Claude Project** vía GitHub, junto con el
**MCP de BigQuery** (`claroinsurance-dataplatform`).

## Para qué sirve

Cuando Claude tiene este repo como contexto y acceso al MCP de BigQuery, puede:

- Responder preguntas de negocio ("¿cuántos miembros activos de ACA tenemos?") reutilizando
  lógica SQL ya validada por el equipo, en lugar de adivinar joins o filtros.
- Explicar qué significa una tabla o columna sin tener que preguntarle a un analista.
- Ejecutar consultas de solo lectura directamente contra BigQuery.

## Estructura

| Carpeta | Contenido |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | Instrucciones que Claude lee automáticamente al abrir el proyecto |
| [`context/`](context/) | Glosario de negocio, reglas de filtrado estándar, líneas de negocio |
| [`schema/`](schema/) | Documentación de tablas y vistas de BigQuery relevantes |
| [`queries/`](queries/) | Consultas SQL verificadas, organizadas por dominio de negocio |

## Cómo añadir una nueva consulta verificada

1. Copia [`queries/_template/query.md`](queries/_template/query.md) dentro de la carpeta del
   dominio correspondiente (crea la carpeta si no existe).
2. Rellena el frontmatter y el SQL.
3. Haz commit y push — Claude Project la recogerá automáticamente en la próxima sincronización.

## Cómo añadir/actualizar documentación de una tabla

Copia [`schema/_template.md`](schema/_template.md) (ver plantilla) a `schema/<tabla>.md` y
documenta columnas clave, joins típicos y "gotchas" (columnas ambiguas, valores especiales,
soft-deletes, etc.).
