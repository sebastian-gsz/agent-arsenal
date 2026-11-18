# Inspección directa del esquema de una tabla

## Para tablas y vistas de BigQuery, Spanner y BigLake Iceberg
Usa la skill `@skill:discovering-gcp-data-assets` para encontrar y consultar el esquema.

## Para exploración de bucket o carpeta de GCS
Si el usuario indica un bucket o carpeta de GCS en lugar de archivos específicos,
**DEBES** explorar primero el contenido de la carpeta para identificar los archivos
relevantes con el comando
`gcloud storage ls gs://<GCS_BUCKET>/<PATH>`.

## Para archivo CSV
Observa la primera fila del archivo CSV

### Para archivo CSV en GCS
Usa `gcloud storage cat gs://bucket/file.csv | head -n 1`

### Para archivo CSV local
Usa `head -n 1 file.csv`
