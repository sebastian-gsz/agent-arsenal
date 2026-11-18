---
name: gcp-spark
description: |
  Desarrolla y ejecuta código Spark en clústeres de Dataproc y Dataproc Serverless.
  Lee y escribe datos usando catálogos BigLake Iceberg, BigQuery y Spanner.
  Depura fallos de ejecución.
  Úsala cuando:
  - Escribas pipelines ETL de Spark en GCP.
  - Entrenes o ejecutes inferencia con modelos de ML con spark en GCP.
  - Gestiones clústeres, jobs, batches y sesiones interactivas de Spark.
  No la uses cuando:
  - Escribas scripts genéricos de Python que no usan Spark.
  - Realices queries SQL simples que pueden hacerse directamente en BigQuery.
license: Apache-2.0
metadata:
  version: v2
  publisher: google
---

# Spark on Dataproc

> [!IMPORTANT]
>
> DEBES seguir SIEMPRE el Task Execution Workflow al escribir código Spark.

## Task Execution Workflow

1.  **Entiende los esquemas**: **SIEMPRE** usa la skill
    `@skill:discovering-gcp-data-assets` o `references/schema_direct_inspection.md`
    para entender los esquemas de entrada y salida. Incluye el esquema en tu proceso
    de razonamiento ANTES de generar cualquier código. NO inventes nombres de columnas.
2.  **Genera el código Spark**:
    *   **Formato de salida**: **SIEMPRE** genera el código en formato **Python
        Notebooks (.ipynb)**. Genera scripts (.py) solo si se solicita explícitamente.
    *   **Leer y escribir datos**: **SIEMPRE** consulta
        `references/read_write_data.md` al leer o escribir datos.
    *   **Tareas de ML**: consulta la skill `@skill:ml-best-practices` y
        `references/ml_tasks.md` al generar código de ML.
    *   **Optimizaciones de Spark**: **SIEMPRE** consulta
        `references/spark_optimizations.md` al generar código Spark y aplica las
        optimizaciones siempre que sea aplicable.
3.  **Verifica el esquema antes de escribir**: **SIEMPRE** verifica que el esquema
    del dataframe y del destino coincidan; usa `df.printSchema()` para el esquema del
    dataframe y consulta la skill `@skill:discovering-gcp-data-assets` o
    `references/schema_direct_inspection.md` para verificar el esquema del destino.
4.  **Compila el código antes de ejecutarlo**: para notebooks, conviértelos primero a
    script de Python con `jupyter nbconvert --to script your-notebook.ipynb` y luego
    compila el código con `python3 -m py_compile your-notebook.py`.
5.  **Ejecuta el script**: SOLO cuando generes un script `.py` consulta
    `references/gcloud_dataproc.md` para escribir el comando que ejecuta el código
    generado en Dataproc. Esto NO aplica cuando generas notebooks.

--------------------------------------------------------------------------------

## Checklist de errores comunes

> [!CAUTION]
>
> Asegúrate de verificar esta checklist para evitar errores

Antes de enviar un job, verifica:

-   [ ] **Todas las importaciones presentes** (`col`, `when`, `lit`, etc. de
    `pyspark.sql.functions`)
-   [ ] **`vector_to_array` del módulo correcto** usa `from pyspark.ml.functions
    import vector_to_array` (NO `pyspark.sql.functions`)
-   [ ] **El esquema del DataFrame coincide con la tabla Iceberg de destino** verifícalo
    con `df.printSchema()` antes de escribir
-   [ ] **Archivos CSV leídos con `header` y `inferSchema`** sin estas opciones, la
    fila de cabecera se convierte en datos y todas las columnas quedan como strings
-   [ ] **Evita toPandas()** Convertir un dataframe de pyspark a pandas llamando a
    toPandas() puede provocar errores de falta de memoria. Solo es aceptable para crear
    visualizaciones en Spark 3.5

--------------------------------------------------------------------------------

## Requisitos de IAM

La cuenta de servicio de Dataproc necesita:

*   `roles/dataproc.worker`: Ejecución de jobs
*   `roles/biglake.admin`: Gestión de tablas Iceberg
*   `roles/bigquery.jobUser`: Materialización de queries
*   `roles/storage.objectUser`: Lectura/escritura en GCS
*   `roles/spanner.databaseUser`: Escrituras en Spanner

--------------------------------------------------------------------------------

## Gestión de recursos de Spark

Consulta `references/gcloud_dataproc.md` para ver las guías detalladas sobre la
gestión de clústeres, jobs, batches y sesiones interactivas de Spark.
