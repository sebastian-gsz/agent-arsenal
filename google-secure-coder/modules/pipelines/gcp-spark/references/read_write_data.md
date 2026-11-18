# Guía definitiva para leer y escribir tablas usando código spark

> [!IMPORTANT]
> DEBES seguir SIEMPRE el Task Execution Workflow al escribir código spark.

## Task Execution Workflow
1. Configura la sesión de spark para leer o escribir datos desde el origen o el
   destino requerido
2. Lee o escribe los datos desde el origen o el destino requerido

## BigQuery

### Configuración de la sesión de Spark
No se necesita configuración de sesión para leer de BigQuery; todos los jars y la
configuración se proporcionan por defecto en Dataproc.

### Leer de BigQuery
#### Ejemplo básico para leer una tabla de Big Query
```python
df = spark.read.format("bigquery") \
    .option("table", "<PROJECT_ID>.<DATASET_NAME>.<TABLE_NAME>") \
    .load()
```

#### Ejecutar una query SQL de BigQuery
El conector de Spark BigQuery te permite ejecutar queries SELECT de Standard SQL
directamente sobre BigQuery y cargar los resultados en un dataframe de spark.
> [!IMPORTANT] Debes habilitar la opción views para ejecutar queries directas

```python
# Define Standard SQL query
sql = """
  SELECT name, SUM(number) as total
  FROM `bigquery-public-data.usa_names.usa_1910_2013`
  GROUP BY name
  LIMIT 10
"""

df = spark.read.format("bigquery").option("viewsEnabled", "true").load(sql)
```

#### Patrones de join con BigQuery
Al enriquecer datos con tablas de referencia de BigQuery:

```python
# Read BigQuery table directly (connector is built-in)
reference_df = spark.read.format("bigquery") \
    .option("table", "<PROJECT_ID>.<DATASET_NAME>.<TABLE_NAME>") \
    .load()

# Join with your main DataFrame
enriched_df = main_df.join(
    reference_df.select("entity_id", "entity_name", "category"),
    on="entity_id",
    how="left"
)
```

#### Consejos de optimización
1. Selecciona solo las columnas necesarias de BigQuery
```python
ref_df = spark.read.format("bigquery") \
    .option("table", "<PROJECT_ID>.<DATASET_NAME>.<LARGE_TABLE_NAME>") \
    .option("filter", "active = true") \
    .load() \
    .select("id", "name")
```

2. Usa broadcast joins para tablas de referencia pequeñas
```python
from pyspark.sql.functions import broadcast

enriched_df = main_df.join(
    broadcast(small_ref_df),  # < 10MB
    on="key"
)
```
3. Haz cache de los datos de referencia si se usan varias veces, ejemplo: `ref_df.cache()`

### Escribir en BigQuery

#### Datos sin particionar
Escribe el DataFrame en BigQuery usando escrituras directas (direct writes),
 - Establece writeMethod en direct
 - Establece mode en uno de `overwrite`, `append`, `errorifexists` según la tarea
```python
df.write \
    .format("bigquery") \
    .option("table", "<PROJECT_ID>.<DATASET_NAME>.<TABLE_NAME>") \
    .option("writeMethod", "direct") \
    .mode("overwrite") \
    .save()
```
#### Datos particionados
Escribe el DataFrame en una tabla de BigQuery particionada usando escrituras
indirectas. En este método, los datos se escriben primero en GCS y luego se cargan
en BigQuery. Se requiere un bucket de GCS como ubicación temporal de los datos.
 - Establece temporaryGcsBucket en un nombre de bucket de GCS
 - Establece una ubicación de path para el checkpoint
```python
df.write \
  .format("bigquery") \
  .option("temporaryGcsBucket","<STAGING_GCS_BUCKET>") \
  .option("checkpointLocation", "<STAGING_PATH>") \
  .save("<PROJECT_ID>.<DATASET_NAME>.<TABLE_NAME>")
```

## BigLake Iceberg Catalog

### Configuración de la sesión de Spark
- [!WARNING] El catálogo BigLake Iceberg debe configurarse en la sesión de spark
  antes de poder leer las tablas.
- Se pueden configurar varios catálogos Iceberg en la misma sesión de spark
- [!IMPORTANT] **DEBES** identificar la capa de almacenamiento de la tabla BigLake
  Iceberg antes de configurar la sesión de spark. Usa
  `@skill:discovering-gcp-data-assets` para **buscar** y **consultar** los detalles
  de la tabla. Revisa el valor del campo `metadataPath` de los resultados de la
  consulta; si empieza con `gs:` continúa con almacenamiento GCS, si empieza con
  `s3:` continúa con almacenamiento S3.
- [!WARNING] **NO** asumas catalog_name, project_id ni implementación de io de una
  tabla, **CONSULTA SIEMPRE** los detalles de la tabla antes de continuar.

Ejemplo con almacenamiento GCS:
```python
spark = SparkSession.builder \
    .appName("<APP_NAME>") \
    .config(
        "spark.sql.extensions",
        "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
    ) \
    .config(
        "spark.sql.catalog.<GCS_CATALOG_NAME>",
        "org.apache.iceberg.spark.SparkCatalog",
    ) \
    .config("spark.sql.catalog.<GCS_CATALOG_NAME>.type", "rest") \
    .config(
        "spark.sql.catalog.<GCS_CATALOG_NAME>.uri",
        "https://biglake.googleapis.com/iceberg/v1/restcatalog",
    ) \
    .config(
        "spark.sql.catalog.<GCS_CATALOG_NAME>.warehouse",
        "gs://<GCS_CATALOG_NAME>",
    ) \
    .config(
        "spark.sql.catalog.<GCS_CATALOG_NAME>.header.x-goog-user-project",
        "<CATALOG_PROJECT_ID>",
    ) \
    .config(
        "spark.sql.catalog.<GCS_CATALOG_NAME>.rest.auth.type",
        "org.apache.iceberg.gcp.auth.GoogleAuthManager",
    ) \
    .config(
        "spark.sql.catalog.<GCS_CATALOG_NAME>.io-impl",
        "org.apache.iceberg.gcp.gcs.GCSFileIO",
    ) \
    .getOrCreate()
```

Ejemplo con almacenamiento S3:
```python
spark = SparkSession.builder \
    .appName("<APP_NAME>") \
    .config(
        "spark.sql.extensions",
        "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>",
        "org.apache.iceberg.spark.SparkCatalog",
    ) \
    .config("spark.sql.catalog.<CATALOG_NAME>.type", "rest") \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.uri",
        "https://biglake.googleapis.com/iceberg/v1/restcatalog",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.warehouse",
        f"bl://projects/<CATALOG_PROJECT_ID>/catalogs/<CATALOG_NAME>",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.header.x-goog-user-project",
        f"<CATALOG_PROJECT_ID>",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.rest.auth.type",
        "org.apache.iceberg.gcp.auth.GoogleAuthManager",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.io-impl",
        "org.apache.iceberg.aws.s3.S3FileIO",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.header.X-Iceberg-Access-Delegation",
        "vended-credentials",
    ) \
    .config(
        "spark.sql.catalog.<CATALOG_NAME>.s3.client-factory-impl",
        "org.apache.iceberg.aws.s3.DefaultS3FileIOAwsClientFactory",
    ) \
    .getOrCreate()
```

### Leer del catálogo BigLake Iceberg
- **DEBES** establecer el catálogo de spark actual antes de intentar leer, usando
  `spark.catalog.setCurrentCatalog("<CATALOG_NAME>")`. **NO** uses backticks
  en esta llamada.
- [!IMPORTANT] **DEBES RODEAR SIEMPRE** el nombre del catálogo con backticks en las
  sentencias SQL y en las llamadas a `load()` para garantizar el manejo correcto de
  caracteres especiales (p. ej., `catalog_name`).
```python
spark.catalog.setCurrentCatalog("<CATALOG_NAME>")
df = spark.read.format("iceberg") \
    .load("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>")
```

### Escribir en el catálogo BigLake Iceberg
[!IMPORTANT] Sigue estas instrucciones antes de escribir código para escribir en
tablas iceberg de BigLake
- **SIEMPRE** usa la API `DataFrameWriterV2` de Spark (`writeTo`) para escribir en
  tablas Iceberg. Ofrece una API más rica con modos de escritura específicos.
- **DEBES** establecer el catálogo de spark actual antes de intentar escribir, usando
  `spark.catalog.setCurrentCatalog("<CATALOG_NAME>")`. **NO** uses backticks
  en esta llamada.
- **SIEMPRE** asegúrate de que el namespace exista antes de escribir, usando:
    ```python
    spark.sql(
        "CREATE NAMESPACE IF NOT EXISTS `<CATALOG_NAME>`.<NAMESPACE_NAME>"
    )
    ```
> [!IMPORTANT] **DEBES RODEAR SIEMPRE** el nombre del catálogo con backticks en las
> sentencias SQL y en las llamadas a `writeTo()` para garantizar el manejo correcto
> de caracteres especiales.
- [!WARNING] El catálogo BigLake Iceberg debe configurarse antes de poder leer las
  tablas.
```python
# The basic pattern
spark.catalog.setCurrentCatalog("<CATALOG_NAME>")
df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>").using("iceberg")
```

#### Modos de escritura

- **DataFrameWriterV2 `createOrReplace()`**: Crea la tabla si no existe, o la
  reemplaza si existe.
    ```python
    df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
        .using("iceberg") \
        .createOrReplace()
    ```

- **DataFrameWriterV2 `append()`**: Equivale a `INSERT INTO`. Añade datos a una
  tabla existente.
    ```python
    df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
        .using("iceberg") \
        .append()
    ```
- **SPARK SQL MERGE INTO**: Realiza updates, inserts y deletes a nivel de fila
  reescribiendo los archivos de datos que contienen las filas afectadas. La tabla
  fuente puede persistirse como temp view para permitir usarla en la sentencia
  MERGE INTO. La vista fuente y la tabla de destino deben unirse por la(s) clave(s)
  primaria(s) para garantizar updates, inserts y deletes correctos a nivel de fila.
    ```python
    df.createOrReplaceTempView("source_updates")

    spark.sql(f"""
        MERGE INTO `<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME> t
        USING source_updates s
        ON t.id = s.id
        WHEN MATCHED THEN UPDATE SET *
        WHEN NOT MATCHED THEN INSERT *
    """)
    ```

- [!WARNING] Estos son modos de escritura menos preferidos, úsalos SOLO si se te
  pide explícitamente:
*   **`create()`**: Equivale a `CREATE TABLE AS SELECT`. Falla si la tabla ya
    existe.
    ```python
    df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
        .using("iceberg") \
        .create()
    ```
*   **`replace()`**: Equivale a `REPLACE TABLE AS SELECT`.
    ```python
    df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
        .using("iceberg") \
        .replace()
    ```
*   **`overwritePartitions()`**: Equivale a `INSERT OVERWRITE` dinámico.
    Sobrescribe las particiones de la tabla que coinciden con los datos del
    DataFrame.
    ```python
    df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
        .using("iceberg") \
        .overwritePartitions()
    ```
*   **`overwrite(condition)`**: Sobrescribe los datos que cumplen una condición
    específica (p. ej., overwrite estático).
    ```python
    from pyspark.sql.functions import col
    df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
        .using("iceberg") \
        .overwrite(col("level") == "INFO")
    ```

#### Escrituras avanzadas de Iceberg

##### Escritura con particiones
Usa funciones de transformación de particionado si escribes en tablas particionadas
usando campos de tipo timestamp. Ej.: years, months, days, hours, etc.
Ejemplo: 
```python
from pyspark.sql.functions import days

df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
    .using("iceberg") \
    .partitionedBy("level", days("ts")) \
    .createOrReplace()
```

##### Evolución del esquema
Iceberg soporta evolución segura del esquema. Puedes fusionar cambios de esquema
durante un append.
```python
df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>") \
    .option("mergeSchema", "true") \
    .append()
```

##### Escribir en un branch específico
```python
# To insert into an `audit` branch
df.writeTo("`<CATALOG_NAME>`.<NAMESPACE_NAME>.<TABLE_NAME>.branch_audit") \
    .append()
```

## Google Cloud Storage (GCS)
No se necesita configuración de sesión para leer de GCS; todos los jars y la
configuración se proporcionan por defecto en Dataproc.

### Leer de GCS

#### Formato CSV
Lee con `header` y `inferSchema`; sin estas opciones, la fila de cabecera se
convierte en datos y todas las columnas quedan como strings
```python
df = spark.read.option("header", "true") \
    .option("inferSchema", "true") \
    .csv("gs://<BUCKET>/<PATH_TO_CSV>")
```

#### Formato Parquet
`df = spark.read.parquet("gs://<BUCKET>/<PATH_TO_PARQUET>")`

#### Formato Json
`df = spark.read.json("gs://<BUCKET>/<PATH_TO_JSON>")`

### Escribir en GCS

#### Formato Csv
`df.write.csv("gs://bucket/output_path")`

#### Artefactos de modelos
```python
# Warehouse already includes gs://, don't add it again
warehouse = spark.conf.get(f"spark.sql.catalog.<CATALOG_NAME>.warehouse")
model_path = f"{warehouse}/models/my_model"
model.write().overwrite().save(model_path)
```

## Spanner
### Configuración de la sesión de Spark
El jar de spanner debe configurarse en la sesión de spark antes de poder leer las
tablas. En el código del notebook SOLO podemos validar que el conector de spanner
está configurado.
Ejemplo:
```python
spark = SparkSession.builder \
    .appName("<APP_NAME>") \
    .config(
        "spark.jars.packages",
        "gs://spark-lib/spanner/spark-3.5-spanner-1.3.0.jar",
    ) \
    .getOrCreate()
```

#### Leer tablas de Spanner
```python
df = spark.read.format('cloud-spanner') \
  .option("projectId", "<PROJECT_ID>") \
  .option("instanceId", "<INSTANCE_ID>") \
  .option("databaseId", "<DATABASE_ID>") \
  .option("table", "<TABLE_NAME>")
  .load()
```

### Escribir en Spanner
El conector soporta escribir en tablas de Spanner usando la API `write` de
DataFrame.

#### Modos de escritura
*   **`Append`**: Inserta filas en la tabla de Spanner existente. Es el modo de
    escritura por defecto.
    ```python
    df.write.format("cloud-spanner") \
        .option("projectId", "<PROJECT_ID>") \
        .option("instanceId", "<INSTANCE_ID>") \
        .option("databaseId", "<DATABASE_ID>") \
        .option("table", "<TABLE_NAME>") \
        .mode("append") \
        .save()
    ```

*   **`Overwrite`**: Borra los datos existentes antes de escribir.
    ```python
    df.write.format("cloud-spanner") \
        .option("projectId", "<PROJECT_ID>") \
        .option("instanceId", "<INSTANCE_ID>") \
        .option("databaseId", "<DATABASE_ID>") \
        .option("table", "<TABLE_NAME>") \
        .mode("overwrite") \
        .save()
    ```

