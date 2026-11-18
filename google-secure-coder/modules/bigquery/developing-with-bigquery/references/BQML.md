# Funciones de BigQuery ML (BQML) y AI

Reglas y estándares de sintaxis para las funciones de AI y Machine Learning de BigQuery.

## 1. Restricciones globales

* **Connection ID**: Usa `'DEFAULT'` para el argumento `connection` en sentencias `CREATE MODEL` remotas.
* **Creación de datasets**: Usa `CREATE SCHEMA IF NOT EXISTS <project>.<dataset>;`.

## 2. Enrutamiento obligatorio de funciones

Función/Caso de uso         | Archivo de referencia requerido
------------------------- | ----------------------------------------------
**AI.FORECAST**           | [ai-forecast.md](ai-forecast.md)
**AI.EVALUATE**           | [ai-evaluate.md](ai-evaluate.md)
**AI.GENERATE_TABLE**     | [ai-generate-table.md](ai-generate-table.md)
**AI.GENERATE_EMBEDDING** | [ai-generate-embedding.md](ai-generate-embedding.md)
**Modelos remotos**         | [remote-models.md](remote-models.md)
**CONTRIBUTION_ANALYSIS** | [ml-contribution-analysis.md](ml-contribution-analysis.md)
**VECTOR_SEARCH**         | [vector-search.md](vector-search.md)

## 3. Verificaciones de sintaxis obligatorias

* **Table-Valued Functions (TVFs)**: `AI.GENERATE_TABLE`, `AI.FORECAST`, `AI.EVALUATE` y `AI.GENERATE_EMBEDDING` DEBEN ubicarse en la cláusula `FROM`.
* **Argumentos nombrados**: `AI.FORECAST` y `AI.EVALUATE` requieren el operador `=>` para los argumentos opcionales.
* **El alias "prompt"**: Para `AI.GENERATE_TABLE`, la subquery de entrada debe contener una columna aliada como `prompt`.
* **Comillas del esquema**: Asegúrate de que la cadena `output_schema` esté entrecomillada.

## 4. Selección de modelos

* **Series temporales**: `AI.FORECAST` usa endpoints de **TimesFM**.
* **Generativas**: `AI.GENERATE_TABLE` usa endpoints de **Gemini**.
* **Vigencia**: Prefiere modelos actuales (p. ej., `gemini-2.5-flash`) por encima de los deprecados.
