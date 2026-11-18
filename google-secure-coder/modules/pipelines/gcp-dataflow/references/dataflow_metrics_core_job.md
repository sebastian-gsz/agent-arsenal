# Métricas de Core Job

*Útiles para analizar la salud general del pipeline, los watermarks y las tasas de
mensajes.*

<!-- disableFinding(LIST_NO_LINE) -->
## `job/backlog_bytes`
*   **Nombre visible**: Per-stage backlog in bytes
*   **Resumen**: cantidad de entrada conocida y sin procesar de un stage, en bytes.
*   **Tipo/clase**: GAUGE, INT64, By
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/backlog_elements`
*   **Nombre visible**: Per-stage backlog in elements
*   **Resumen**: cantidad de entrada conocida y sin procesar de un stage, en
    elementos.
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/bundle_user_processing_latencies`
*   **Nombre visible**: Bundle user processing latencies
*   **Resumen**: latencias de procesamiento de usuario de los bundles de un stage
    concreto. Disponible para jobs que se ejecutan en Streaming.
*   **Tipo/clase**: GAUGE, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/data_watermark_age`
*   **Nombre visible**: Data watermark lag
*   **Resumen**: la edad (tiempo desde el event timestamp) hasta la cual todos los
    datos han sido procesados por el pipeline.
*   **Tipo/clase**: GAUGE, INT64, s
*   **Etiquetas de filtro**: `job_id`

### `job/dofn_latency_average`
*   **Nombre visible**: Average message processing time per DoFn.
*   **Resumen**: el tiempo medio de procesamiento de un único mensaje en un DoFn
    dado (en la ventana de los últimos 3 min).
*   **Tipo/clase**: GAUGE, DOUBLE, ms
*   **Etiquetas de filtro**: `job_id`, `do_fn`

### `job/estimated_timer_backlog_processing_time`
*   **Nombre visible**: Estimated time (in seconds) for timers to complete.
*   **Resumen**: tiempo estimado (en segundos) para que los timers se completen.
    Solo disponible para jobs de Streaming Engine.
*   **Tipo/clase**: GAUGE, INT64, s
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/is_bottleneck`
*   **Nombre visible**: Bottleneck Status and Likely Causes
*   **Resumen**: si un stage concreto del pipeline de Dataflow es un cuello de
    botella, junto con su tipo de cuello de botella y su causa probable.
*   **Tipo/clase**: GAUGE, BOOL,
*   **Etiquetas de filtro**: `job_id`, `stage`, `likely_cause`, `bottleneck_kind`

### `job/longest_processing_time`
*   **Nombre visible**: Longest Processing Times
*   **Resumen**: los 10 tiempos de operación activa más antiguos agrupados por
    user worker. Muestreados cada 60 segundos.
*   **Tipo/clase**: GAUGE, INT64, us
*   **Etiquetas de filtro**: `job_id`, `user_worker_id`

### `job/oldest_active_message_age`
*   **Nombre visible**: Oldest active message processing time per DoFn.
*   **Resumen**: cuánto tiempo lleva procesándose el mensaje activo más antiguo de
    un DoFn.
*   **Tipo/clase**: GAUGE, INT64, ms
*   **Etiquetas de filtro**: `job_id`, `do_fn`

### `job/per_stage_data_watermark_age`
*   **Nombre visible**: Per-stage data watermark lag
*   **Resumen**: la edad (tiempo desde el event timestamp) hasta la cual todos los
    datos han sido procesados por este stage del pipeline.
*   **Tipo/clase**: GAUGE, INT64, s
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/per_stage_system_lag`

*   **Nombre visible**: Per-stage system lag
*   **Resumen**: la duración máxima actual que un elemento de datos ha estado
    procesándose o esperando procesamiento en este stage.
*   **Tipo/clase**: GAUGE, INT64, s
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/system_lag`

*   **Nombre visible**: System lag
*   **Resumen**: la duración máxima actual que un elemento de datos ha estado
    procesándose o esperando procesamiento.
*   **Tipo/clase**: GAUGE, INT64, s
*   **Etiquetas de filtro**: `job_id`

### `job/thread_time`

*   **Nombre visible**: Thread Time
*   **Resumen**: tiempo estimado en milisegundos dedicado a ejecutarse en la
    función del ptransform, acumulado entre todos los threads.
*   **Tipo/clase**: DELTA, INT64, ms
*   **Etiquetas de filtro**: `job_id`, `ptransform`, `function`

### `job/user_counter`

*   **Nombre visible**: User Counter
*   **Resumen**: una métrica de contador definida por el usuario. Muestreada cada
    60 segundos.
*   **Tipo/clase**: GAUGE, DOUBLE, 1
*   **Etiquetas de filtro**: `metric_name`, `job_id`, `ptransform`

### `job/dofn_latency_max`
*   **Nombre visible**: Maximum message processing time per DoFn.
*   **Resumen**: el tiempo máximo de procesamiento de un único mensaje en un DoFn
    dado (en la ventana de los últimos 3 min).
*   **Tipo/clase**: GAUGE, INT64, ms
*   **Etiquetas de filtro**: `job_id`, `do_fn`

### `job/dofn_latency_min`
*   **Nombre visible**: Minimum message processing time per DoFn.
*   **Resumen**: el tiempo mínimo de procesamiento de un único mensaje en un DoFn
    dado (en la ventana de los últimos 3 min).
*   **Tipo/clase**: GAUGE, INT64, ms
*   **Etiquetas de filtro**: `job_id`, `do_fn`

### `job/dofn_latency_num_messages`
*   **Nombre visible**: Number of messages processed per DoFn.
*   **Resumen**: el número de mensajes procesados por un DoFn dado (en la ventana
    de los últimos 3 min).
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `do_fn`

### `job/dofn_latency_total`
*   **Nombre visible**: Total message processing time per DoFn.
*   **Resumen**: el tiempo total de procesamiento de todos los mensajes en un DoFn
    dado (en la ventana de los últimos 3 min).
*   **Tipo/clase**: GAUGE, INT64, ms
*   **Etiquetas de filtro**: `job_id`, `do_fn`

### `job/element_count`
*   **Nombre visible**: Element count
*   **Resumen**: número de elementos añadidos a la pcollection hasta ahora.
    Muestreado cada 60 segundos. Tras el muestreo
*   **Tipo/clase**: GAUGE, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `pcollection`

### `job/elements_produced_count`
*   **Nombre visible**: Elements Produced
*   **Resumen**: el número de elementos producidos por cada PTransform. Muestreado
    cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `pcollection`, `ptransform`

### `job/estimated_backlog_processing_time`
*   **Nombre visible**: Estimated time to process current backlog per stage
*   **Resumen**: tiempo estimado (en segundos) para consumir el backlog actual si
    no entran datos nuevos y el throughput se mantiene igual.
*   **Tipo/clase**: GAUGE, INT64, s
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/estimated_byte_count`
*   **Nombre visible**: Estimated byte count
*   **Resumen**: número estimado de bytes añadidos a la pcollection hasta ahora.
*   **Tipo/clase**: GAUGE, INT64, By
*   **Etiquetas de filtro**: `job_id`, `pcollection`

### `job/estimated_bytes_active`
*   **Nombre visible**: Active Size
*   **Resumen**: número estimado de bytes activos en este stage del job.
*   **Tipo/clase**: GAUGE, INT64, By
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/estimated_bytes_consumed_count`
*   **Nombre visible**: Throughput
*   **Resumen**: número estimado de bytes consumidos por el stage de este job.
*   **Tipo/clase**: DELTA, INT64, By
*   **Etiquetas de filtro**: `job_id`, `stage`

### `job/estimated_bytes_produced_count`
*   **Nombre visible**: Estimated Bytes Produced
*   **Resumen**: el tamaño total estimado en bytes de los elementos producidos por
    cada PTransform. Muestreado cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `pcollection`, `ptransform`
