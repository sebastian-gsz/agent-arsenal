# VECTOR_SEARCH

La función `VECTOR_SEARCH` de BigQuery es una table-valued function que se usa
para encontrar entidades semánticamente similares, buscando embeddings de una
tabla base contra los embeddings de una tabla de query.

## Sintaxis

```sql
VECTOR_SEARCH(
  { TABLE base_table | (base_table_query) },
  column_to_search,
  { TABLE query_table | (query_table_query) },
  [, query_column_to_search => query_column_to_search_value]
  [, top_k => top_k_value ]
  [, distance_type => distance_type_value ]
  [, options => options_value ]
)
```

### Argumentos
| Argumento | Requisito | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **`base_table`** | **Uno de**  | TABLE | La tabla que contiene los embeddings a buscar. Mutuamente excluyente con base_table_query. |
| **`base_table_query`** | **Uno de** | QUERY | Una query que puedes usar para pre-filtrar la tabla base. En esta query solo se permiten las cláusulas SELECT, FROM y WHERE. No apliques filtros sobre la columna de embeddings. Mutuamente excluyente con base_table. |
| **`column_to_search`** | **Obligatorio** | String | El nombre de la columna de la tabla base que contiene los embeddings (`ARRAY<FLOAT64>` o `STRING`). |
| **`query_table`** | **Uno de**  | TABLE | La tabla que provee los embeddings para los cuales se buscan los vecinos más cercanos. Todas las columnas se pasan a la salida como columnas de salida. Mutuamente excluyente con query_table_query |
| **`query_table_query`** | **Uno de**  | QUERY | Una query que provee los embeddings para los cuales se buscan los vecinos más cercanos. Todas las columnas se pasan a la salida como columnas de salida. Mutuamente excluyente con query_table |
| **`query_column_to_search`** | Opcional | String | El nombre de la columna en la tabla de query. Por defecto `column_to_search`. El valor puede ser `STRING` o `ARRAY<FLOAT64>`. Si es `STRING`, la base_table debe tener habilitada la generación autónoma de embeddings. Los valores de cadena se convierten en embeddings en tiempo de ejecución usando la misma conexión y endpoint especificados para la generación de embeddings de la tabla base. Si el valor es un `ARRAY<FLOAT64>`, todos los elementos del array deben ser no-NULL y todos los valores de la columna deben tener las mismas dimensiones de array que los valores de la columna column_to_search.|
| **`top_k`** | Opcional | Int64 | La cantidad de vecinos más cercanos a devolver. Por defecto 10. |
| **`distance_type`** | Opcional | String | La métrica a usar: `'EUCLIDEAN'`, `'COSINE'` o `'DOT_PRODUCT'`. Por defecto `'EUCLIDEAN'`. |
| **`options`** | Opcional | String | Cadena JSON con las opciones de búsqueda (p. ej., `use_brute_force`, `fraction_lists_to_search`). |

### Columnas de salida
| Columna | Tipo | Descripción |
| :--- | :--- | :--- |
| **`base`** | STRUCT | Un valor STRUCT que contiene todas las columnas de base_table o un subconjunto de las columnas de base_table que seleccionaste en la query base_table_query. |
| **`query`** | STRUCT | Un valor STRUCT que contiene todas las columnas seleccionadas de los datos de query. Esta columna solo se incluye en la salida si usas la sintaxis de búsqueda batch. Para búsquedas de un solo vector, esta columna se omite.|
| **`distance`** | FLOAT64 | Un valor FLOAT64 que representa la distancia entre los datos base y los datos de query. |

---

## Ejemplo: Búsqueda vectorial combinada con AI.GENERATE_EMBEDDING

```sql
SELECT
  query.content AS search_query,
  base.product_name,
  base.description,
  distance
FROM
  VECTOR_SEARCH(
    TABLE `my_project.my_dataset.products`,
    'product_embedding',
    (
      SELECT
        content,
        embedding
      FROM
        AI.GENERATE_EMBEDDING(
          MODEL `my_project.my_dataset.embedding_model`,
          (SELECT "high-performance running shoes" AS content)
        )
    ),
    top_k => 5,
    distance_type => 'COSINE'
  );
```

---

## Ejemplo: Búsqueda única optimizada

Esta versión está optimizada para encontrar vecinos de un único valor.

```sql
SELECT
  base.product_name,
  base.description,
  distance
FROM
  VECTOR_SEARCH(
    TABLE `my_project.my_dataset.products`,
    'product_embedding',
    query_value => (
      SELECT embedding
      FROM AI.GENERATE_EMBEDDING(
        MODEL `my_project.my_dataset.embedding_model`,
        (SELECT "high-performance running shoes" AS content)
      )
    ),
    top_k => 5
  );
```
