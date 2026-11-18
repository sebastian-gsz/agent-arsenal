# AI.GENERATE_EMBEDDING
Se usa para generar vectores numéricos de alta dimensión (embeddings) para
contenido textual o visual. Estos embeddings pueden emplearse en búsqueda
semántica, clustering y sistemas de recomendación.

## Sintaxis
```sql
SELECT
  *
FROM
  AI.GENERATE_EMBEDDING(
    MODEL `project.dataset.model`,
    { TABLE `project.dataset.table` | (QUERY_STATEMENT) }
    [, STRUCT(
      [TASK_TYPE AS task_type]
      [, OUTPUT_DIMENSIONALITY AS output_dimensionality]) ]
  )
```

## Argumentos de entrada
| Argumento | Requisito | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **`model`** | **Obligatorio** | | El recurso del modelo (p. ej., `project.dataset.model`). |
| **`input_data`** | **Obligatorio*** | | La tabla o sentencia `SELECT` de origen que contiene los datos a convertir en embeddings. |
| **`task_type`** | Opcional | String | La aplicación posterior prevista (p. ej., `RETRIEVAL_QUERY`, `RETRIEVAL_DOCUMENT`). |
| **`output_dimensionality`** | Opcional | Int64 | La cantidad de dimensiones a usar al generar los embeddings. |

\* *`input_data` es opcional para modelos de factorización matricial.*

## Esquema de salida
| Columna | Tipo | Descripción |
| :--- | :--- | :--- |
| **`[Input Columns]`** | (Como entrada) | Cada columna incluida en tus datos de entrada. |
| **`embedding`** | ARRAY<FLOAT64> | El vector numérico generado. |
| **`statistics`** | JSON | Metadatos sobre la generación, como el conteo de tokens. |
| **`status`** | STRING | Estado de la ejecución; contiene mensajes de error en caso de falla. |

## Ejemplo: Generación de embeddings de texto
```sql
-- Generate embeddings for a literal string
SELECT *
FROM
  AI.GENERATE_EMBEDDING(
    MODEL `mydataset.text_embedding`,
    (SELECT "Example text to embed" AS content)
);
```
