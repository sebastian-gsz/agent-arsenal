# Referencia de diagnósticos de Dataflow

> [!IMPORTANT] Para realizar un Root Cause Analysis (RCA) completo, DEBES obtener
> los **Job Messages/Events** y tanto **Monitoring** como **Logging** para
> correlacionar los picos/caídas de métricas con los errores de los logs. Sigue
> siempre las queries de ejemplo proporcionadas en las secciones siguientes.

## 1. Monitoring

### Jobs de streaming

Métricas clave a revisar para jobs de streaming:

*   **Data Freshness**:
    *   `job/per_stage_data_watermark_age` (Filtros: `job_id`, `stage`)
    *   `job/data_watermark_age` (Filtros: `job_id`)
*   **Throughput**:
    *   `job/elements_produced_count` (Filtros: `job_id`, `pcollection`,
        `ptransform`)
    *   `job/estimated_bytes_produced_count` (Filtros: `job_id`, `pcollection`,
        `ptransform`)
*   **Backlog**:
    *   `job/estimated_backlog_processing_time` (Filtros: `job_id`, `stage`)
    *   `job/backlog_bytes` (Filtros: `job_id`, `stage`)
*   **System Latency**:
    *   `job/per_stage_system_lag` (Filtros: `job_id`, `stage`)
    *   `job/system_lag` (Filtros: `job_id`)
*   **Autoscaling**:
    *   `job/horizontal_worker_scaling` (Filtros: `job_id`, `rationale`,
        `direction`)
*   **Cuellos de botella**:
    *   `job/is_bottleneck` (Filtros: `job_id`, `stage`, `likely_cause`,
        `bottleneck_kind`)
    *   `job/backlogged_keys` (Filtros: `job_id`, `stage`)
*   **Utilización de recursos (CPU/Memoria)**:
    *   `compute.googleapis.com/instance/cpu/utilization`
    *   `compute.googleapis.com/guest/memory/bytes_used`

### Jobs batch

Métricas clave a revisar para jobs batch:

*   **Utilización de recursos (CPU/Memoria)**:
    *   `compute.googleapis.com/instance/cpu/utilization`
    *   `compute.googleapis.com/guest/memory/bytes_used`
*   **Throughput**:
    *   `job/elements_produced_count` (Filtros: `job_id`, `pcollection`,
        `ptransform`) *(el throughput es el indicador principal para jobs batch)*

### Petición a la Monitoring API

```bash
curl -s -G -H \"Authorization: Bearer \$(gcloud auth print-access-token)\" \\
  --data-urlencode 'filter=metric.type=\"<METRIC_TYPE>\" AND metric.labels.job_id=\"<JOB_ID>\"' \\
  --data-urlencode 'interval.startTime=<START_TIME>' \\
  --data-urlencode 'interval.endTime=<END_TIME>' \\
  \"https://monitoring.googleapis.com/v3/projects/<PROJECT_ID>/timeSeries\"
```

Placeholders: reemplaza `<METRIC_TYPE>`, `<JOB_ID>`, `<START_TIME>`, `<END_TIME>`
y `<PROJECT_ID>` con los valores reales. Timestamps: deben estar en formato ISO 8601
(p. ej., 2026-04-20T23:16:03Z).

## 2. Análisis de logs

Usa `gcloud logging read` para obtener los logs del job de Dataflow

## 3. Dataflow REST API (queries directas)

Usa la Dataflow REST API (v1b3) para obtener métricas de snapshot actuales o
job messages de alto nivel.

**Obtener métricas del job (snapshot):** `bash curl -X GET \ -H "Authorization: Bearer
$(gcloud auth print-access-token)" \
"https://dataflow.googleapis.com/v1b3/projects/<PROJECT_ID>/locations/<REGION>/jobs/<JOB_ID>/metrics"`

**Obtener Job Messages/Events de alto nivel:** `bash curl -X GET \ -H "Authorization:
Bearer $(gcloud auth print-access-token)" \
"https://dataflow.googleapis.com/v1b3/projects/<PROJECT_ID>/locations/<REGION>/jobs/<JOB_ID>/messages"`

## 4. Métricas adicionales de Dataflow
Solo cuando necesites más métricas no cubiertas por las secciones anteriores, o
quieras comprobar el rendimiento basado en source/sink, consulta:

* [Métricas de Core Job](dataflow_metrics_core_job.md)
* [Métricas de Pub/Sub](dataflow_metrics_pubsub.md)
* [Métricas de BigQuery](dataflow_metrics_bigquery.md)
* [Métricas de Streaming Engine](dataflow_metrics_streaming_engine.md)
