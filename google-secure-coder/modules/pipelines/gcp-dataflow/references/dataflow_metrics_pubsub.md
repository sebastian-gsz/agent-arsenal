## Métricas de Pub/Sub

*Útiles para depurar jobs de Dataflow con fuente/sink de Pubsub.*

<!-- disableFinding(LIST_NO_LINE) -->
### `job/pubsub/late_messages_count`
*   **Nombre visible**: Pub/Sub Late Messages Count
*   **Resumen**: número de mensajes de Pub/Sub con timestamp anterior al watermark
    estimado. Muestreados cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`, `subscription_project_id`,
    `subscription_id`, `topic_project_id`, `topic_id`

### `job/pubsub/published_messages_count`
*   **Nombre visible**: Pub/Sub Published Messages Count
*   **Resumen**: número de mensajes de Pub/Sub publicados, desglosado por topic
    y status. Muestreados cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`, `topic_project_id`, `topic_id`,
    `status`

### `job/pubsub/pulled_message_ages`
*   **Nombre visible**: Pub/Sub Pulled Message Ages
*   **Resumen**: distribución de las edades de los mensajes de Pub/Sub extraídos
    (pulled) pero sin confirmar (unacked). Muestreados cada 60 segundos.
*   **Tipo/clase**: GAUGE, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `stage`, `subscription_project_id`,
    `subscription_id`, `topic_project_id`, `topic_id`

### `job/pubsub/read_count`
*   **Nombre visible**: PubsubIO.Read requests from Dataflow jobs
*   **Resumen**: solicitudes Pull de Pub/Sub. Para Streaming Engine, esta métrica
    está deprecada.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `status`, `job_id`, `ptransform`,
    `subscription_project_id`, `subscription_id`, `topic_project_id`, `topic_id`

### `job/pubsub/read_latencies`
*   **Nombre visible**: Pub/Sub Pull Request Latencies
*   **Resumen**: latencias de las solicitudes Pull de Pub/Sub de PubsubIO.Read en
    jobs de Dataflow. Para Streaming Engine, esta métrica está deprecada.
*   **Tipo/clase**: DELTA, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `ptransform`, `subscription_project_id`,
    `subscription_id`, `topic_project_id`, `topic_id`

### `job/pubsub/streaming_pull_connection_status`
*   **Nombre visible**: Percentage of active/terminated Streaming Pull connections
*   **Resumen**: porcentaje de todas las conexiones Streaming Pull que están
    activas (status OK) o terminadas por algún error.
*   **Tipo/clase**: GAUGE, DOUBLE, %
*   **Etiquetas de filtro**: `status`, `job_id`, `ptransform`,
    `subscription_project_id`, `subscription_id`, `topic_project_id`, `topic_id`

### `job/pubsub/timestamp_events`
*   **Nombre visible**: Pub/Sub Event Timestamp Status
*   **Resumen**: número de mensajes de Pub/Sub con event timestamps, desglosado
    por topic y status. Muestreados cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `job_id`, `stage`, `topic_project_id`, `topic_id`,
    `status`

### `job/pubsub/write_count`
*   **Nombre visible**: Pub/Sub Publish Requests
*   **Resumen**: solicitudes Publish de Pub/Sub de PubsubIO.Write en jobs de
    Dataflow. Muestreadas cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `status`, `job_id`, `ptransform`, `topic_project_id`,
    `topic_id`

### `job/pubsub/write_latencies`
*   **Nombre visible**: Pub/Sub Publish Request Latencies
*   **Resumen**: latencias de las solicitudes Publish de Pub/Sub de PubsubIO.Write
    en jobs de Dataflow. Muestreadas cada 60 segundos.
*   **Tipo/clase**: DELTA, DISTRIBUTION, ms
*   **Etiquetas de filtro**: `job_id`, `ptransform`, `topic_project_id`, `topic_id`
