---
name: ml-best-practices
description: |
  REGLA CRÍTICA: DEBES usar esta skill siempre que la tarea implique tareas de machine learning o análisis de datos.
  Usa esta skill si el prompt o los requisitos del usuario mencionan alguno de los siguientes puntos:
    * Clustering
    * Clasificación
    * Regresión
    * Predicción de series temporales
    * Pruebas estadísticas
    * Comparación de modelos
    * ML
    * Análisis de datos

  TRASPASO SQL/BigQuery ML: si el usuario requiere una solución SQL, usa esta skill para dictar los PASOS DE ANÁLISIS (p. ej., celdas de análisis en markdown, lógica de visualización), pero delega en `bigquery` toda la sintaxis SQL.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Buenas prácticas de ML

Quiero leer una historia sobre los datos, no solo ejecutar código. Asegúrate de
que cada celda de código vaya seguida de una celda de markdown que analice los
resultados. Termina el notebook con un resumen que responda de forma integral al
prompt.

Si hay una buena correspondencia entre la solicitud del usuario y un plan de
ejemplo asociado, adapta ese plan de ejemplo para responder por completo a la
solicitud del usuario:

## Clustering:

Identifica grupos distintos según sus features.

-   Comprende el esquema y las descripciones de los campos.
-   Visualiza las features mencionadas en el prompt (p. ej., con histogramas o
    scatterplots).
-   Transforma las fechas en timestamps.
-   Antes de aplicar encoders, verifica si el dataset ya contiene features
    pre-codificadas y prefiere las representaciones numéricas existentes.
-   Prefiere conservar los datos en lugar de eliminarlos cuando sea posible.
-   Transforma los datos ordinales con un ordinal encoder.
-   Transforma los datos nominales con un one hot encoder.
-   Estandariza las features numéricas.
-   Realiza clustering con un rango de valores y registra el silhouette score.
-   Elige el número óptimo de clusters según el silhouette score.
-   Usa reducción de dimensionalidad (p. ej., PCA) para proyectar los datos a
    dos dimensiones.
-   Representa las muestras en un scatterplot de dos dimensiones usando las
    etiquetas de cluster como hue.
-   Representa las muestras en un scatterplot de dos dimensiones usando una
    feature discreta como hue.
-   Describe los clusters en texto mediante las distribuciones de las features
    o sus valores típicos.
-   Conclusión: responde de forma integral al prompt en una celda final de
    markdown.

## Predicción de series temporales:

Desarrolla un modelo predictivo para estimar valores futuros basándote en
tendencias históricas. ¿Cómo pueden los distintos enfoques de modelado afectar
la precisión de la predicción?

-   Comprende el esquema y las descripciones de los campos.
-   Visualiza la feature objetivo a lo largo del tiempo con una granularidad
    razonable.
-   Realiza siempre una división cronológica de los datos para crear los
    conjuntos de entrenamiento, validación y prueba.
-   ¿Hay tendencias estacionales?
-   Comprueba la estacionariedad.
-   Analiza los posibles enfoques de modelado. ¿Cómo pueden los distintos
    enfoques de modelado afectar la precisión de la predicción?
-   Entrena dos modelos de forecasting de series temporales para predecir la
    feature objetivo. Usa la información previa de estacionalidad y
    estacionariedad como hiperparámetros del modelo.
-   Predice la feature objetivo para los conjuntos de entrenamiento y
    validación.
-   Opcionalmente, ajusta los hiperparámetros de los modelos con el conjunto de
    validación.
-   Visualiza la feature objetivo real y la predicha frente al tiempo para cada
    modelo, en los conjuntos de entrenamiento y validación.
-   Evalúa el rendimiento en validación con métricas de error.
-   Selecciona un modelo.
-   Reentrena el modelo seleccionado con los conjuntos de prueba y validación.
-   Predice los valores de prueba con el modelo seleccionado.
-   Visualiza la feature objetivo promedio y los valores de prueba predichos.
-   Conclusión: responde de forma integral al prompt en una celda final de
    markdown.

## Análisis exploratorio de datos / Detección de anomalías:

Identifica y describe cualquier outlier, patrón inusual o tendencia
significativa observada en los datos. Aporta visualizaciones que respalden tus
hallazgos.

-   Comprende el esquema y las descripciones de los campos.
-   Visualiza la distribución de la feature objetivo de forma que se muestren
    los outliers.
-   Identifica y describe cualquier outlier en la feature objetivo.
-   Visualiza las relaciones entre la feature objetivo y las demás features.
-   Identifica y describe patrones inusuales o tendencias significativas.
-   Visualiza los patrones y las tendencias.
-   Conclusión: responde de forma integral al prompt en una celda final de
    markdown.

## Clasificación:

Dados los datos, ¿podemos clasificar según la feature objetivo?

-   Comprende el esquema y las descripciones de los campos.
-   Identifica las filas que no tienen sentido. ¿Cuántas hay y qué contienen?
-   Identifica las filas sin valor objetivo. ¿Cuántas hay y qué contienen?
-   Elimina las filas que no cumplan el esquema o que no tengan valor objetivo
    (si es razonable hacerlo).
-   Divide los datos en conjuntos de entrenamiento, validación y prueba.
-   Crea features que representen cuándo faltan datos, si esto tiene sentido.
-   Gestiona los datos faltantes. Prefiere conservar los datos en lugar de
    eliminarlos cuando sea posible.
-   Antes de aplicar encoders, verifica si el dataset ya contiene features
    pre-codificadas y prefiere las representaciones numéricas existentes.
-   Transforma los datos ordinales con un ordinal encoder.
-   Transforma los datos nominales con un one hot encoder.
-   Estandariza las features numéricas.
-   Entrena varios modelos.
-   Si hay evidencia de overfitting, regulariza y reentrena el modelo.
-   Si hay evidencia de underfitting, considera añadir o ingeniar features.
-   Evalúa los modelos.
-   Crea matrices de confusión.
-   Conclusión: responde de forma integral al prompt en una celda final de
    markdown.

## Regresión:

Predice la feature objetivo de valor continuo.

-   Comprende el esquema y las descripciones de los campos.
-   Identifica las filas que no tienen sentido. ¿Cuántas hay y qué contienen?
-   Identifica las filas sin valor objetivo. ¿Cuántas hay y qué contienen?
-   Desarrolla una comprensión de los datos y determina cómo manejar los
    valores faltantes. Esto debe tener sentido en el contexto del negocio.
-   Identifica cualquier fuente potencial de group leakage. Agrega cuando sea
    apropiado para prevenirlo.
-   Visualiza la feature objetivo.
-   Divide los datos en conjuntos de entrenamiento, validación y prueba.
-   Gestiona los datos faltantes. Prefiere conservar los datos en lugar de
    eliminarlos cuando sea posible.
-   Antes de aplicar encoders, verifica si el dataset ya contiene features
    pre-codificadas y prefiere las representaciones numéricas existentes.
-   Transforma los datos ordinales con un ordinal encoder.
-   Transforma los datos nominales con un one hot encoder. Restringe las
    features categóricas de alta cardinalidad a un tamaño manejable.
-   Estandariza las features numéricas.
-   Entrena varios modelos.
-   Visualiza los valores reales frente a los predichos en los datos de
    entrenamiento y validación.
-   Si hay evidencia de overfitting, regulariza y reentrena el modelo.
-   Si hay evidencia de underfitting, considera añadir o ingeniar features.
-   Evalúa el error del modelo.
-   Conclusión: responde de forma integral al prompt en una celda final de
    markdown.

## Comparación de modelos ML:

Evalúa y compara varios modelos para determinar cuál es el más adecuado para
producción según su poder predictivo, robustez y viabilidad.

-   Comprende el esquema y alinea las métricas con los objetivos del negocio
    (p. ej., costo de falsos positivos vs. falsos negativos).
-   Establece líneas base: define un baseline ingenuo (clase mayoritaria/media)
    y un baseline ML simple (p. ej., regresión logística/lineal).
-   Garantiza una validación rigurosa: usa divisiones de datos idénticas y
    fijas para todos los modelos y realiza validación cruzada $k$-fold.
-   Si los datos son temporales, usa divisiones cronológicas para la validación.
-   Selecciona y reporta métricas más allá del accuracy (p. ej., F1-Score,
    PR-AUC, MAE, RMSE) que reflejen el impacto en el negocio.
-   Usa bootstrapping para calcular intervalos de confianza del 95% en las
    métricas clave y determinar la significancia estadística.
-   Realiza un análisis de error por segmentos (slices): evalúa el rendimiento
    del modelo en subpoblaciones y datos demográficos clave para identificar
    sesgos o modos de fallo específicos.
-   Inspecciona y compara matrices de confusión, gráficos de residuos y curvas
    de calibración.
-   Evalúa los trade-offs operativos: considera la latencia de inferencia, el
    tiempo de entrenamiento, el costo de cómputo y el tamaño del modelo.
-   Evalúa la interpretabilidad con herramientas como SHAP o LIME donde se
    requiera transparencia.
-   Conclusión: recomienda el modelo óptimo para el caso de uso específico,
    justificando la elección tanto por el rendimiento como por la viabilidad en
    producción.

## Sin coincidencia:

-   Comprende el esquema y las descripciones de los campos.
-   Identifica las filas que no tienen sentido. ¿Cuántas hay y qué contienen?
-   Identifica las filas sin valor objetivo. ¿Cuántas hay y qué contienen?
-   Elimina las filas que no cumplan el esquema o que no tengan valor objetivo
    (si es razonable hacerlo).
-   Crea features que representen cuándo faltan datos, si esto tiene sentido.
-   Gestiona los datos faltantes. Prefiere conservar los datos en lugar de
    eliminarlos cuando sea posible.
-   Antes de aplicar encoders, verifica si el dataset ya contiene features
    pre-codificadas y prefiere las representaciones numéricas existentes.
-   Transforma los datos ordinales con un ordinal encoder.
-   Transforma los datos nominales con un one hot encoder.
-   Estandariza las features numéricas.
-   Conclusión: responde de forma integral al prompt en una celda final de
    markdown.

## Prácticas esenciales de ML

[!IMPORTANT] Sigue SIEMPRE estas prácticas de ML

-   **Orden estricto de featurización**: para aprendizaje supervisado,
    **SIEMPRE** divide el dataset en datos de entrenamiento y de prueba
    **ANTES** de ajustar los pipelines de preprocesamiento (p. ej., escalado,
    codificación). Ajusta los pipelines sobre los datos de entrenamiento y
    sobre los datos de prueba de forma independiente.

-   **Manejo de valores faltantes o NULL**: **SIEMPRE** verifica y maneja los
    valores faltantes y los NULL. Primero, analiza su frecuencia. Luego,
    decide si conservarlos, eliminarlos o imputarlos con un valor apropiado al
    contexto, y explica tu razonamiento.
