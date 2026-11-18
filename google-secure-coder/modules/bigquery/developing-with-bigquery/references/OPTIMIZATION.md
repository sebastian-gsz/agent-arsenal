# Optimización de BigQuery

Pautas de rendimiento y eficiencia para queries SQL de BigQuery.

## Reglas de optimización SQL

> [!TIP]
> Incluye siempre una sección **"Resumen de optimizaciones"** que liste solo las optimizaciones aplicadas.

### Aplicar siempre (automático)

| Optimización | Descripción |
| --- | --- |
| **Poda de columnas** | Elimina columnas innecesarias en todas las etapas de la query. |
| **Reutilización de subexpresiones comunes** | Factoriza expresiones idénticas para evitar cálculo redundante. |
| **Predicate pushdown** | Aplica los filtros `WHERE` lo antes posible. |
| **Agregación temprana** | Realiza `GROUP BY` antes de los joins cuando sea posible. |
| **Materialización intermedia** | Elige `VIEW` vs `TABLE` para los nodos intermedios según la eficiencia. |

#### Estrategia para nodos intermedios

| Estrategia | Cuándo usarla |
| --- | --- |
| **`VIEW`** | Datasets pequeños o transformaciones simples. |
| **`TABLE`** | Datasets grandes, cálculos costosos o nodos reutilizados varias veces. |

### Reescribir siempre (obligatorio)

| Patrón | Reemplazar por |
| --- | --- |
| `WHERE <col> IN (SELECT ...)` | `WHERE EXISTS (SELECT 1 FROM ...)` |
| `WHERE (SELECT COUNT(*) ...) > 0` | `WHERE EXISTS (SELECT 1 FROM ...)` |

### Proponer con confirmación (condicional)

- **`UNION` → `UNION ALL`**: Más rápido (omite la deduplicación), pero permite filas duplicadas.
- **`COUNT(DISTINCT)` → `APPROX_COUNT_DISTINCT`**: Más rápido y con menor uso de memoria, pero aproximado.
