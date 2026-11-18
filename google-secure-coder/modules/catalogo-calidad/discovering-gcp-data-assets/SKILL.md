---
name: discovering-gcp-data-assets
description: |
  Encuentra e inspecciona data assets dentro de Google Cloud.
  Relevante cuando se cumple alguna de las siguientes condiciones:
    1. La solicitud del usuario implica encontrar, explorar o inspeccionar data assets
       en Google Cloud, tales como:
         - datasets, tablas o vistas de BigQuery
         - catálogo o tablas de BigLake
         - instancias, bases de datos o tablas de Spanner
         - etc.
    2. Necesitas recuperar el esquema, los metadatos o las políticas de gobernanza de
       un data asset de GCP.
    3. Tienes una palabra clave o un tema (p. ej., "datos de ventas") pero careces
       del ID específico de la tabla o del recurso.
    4. Estás intentando encontrar datos con `bq ls`, ya que esta skill ofrece un
       enfoque superior.
  No usar cuando:
    - Los assets están fuera de Google Cloud
license: Apache-2.0
metadata:
  version: v4
  publisher: google
---

# Instrucciones

## Paso 1: Prioriza los assets de la conversación

Si el asset fue creado o mencionado antes en la misma conversación, continúa con
ese asset en lugar de buscar. Omite los pasos 2, 3 y 4.

## Paso 2: Gestiona los datasets públicos o pasa a la búsqueda

Dataplex Lookup Context proporciona los metadatos más completos para los data
assets. Debes priorizar su uso para todos los assets de Google Cloud, incluso si
ya conoces sus IDs.

-   **Datasets públicos (inspección directa)**: si el asset solicitado pertenece
    al proyecto `bigquery-public-data`, Dataplex Lookup Context fallará. Debes
    omitir los pasos 3 y 4 e inspeccionar la tabla directamente usando la CLI de
    `bq` o las herramientas MCP de BigQuery.
-   **Todos los demás assets (pasa al Paso 3)**: para todos los demás data
    assets de BigQuery, Cloud Storage, Spanner, BigLake Iceberg o de GCP en
    general (tanto si sus IDs se conocen como si faltan), debes pasar al
    **Paso 3** para buscar en el catálogo de Dataplex y obtener su nombre de
    Entry completo.

## Paso 3: Ejecuta la búsqueda de descubrimiento

Debes usar el comando de búsqueda de Dataplex para descubrir assets y recuperar
sus nombres de Entry completos `projects/...`. Este paso es obligatorio incluso
si ya conoces el ID corto del asset (p. ej., `my_dataset.my_table`), porque el
Paso 4 exige estrictamente el nombre de Entry completo.

> [!IMPORTANT]
>
> El parámetro `--project` DEBE proporcionarse SIEMPRE. Este project_id se usa
> solo para atribuir la búsqueda y NO restringe el alcance de la búsqueda. El
> proyecto debe tener la API de dataplex habilitada y el usuario debe tener los
> permisos `dataplex.entries.get`.

### A. Búsqueda semántica (intención en lenguaje natural)

Úsala cuando el usuario describe el **significado** o la **intención** de los
datos (p. ej., "encuentra los datos de ventas de productos del Q4").

Usa la herramienta MCP `search_entries`

O

```bash
gcloud dataplex entries search "<NATURAL_LANGUAGE_QUERY>" \
  --project="<PROJECT_ID>" \
  --semantic-search \
  --limit=50
```

### B. Búsqueda por palabras clave (cadenas técnicas)

Úsala para coincidencias exactas de palabras clave o cadenas técnicas (p. ej.,
`name:order_v2`).

#### Reglas de la consulta de búsqueda (OBLIGATORIAS)

-   **Sintaxis específica por modo**:
    -   **Búsqueda semántica**: los operadores lógicos (`AND`, `OR`) DEBEN estar
        en **MAYÚSCULAS**. Usa `labels.` en plural para los filtros de labels
        (p. ej., `labels.env=prod`).
    -   **Búsqueda por palabras clave**: los operadores no distinguen mayúsculas
        ni minúsculas. Usa `label.` en singular para los filtros de labels
        (p. ej., `label.env=prod`).
-   **Lógica abreviada**: usa `|` para OR y `,` para AND dentro de paréntesis
    para acortar las consultas (p. ej., `projectid:(prod|staging)` o
    `column:(id,name)`).
-   **Coincidencia exacta vs. por token**:
    -   Usa `:` para coincidencias por token o subcadena (p. ej., `name:sales`).
    -   Usa `=` para coincidencias exactas. OBLIGATORIO para `system`, `type` y
        `location`.
-   **Palabras clave en singular**: al realizar búsquedas por palabras clave,
    convierte SIEMPRE los plurales a singular (p. ej., "producto", NO
    "productos"). La búsqueda semántica maneja automáticamente las variaciones
    de singular/plural y los sinónimos.
-   **Restricción del alcance**: DEBERÍAS restringir el alcance de la búsqueda
    con un filtro `parent` si conoces el proyecto o el dataset (p. ej.,
    `parent:projects/<PROJECT_ID>`).

#### Referencia de sintaxis de búsqueda de Dataplex

-   **`name:x`**: coincidencia por subcadena/token sobre el ID del recurso.
-   **`displayname:x`**: coincidencia por subcadena/token sobre el display name.
-   **`projectid:x`**: coincidencia por subcadena/token sobre el ID del proyecto
    de GCP.
-   **`parent:x`**: coincidencia por subcadena sobre la ruta jerárquica (p. ej.,
    `projects/my-proj`).
-   **`location=x`**: coincidencia exacta sobre la ubicación (p. ej.,
    `us-central1`, `us`).
-   **`column:x`**: coincidencia por subcadena/token sobre los nombres de
    columna del esquema.
-   **`system=x`**: coincidencia exacta sobre el sistema de origen. Valores
    comunes: `bigquery`, `storage`, `biglake`, `cloud_sql`, `cloud_spanner`,
    `cloud_bigtable`, `pubsub`.
-   **`type=x`**: coincidencia exacta sobre el tipo de Entry (p. ej.,
    `bigquery-table`, `storage-bucket`, `storage-folder`).
-   **`labels.key=value`**: (SOLO en modo semántico) coincidencia exacta sobre
    un label.
-   **`label.key=value`**: (SOLO en modo de palabras clave) coincidencia exacta
    sobre un label.
-   **`createtime[>|<|=]x`**: busca assets creados después/antes de la fecha
    `YYYY-MM-DD`.
-   **`fully_qualified_name=x`**: coincidencia exacta sobre el FQN (p. ej.,
    `bigquery:project.dataset.table`).

> [!TIP]
>
> Los resultados de búsqueda de Dataplex dependen de que los metadatos se hayan
> ingresado en el Universal Catalog (a menudo mediante **Discovery Scans**). Si
> un asset no aparece en la búsqueda, puede que no esté indexado. - **Fallback
> 1**: prueba a buscar con el calificador `fully_qualified_name`. - **Fallback
> 2**: usa herramientas nativas (p. ej., `bq show`, `gcloud storage`) o skills
> específicas para ese tipo de asset si ya conoces el ID.

```bash
gcloud dataplex entries search "<KEYWORD_SEARCH_QUERY>" \
  --project="<PROJECT_ID>" \
  --limit=50
```

> [!IMPORTANT]
>
> Manejo de los resultados de búsqueda y evitación de bucles:
>
> 1.  **Sin resultados:** si la búsqueda no devuelve ninguna entrada:
>     *   **Regla de variación:** puedes probar COMO MÁXIMO 3 variaciones de la
>         consulta de búsqueda (p. ej., cambiar las cláusulas AND/OR,
>         añadir/quitar `parent:`, quitar `projectid:` o `location:`, probar
>         `fully_qualified_name=`).
>     *   **Regla de parada:** si tras 3 intentos no se encuentran resultados,
>         DETÉNTE e informa al usuario. Pide una aclaración, concretamente el
>         **nombre de Entry completo** de Dataplex si se conoce, o
>         identificadores como el **ID del proyecto**, el **ID del dataset** o
>         el **ID de la instancia** para ayudar a acotar la búsqueda. Ejemplo:
>         "No pude encontrar ninguna tabla que coincida con esa descripción
>         tras varios intentos. Si conoces el nombre de Entry completo de
>         Dataplex (`projects/...`), proporciónamelo. De lo contrario,
>         indícame cualquier identificador que conozcas, como el nombre del
>         proyecto, del dataset o de la instancia, para ayudar a localizar el
>         asset."
> 2.  **Múltiples resultados:**
>     *   Si se devuelven más de 10 resultados, indica que se encontraron tantas
>         coincidencias. Muestra los nombres de las primeras 5 entradas y pide
>         una aclaración.
>     *   Si se devuelven entre 2 y 10 resultados y no puedes elegir con
>         certeza, enuméralos y pregúntale al usuario.
> 3.  **Resultado único:** pasa al Paso 3 con el nombre de Entry completo.
> 4.  **Evita los bucles infinitos:** NO debes volver a ejecutar consultas
>     idénticas o casi idénticas. Si Dataplex no devuelve el asset esperado,
>     prioriza pedirle al usuario el ID exacto del recurso o usar el Fallback 2
>     (herramientas nativas).

*Criterio*: una vez que se devuelvan los assets candidatos, pasa al Paso 4
usando los **nombres de Entry completos** de los resultados de búsqueda.

## Paso 4: Lookup Context

Debes usar el comando **Lookup Context** para obtener el esquema y los
metadatos profundos de los resultados relevantes obtenidos en el Paso 3.

> [!IMPORTANT]
>
> El parámetro `--resources` DEBE ser el **nombre completo** (que empieza con
> `projects/`) devuelto por el resultado de la búsqueda. Pasar IDs cortos de
> tabla, URIs de GCS o prefijos `bigquery:` totalmente calificados está
> PROHIBIDO y fallará.

### Ejecución del comando

Usa la herramienta MCP `lookup_context`

O

```bash
gcloud dataplex context lookup --resources="<FULL_ENTRY_NAME>"
```

*Criterio de finalización*: el comando devuelve el esquema detallado y el
contexto de negocio.

--------------------------------------------------------------------------------

## Solución de problemas

### Context Lookup falla o "Resource not found"

-   **Causa**: se usaron nombres cortos de tabla de manera incorrecta.
-   **Solución**: asegúrate de usar el formato correcto del nombre de Entry de
    los resultados de búsqueda (que empieza con `projects/`).

### La búsqueda no devuelve resultados

-   **Causa**: términos en plural en la búsqueda por palabras clave o falta de
    acotamiento.
-   **Solución**: cambia a palabras clave en singular. Para la búsqueda
    semántica, prueba con lenguaje natural más descriptivo.

### Context Lookup falla con "NOT_FOUND" (a pesar del formato correcto)

-   **Causa**: la tabla pertenece a un proyecto (p. ej., `bigquery-public-data`)
    que no ha sincronizado completamente sus metadatos con el Dataplex Universal
    Catalog. Aunque la entrada aparece en la búsqueda, `context lookup` no está
    disponible.
-   **Solución**: recurre a la inspección directa usando herramientas nativas
    (p. ej., la CLI de `bq`).
-   **Regla de parada:** si la herramienta nativa (p. ej., `bq show`) también
    devuelve "Not Found", DETÉNTE. No reinicies el bucle de descubrimiento de
    Dataplex. Pide específicamente al usuario que verifique el **ID del
    proyecto** y el **ID de la tabla**.

### Rompe el bucle de investigación

Si te encuentras buscando repetidamente el mismo asset:

1.  **DETÉNTE.**
2.  Explica lo que has intentado (p. ej., "busqué en Dataplex con X y Y, y
    verifiqué con `bq show`").
3.  Pide al usuario el ID exacto del proyecto, del dataset y de la tabla.

### La búsqueda falla con "--project: Must be specified."

-   **Causa**: no se proporcionaron los argumentos `--project <PROJECT_ID>`
-   **Solución**: proporciona un proyecto, que se usará para autorizar y
    atribuir la solicitud de búsqueda.

### La búsqueda falla con "PERMISSION_DENIED"

-   **Causa**: el project_id proporcionado en los argumentos
    `--project <PROJECT_ID>` no tiene la API de Dataplex habilitada o al usuario
    le faltan los permisos IAM necesarios.
-   **Solución**: pregúntale al usuario si tiene un proyecto con la API de
    Dataplex habilitada y con el permiso dataplex.entries.get
