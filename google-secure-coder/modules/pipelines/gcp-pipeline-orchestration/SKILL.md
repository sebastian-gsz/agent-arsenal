---
name: gcp-pipeline-orchestration
description: Esta skill ayuda al agente a generar o actualizar definiciones de pipelines
  de orquestación para Google Cloud Composer, para inicializar un pipeline de orquestación
  o actualizar la definición de orquestación de diversos pipelines de datos, como pipelines
  dbt, notebooks, jobs de Spark, Dataform, scripts de Python o queries SQL inline de
  BigQuery. Esta skill también ayuda a desplegar y disparar pipelines de orquestación.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

## Enrutado obligatorio a referencias

Si aplica, consulta los archivos de referencia asociados antes de actuar.
Usa la tabla de abajo para determinar qué archivo de referencia recuperar en
los distintos escenarios que involucren funciones concretas. [!IMPORTANT]: NO
ADIVINES nombres de archivo. DEBES usar únicamente las rutas exactas de abajo.

Función/Caso de uso                 | Archivo de referencia requerido                | Capacidades y palabras clave
---------------------------------- | ---------------------------------------------- | ------------------------------
**esquema orchestration-pipelines** | `references/orchestration-pipelines-schema.md` | orquestar, generar, crear, actualizar

## Cómo usar esta skill

Los pipelines de orquestación requieren crear dos archivos para asegurar un
pipeline completo y desplegable:

1.  `Archivo de orquestación` (p. ej., `orchestration-pipeline.yaml`,
    `test-pipeline.yaml`): define la lógica, las tareas y la programación del
    pipeline. **IMPORTANTE:** Comprueba si existe un archivo `deployment.yaml`
    que referencie un archivo de orquestación existente. Si es así, **debes
    actualizar el archivo de orquestación existente** (p. ej.,
    `test_pipeline.yaml`) en vez de crear uno nuevo. El nombre del archivo puede
    personalizarse pero debe estar referenciado en `deployment.yaml`.
2.  `deployment.yaml`: define las configuraciones por entorno (p. ej.,
    `dev`, `prod`). `deployment.yaml` solo debe existir en la raíz del
    repositorio y debe llamarse `deployment.yaml`.

-   Todos los archivos deben mantenerse siempre juntos. Y todos los archivos
    deben colocarse en la raíz de la carpeta del workspace.

-   Esta skill es útil para crear o actualizar archivos de configuración para
    orquestar pipelines de datos.

## Cómo usar esta skill

### Paso 1: Evaluar el estado del pipeline de orquestación e inicializar si es necesario

Examina el directorio raíz del repositorio en busca de un archivo `deployment.yaml`.

1.  **Comprobar configuración existente**: la ausencia de `deployment.yaml`
    indica que la orquestación no está configurada.
2.  **Determinar si se requiere inicialización**: la inicialización se requiere
    si falta `deployment.yaml`. DEBES ejecutar el comando `init` del Paso 3 para
    hacer el scaffolding del proyecto si falta `deployment.yaml`. NO crees los
    archivos manualmente.
3.  **Nombre del pipeline**: si se necesita inicializar, pregunta al usuario el
    nombre del pipeline. Si el usuario no ha dado el nombre del pipeline de
    orquestación, el nombre debe ser "orchestration_pipeline".
4.  **Nombre del entorno**: si se necesita inicializar, DEBES preguntar al
    usuario el nombre del entorno. Si no lo proporciona, usa **dev** como
    valor por defecto.

5.  **Ejecutar la inicialización**: cuando tengas el nombre del pipeline,
    ejecuta el siguiente comando:

```
# Replace <ORCHESTRATION_PIPELINE_NAME> with the actual name
# Replace <ENV_NAME> with the actual environment name
gcloud beta orchestration-pipelines init <ORCHESTRATION_PIPELINE_NAME> --environment=<ENV_NAME>
```

### Paso 2: Revisar la estructura del código del pipeline de orquestación y su sintaxis

*** Modelos de pipeline (mapeo a YAML)

> [!IMPORTANT]
>
> Aunque los modelos internos del pipeline se definen con protobuf (que
> normalmente usa `snake_case`), **la configuración YAML espera `camelCase`**
> para casi todos los nombres de campo.
>
> **Regla de mapeo:** convierte siempre los campos proto en `snake_case`
> (p. ej., `pipeline_id`) a `camelCase` en el YAML (p. ej., `pipelineId`).

#### Estructura y sintaxis YAML de orchestration-pipelines

Consulta el archivo `references/orchestration-pipelines-schema.md`.

**Tags requeridos (top-level)**: DEBES añadir un campo `tags` en el nivel
superior de la definición YAML del pipeline de orquestación. El valor de este
campo depende del entorno de IDE:

-   En Antigravity, usa `["job:datacloud:antigravity"]`.
-   En VS Code, usa `["job:datacloud:vscode"]`.
-   En cualquier otro entorno, usa `["job:datacloud:other"]`.

#### Estructura y sintaxis del deployment.yaml

**Estructura top-level:** la raíz del YAML debe ser un objeto con los
siguientes campos:

-   `environments` (diccionario): un mapa cuyas claves son nombres de entorno
    (p. ej., 'dev', 'prod', etc.) y cuyos valores son objetos Environment.

**Environment:** cada objeto de entorno contiene los siguientes campos:

-   `project` (string): el ID del proyecto de Google Cloud.
-   `region` (string): la región de Google Cloud (p. ej., 'us-central1').
-   `composer_environment` (string): el nombre del entorno de Cloud Composer.
-   `artifact_storage`
    -   `bucket` (string): bucket de GCS
    -   `path_prefix` (string): prefijo de la ruta que queremos poner en el
        bucket
-   `pipelines`
    -   `- source` (string): nombres de archivos YAML de pipelines de
        orquestación. Puede haber varios.
-   `variables` (diccionario, opcional): pares clave-valor que representan
    variables de entorno. Los valores pueden ser strings, números o booleanos.

> [!TIP]
>
> Si el usuario no proporciona rutas concretas para scripts, proyectos dbt o
> datos de GCP (ID de proyecto, región), usa herramientas como `find_by_name`
> para buscar en el repositorio y comandos `gcloud` (p. ej.,
> `gcloud config get-value project`) para recuperar la información necesaria.

### Paso 3: Generar los archivos del pipeline

-   Antes de generar, comprueba si ya existen un archivo de definición de
    pipeline de orquestación y un `deployment.yaml` en el directorio actual. Si
    existen, informa al usuario y pregúntale si quiere actualizar los archivos
    existentes o crear unos nuevos con otros nombres. No sobrescribas sin
    confirmación.

-   Primero, antes de crear el archivo de definición del pipeline de
    orquestación, **debes** ejecutar el siguiente comando para obtener la lista
    de entornos Dataproc disponibles para el proyecto del usuario. Esto evita
    usar valores placeholder para lanzar los jobs.

    ```
    # Replace <PROJECT_ID> with the actual project_id
    # Replace <REGION> with the actual region
    gcloud dataproc clusters list \
    --project <PROJECT_ID> \
    --region <REGION> \
    ```

    > [!TIP]
    >
    > Ejecutar el comando sin `--format=yaml` da una salida tabular clara y
    > más fácil de leer.

-   Después usa la lista Dataproc devuelta con sus detalles para crear el
    archivo de definición del pipeline de orquestación según los requisitos del
    usuario para la lógica y la programación del pipeline. **IMPORTANTE:** toda
    programación **debe** incluir un `endTime`. Toda programación **debe** usar
    la fecha actual como `startTime` si el usuario no ha especificado otra.

    > [!IMPORTANT]
    >
    > Un entorno de Composer no es un cluster de Dataproc. Si no hay clusters de
    > Dataproc disponibles, no uses un entorno de Composer para el
    > `sparkHistoryServerConfig`. Es mejor omitir esa configuración si no hay
    > un Spark History Server dedicado disponible.

-   Si quieres programar el job de Python, revisa el contenido del código Python
    para determinar si es un job de Spark. Si lo es, usa `pyspark` como type en
    lugar de script como type.

-   Antes de crear o actualizar el archivo `deployment.yaml`, **debes** ejecutar
    primero el siguiente comando para obtener la lista de entornos Composer
    disponibles para el proyecto del usuario.

    ```
    # Replace <PROJECT_ID> with the actual project_id
    # Replace <REGION> with the actual region
    gcloud composer environments list \
    --project <PROJECT_ID> \
    --locations <REGION> \
    ```

    Tras listar los entornos Composer disponibles, **debes** comprobar cada
    entorno para asegurar que Composer usa la versión de imagen correcta o tiene
    instalados los paquetes PyPI adecuados. Ejecuta el siguiente comando para
    cada entorno:

    ```
    # Replace <ENVIRONMENT_NAME> with the Composer environment name
    # Replace <REGION> with the region
    gcloud composer environments describe <ENVIRONMENT_NAME> \
    --location <REGION> \
    --format="json(config.softwareConfig.imageVersion, config.softwareConfig.pypiPackages)"
    ```

    De la salida, selecciona un entorno cuyo valor de imageVersion sea uno de
    "composer-3-airflow-3.1.7-build.x, composer-3-airflow-2.11.1-build.x,
    composer-3-airflow-2.10.5-build.x, composer-3-airflow-2.9.3-build.x,
    composer-2.16.11-airflow-2.11.1, composer-2.16.11-airflow-2.10.5,
    composer-2.16.11-airflow-2.9.3" o selecciona un entorno donde aparezca el
    campo `orchestration-pipelines` en los paquetes PyPI. Esto garantiza que el
    entorno seleccionado es compatible con los pipelines de orquestación.

-   Tercero, antes de generar el archivo `deployment.yaml`, **debes pedir al
    usuario** el nombre del bucket de `artifact_storage`. Ten en cuenta que el
    bucket de `artifact_storage` suele inicializarse como placeholder (p. ej.,
    `YOUR_BUCKET`) por el comando `init` del Paso 1. Debes identificar esos
    placeholders, pedir al usuario el nombre real del bucket y actualizar el
    `deployment.yaml` con el valor proporcionado.

    Usa la lista de Composer devuelta con sus detalles, junto con el ID de
    proyecto, la región y el nombre de bucket aportado por el usuario, para
    generar o actualizar el archivo `deployment.yaml`. Al generar o actualizar
    `deployment.yaml`, **debes** sustituir los placeholders (p. ej.,
    "<YOUR_PROJECT_ID>", "<YOUR_REGION>", "<YOUR_COMPOSER>", "<YOUR_BUCKET>")
    por los valores reales obtenidos y proporcionados. Además, **debes**
    eliminar los comentarios `# TODO:` asociados una vez sustituidos los
    placeholders.

-   Asegúrate de que ambos archivos respetan las estructuras de código y la
    sintaxis especificadas en este documento.

-   **Renombrar pipelines**: si te piden cambiar el nombre del pipeline de
    orquestación, renombra en consecuencia el archivo YAML de orquestación
    (p. ej., de `dbt_clean_pipeline.yaml` a `new_name.yaml`) y actualiza el
    campo `source` de la lista `pipelines` de `deployment.yaml` para que
    coincida con el nuevo nombre.

> [!IMPORTANT]
>
> **Formato de tiempo**: NO incluyas el sufijo `Z` en `startTime` ni `endTime`.
> Usa el formato `"YYYY-MM-DDTHH:MM:SS"` (p. ej., `"2025-10-01T00:00:00"`).

### Paso 4: Validar el contenido (OBLIGATORIO)

Tras crear o editar los archivos del pipeline, **DEBES** validarlos con el
comando `gcloud beta orchestration-pipelines validate`. Debes: a. Leer el
archivo `deployment.yaml` para identificar todos los entornos definidos.
b. Ejecutar el comando `validate` de abajo para **cada** entorno encontrado en
`deployment.yaml`.

```
# Replace <ENV_NAME> with the identified environment name
gcloud beta orchestration-pipelines validate --environment=<ENV_NAME>
```

### Paso 5: Gestionar errores de validación

1.  Revisa la salida del comando de validación.

2.  Si el comando devuelve un mensaje de error o fallo:

    -   Lee el mensaje de error con atención.
    -   Edita los archivos de orquestación y despliegue para corregir el
        problema concreto mencionado.

3.  Vuelve a ejecutar el comando de validación para confirmar la corrección. No
    marques la tarea como completada hasta que la validación pase (código de
    salida 0), y no recurras a crear un DAG de Airflow en Python si la
    validación falla.

## Plantillas de pipelines declarativos

Cuando te pidan generar o verificar archivos de pipelines declarativos,
asegúrate de que siguen estas estructuras conformes. **No uses los valores
exactos de abajo; adáptalos a los detalles de proyecto, región y entorno del
usuario.**

### Plantilla de `deployment.yaml` - IMPORTANTE: EL FORMATO DEBE COINCIDIR-

```yaml
environments:
  <environment_name>: # e.g., dev, prod
    project: <PROJECT_ID>
    region: <REGION>
    composer_environment: <COMPOSER_ENVIRONMENT_NAME>
    gcs_bucket: "" # Optional
    artifact_storage:
      bucket: <ARTIFACT_BUCKET_NAME>
      path_prefix: "<prefix>-" # e.g., namespace or username prefix
    pipelines:
      - source: '<orchestration-pipeline.yaml>' # e.g., list of pipeline yaml names
```

### Paso 6: Desplegar el pipeline de orquestación (opcional)

Si te piden **desplegar** el pipeline de orquestación:

1.  DEBES preguntar al usuario a qué entorno desplegar. Si no se proporciona
    nombre de entorno, lista los entornos disponibles de `deployment.yaml` y
    pide al usuario elegir uno, con `dev` por defecto si existe.

2.  Lee el YAML de orquestación para extraer el `pipelineId`.

3.  Despliega con `--local`. Esto sube el DAG sin ejecutarlo:

    ```
    # Replace <ENV_NAME> with the target environment
    # Replace <PIPELINE_SOURCE> with the orchestration YAML filename
    gcloud beta orchestration-pipelines deploy \
      --environment=<ENV_NAME> --local
    ```

4.  Analiza la salida del deploy para extraer el **bundle ID** (versión). La
    salida incluye una línea como: `Pipeline deployment successful for version
    local-b32d15e307b5`. El string de versión (p. ej., `local-b32d15e307b5`) es
    el bundle ID.

> [!IMPORTANT]
>
> Los despliegues con `--local` ahora usan por defecto `--paused=true`. El DAG
> desplegado será visible en Airflow como un DAG pausado sin programación. **No**
> se ejecutará solo. Usa el Paso 7 para dispararlo.

### Paso 7: Disparar la ejecución del pipeline de orquestación (opcional)

Si te piden **disparar/ejecutar** el pipeline de orquestación, DEBES seguir el
flujo Desplegar → Sondear → Disparar.

1.  **Preguntar el entorno**: DEBES preguntar al usuario qué entorno usar. Por
    defecto `dev` si existe en `deployment.yaml`.

2.  **Desplegar primero** (Paso 6): despliega siempre antes de disparar para
    asegurar que la ejecución usa el código más reciente. Extrae el
    `bundle ID` de la salida del deploy y el `pipelineId` del YAML de
    orquestación.

3.  **Sondear la disponibilidad del DAG**: espera a que el DAG quede registrado
    en Composer.

    ```bash
    # Initial delay: wait 30 seconds after deploy
    sleep 30

    # Poll every 15 seconds, up to 2 minutes total
    # Replace <ENV_NAME>, <BUNDLE_ID> with actual values

    gcloud beta orchestration-pipelines list \
    --environment=<ENV_NAME> \
    --bundle=<BUNDLE_ID>
    ```

    El pipeline está listo cuando aparece en la salida del listado. Si no
    aparece tras 2 minutos, informa del fallo y recomienda al usuario revisar la
    validez del YAML.

4.  **Disparar el pipeline**:

    ```
    # Replace <ENV_NAME>, <BUNDLE_ID>, <PIPELINE_ID> with actual values
    gcloud beta orchestration-pipelines trigger \
    --environment=<ENV_NAME> \
    --bundle=<BUNDLE_ID> \
    --pipeline=<PIPELINE_ID>
    ```

5.  **Verificar que la ejecución arrancó**:

    ```
    gcloud beta orchestration-pipelines runs list \
    --environment=<ENV_NAME> \
    --bundle=<BUNDLE_ID> \
    --pipeline=<PIPELINE_ID>
    ```

> [!TIP]
>
> **Solo disparar (sin deploy):** si el usuario quiere disparar un pipeline ya
> desplegado, salta el Paso 6. Usa `gcloud beta orchestration-pipelines list
> --environment=<ENV_NAME>` para encontrar el bundle ID y dispara directamente
> con el Paso 7.4.

> [!IMPORTANT]
>
> **Fallback:** si `gcloud trigger` falla, usa el script incluido: ejecuta el
> script con --help para descubrir y aprender su interfaz.
>
> ```
> python scripts/trigger/airflow_trigger.py \ --project <PROJECT_ID>
> --location <REGION> \ --environment <COMPOSER_ENV> --dag_id <PIPELINE_ID>
> ```
>
> Obtén `project`, `region` y `composer_environment` de `deployment.yaml`.

## Definición de terminado

-   El archivo `deployment.yaml` se creó correctamente.
-   El archivo del pipeline de orquestación (p. ej., `orchestration_pipeline.yaml`)
    se creó correctamente, incluye un `endTime` obligatorio en toda programación
    y pasa el comando de validación:
    `gcloud beta orchestration-pipelines validate --environment=<ENV_NAME>`
-   Si el usuario pidió **desplegar** el pipeline de orquestación, el comando
    `gcloud beta orchestration-pipelines deploy --environment=<ENV_NAME> --local`
    debe devolver un mensaje de éxito con un ID de versión/bundle.
-   Si el usuario pidió **disparar/ejecutar** el pipeline de orquestación:
    1.  El deploy tuvo éxito (bundle ID extraído de la salida)
    2.  El DAG apareció en `gcloud beta orchestration-pipelines list` en menos
        de 2 min
    3.  `gcloud beta orchestration-pipelines trigger` devolvió éxito
    4.  La ejecución es visible en `gcloud beta orchestration-pipelines runs list`

## Otras acciones

Si te piden pausar/detener el pipeline de orquestación, usa

```bash
    # Replace <ENV_NAME>, <BUNDLE_ID>, <PIPELINE_ID> with actual values
    gcloud beta orchestration-pipelines pause \
    --environment=<ENV_NAME> \
    --bundle=<BUNDLE_ID> \
    --pipeline=<PIPELINE_ID>
```

Si te piden reanudar el pipeline de orquestación, usa

```bash
    # Replace <ENV_NAME>, <BUNDLE_ID>, <PIPELINE_ID> with actual values
    gcloud beta orchestration-pipelines unpause \
    --environment=<ENV_NAME> \
    --bundle=<BUNDLE_ID> \
    --pipeline=<PIPELINE_ID>
```
