# Creación de modelos remotos
Las funciones generativas y de embeddings requieren un modelo remoto que apunte a
un endpoint específico (p. ej., Gemini). Los modelos remotos se crean usando una
conexión para interactuar con Vertex AI.

### Sintaxis
```sql
CREATE [OR REPLACE] MODEL `project.dataset.model_name`
REMOTE WITH CONNECTION { DEFAULT | `project.region.connection_id` }
OPTIONS (ENDPOINT = 'endpoint_name');
```

### Endpoints disponibles

* **Generativos**: `gemini-2.5-pro`, `gemini-2.5-flash`
* **Embeddings**: `text-embedding-005`, `text-multilingual-embedding-002`,
  `gemini-embedding-001`.

### Uso de conexiones
Si se usa `REMOTE WITH CONNECTION DEFAULT`, BigQuery intenta automáticamente usar
una conexión por defecto en la región del modelo. La conexión por defecto debería
ser el comportamiento predeterminado. De lo contrario, debes especificar un ID de
conexión completamente calificado (p. ej., `my-project.us.my-connection`).
