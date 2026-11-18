# Análisis de contribución

El análisis de contribución, también llamado análisis de drivers clave, es un
método que se usa para generar insights sobre cambios en métricas clave de tus
datos multidimensionales.

## Sintaxis

Para usar el análisis de contribución en BigQuery ML, crea un modelo de análisis
de contribución con la sentencia CREATE MODEL.

Una vez creado el modelo de análisis de contribución, puedes usar la función
ML.GET_INSIGHTS para recuperar la información de métricas calculada por el
modelo.

### Sintaxis de CREATE MODEL

```sql
CREATE OR REPLACE MODEL `project.dataset.model_name`
OPTIONS (
  MODEL_TYPE = 'CONTRIBUTION_ANALYSIS',
  CONTRIBUTION_METRIC = 'contribution_metric',
  IS_TEST_COL = 'is_test_col'
  [, DIMENSION_ID_COLS = dimension_column_array]
  [, MIN_APRIORI_SUPPORT = min_apriori_support]
  [, TOP_K_INSIGHTS_BY_APRIORI_SUPPORT = top_k_insights_by_apriori_support]
  [, PRUNING_METHOD = {'NO_PRUNING', 'PRUNE_REDUNDANT_INSIGHTS'}]
)
AS query_statement;
```

### Opciones
| Opción | Requisito | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **`MODEL_TYPE`** | **Obligatorio** | String | Debe ser `'CONTRIBUTION_ANALYSIS'`. |
| **`CONTRIBUTION_METRIC`** | **Obligatorio** | String | La métrica a analizar. La métrica soportada es `SUM(metric_column_name)`. |
| **`IS_TEST_COL`** | **Obligatorio** | String | El nombre de la columna booleana que distingue los datos de test (TRUE) de los datos de control (FALSE). |
| **`DIMENSION_ID_COLS`** | Opcional | Array<String> | Las columnas a usar como dimensiones para segmentar los datos. Por defecto, todas las columnas de los datos de entrada no especificadas en otras opciones. Restringido a columnas de tipo `INT64`, `BOOL` o `STRING`. Limitado a un máximo de 50 columnas.|
| **`MIN_APRIORI_SUPPORT`** | Opcional | Float64 | Umbral mínimo de soporte para podar segmentos pequeños. No puede usarse junto con `TOP_K_INSIGHTS_BY_APRIORI_SUPPORT`. Por defecto 0.1 si no se especifica ninguno. El valor está en el rango [0, 1]. |
| **`TOP_K_INSIGHTS_BY_APRIORI_SUPPORT`** | Opcional | Int64 | La cantidad máxima de insights a mostrar con los valores de soporte apriori más altos. No puede usarse junto con `MIN_APRIORI_SUPPORT`. Este valor debe estar dentro del intervalo [1, 1,000,000]|
| **`PRUNING_METHOD`** | Opcional | String | El método de poda usado para filtrar insights. Valores soportados: `'NO_PRUNING'`, `'PRUNE_REDUNDANT_INSIGHTS'`. Por defecto `'NO_PRUNING'`. `PRUNE_REDUNDANT_INSIGHTS` poda los segmentos redundantes, excluyendo el segmento `all`. Dos segmentos son redundantes si se cumplen dos condiciones: 1) sus valores de métrica son iguales 2) los IDs de dimensión y los valores correspondientes de una fila son un subconjunto de los IDs de dimensión y los valores correspondientes de la otra. En ese caso, se conserva la fila con más IDs de dimensión (la fila más descriptiva).|

---

## Sintaxis de ML.GET_INSIGHTS

Usa `ML.GET_INSIGHTS` para recuperar los resultados del modelo:

```sql
ML.GET_INSIGHTS(
  MODEL `PROJECT_ID.DATASET.MODEL_NAME`
)
```

### Esquema de salida
| Columna | Descripción |
| :--- | :--- |
| **`contributors`** | Los valores de dimensión para un segmento dado (p. ej., `['dim1=val1', 'dim2=val2']`). |
| **`metric_test`** | Un valor numérico que contiene la suma del valor de la columna de métrica en el dataset de test para el segmento dado. |
| **`metric_control`** | Un valor numérico que contiene la suma del valor de la columna de métrica en el dataset de control para el segmento dado. |
| **`difference`** | Un valor numérico que contiene la diferencia entre los valores metric_test y metric_control: `metric_test - metric_control`. |
| **`relative_difference`** | Un valor numérico que contiene el cambio relativo del valor del segmento entre los datasets de test y de control. |
| **`unexpected_difference`** | Un valor numérico que contiene la diferencia inesperada entre el valor metric_test real del segmento y el valor metric_test esperado del segmento, que se determina comparando el ratio de cambio de este segmento contra el ratio de cambio del complemento. |
| **`relative_unexpected_difference`** | Un valor numérico que contiene el ratio entre el valor unexpected_difference y el valor expected_metric_test. |
| **`apriori_support`** | La proporción de la métrica total en los conjuntos de test y de control que representa este segmento. |
| **`contribution`** | Un valor numérico que contiene el valor absoluto del valor de la diferencia. |
---

## Ejemplo: Análisis del cambio de ingresos

```sql
-- 1. Create the model
CREATE OR REPLACE MODEL `my_project.my_dataset.revenue_analysis`
OPTIONS (
  MODEL_TYPE = 'CONTRIBUTION_ANALYSIS',
  is_test_col = 'is_current_year',
  dimension_id_cols = ['region', 'product_category', 'customer_segment'],
  contribution_metric = 'SUM(revenue)'
) AS
SELECT
  region,
  product_category,
  customer_segment,
  revenue,
  IF(date >= '2024-01-01', TRUE, FALSE) AS is_current_year
FROM
  `my_project.my_dataset.sales_data`;

-- 2. Get insights
SELECT
  *
FROM
  ML.GET_INSIGHTS(MODEL `my_project.my_dataset.revenue_analysis`)
ORDER BY
  unexpected_difference DESC;
```
