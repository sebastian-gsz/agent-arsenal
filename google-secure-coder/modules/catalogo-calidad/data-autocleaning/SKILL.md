---
name: data-autocleaning
description: Capacidades automatizadas de calidad y transformación de datos para
  pipelines de Dataform/dbt/BigQuery. Procesa datos provenientes de BigQuery o Cloud
  Storage (GCS), aplicando buenas prácticas de ingesta, movimiento, mapeo de esquemas
  y limpieza integral de datos.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Skill de Data Autocleaning

Perfilado automatizado de datos, evaluación de calidad y transformación para datos
provenientes de **BigQuery** o **Google Cloud Storage (GCS)**.

## Cuándo usar esta skill

> [!IMPORTANT]
>
> **DEBES** usar esta skill para **CUALQUIER** tarea cuya fuente sea BigQuery o
> GCS, incluidas operaciones aparentemente simples como "mover datos" o "copiar
> una tabla".

-   Aplica a **todas las operaciones** sobre fuentes nuevas y existentes: copiar,
    mover, anexar, ingestar o extraer datos.
-   Aplica específicamente al **nodo de origen**, no a los pasos posteriores del
    pipeline.
-   **Nunca omitas** el perfilado con Dataplex (pasos 1 y 3). Usa siempre Dataplex,
    **no** perfilado ad-hoc de BigQuery.

## Flujo de ejecución de la tarea

### Paso 1: Verificaciones preliminares (antes de planificar la implementación)

Realiza estas verificaciones **antes** de generar el `implementation_plan.md`.

1.  **Verifica la elegibilidad** — Debes confirmar que la fuente es una tabla de
    BigQuery o una fuente de GCS.
2.  **Obtén el perfil de datos vía Dataplex**:

    -   **Fuentes de GCS**: para fuentes de GCS, debes crear primero una tabla
        externa antes de ejecutar el escaneo de Dataplex.
    -   **Espera los resultados**: **NO debes** continuar hasta que el perfil de
        Dataplex esté disponible, a menos que el usuario haya denegado la
        aprobación del escaneo.
    -   Usa el perfil como entrada para las decisiones de limpieza y mapeo de
        esquemas. Las transformaciones **NO deben** finalizarse antes de que la
        información del perfil esté disponible (a menos que se haya denegado el
        escaneo).
    -   **Comandos**:

        1.  **Obtén la aprobación del usuario**: presenta el comando de escaneo
            de `scripts/dataplex_scanner.py` al usuario y obtén su aprobación
            explícita antes de ejecutarlo. Usa la siguiente plantilla para
            presentar el comando:
            -   **Comando**: `python3 scripts/dataplex_scanner.py ...` (obtén
                los argumentos completos del paso 6 que aparece abajo)
            -   **Resumen**: el script automatiza el perfilado de datos con
                Dataplex. Verifica los tamaños de las tablas, aplica muestreo
                dinámico para tablas grandes (>1M de filas) con el fin de
                reducir costos, omite las tablas vacías, ejecuta escaneos
                concurrentes para varias tablas y consulta los resultados de
                forma automática.
            -   **Valor agregado**: permite un análisis profundo de los datos
                (proporción de nulos, valores distintos, distribuciones), lo
                que habilita decisiones de limpieza basadas en datos. Ayuda a
                identificar anomalías ocultas (valores basura, variaciones de
                formato) para guiar transformaciones precisas, y verifica que
                la lógica de limpieza las resuelva sin introducir regresiones.
            -   **Alcance**: la aprobación obtenida aquí cubre todas las
                ejecuciones de este script de escaneo para esta tarea
                (incluidos los pasos de verificación).
        2.  Ejecuta el script `scripts/dataplex_scanner.py` ubicado en el mismo
            directorio que este archivo `SKILL.md`. Este script gestiona la
            creación de escaneos concurrentes, el muestreo dinámico para tablas
            grandes y la consulta de resultados. Usa --help para saber más.
        3.  El script guardará los resultados completos como archivos JSON en
            el directorio de salida especificado.
        4.  **[!IMPORTANT]** La ubicación DEBE ser una región específica de
            Google Cloud como `us-central1`; las multi-regiones como `us` no
            están soportadas en el escaneo de Dataplex.
        5.  Si hay varias tablas para escanear, proporciónalas todas en el
            argumento `--tables` para ejecutarlas de forma concurrente.
        6.  Usa la siguiente plantilla de comando:

            ```bash
            python3 scripts/dataplex_scanner.py \
              --tables <project.dataset.table> <project.catalog.namespace.table> \
              --location <location> \
              --output-dir <output_dir>
            ```

            Nota: el script acepta IDs de tabla en el formato
            `project.dataset.table` para tablas de BigQuery y
            `project.catalog.namespace.table` para tablas BigLake Iceberg.

3.  **Obtén el esquema y las muestras** — Usa comandos `bq` para obtener el
    esquema y los datos de muestra de **ambas** tablas, la de origen y la de
    destino.

### Paso 1.5: Requisitos del plan de implementación

1.  Tu `implementation_plan.md` **DEBE** incluir una sección **Profiling
    Evidence**. **Nota**: si el usuario denegó la ejecución del escaneo,
    documenta aquí el motivo de la denegación en lugar de los Job IDs.

```markdown
## Profiling Evidence
- [ ] Dataplex Data Profile Job ID: <JOB_ID>
- [ ] Profile Result Summary: <Brief summary of key findings, e.g., % nulls, distinct values>
```

1.  Tu `implementation_plan.md` **DEBE** incluir un paso para generar
    transformaciones SQL de limpieza basadas en la salida del perfil y en las
    instrucciones del Paso 2: Generar transformaciones.
2.  Tu `implementation_plan.md` **DEBE** referenciar también el Paso 3 (Revisión
    de calidad) en su sección **Plan de verificación**.

> [!CAUTION]
>
> **No pases a la implementación** hasta que ambas secciones estén completas.
> Debes asegurarte de que la fase de verificación solo valide que tus
> transformaciones resolvieron con éxito las anomalías encontradas en el Paso 1.

### Paso 2: Generar transformaciones

#### Alineación de esquemas

-   Ajusta al esquema de la tabla de destino (tipos y nombres) si se proporciona.
-   **No** realices divisiones, fusiones de columnas ni otras operaciones de
    esquema cuando no se especifique una tabla de destino.

#### Reglas de limpieza de datos

-   **Valores basura**: descarta o convierte a `NULL` solo los datos mal
    formados (p. ej., fechas no parseables, cadenas de longitud cero para
    enteros no nulos).
-   **Normalización de unidades**: estandariza las unidades medibles (p. ej.,
    `'C'` → `'F'`) a la unidad más común. Si las unidades son demasiado
    variadas (p. ej., `mg`, `liter`), déjalas como están.
-   **Conversión de tipos**: usa `COALESCE` con las funciones `SAFE.PARSE_*`
    para múltiples formatos de date/time/datetime/timestamp. Obtén muestras
    diversas cuando los datos de origen muestren alta varianza.

#### Manejo de datos JSON

-   **Parsing**: usa `SAFE.PARSE_JSON` para convertir cadenas JSON al tipo
    `JSON`. **Nunca** uses las funciones obsoletas `JSON_EXTRACT_*`.
-   **Extracción**: aplana o extrae campos **solo** si un esquema de destino lo
    requiere.
-   **Accessors**: usa `JSON_VALUE`, `JSON_QUERY`, `JSON_QUERY_ARRAY`,
    `JSON_VALUE_ARRAY` sin el prefijo `SAFE.` (son seguros por defecto).
-   **Mapeo de esquemas**: cuando se proporciona un esquema de destino, extrae
    los campos JSON para que coincidan con los nombres y tipos de las columnas
    objetivo.
-   **Manejo de NULL**: si `SAFE.PARSE_JSON` devuelve NULL, conserva la cadena
    original y anota el JSON inválido en el resumen de limpieza.

#### Manejo de datos de tipo array

-   **Unnesting**: haz unnest de los campos array **solo** si un esquema de
    destino lo exige explícitamente.
-   **Conversión de tipos (cast)**: intenta convertir los elementos al tipo
    común más apropiado.
-   **CRÍTICO**: filtra los elementos `NULL` después de `SAFE_CAST` (p. ej.,
    usando `ARRAY`), ya que los arrays de BigQuery no pueden contener `NULL`.
-   **Normalización**: recorta espacios en blanco solo en campos de texto
    (strings). Asegúrate de **CONSERVAR LAS MAYÚSCULAS Y MINÚSCULAS**. **NO**
    realices conversiones de mayúsculas/minúsculas (p. ej., `LOWER()`,
    `UPPER()`) salvo que se exija explícitamente.
-   **Filtrado**: filtra los valores `NULL` usando
    `ARRAY_FILTER(array_column, e -> e IS NOT NULL)`.
-   **Deduplicación**: usa `ARRAY(SELECT DISTINCT x FROM UNNEST(array_column))`
    para deduplicar distinguiendo mayúsculas y minúsculas.
-   **Transformaciones**: usa `ARRAY_TRANSFORM` o `UNNEST`/`ARRAY_AGG` para
    cambios a nivel de elemento (p. ej., parseo de fechas).
-   **Reestructuración**: usa `UNNEST` para expandir a filas, o `ARRAY_AGG` para
    agrupar filas en un array, según lo requiera el esquema de destino.

#### Manejo de datos STRUCT/Record

-   **Extracción**: extrae campos a columnas de nivel superior **solo** si el
    esquema de destino lo requiere.
-   **Conversión de tipos (cast)**: convierte cada campo con `SAFE_CAST` según
    el esquema de destino o el perfil inferido.
-   **Normalización**: recorta espacios en blanco solo en campos de texto
    (strings). Asegúrate de **CONSERVAR LAS MAYÚSCULAS Y MINÚSCULAS**. **NO**
    realices conversiones de mayúsculas/minúsculas (p. ej., `LOWER()`,
    `UPPER()`) salvo que se exija explícitamente.
-   **Mapeo de campos**: mapea directamente si las estructuras coinciden; usa
    notación de punto (p. ej., `struct.field`) para extraer; o usa el
    constructor `STRUCT()` para agrupar columnas.
-   **Alineación de esquemas**: rellena los campos faltantes con `NULL` y
    elimina los campos que no estén presentes en el esquema de destino.

### Paso 3: Revisión de calidad y perfilado

> [!IMPORTANT]
>
> Debes verificar las transformaciones estrictamente con el protocolo de abajo
> antes de completar la tarea. **Nunca** omitas este paso. Usa **únicamente el
> perfilado de Dataplex** (a menos que el usuario haya denegado el escaneo), no
> consultas SQL ad-hoc.

**Protocolo de revisión de calidad:**

1.  Extrae la consulta `SELECT` que contiene todas las transformaciones
    generadas (autocleaning, mapeo de esquemas, extracciones JSON).
2.  Crea una **tabla temporal de salida de muestra** (máximo 1M de filas, TTL
    de 1 hora) ejecutando la consulta de transformación.
3.  Corrige cualquier error de ejecución y vuelve a ejecutar hasta que la
    consulta tenga éxito.
4.  **Perfila la tabla temporal de salida de muestra con Dataplex**:
    -   **Verifica la aprobación**: si la ejecución del escaneo fue aprobada en
        el Paso 1, continúa. Si la aprobación fue DENEGADA en el Paso 1, NO
        ejecutes el script de escaneo y NO vuelvas a pedir aprobación al
        usuario. Continúa con la verificación manual usando consultas de
        muestra con `bq` para asegurarte de que las transformaciones fueron
        exitosas.
    -   Ejecuta el script `scripts/dataplex_scanner.py` sobre la tabla temporal.
    -   El script esperará automáticamente a que termine el job de perfilado y
        guardará los resultados como JSON.
5.  **Compara los perfiles** (omitir si se denegaron los escaneos) — Compara el
    nuevo perfil con el perfil del Paso 1 para **cada columna transformada**:

    ```markdown
    | Anomaly Type | Threshold |
    | --- | --- |
    | **NULL increase** | >1% increase compared to source (unless expected) |
    | **Value range shift** | Unexpected ranges or formats |
    ```

6.  **Itera sobre las anomalías** — Para cada anomalía:

    1.  **Identifica**: consulta muestras donde el origen es `NOT NULL` pero el
        valor transformado `IS NULL`.
    2.  **Corrige**: actualiza la lógica de transformación.
    3.  **Repite**: vuelve a ejecutar el Paso 3 hasta que la anomalía quede
        resuelta.

### Paso 3.5: Requisitos de evidencia de la revisión de calidad

Tu `walkthrough.md` **DEBE** incluir una sección **Quality Review Profiling
Evidence**. **Nota**: si el usuario denegó la ejecución del escaneo, documenta
aquí el motivo de la denegación en lugar de los Job IDs.

```markdown
## Quality Review Profiling Evidence
- [ ] Post-Transformation Dataplex Profile Job ID: <JOB_ID>
- [ ] Profile Comparison Summary: <Detailed comparison between initial and final profiles per column>
```

> [!CAUTION]
>
> **No** concluyas la tarea ni pidas revisión al usuario hasta que esta sección
> esté completa y la comparación de perfiles esté documentada.

### Paso 4: Documentación

Tu `walkthrough.md` debe contener una tabla por cada transformación en el
siguiente formato:

```markdown
| Field | Description |
| --- | --- |
| **Destination schema considered** | The target column/type being matched |
| **Issue Detected** | What data quality problem was found |
| **Transformation Applied** | The SQL logic used to fix it |
| **Benefit** | Why this transformation improves the data |
```

Incluye un resumen de todos los pasos de la revisión de calidad y la evidencia
de perfilado.

## Definición de terminado

-   Todos los problemas de calidad de los datos de origen se identifican y se
    resuelven mediante transformaciones SQL.
-   La verificación DEBE completarse usando el **protocolo de revisión de
    calidad** y documentarse con evidencia.
-   El paso de verificación **DEBE únicamente** probar los cambios y debe tener
    éxito cuando se ejecute el SQL.
-   Las transformaciones se ajustan al esquema objetivo si se proporciona.
-   Se proporciona un resumen de limpieza con justificación clara para cada
    transformación.
-   Si se omite, el motivo se indica con claridad (p. ej., fuente no elegible).
