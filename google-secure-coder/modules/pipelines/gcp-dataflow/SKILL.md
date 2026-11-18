---
name: gcp-dataflow
description: |
  Guía la escritura, el empaquetado, la ejecución y la resolución de problemas de pipelines de Apache Beam en Dataflow. Úsala al crear pipelines nuevos, configurar Flex Templates o analizar el rendimiento de jobs de Dataflow. Las capacidades incluyen configuración de Java/Python/Go, integración con Cloud Build y análisis diagnóstico profundo de la salud del job y del autoscaling.
  Úsala cuando: - Crear un pipeline de Apache Beam en Dataflow. - Crear un Flex Template de Google Dataflow. - Usar un Google Dataflow Template existente. - Depurar un pipeline de Dataflow - Resolver problemas de un pipeline de Dataflow - Analizar el rendimiento de un pipeline de Dataflow.
  Capacidades clave: configuración de proyectos Java/Python/Go, Flex Templates (con Cloud Build) y diagnósticos de salud de jobs de streaming, cuellos de botella y autoscaling.
  NO la uses para: - Gestión general de recursos de GCP no relacionada con Dataflow. - Problemas de otros servicios de GCP (p. ej., GCE, GCS, BigQuery) a menos que afecten
    directamente a la ejecución del pipeline de Dataflow.
  - Tecnologías de pipeline distintas de Apache Beam en Dataflow.
license: Apache-2.0
metadata:
  version: v4
  publisher: google
---

# Pipelines de Apache Beam en Cloud Dataflow

## Autoría de pipelines

Usa esta sección al implementar la lógica de un pipeline de Dataflow con Apache Beam.

### Comprobar si existe un Google Dataflow Template existente

Google ofrece una variedad de Dataflow templates preconstruidos y de código abierto
que se pueden usar para escenarios comunes. Antes de implementar un pipeline desde
cero, DEBES seguir los pasos siguientes para comprobar si ya existe un Dataflow
template para la lógica de pipeline que necesitas implementar.

-   **Paso 1: Buscar un Google Dataflow Template que coincida**

    -   Identifica la **fuente** y el **sink** (p. ej., GCS a BigQuery) de la
        petición del usuario. *Nota*: NO debes continuar hasta que la fuente y el
        sink estén claramente identificados.
    -   **Acción**: Lista los templates del bucket público `dataflow-templates`:
        *   Para Classic templates, revisa `gs://dataflow-templates/latest`.
        *   Para Flex templates, revisa `gs://dataflow-templates/latest/flex`. Usa
            `gcloud storage ls` para listar el contenido.
    -   Empareja los templates por nombre o descripción con la fuente y el sink.
    -   Si no se encuentra ningún template que coincida, ve a **Crear un pipeline
        nuevo desde cero**.

-   **Paso 2: Confirmar la selección del template**

    -   Presenta al usuario el/los template(s) emparejados con una breve explicación
        de por qué coinciden, y toma nota de si es un template Classic o Flex.
    -   **Acción**: Pide al usuario una confirmación explícita para continuar con
        este template.
    -   Si el usuario lo rechaza o prefiere una solución personalizada, continúa con
        **Crear un pipeline nuevo desde cero**.

### Crear un pipeline nuevo desde cero

Usa esta sección al crear un proyecto nuevo para un pipeline de Dataflow desde
cero.

-   Si el usuario no dice explícitamente qué lenguaje (Java, Python, Go) se debe
    usar para escribir el pipeline, DEBES confirmar el lenguaje.
-   Determina qué versión del Beam SDK se debe usar buscando la versión de Apache
    Beam publicada más recientemente, a menos que el usuario ya use una versión
    concreta.
    -   **Acción**: Ejecuta una búsqueda web del último release del Apache Beam SDK.
-   DEBES usar la misma versión de Apache Beam de forma consistente en todo el
    proyecto, en Dockerfiles, `requirements.txt` y otros archivos similares donde se
    especifiquen versiones.

### Proyectos Java que usan Gradle

Usa esta sección al configurar un proyecto de pipeline Java de Dataflow usando
gradle.

-   **Shadow Jars (Fat Jars)**: NO propongas usar el plugin Shadow
    (`com.github.johnrengelman.shadow`) a menos que el usuario solicite
    explícitamente un Fat Jar.
-   **Paso de parámetros por línea de comandos**: usa el plugin `application` para
    pasar parámetros por línea de comandos.
-   **Alineación de dependencias de logging SLF4J**:
    -   Verifica la versión de `slf4j-api` que Apache Beam trae de forma transitiva.
    -   DEBES configurar el backend de logging de la aplicación (`slf4j-simple`,
        `logback-classic`, etc.) para que coincida exactamente con la versión
        major/minor del `slf4j-api` resuelto.

### Empaquetar un pipeline como Flex Template

Usa esta sección para empaquetar código de pipeline como Flex template.

Los Flex Templates ofrecen un entorno de lanzamiento hermético y reproducible para
un pipeline. Son fáciles de lanzar con `gcloud` o con orquestadores como Cloud
Composer. **DEBES** empaquetar el pipeline como Flex Template al crear proyectos
nuevos de pipelines de Dataflow.

Sigue los pasos siguientes:

-   **Proporciona instrucciones**: entrega al usuario, en el walkthrough,
    instrucciones sobre cómo reconstruir y ejecutar Flex Templates.
-   **Usa una única imagen Docker para pipelines Python**: para Flex Templates de
    Python, es mejor usar una sola imagen tanto para la imagen del template launcher
    como para el entorno de ejecución del worker (`--sdk_container_image`). ¿El
    pipeline de Python requiere dependencias adicionales (p. ej., usar
    `--requirements_file`, `--setup_file` o `--extra_package`)? Si es así,
    **DEBES recomendar la** **Configuración de Imagen Docker Única** para el Flex
    Template. Consulta [python_flex_template_reference.md][py-flex-ref] para más
    detalles.
-   **Prefiere Cloud Build antes que Docker local**:
    -   NO asumas que hay Docker local disponible en la máquina del workspace.
    -   **Acción**: sugiere y proporciona un `cloudbuild.yaml` listo para usar para
        construir y subir imágenes, a menos que se solicite explícitamente una
        configuración local.
    -   Al construir imágenes con Cloud Build en segundo plano, DEBES proporcionar el
        enlace donde el usuario puede monitorizar la operación de larga duración.
-   **Entrega de certificados SSL y secrets a los workers**:
    -   Si los certificados o las claves se almacenan en Secret Manager, **NUNCA**
        los incrustes en las capas de la imagen Docker. En su lugar, recupéralos
        dinámicamente en runtime dentro del ciclo de vida `DoFn.setup()` de Apache
        Beam usando la librería cliente de Secret Manager (escribiéndolos en el
        disco efímero del worker como `/tmp` solo si se requieren estrictamente rutas
        de archivos físicas). Asegúrate de que la Dataflow Worker Service Account
        tenga el rol `roles/secretmanager.secretAccessor`.

## Configurar templates proporcionados por Google

Usa esta sección cuando el usuario haya seleccionado un template proporcionado por
Google (Classic o Flex) y necesites configurarlo.

-   **Paso 1: Obtener los metadatos del template**

    -   Identifica el tipo de template:
        *   **Classic**: los archivos de metadatos están en `gs://dataflow-templates`
            y terminan con `_metadata` (p. ej.,
            `gs://dataflow-templates/latest/Word_Count_metadata`).
        *   **Flex**: los metadatos están incrustados en el archivo spec del template
            bajo `gs://dataflow-templates/latest/flex` (p. ej.
            `gs://dataflow-templates/latest/flex/Cloud_Datastream_to_BigQuery`).
    -   Lee el archivo de metadatos del template correspondiente para identificar los
        parámetros requeridos.
    -   **Nota**:
        *   Asegúrate de ejecutar una búsqueda recursiva sobre el bucket si es
            necesario para localizar los metadatos.
        *   Si los parámetros del template incluyen campos relacionados con UDF
            (p. ej., `javascriptTextTransformGcsPath`,
            `javascriptTextTransformFunctionName`), consulta la
            [guía de UDF][udf-guide] para escribir y configurar la UDF.
        *   **Staging de SSL / secrets basado en parámetros**: si el template
            proporcionado por Google requiere certificados SSL locales o secrets de
            Secret Manager, pasa rutas de GCS separadas por comas mediante el
            parámetro `extraFilesToStage`. El runner las dejará en `/extra_files` en
            las VM de los workers. Consulta la [guía de certificados SSL][ssl-cert-guide]
            para la sintaxis de referencia local (`/extra_files/...`).

-   **Paso 2: Obtener la configuración de red**

    -   **Acción**: ejecuta comandos `gcloud` para listar redes y subredes.
    -   Confirma con el usuario la red y la subred a usar.

-   **Paso 3: Identificar los parámetros requeridos y preparar los recursos**

    -   Extrae los parámetros requeridos de los metadatos del template.
    -   > [!IMPORTANT]
    -   > **Validación estricta de parámetros**: cualquier parámetro del JSON de
    -   > metadatos que **NO** tenga explícitamente `"isOptional": true` es
    -   > **estrictamente requerido** por la Dataflow API.
    -   > Esto aplica incluso si la descripción sugiere que tiene un valor por
    -   > defecto (p. ej., `csvFormat` o `badRecordsOutputTable` en algunos
    -   > templates).
    -   > Debes identificarlos y suministrarlos todos.
    -   Identifica qué parámetros los proporciona el usuario y cuáles deben ser
        resueltos o creados por ti.
        *   **Acción**: presenta estos parámetros al usuario usando Markdown
            clave-valor (viñetas) para mayor claridad y confirmación.
    -   **Manejo de esquemas**: si se requiere un parámetro de esquema JSON (como
        `schemaJSONPath` o `JSONPath`):
        *   **Acción**: pide al usuario la ruta de GCS de un archivo de esquema
            existente o el contenido JSON.
        *   Si el usuario no tiene un archivo de esquema, pídele los nombres de los
            campos y sus tipos. Construye el JSON de esquema localmente y preséntalo
            al usuario para validación.
        *   Una vez confirmado por el usuario, escribe el archivo JSON de esquema
            localmente y súbelo a una ubicación de staging en GCS; luego suministra
            esa ruta al parámetro.
    -   **Precrear el sink de destino (buena práctica)**: para garantizar
        estabilidad y evitar desajustes de esquema en la creación en runtime:
        *   **Acción**: aclara con el usuario si el sink de destino (p. ej., tabla
            de BigQuery, base de datos/tabla de Spanner) ya existe. Si el usuario
            confirma que existe, continúa con los pasos restantes tal cual.
        *   Si no existe, pide permiso para crearlo. Si te lo permiten, créalo
            tú mismo. De lo contrario, entrega al usuario los comandos exactos de
            creación.
        *   Si el sink es BigQuery, consulta los
            [Prerrequisitos específicos del destino][dest-prereqs] para la
            configuración crucial de la tabla y la tabla de errores.
    -   Incluye parámetros adicionales como service accounts, detalles de red y
        otras opciones del pipeline.
        *   **Especificación de opciones**: para Flex Templates proporcionados por
            Google, consulta la guía
            [Specifying options for Flex Templates][flex-template-options] para saber
            cómo pasar parámetros y experiments adicionales.

### Prerrequisitos específicos del destino

Distintos templates pueden requerir que se preparen recursos específicos en el
sink de destino antes de la ejecución. Sigue las instrucciones para tu sink de
destino a continuación.

#### BigQuery

Al ejecutar templates que escriben en BigQuery, DEBES asegurarte de que se
preparen los recursos siguientes para evitar fallos del job:

-   **Precrear la tabla de destino**: crea la tabla de BigQuery de destino (p. ej.,
    usando `bq mk`) antes de lanzar el job. Asegúrate de que el esquema coincida con
    las expectativas del template.
-   **Precrear la tabla de errores/registros fallidos**: muchos templates que
    escriben en BigQuery tienen un parámetro para redirigir los registros fallidos
    (p. ej., `badRecordsOutputTable` u `outputDeadletterTable`). Algunos templates
    intentan autocreación de esta tabla. Sin embargo, precrearla es una buena
    práctica. Esto garantiza el esquema y los permisos correctos.
    *   **Cómo determinar el esquema de errores**: sigue la definición del esquema
        en el [repositorio público de DataflowTemplates en GitHub][df-templates-repo]:
        1.  Localiza el código fuente o el README del template que estás usando
            (p. ej., en los directorios `v1/` o `v2/`).
        2.  Identifica el nombre del parámetro usado para la tabla de errores
            (p. ej., `badRecordsOutputTable` u `outputDeadletterTable`).
        3.  Busca en el código fuente cómo se define o carga el esquema para ese
            parámetro:
            *   **Ejemplo (referencia de código)**: en `PubSubToBigQuery.java`, el
                esquema se establece con
                `ResourceUtils.getDeadletterTableSchemaJson()`. Siguiendo
                `ResourceUtils.java` se ve que carga el esquema desde
                [streaming_source_deadletter_table_schema.json en GitHub][deadletter-schema].
            *   **Ejemplo (referencia de documentación)**: para templates más
                sencillos, el esquema puede aparecer en la documentación oficial,
                como el esquema `RawContent`/`ErrorMsg` mostrado en la
                [documentación de CSV to BigQuery en DevSite][csv-bq-doc].

## Configurar pipelines personalizados (Dataflow Runner)

Usa esta sección al prepararte para ejecutar un pipeline personalizado de Apache
Beam en Dataflow.

-   Al lanzar pipelines de Python sin un Flex Template con `DataflowRunner`, DEBES
    escanear el directorio del proyecto del pipeline buscando los archivos
    siguientes:
    -   **`requirements.txt`**:
        -   Si se encuentra, DEBES incluir la opción de pipeline `--requirements_file`.
    -   **`setup.py`**:
        -   Si se encuentra, DEBES incluir la opción de pipeline `--setup_file`. Esto
            es crítico si el pipeline usa módulos o paquetes locales.

-   Al lanzar pipelines de Python con un Flex Template, si la imagen del Flex
    Template es también la imagen del SDK Container (Configuración de Imagen Docker
    Única), entonces DEBES suministrar la imagen en el parámetro
    `sdk_container_image`.

### Consultar los recursos del entorno en lugar de usar valores de placeholder

-   Evita usar placeholders genéricos (p. ej., `your-gcp-project-id`) para recursos
    de GCP al redactar scripts de ejecución o configs. **Acción**: si los valores son
    desconocidos, ejecuta proactivamente comandos como `gcloud config get-value
    project` para encontrar los recursos activos y prellenar los scripts para el
    usuario. Confirma los valores con el usuario antes de continuar.

## Ejecución del job

Usa esta sección cuando la configuración esté completa y estés listo para lanzar
cualquier job de Dataflow (template proporcionado por Google, Flex template
personalizado o pipeline independiente).

### Universal Execution Workflow

1.  **Construir el comando de lanzamiento**: redacta el comando de lanzamiento
    completo según el tipo de pipeline (p. ej., `gcloud dataflow flex-template run`
    o `python main.py --runner=DataflowRunner`). Asegúrate de que los workers usen
    por defecto la configuración de IP privada, salvo que se especifique otra cosa,
    y verifica los permisos del proyecto de destino.
2.  **Confirmación obligatoria antes del lanzamiento**: presenta al usuario el
    comando redactado *completo* de una sola vez. Explica el propósito de todos los
    parámetros (incluidos los flags experimentales) y permite al usuario revisar y
    corregir el comando en bloque, en lugar de confirmar por partes. **NO
    continúes** con la ejecución hasta que se apruebe explícitamente.
3.  **Lanzar el job**: una vez aprobado, ejecuta el comando y anota el Job ID
    resultante (mostrándolo al usuario).
4.  **Mostrar la URL de la consola**: construye y presenta la URL directa de
    monitorización de Cloud Console:
    https://console.cloud.google.com/dataflow/jobs/<region>/<job_id>?project=<project_id>

## Monitorización del job

Usa esta sección para monitorizar el progreso de un job de Dataflow en ejecución.

-   Comprueba el estado del job de Dataflow lanzado usando el job ID.
-   Ejecuta la comprobación cada 30 segundos durante los primeros 2 minutos y luego
    cada 3 minutos, salvo que el usuario especifique otra cosa.
-   **Nota**: NO ejecutes queries de verificación de datos sobre el sink hasta que
    el job haya alcanzado un estado estable `RUNNING` o `DONE`.

## Diagnósticos y resolución de problemas

> [!IMPORTANT] DEBES usar esta sección cuando el usuario pregunte por el
> rendimiento de sus pipelines de Dataflow. Puede usarse para depurar problemas
> como lentitud del pipeline, fallos del pipeline, etc.

### Task Execution Workflow

1.  **Entender la petición del usuario**: extrae el Job ID, el Project ID, el
    Transform Name (opcional) y la ventana de tiempo.
2.  **Mapeo del Transform Name**: si el usuario requiere depuración basada en
    transforms, mapea los Transform Names proporcionados por el usuario al `stage`
    o `ptransform` real de Dataflow y aplícalos a los filtros al consultar:

    Este mapeo puede extraerse de `gcloud dataflow jobs describe JOB_ID
    --full --format="json(pipelineDescription.executionPipelineStage)"`.

    1.  **Extraer los objetivos**:
        *   Obtén el stage_id: propiedad **`name`** a nivel del stage padre. Esto
            coincide con `"F[digit]"` (p. ej. `"F6"`).
        *   Obtén el ptransform: dentro del array `componentTransform`, lee
            precisamente de **`userName`** o **`originalTransform`** (p. ej.
            `"RateLimitAndLog/ParMultiDo(RateLimitAndLog)"`) y úsalo como
            **`ptransform`**.
    2.  **Aplicar los filtros siguiendo estrictamente la mecánica del mapeo**:
        *   **Para queries de Cloud Logging**: aplica el nombre del ptransform
            extraído al filtro `resource.labels.step_id="[Extracted ptransform name]"`.
        *   **Para queries de Monitoring**: usa los filtros stage_id/ptransform según
            los filtros soportados por la métrica:
            `metric.labels.ptransform="[Extracted ptransform name]"` o
            `metric.labels.stage="[Extracted stage_id]"`.

3.  **Consultar la telemetría**:

    *   Usa la Dataflow REST API para obtener los Job Messages/Events de alto nivel
        que ocurrieron en el job.
    *   Consulta [dataflow_diagnostics_reference.md][diag-ref] para las métricas
        clave y los patrones de queries de logging según el tipo de job.
    *   Usa la Monitoring REST API para obtener métricas.
    *   Usa el comando de GCloud Logging para obtener logs.
    *   Usa la Dataflow REST API para obtener métricas de snapshot actuales cuando
        no se necesiten series temporales históricas.

4.  **Análisis**:

    *   Para jobs de streaming
        *   Salud general del job: DEBES consultar
            [streaming_job_health](references/streaming_job_health.md) para analizar
            la salud general del job de streaming.
        *   Analiza cuellos de botella y parallelism. DEBES consultar
            [bottlenecks_and_parallelism_context][bottlenecks-context] e interpretar
            las métricas de cuellos de botella y parallelism en ese contexto.
        *   Analiza el comportamiento del autoscaling. DEBES consultar
            [streaming_horizontal_autoscaling_analysis.md][autoscaling-analysis-link]
    *   Para jobs batch
        *   Correlaciona los picos/caídas de métricas con los errores de los logs.
        *   Identifica los problemas.

5.  **Salida**: entrega un diagnóstico sintetizado que contenga síntomas, causas
    raíz y enlaces al código objetivo (usando el formato `file:///...`). Sigue
    estrictamente la estructura de respuesta apropiada para el tipo de job:

    **Para jobs de streaming:**

    1.  **Estado general del job**: categorización del estado (Healthy, Mostly
        Healthy, Not Healthy) según
        [streaming_job_health](references/streaming_job_health.md).
    2.  **Eventos del job de alto nivel**: eventos notables del control plane,
        errores o fallos de stage extraídos de los job messages.
    3.  **Frescura de los datos**: retraso actual de los datos usando
        `job/data_watermark_age` / `job/per_stage_data_watermark_age` y el system
        lag.
    4.  **Throughput**: tendencias de la tasa de procesamiento usando
        `job/elements_produced_count` / `job/estimated_bytes_produced_count`.
    5.  **Backlog**: backlog de entrada (si es el stage de origen) o backlog
        entre stages usando `job/estimated_backlog_processing_time` /
        `job/backlog_bytes`.
    6.  **Cuellos de botella y parallelism**: diagnósticos de queue delay usando
        `job/is_bottleneck` (interpretando `likely_cause` / `bottleneck_kind`) y las
        métricas clave `job/backlogged_keys` /
        `job/processing_parallelism_keys` interpretadas en el contexto de
        [bottlenecks_and_parallelism_context][bottlenecks-context].
    7.  **Análisis de autoscaling**: tendencias de escalado usando
        `job/horizontal_worker_scaling` (y la etiqueta `rationale`), los límites de
        clamp (`job/max_worker_instances_limit` / `job/min_worker_instances_limit`)
        y las pistas de utilización en el contexto de
        [streaming_horizontal_autoscaling_analysis][autoscaling-analysis-link].
    8.  **Recomendaciones**: planes de remediación directos (in-flight updates,
        configuraciones del lado del cliente o correcciones de código enlazadas
        mediante URIs absolutas `file:///`).

    **Para jobs batch:**

    1.  **Eventos del job de alto nivel**: eventos notables del control plane,
        errores o fallos de stage extraídos de los job messages.
    2.  **Throughput**: tendencias de la tasa de procesamiento usando
        `job/elements_produced_count` (indicador principal de rendimiento).
    3.  **Recomendaciones**: planes de remediación directos para ejecuciones
        futuras (configuraciones del lado del cliente o correcciones de código
        enlazadas mediante URIs absolutas `file:///`).

[py-flex-ref]: references/python_flex_template_reference.md
[udf-guide]: https://docs.cloud.google.com/dataflow/docs/guides/templates/create-template-udf
[ssl-cert-guide]: https://docs.cloud.google.com/dataflow/docs/guides/templates/ssl-certificates
[dest-prereqs]: #prerrequisitos-específicos-del-destino
[flex-template-options]: https://docs.cloud.google.com/dataflow/docs/guides/templates/run-flex-templates#specify-options
[df-templates-repo]: https://github.com/GoogleCloudPlatform/DataflowTemplates
[deadletter-schema]: https://github.com/GoogleCloudPlatform/DataflowTemplates/blob/main/v2/common/src/main/resources/schema/streaming_source_deadletter_table_schema.json
[csv-bq-doc]: https://cloud.google.com/dataflow/docs/guides/templates/provided/cloud-storage-csv-to-bigquery#GcsCSVToBigQueryBadRecordsSchema
[diag-ref]: references/dataflow_diagnostics_reference.md
[bottlenecks-context]: references/bottlenecks_and_parallelism_context.md
[autoscaling-analysis-link]: references/streaming_horizontal_autoscaling_analysis.md
