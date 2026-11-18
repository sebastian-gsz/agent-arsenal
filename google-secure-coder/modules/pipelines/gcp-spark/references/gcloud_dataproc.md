# Integración de Dataproc y Spark
Gestiona recursos de Spark en Google Cloud Dataproc Clusters y Serverless,
  incluyendo la configuración de clústeres; el lanzamiento de jobs y batches; la
  gestión de session templates de serverless, y la inspección de salidas.

## Contexto

Dataproc es el servicio administrado de Google Cloud para ejecutar cargas de
trabajo de Hadoop y Spark. Las dos modalidades básicas son:

-   **Clusters** aka **Dataproc on GCE**: los usuarios crean un clúster y luego
    envían uno o más jobs de Spark u otros. Los usuarios tienen control sobre los
    recursos de las VM subyacentes.
-   **Serverless Spark** aka **Dataproc Serverless**, donde los usuarios no controlan
    los recursos de las VM subyacentes:
    -   Los usuarios pueden enviar **batches**, que aprovisionan los recursos
        subyacentes, lanzan un job y destruyen los recursos, todo en una sola
        operación.
    -   Los usuarios también pueden crear **sesiones interactivas** persistentes.
        Estas generalmente se crean a través de una interfaz Jupyter en lugar de
        gcloud, pero las sesiones existentes pueden inspeccionarse con gcloud.
    -   Los usuarios pueden crear **session templates** como forma de crear varias
        sesiones usando la misma configuración.

Los usuarios no siempre conocen la terminología técnicamente correcta para
Clusters vs. Serverless; por ejemplo, pueden pedir "jobs" o "spark jobs" pero
referirse a Serverless Batches.

## Configuración

## Preferencias de proyecto y región/ubicación

Los usuarios configuran `gcloud` para apuntar al proyecto y región/ubicación
deseados. Asume que gcloud ya está instalado.

Consulta la configuración con:

```
gcloud config get project
gcloud config get dataproc/region
gcloud config get dataproc/location
gcloud config get compute/region
```

Si la región y la ubicación no están establecidas, puedes sugerir usar la
compute region.

### Prefiere MCP si es posible

> [!IMPORTANT] Si tienes acceso a uno o más servidores MCP relacionados con Dataproc
> o Serverless Spark, DEBES usar esas herramientas MCP en lugar de gcloud. SOLO
> recurre a gcloud si las herramientas MCP no están disponibles.

Al usar MCP, lo ÚNICO para lo que usas gcloud es consultar
proyecto/región/ubicación para pasarlos como argumentos a las herramientas MCP.

### gcloud como respaldo

Si no hay servidores MCP disponibles o no hay herramientas que se puedan usar para
tu caso de uso, usa `gcloud` para interactuar con Dataproc.

En general, los comandos de Dataproc Clusters y Serverless Batches aceptan
`--region`. Los comandos de Dataproc Serverless Sessions aceptan `--location`.

## Dataproc Clusters

Usa esta sección si el usuario pide "spark jobs", "spark clusters",
"clusters", "cluster jobs" o simplemente "jobs". **No** uses esta sección si el
usuario pide "serverless jobs", "serverless batches" o "batches".

### Listar clústeres

Prefiere MCP si está disponible. Si usas gcloud, usa esta plantilla de comando:

```
gcloud dataproc clusters list \
    --format="json(\
clusterName,\
clusterUuid,\
projectId,\
region,\
creator,\
status)" \
    --sort-by="~status.stateStartTime" \
    --limit=100
```

Consejos:

-   **Importante:** incluye siempre un límite; el valor por defecto es sin límite,
    lo que puede producir demasiada salida para procesar.
-   Añade un `--filter` para limitar los resultados, p. ej. `status.state = ACTIVE AND
    clusterName = mycluster AND labels.env = staging AND labels.starred = *`

### Listar jobs

Prefiere MCP si está disponible. Si usas gcloud, usa esta plantilla de comando:

```
gcloud dataproc jobs list \
    --format="json(\
jobType,\
reference,\
placement.clusterName,\
status.state,\
status.stateStartTime)" \
    --sort-by="~status.stateStartTime" \
    --limit=100
```

Consejos:

-   **Importante:** incluye siempre un límite; el valor por defecto es sin límite,
    lo que puede producir demasiada salida para procesar.
-   Añade un `--filter` para limitar los resultados, p. ej. `status.state = ACTIVE AND
    labels.env = staging AND labels.starred = *`

## Dataproc Serverless

Usa esta sección si el usuario pide:

-   batches, serverless batches, spark batches, serverless jobs
-   spark sessions, serverless spark sessions, spark interactive sessions,
    serverless interactive sessions, spark notebooks, spark kernels

**No** uses esta sección si el usuario pide
spark jobs, cluster jobs o simplemente jobs, **a menos que** hayas confirmado con
el usuario que está usando Serverless.

### Listar batches

Prefiere MCP si está disponible. Si usas gcloud, usa esta plantilla de comando:

```
gcloud dataproc batches list \
    --format="json(batchType, createTime, creator, name, state, stateTime)" \
    --sort-by="~stateTime" \
    --limit=100
```

Consejos:

-   **Importante:** incluye siempre un límite; el valor por defecto es sin límite,
    lo que puede producir demasiada salida para procesar.
-   Añade un `--filter` para limitar los resultados, p. ej. `(state = RUNNING and create_time <
    "2023-01-01T00:00:00Z") or labels.environment=production`

### Lanzar batches

> [!WARNING] Esto NO aplica a la ejecución de **Python Notebooks (.ipynb)**.
> [!IMPORTANT] Consulta esta guía SOLO para ejecutar **PySpark Script (.py)**.

Determina las propiedades y la configuración que requiere el script pyspark antes
de ejecutar el comando de envío del job

#### Comando básico de envío de batch

Prefiere MCP si está disponible. Si usas gcloud, usa esta plantilla de comando:

Amplía el comando básico con argumentos relacionados con iceberg, spanner o
xgboost según lo que necesite el script a ejecutar.

```
gcloud dataproc batches submit pyspark <SCRIPT_PATH.py> \
    --project=<PROJECT_ID> \
    --region=<GCP_REGION> \
    --version=2.3 \
    --deps-bucket=<GCS_PATH>
```
DEBES establecer `--deps-bucket` en una ruta de GCS para subir las dependencias
de la carga de trabajo.

> [!IMPORTANT] Puede esperarse que los batches de Dataproc Serverless tarden mucho
> tiempo. **Tiempo típico de ejecución inicial:** 10-15 minutos. Es un comportamiento
> **NORMAL**.
> [!WARNING]
> **¡NO CANCELES PREMATURAMENTE!**

#### Lectura o escritura en un catálogo BigLake Iceberg
Si el script pyspark lee o escribe datos en un catálogo BigLake Iceberg, establece
 estas propiedades
```
--properties="\
spark.sql.catalog.<CATALOG_NAME>=org.apache.iceberg.spark.SparkCatalog,\
spark.sql.catalog.<CATALOG_NAME>.type=rest,\
spark.sql.catalog.<CATALOG_NAME>.uri=\
https://biglake.googleapis.com/iceberg/v1/restcatalog,\
spark.sql.catalog.<CATALOG_NAME>.io-impl=\
org.apache.iceberg.gcp.gcs.GCSFileIO,\
spark.sql.catalog.<CATALOG_NAME>.header.x-goog-user-project=<PROJECT_ID>,\
spark.sql.catalog.<CATALOG_NAME>.warehouse=<WAREHOUSE>,\
spark.sql.catalog.<CATALOG_NAME>.rest.auth.type=\
org.apache.iceberg.gcp.auth.GoogleAuthManager,\
spark.sql.extensions=\
org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions"
```
Si el BigLake es de tipo GCS Catalog entonces `WAREHOUSE="gs://<CATALOG_NAME>"`, si
 es de tipo BQ federated entonces `WAREHOUSE="bq://projects/<PROJECT_ID>"`

#### Lectura de datos de spanner
Incluye el jar de conexión de spark spanner usando el argumento
 `--jars=gs://spark-lib/spanner/spark-3.5-spanner-1.2.2.jar`

#### XGBoost
XGBoost requiere que la asignación dinámica de spark esté desactivada. Establece
 propiedades adicionales:
```
--properties="spark.dynamicAllocation.enabled=false"
```

### Crear sesiones

**No** crees sesiones con gcloud para usarlas en notebooks. En su lugar, indica al
usuario que asocie el notebook con un kernel usando el Kernel Selector:

1. Haz clic en "Remote Spark Kernels"
2. Elige un nombre de kernel que termine en "on Serverless Spark"

Es normal que la creación de kernels Serverless tarde aproximadamente 2 minutos
o más.

### Listar sesiones

Prefiere MCP si está disponible. Si usas gcloud, usa esta plantilla de comando:

```
gcloud beta dataproc sessions list \
    --format="json(createTime, uuid, creator, state, jupyterSession, sparkConnectSession)" \
    --sort-by="~createTime" \
    --limit=100
```

Consejos:

-   **Importante:** incluye siempre un límite; el valor por defecto es sin límite,
    lo que puede producir demasiada salida para procesar.
-   Añade un `--filter` para limitar los resultados, p. ej. `state = ACTIVE AND
    labels.env = staging AND create_time >= "2023-01-01T00:00:00Z"`
