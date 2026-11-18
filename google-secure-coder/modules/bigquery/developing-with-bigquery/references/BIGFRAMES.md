# Generación de código BigFrames

Pautas para generar código válido con la biblioteca BigFrames (BigQuery DataFrame).

## Pautas generales de codificación

*   **Evita `.to_pandas()`**: NO debes usar `.to_pandas()` para descargar el dataset completo a memoria. Hay algunas excepciones:
    *  Un mensaje de error te pide explícitamente usar `to_pandas()`
    *  Vas a visualizar los datos, **y** la biblioteca de visualización no acepta instancias de DataFrame/Series de BigFrames. En ese caso, reduce la cantidad de datos a descargar antes de llamar a `.to_pandas()`
*   **Evita `read_gbq()` para SQL**: No escribas queries SQL y las ejecutes con `read_gbq()`. Usa en su lugar los métodos de DataFrame/Series de BigFrames.
*   **Usa el paquete BigFrames ML para tareas de Machine Learning**: No uses Scikit-learn ni otras bibliotecas de ML con dataframes de BigFrames. Importa tus herramientas/clases desde `bigframes.ml`.
*   **Permanece en la nube**: Realiza la limpieza, transformación y análisis de datos con métodos de BigFrames para aprovechar la escala de BigQuery.
*   **Accessors por encima de UDFs/lambdas**:
    *   Prefiere los accessors integrados (p. ej., `df.col.str.*`, `df.col.dt.*`) por encima de las UDFs remotas.
    *   **No uses lambdas** con `Series.map()` ni `DataFrame.apply()`.
*   **Verificación del esquema**: No asumas el esquema de las salidas intermedias. Revisa `.dtypes` después de cargar, y usa `display()` con `.head()` o `.peek()`.
*   **Visualización**: Los DataFrames de BigFrames en general funcionan directamente con Matplotlib, Seaborn y otras bibliotecas de gráficos. Si el intento no funcionó, prueba con el accessor "plot". Si tampoco funcionó, DEBES muestrear o agregar tus datos para reducirlos lo suficiente antes de llamar a "to_pandas()".

## Desarrollo de modelos

* **A diferencia de Scikit-learn**: el método `predict()` de BigFrames siempre devuelve un **DataFrame** que contiene tanto predicciones como features (no solo una serie de predicciones).
* **Sin `random_state`**: No pases un argumento `random_state` al instanciar modelos de ML de BigFrames.
* **Escalado automático**: No uses `OneHotEncoder` ni `StandardScaler` salvo que se solicite explícitamente (se maneja automáticamente).
* **Ajuste de hiperparámetros**: Debes escribir bucles personalizados (BigFrames no cuenta con `GridSearchCV` ni `RandomizedSearchCV`).
* **ARIMA Plus** (forecasting):
    - Importa desde `bigframes.ml.forecasting`.
    - Ordena los datos cronológicamente y divídelos alrededor de un punto temporal antes de entrenar.
    - El horizonte de predicción debe ser menor o igual al horizonte de entrenamiento.
* **PCA**: la clase PCA de BigFrames carece de un método `transform()` simple. Usa `predict()` en su lugar.
* **Persistencia de modelos**: para persistir un modelo, usa `model.to_gbq()`. Para cargar un modelo persistido, usa `bpd.read_gbq_model()`.
