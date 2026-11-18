---
name: bigquery-data-transfer-service
description: Descubre e inspecciona configuraciones de BigQuery Data Transfer Service (DTS).
  Úsala para identificar pipelines de ingesta existentes y extraer metadatos de datasource
  o de transfer config para pipelines de datos. Úsala cuando el usuario pida escenarios
  de ingesta mientras construye o gestiona pipelines de datos, o cuando pida "ingestar"
  o "agregar" datos que ya podrían estar gestionados por una transferencia de DTS.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# BigQuery Data Transfer Service (DTS)

## Directrices obligatorias

> [!IMPORTANT]
>
> Toda configuración nueva de BigQuery Data Transfer Service (DTS) **DEBE** ser
> aprovisionada a través del framework **gcp pipeline resource provisioning**,
> que incluye generar un `deployment.yaml`.
>
> -   **NO** uses comandos CLI imperativos (p. ej., `bq mk` o `gcloud`) para
>     crear o actualizar configuraciones.
> -   Los comandos CLI se permiten **solo** para descubrimiento (listar/mostrar)
>     y para disparar ejecuciones manuales.

Esta guía permite descubrir los recursos de ingesta existentes y provee los
metadatos relacionados con la ingesta cuando se necesiten.

## Flujo de trabajo

### Paso 0: Descubrir los parámetros del entorno

Antes de generar configuraciones, descubre los valores reales del proyecto
y la región objetivo.

> [!TIP]
>
> Si `deployment.yaml` ya existe en la raíz del repositorio, prioriza
> extraer `project` y `region` de la configuración del entorno objetivo
> (p. ej., `dev`).

1.  **Proyecto**: `gcloud config get project`
2.  **Región**: `gcloud config get compute/region`

> [!TIP]
>
> Usa estos comandos para reemplazar placeholders como `<PROJECT_ID>` con valores
> reales. Una vez reemplazados, elimina siempre los comentarios asociados que
> empiezan con TODO.

### Paso 1: Verificar transferencias existentes

Antes de asumir que se necesita una transferencia nueva, verifica las existentes
en la región objetivo.

1.  **Listar transferencias**:

    ```bash
    bq ls --transfer_config \
      --transfer_location=<REGION> \
      --project_id=<PROJECT_ID>
    ```

2.  **Analizar las transferencias existentes**:

    -   **Se encontró una única transferencia**:

        -   Verifica si la transferencia tiene al menos una ejecución exitosa: `bq ls
            --transfer_run --transfer_config=<RESOURCE_NAME>`
        -   Si se encuentra: usa la transfer config existente.
        -   Si no se encuentra: confirma con el usuario si está bien disparar la
            ejecución de la transferencia.

    -   **Se encontraron múltiples transferencias**:

        -   Intenta deducir la correcta según el contexto.
        -   Pide al usuario que confirme.

    -   **Se encontraron transferencias deshabilitadas**:

        -   Pregunta al usuario si quiere habilitarla o crear una nueva.
        -   Para habilitarla: indica al usuario que actualice la configuración de la
            transferencia en su archivo `deployment.yaml` estableciendo el campo
            `disabled` en `false` para el recurso de transferencia específico.

    -   **No se encontraron transferencias**: procede a crear una nueva si es necesario.

### Paso 2: Descubrir y validar parámetros (transferencias nuevas)

Si se crea una transferencia nueva, descubre los parámetros requeridos usando la
API REST y valídalos con el usuario.

> [!TIP]
>
> Si `<DATA_SOURCE_ID>` es desconocido, ejecuta el script de descubrimiento sin
> el argumento `<DATA_SOURCE_ID>` para listar los IDs de fuente disponibles (p. ej.,
> `google_cloud_storage`). Usa el proyecto y la ubicación derivados del Paso 0.

```bash
python3 scripts/bigquery_dts.py --project_id=<PROJECT_ID>
```

1.  **Ejecutar el script de descubrimiento**: usa el script `bigquery_dts.py`
    para inspeccionar los parámetros del Data Source vía la API REST.

    ```bash
    # Passes the derived project and region to the script.
    python3 scripts/bigquery_dts.py --project_id=<PROJECT_ID> <DATA_SOURCE_ID> <REGION>
    ```

    > [!IMPORTANT]
    >
    > Ejecuta este comando cada vez que se planee una nueva transferencia.

2.  > [!CAUTION]
    >
    > **Cuestionario obligatorio para el usuario (CRÍTICO)**:

    -   **Identifica explícitamente TODOS los parámetros específicos** que devuelva
        el script de descubrimiento. **NO debes generalizarlos ni resumirlos
        vagamente.**
    -   **Autorización OAuth (fuentes de datos de Google)**: para las fuentes de
        datos del ecosistema de Google (Google Ads, Youtube, etc.), si el usuario
        no usa una service account para configurar la transfer config de DTS
        (es decir, usa End User Credentials o EUC para configurar la transfer
        config), genera un OAuth URI. Pide al usuario que visite esa URL para
        autorizar. Una vez que el usuario provea el código versionInfo, úsalo como
        `definition.versionInfo` en `deployment.yaml` y entonces puedes continuar.
    -   Si algún parámetro está relacionado con la autenticación, pide
        explícitamente al usuario que provea el Secret Manager Resource ID (p. ej.,
        projects/my-project/secrets/my-secret) para esos parámetros
    -   Presenta al usuario cada parámetro requerido ANTES de generar los archivos
        de configuración.
    -   Pide la verificación de los assets/tablas que se van a ingerir.

3.  **Esperar la respuesta del usuario**: NO debes continuar hasta que los
    parámetros estén confirmados.

### Paso 3: Extraer los datos de la transfer config

Recupera los detalles de configuración de la transferencia seleccionada.

```bash
bq show --format=prettyjson --transfer_config <RESOURCE_NAME>
```

### Paso 4: Disparar y verificar la transferencia

Después de que la transferencia se despliegue vía el framework de aprovisionamiento
de recursos, DEBES asegurarte de que haya al menos una ejecución exitosa antes de
continuar con el resto de las tareas.

1.  **Disparar una ejecución manual**: si no se encuentran ejecuciones exitosas
    o en curso, o la transferencia acaba de crearse, dispara una ejecución manual
    para la hora actual.

    ```bash
    bq mk --transfer_run \
      --transfer_config=<RESOURCE_NAME> \
      --run_time=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
    ```

2.  **Consultar hasta la finalización (regla de los 5 minutos)**: intenta verificar
    el estado de la ejecución cada 30-60 segundos durante un máximo de **5 minutos**.

    ```bash
    bq ls --format=prettyjson --transfer_run --transfer_config=<RESOURCE_NAME>
    ```

    -   **Éxito**: si la ejecución se completa con éxito, continúa con el resto del
        pipeline.
    -   **Falla**: si la ejecución falla, analiza los logs y pide ayuda al usuario.
    -   **Timeout (5 min)**: si la ejecución sigue en curso después de 5 minutos,
        **DETENTE** y pregunta al usuario: "La ingesta del Data Transfer Service
        sigue en curso. Provee 'proceed guidance' una vez que la ingesta haya
        terminado para que pueda continuar construyendo el resto del pipeline de
        datos usando el esquema y las muestras ingeridas."

3.  **Esperar la guía del usuario**: NO continúes hasta que el usuario confirme
    que la ingesta está completa o provea una guía.

4.  Una vez que el usuario confirme continuar, empieza a trabajar en el resto de
    las tareas.

## Definición de terminado

-   Una configuración de transferencia de BigQuery DTS fue descubierta o
    aprovisionada declarativamente (vía **gcp pipeline resource provisioning**
    con un `deployment.yaml` generado).
-   Los parámetros obligatorios del datasource fueron identificados y confirmados
    con el usuario.
-   Una ejecución manual de la transferencia fue disparada y monitoreada.
-   La ejecución de la transferencia se completó con éxito O el usuario proveyó
    "proceed guidance" para una transferencia de larga duración.
