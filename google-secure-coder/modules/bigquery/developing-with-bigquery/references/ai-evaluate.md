# AI.EVALUATE
Evalúa la precisión de un modelo de forecasting comparando los valores predichos
con los datos históricos reales. Calcula métricas de regresión estándar como
MAE, MSE y RMSE.

## Sintaxis
```sql
SELECT
  *
FROM
  AI.EVALUATE(
    { TABLE `project.dataset.history_table` | (HISTORY_QUERY) },
    { TABLE `project.dataset.actual_table` | (ACTUAL_QUERY) },
    data_col => 'DATA_COL',
    timestamp_col => 'TIMESTAMP_COL'
    [, model => 'MODEL']
    [, id_cols => ID_COLS]
    [, horizon => HORIZON]
  )
```

## Argumentos de entrada
| Argumento | Requisito | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **`history_data`** | **Obligatorio** | | La tabla o query que contiene los datos históricos usados para el forecast. |
| **`actual_data`** | **Obligatorio** | | La tabla o query que contiene los datos reales ("ground truth"). |
| **`data_col`** | **Obligatorio** | String | La columna numérica a evaluar. |
| **`timestamp_col`** | **Obligatorio** | String | La columna que contiene fechas/timestamps. |
| **`id_cols`** | Opcional | Array<String> | Columnas de agrupación para múltiples series. |
| **`horizon`** | Opcional | Int64 | Cantidad de puntos a evaluar. Por defecto 1024. |
| **`model`** | Opcional | String | Versión del modelo (p. ej., `'TimesFM 2.0'`). |

## Esquema de salida
| Columna | Tipo | Descripción |
| :--- | :--- | :--- |
| **`id_cols`** | (Como entrada) | Identificadores originales de las series (si se proporcionaron). |
| **`mean_absolute_error`** | FLOAT64 | Magnitud promedio de los errores en las predicciones. |
| **`mean_squared_error`** | FLOAT64 | Promedio de los cuadrados de los errores. |
| **`root_mean_squared_error`**| FLOAT64 | Raíz cuadrada del error cuadrático medio. |
| **`mean_absolute_percentage_error`** | FLOAT64 | Error porcentual absoluto medio de la serie. |
| **`symmetric_mean_absolute_percentage_error`** | FLOAT64 | Error porcentual absoluto medio simétrico de la serie. |
| **`ai_evaluate_status`** | STRING | Mensajes de error o cadena vacía en caso de éxito. |

## Ejemplo: Evaluación de la calidad del forecast
```sql
SELECT * FROM AI.EVALUATE(
  (SELECT * FROM `sales` WHERE date < '2024-01-01'),
  (SELECT * FROM `sales` WHERE date >= '2024-01-01'),
  data_col => 'total_sales',
  timestamp_col => 'date',
  id_cols => ['store_id']
);
```
