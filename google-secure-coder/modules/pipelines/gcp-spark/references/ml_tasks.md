# ML on Dataproc

**Patrones verificados** para entrenamiento de ML:
- **XGBoost**: usa `SparkXGBClassifier`
- **Spark ML nativo**: `GBTClassifier`, `RandomForestClassifier`,
  `LogisticRegression`

## LightGBM en Dataproc

> [!WARNING]
> **LightGBM tiene conflictos de dependencias en Dataproc Serverless.** El wrapper
> de LightGBM de SynapseML entra en conflicto con las librerías internas de Dataproc.

**Alternativas:**
1. **Usa XGBoost** — Rendimiento similar, soporte nativo en Spark.
2. **Usa Spark ML nativo** — `GBTClassifier` ofrece un gradient boosting similar
3. **Usa Vertex AI** — Entrena LightGBM en Vertex, exporta el modelo y cárgalo en Spark
   para la inferencia
4. **Usa un clúster de Dataproc** (no Serverless) — Mayor control sobre las dependencias

**Si es imprescindible usar LightGBM**, considera:
- Entrenar en un clúster dedicado de Dataproc creado con los paquetes Spark de LightGBM
  establecidos en las propiedades del clúster:
  `spark:spark.jars.packages=com.microsoft.azure:synapseml_2.12:1.1.3`
- **DEBES** desactivar el Autoscaling de Dataproc en el clúster
- Usar exportación del modelo a ONNX para la inferencia

---

## Restricciones de parámetros de XGBoost

> [!WARNING]
> **`SparkXGBClassifier` NO permite establecer el parámetro `objective` personalizado.**
> El objetivo se infiere automáticamente a partir del tipo de clasificador:
> - `SparkXGBClassifier` → `binary:logistic` (inferido)
> - `SparkXGBRegressor` → `reg:squarederror` (inferido)

> [!WARNING]
> **`SparkXGBClassifier`** requiere `dynamicAllocation=false`

**Prohibido** (provoca `ValueError`):
```python
# ❌ DO NOT do this
xgb = SparkXGBClassifier(objective="binary:logistic", ...)
```

**Correcto**:
```python
# ✅ Do this - objective is automatically set
xgb = SparkXGBClassifier(
    features_col="features",
    label_col="label",
    numRound=100,
    maxDepth=6,
)
```

Consulta la [documentación de XGBoost PySpark]
(https://xgboost.readthedocs.io/en/stable/python/python_api.html#module-xgboost.spark)
para ver los parámetros permitidos.
