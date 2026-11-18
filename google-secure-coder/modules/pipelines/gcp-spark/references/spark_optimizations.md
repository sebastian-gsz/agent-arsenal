# Optimizaciones de Spark

## Broadcast Joins

Al realizar un join estándar entre una tabla de hechos grande y una tabla de
dimensiones diminuta (tabla de lookup), usa siempre el hint de broadcast
`pyspark.sql.functions.broadcast()`. Sin él, Spark puede ejecutar una
operación de shuffle pesada y provocar problemas de rendimiento o errores de
falta de memoria (out-of-memory).
