# AI.GENERATE_TABLE
Usa modelos de Gemini para extraer información hacia un esquema SQL predefinido.
Es una "Table-Valued Function", es decir, se invoca en la cláusula `FROM`.

## Sintaxis
```sql
SELECT
  *
FROM
  AI.GENERATE_TABLE(
    MODEL `project.dataset.model`,
    { TABLE `project.dataset.table` | (QUERY_STATEMENT) },
  STRUCT(
    OUTPUT_SCHEMA AS output_schema
    [, MAX_OUTPUT_TOKENS AS max_output_tokens]
    [, TOP_P AS top_p]
    [, TEMPERATURE AS temperature]
    [, STOP_SEQUENCES AS stop_sequences]
    [, SAFETY_SETTINGS AS safety_settings]
    [, REQUEST_TYPE AS request_type])
  )
```

## Argumentos de entrada
| Argumento | Requisito | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **`model`** | **Obligatorio** | | El recurso del modelo remoto (p. ej., `project.dataset.model`). |
| **`input_data`** | **Obligatorio** | | La tabla o sentencia `SELECT` de origen. Debe contener una columna llamada o aliada como `prompt`. |
| **`output_schema`** | **Obligatorio** | `STRUCT` | Un `STRUCT` con definiciones de columnas al estilo SQL (p. ej., `"name STRING, qty INT64"`). |
| **`max_output_tokens`** | Opcional | Int64 | [1, 8192]. Limita la longitud de la respuesta. |
| **`temperature`** | Opcional | Float64 | [0.0, 2.0]. Grado de aleatoriedad (se recomienda 0.0 para respuestas deterministas). |
| **`top_p`** | Opcional | Float64 | [0.0, 1.0]. Modifica cómo el modelo selecciona los tokens de salida. |
| **`stop_sequences`** | Opcional | Array<String> | Cadenas que detienen la generación del modelo si coinciden. |
| **`safety_settings`** | Opcional | Array<Struct> | Umbrales para filtrar discurso de odio, acoso, etc. |
| **`request_type`** | Opcional | String | `SHARED`, `DEDICATED` o `UNSPECIFIED` (por defecto `UNSPECIFIED`). |

### Construcción del prompt (campos del STRUCT)
Al usar un `QUERY_STATEMENT`, puedes construir prompts multimodales combinando
tipos:

* **`STRING` / `ARRAY<STRING>`**: Texto literal o nombres de columnas.
* **`ObjectRefRuntime`**: Usa `OBJ.GET_ACCESS_URL(col, 'r')` para imágenes/PDFs.

## Esquema de salida

La tabla de salida es un join entre tus datos de origen y el contenido generado
por el modelo. Conserva todas las columnas de entrada originales para mantener
la trazabilidad.

| Columna | Tipo | Descripción |
| :--- | :--- | :--- |
| **`[Input Columns]`** | (Como entrada) | Cada columna incluida en tu `TABLE` o `QUERY_STATEMENT` de entrada. |
| **`[Schema Columns]`**| (Según definición)| Las columnas tipadas que especificaste en el argumento `OUTPUT_SCHEMA`. |
| **`full_response`** | JSON        | La respuesta JSON cruda, sin parsear, del modelo Gemini subyacente. |
| **`status`** | STRING      | Estado de la ejecución; contendrá mensajes de error si una fila falla en la extracción. |

## Ejemplo: Preservación de claves durante el resumen con un LLM

```sql
-- The output includes 'ticket_id' from the input and 'priority' and 'issue_summary' from the AI schema
SELECT
  ticket_id,
  priority,
  issue_summary
FROM AI.GENERATE_TABLE(
  MODEL `prod.models.gemini_flash`,
  (SELECT ticket_id, body AS prompt FROM `support.tickets`),
  STRUCT("priority STRING, issue_summary STRING" AS output_schema)
)
```
