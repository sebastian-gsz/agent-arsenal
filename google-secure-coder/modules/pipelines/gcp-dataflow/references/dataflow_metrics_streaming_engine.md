## Métricas de Streaming Engine

*Útiles para depurar jobs de Dataflow Streaming Engine.*

<!-- disableFinding(LIST_NO_LINE) -->

### `job/backlogged_keys`

*   **Nombre visible**: Backlogged Keys
*   **Resumen**: número de claves con backlog para un stage con cuello de botella.
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/bundle_user_processing_latencies`

*   **Nombre visible**: Bundle user processing latencies
*   **Resumen**: latencias de procesamiento de usuario de los bundles de un stage
    concreto. Disponible para jobs que se ejecutan en Streaming.
*   **Tipo/clase**: GAUGE, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/processing_parallelism_keys`

*   **Nombre visible**: The approximate number of parallel processing keys
*   **Resumen**: número aproximado de claves en uso para el procesamiento de datos
    en cada stage.
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/recommended_parallelism`

*   **Nombre visible**: Recommended Parallelism
*   **Resumen**: el parallelism recomendado para un stage con el fin de reducir el
    cuello de botella.
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/streaming_engine/key_processing_availability`

*   **Nombre visible**: Current processing key-range availability
*   **Resumen**: porcentaje de claves de procesamiento de streaming asignadas a
    workers y disponibles para realizar trabajo.
*   **Tipo/clase**: GAUGE, DOUBLE, 10^2.%
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/streaming_engine/persistent_state/read_bytes_count`

*   **Nombre visible**: Storage bytes read
*   **Resumen**: bytes de almacenamiento leídos por un stage concreto. Disponible
    para jobs que se ejecutan en Streaming Engine. Muestreados cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/streaming_engine/persistent_state/stored_bytes`

*   **Nombre visible**: Current persistence state usage
*   **Resumen**: bytes almacenados actualmente en el persistent state del job.
*   **Tipo/clase**: GAUGE, INT64, By
*   **Etiquetas de filtro**: `job_id`

### `job/streaming_engine/persistent_state/write_bytes_count`

*   **Nombre visible**: Storage bytes written
*   **Resumen**: bytes de almacenamiento escritos por un stage concreto.
    Disponible para jobs que se ejecutan en Streaming Engine. Muestreados cada
    60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/streaming_engine/persistent_state/write_latencies`

*   **Nombre visible**: Storage write latencies
*   **Resumen**: latencias de escritura de almacenamiento de un stage concreto.
    Disponible para jobs que se ejecutan en Streaming Engine.
*   **Tipo/clase**: DELTA, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/streaming_engine/stage_end_to_end_latencies`

*   **Nombre visible**: Per stage end to end latencies.
*   **Resumen**: distribución del tiempo que el streaming engine pasa en cada
    stage del pipeline.
*   **Tipo/clase**: GAUGE, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/timers_pending_count`

*   **Nombre visible**: Timers pending count per stage
*   **Resumen**: número de timers pendientes en un stage concreto. Disponible para
    jobs que se ejecutan en Streaming Engine.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/timers_processed_count`

*   **Nombre visible**: Timers processed count per stage
*   **Resumen**: número de timers completados por un stage concreto. Disponible
    para jobs que se ejecutan en Streaming Engine.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/horizontal_worker_scaling`

*   **Nombre visible**: Horizontal worker scaling
*   **Resumen**: un booleano que indica la dirección y la razón del escalado
    horizontal recomendado. True significa que la decisión de escalado tuvo
    efecto, false en caso contrario.

*   **Tipo/clase**: GAUGE, BOOL, 1

*   **Etiquetas de filtro**: `job_id`, `rationale`, `direction`

### `job/max_worker_instances_limit`

*   **Nombre visible**: Job max worker instances limit
*   **Resumen**: número máximo de workers que el autoscaling puede solicitar.
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`

### `job/min_worker_instances_limit`

*   **Nombre visible**: Job min worker instances limit
*   **Resumen**: número mínimo de workers que el autoscaling está obligado a
    solicitar.
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`

### `job/worker_utilization_hint`

*   **Nombre visible**: Job worker utilization hint
*   **Resumen**: hint de utilización del user worker establecido por los clientes
    para definir un rango objetivo de utilización de CPU del worker para el
    autoscaling horizontal, influyendo en la agresividad del escalado. Un valor de
    0 indica que el hint no está en uso activo.
*   **Tipo/clase**: GAUGE, DOUBLE, 10^2.%
*   **Etiquetas de filtro**: `job_id`

### `job/worker_utilization_hint_is_actively_used`

*   **Nombre visible**: Job worker utilization hint is actively used
*   **Resumen**: informa si el hint de utilización del worker está siendo usado
    activamente por la política de autoscaling horizontal.
*   **Tipo/clase**: GAUGE, BOOL, 1
*   **Etiquetas de filtro**: `job_id`
