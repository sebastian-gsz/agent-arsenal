## Métricas de BigQuery

*Útiles para evaluar fallos o retrasos de escritura en el sink de BigQuery.*

### `job/bigquery/write_count`

*   **Nombre visible**: BigQueryIO.Write Requests
*   **Resumen**: solicitudes de escritura de BigQuery de BigQueryIO.Write en jobs de
    Dataflow. Muestreadas cada 60 segundos.
*   **Tipo/clase**: DELTA, INT64, 1
*   **Etiquetas de filtro**: `status`, `job_id`, `ptransform`, `bigquery_project_id`,
    `bigquery_dataset_id`, `bigquery_table_or_view_id`
