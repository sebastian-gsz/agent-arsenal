---
name: developing-with-bigquery
description: |
  Repositorio de lógica, conocimiento y estándares especializados de BigQuery.
  Usa esta skill siempre que hagas cualquier cosa con BigQuery, incluyendo:
    1. Optimización de queries de BigQuery
    2. Código Python con BigFrames
    3. Funciones de ML/AI de BigQuery.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

Esta skill ofrece guía completa sobre los servicios de BigQuery, sus optimizaciones
y el manejo de datos. Actúa como una tabla de enrutamiento hacia temas
especializados de BigQuery.

> [!IMPORTANT]
>
> Para los estándares generales de ejecución de BigQuery en notebooks (celdas SQL, palabra clave `export`), consulta `@skill:notebook-guidance`.

> [!IMPORTANT]
>
> DEBES verificar el tamaño de los datos antes de decidir qué bibliotecas usar. Usa
> el tamaño de los datos para justificar tu decisión.

Consulta los siguientes recursos para obtener guía experta sobre funcionalidades
específicas de BigQuery:

### 1. Optimización de queries

Pautas de rendimiento y eficiencia para SQL de BigQuery. Incluye reglas para
poda de columnas, pushdown y estrategias de materialización. - **Guía**:
[OPTIMIZATION.md](references/OPTIMIZATION.md)

### 2. BigFrames (BigQuery DataFrames)

Pautas para generar código BigFrames válido para manipulación de datos, desarrollo
de modelos y visualización. - **Guía**:
[BIGFRAMES.md](references/BIGFRAMES.md)

BigFrames debe ser la biblioteca/herramienta por defecto, ya que es más eficiente
que usar la biblioteca cliente de Python de BigQuery.

### 3. Funciones de ML y AI de BigQuery (BQML SQL)

Reglas de uso y estándares de sintaxis para todas las funciones de AI/ML de
BigQuery vía SQL (forecasting, IA generativa, clasificación, etc.). - **Guía**:
[BQML.md](references/BQML.md) - **Referencia de funciones**: -
[AI.FORECAST](references/ai-forecast.md) -
[AI.EVALUATE](references/ai-evaluate.md) -
[AI.GENERATE_TABLE](references/ai-generate-table.md) -
[AI.GENERATE_EMBEDDING](references/ai-generate-embedding.md) -
[Modelos remotos](references/remote-models.md)
[CONTRIBUTION_ANALYSIS](references/ml-contribution-analysis.md)
[VECTOR_SEARCH](references/vector-search.md)

### 4. Celdas SQL en notebooks

Consulta `@skill:notebook-guidance` para los estándares de ejecución de BigQuery
en notebooks.
