# AI.FORECAST
Se usa para forecasting de series temporales sin entrenar modelos. Detecta
automáticamente la frecuencia (diaria, semanal, etc.) según tu columna de
timestamps.

## Sintaxis
```sql
SELECT
  *
FROM
  AI.FORECAST(
    { TABLE `project.dataset.table` | (QUERY_STATEMENT) },
    data_col => 'DATA_COL',
    timestamp_col => 'TIMESTAMP_COL'
    [, model => 'MODEL']
    [, id_cols => ID_COLS]
    [, horizon => HORIZON]
    [, confidence_level => CONFIDENCE_LEVEL]
    [, output_historical_time_series => OUTPUT_HISTORICAL_TIME_SERIES]
    [, context_window => CONTEXT_WINDOW]
  )
```

## Argumentos de entrada
| Argumento | Requisito | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **`input_data`** | **Obligatorio** | | La tabla o subquery de origen que contiene los datos históricos. |
| **`data_col`** | **Obligatorio** | String | La columna numérica a predecir. |
| **`timestamp_col`** | **Obligatorio** | String | La columna que contiene fechas/timestamps. |
| **`id_cols`** | Opcional | Array<String> | Columnas de agrupación para múltiples series (p. ej., `['store_id']`). |
| **`horizon`** | Opcional | Int64 | Cantidad de puntos futuros a predecir. Por defecto 30. |
| **`confidence_level`** | Opcional | Float64 | Intervalo de confianza (0 a 1). Por defecto 0.95. |
| **`model`** | Opcional | String | Versión del modelo. Por defecto `'TimesFM 2.0'`. |
| **`context_window`** | Opcional | Int64 | Cantidad de puntos de datos históricos que el modelo usa para hacer el forecast. Máximo 512. Si no se establece, el modelo lo determina automáticamente. |

## Esquema de salida
El esquema se ajusta según el flag `output_historical_time_series`.

| Columna | Tipo | Se incluye si output_historical_time_series=FALSE | Se incluye si output_historical_time_series=TRUE | Descripción |
| :--- | :--- | :---: | :---: | :--- |
| **`id_cols`** | (Como entrada) | Sí | Sí | Identificadores originales de las series. |
| **`forecast_timestamp`** | TIMESTAMP | **Sí** | No | Timestamp de los puntos predichos. |
| **`forecast_value`** | FLOAT64 | **Sí** | No | Predicción del cuantil 50% (mediana). |
| **`time_series_timestamp`** | TIMESTAMP | No | **Sí** | Columna de timestamps uniforme para el historial y el forecast. |
| **`time_series_data`** | FLOAT64 | No | **Sí** | Columna combinada: valores reales para el historial, mediana para el forecast. |
| **`time_series_type`** | STRING | No | **Sí** | Etiqueta: `'history'` o `'forecast'`. |
| **`prediction_interval_lower_bound`** | FLOAT64 | Sí | Sí | Límite inferior (NULL para las filas históricas). |
| **`prediction_interval_upper_bound`** | FLOAT64 | Sí | Sí | Límite superior (NULL para las filas históricas). |
| **`confidence_level`** | FLOAT64 | Sí | Sí | Nivel de confianza constante utilizado. |
| **`ai_forecast_status`** | STRING | Sí | Sí | Mensajes de error o cadena vacía en caso de éxito. |

## Ejemplo: Forecast con historial

```sql
-- Forecast next 10 days of sales by store and include historical data for charting
SELECT * FROM AI.FORECAST(
  TABLE `my_project.sales.daily_totals`,
  data_col => 'total_sales',
  timestamp_col => 'date',
  id_cols => ['store_id'],
  output_historical_time_series => TRUE
);
```
