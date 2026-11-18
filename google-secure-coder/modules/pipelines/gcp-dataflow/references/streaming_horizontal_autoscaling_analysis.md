# Análisis de autoscaling horizontal de Dataflow Streaming

Usa esta referencia para analizar el comportamiento del autoscaling horizontal
de Dataflow y diagnosticar límites o anomalías. Para telemetría completa,
correlaciona este análisis con las métricas definidas en
[Métricas de Streaming Engine](dataflow_metrics_streaming_engine.md).

## 1. Estándares de salud del autoscaling

*   **Utilización de CPU de los workers**:
    *   **Comportamiento sano**: se estabiliza en torno al **70%** por defecto
        bajo carga.
    *   **Comportamiento anómalo/no sano**: trabajadores extremadamente
        infrautilizados ($<20\%$) o completamente saturados ($>90\%$) mientras
        el pipeline está en mal estado.
*   **Backlog estimado**:
    *   **Comportamiento sano**: se mantiene bajo (consistentemente cerca de
        cero).
    *   **Comportamiento anómalo/no sano**: tiempo de backlog creciendo de
        forma sostenida.
*   **Número de workers**:
    *   **Comportamiento sano**: escala fluidamente arriba/abajo con el
        tráfico.
    *   **Comportamiento anómalo/no sano**: restringido por los límites de
        `job/max_worker_instances_limit` o `job/min_worker_instances_limit`.
*   **Decisiones del autoscaler**: consulta la métrica
    `job/horizontal_worker_scaling` e inspecciona el campo `rationale` para
    identificar los disparadores lógicos concretos detrás de las decisiones de
    escalado. Estos disparadores pueden correlacionarse además con los
    diagnósticos del pipeline.

## 2. Cómo opera el Autoscaler

Esto es una visión general; hay heurísticas y casos límite que impiden la
operación aquí descrita, pero en la mayoría de los casos aplican las
siguientes reglas.

*   **Scale Up estándar**: se dispara cuando el tiempo estimado de procesado
    del backlog supera el umbral de disparo de escalado.
*   **Scale Up proactivo (CPU alta)**: si la utilización de CPU sube de forma
    agresiva, el autoscaler escala hacia arriba *antes* de que se forme un
    backlog, absorbiendo ráfagas de tráfico intensivas en recursos.
*   **Scale Down estándar**: solo se dispara cuando **tanto** el backlog
    estimado como la utilización de CPU están bajos.

## 3. Gotchas críticos y estados anómalos

### Throttling no detectado (escenario de depuración crucial)

*   **Mecanismo**: si un pipeline está limitado por latencia de I/O, locks,
    baja utilización o límites de escritura downstream (throttled), el motor
    heurístico del autoscaler puede no detectar la restricción.
*   **Cascada de síntomas**:
    1.  El backlog estimado sigue subiendo.
    2.  La utilización de CPU global de los workers permanece baja (los
        workers están ociosos esperando RPCs downstream).
    3.  El autoscaler añade workers sin fin, escalando hasta el máximo límite
        de workers.
*   **Causas raíz**:
    *   **Cuellos de botella de I/O**: cuotas de APIs de terceros,
        aprovisionamiento de recursos externos o bloqueos de I/O propios.
    *   **Paralelismo insuficiente / hot keys**: espacios de claves poco
        particionados serializan el trabajo sobre pocas claves. Esto deja a la
        mayoría de los workers ociosos (CPU baja) mientras el backlog se
        acumula, provocando el mismo falsos upscaling.

## 4. Mitigación y controles en caliente

Para hacer troubleshooting u optimizar el comportamiento de escalado,
recomienda actualizar el job en ejecución con los siguientes parámetros (ver
[Actualizaciones en caliente de Dataflow](https://docs.cloud.google.com/dataflow/docs/guides/updating-a-pipeline#in-flight-updates)):

*   **Worker Utilization Hint (`job/worker_utilization_hint`)**:
    *   Personaliza la utilización de CPU objetivo para el downscaling.
    *   *Hint bajo*: evita la terminación agresiva de workers, conservando
        capacidad caliente.
    *   *Hint alto*: maximiza la densidad de workers pero exige workloads muy
        paralelizados para evitar picos de backlog.
*   **Límites de clamping (`job/max_worker_instances_limit` /
    `job/min_worker_instances_limit`)**:
    *   Fuerzan límites estrictos sobre el pool, anulando las decisiones del
        autoscaler para evitar desbordes de coste o estabilizar el rendimiento
        durante picos.

## 5. Sesgo algorítmico al escalar arriba y escalada a soporte

*   **Sesgo de minimización de latencia**: ante incertidumbre de telemetría, el
    motor central de autoscaling se sesga deliberadamente hacia escalar arriba
    para garantizar una baja antigüedad de datos/watermark. Este rasgo valora
    la frescura de los datos por encima del coste de infraestructura y de vez
    en cuando genera picos de coste.
*   **Diversidad de workloads**: las reglas heurísticas de autoscaling no
    pueden modelar perfectamente cualquier estructura de pipeline. Patrones de
    telemetría que funcionan en un workload pueden fallar en otro.
*   **Escalada a soporte**: si no existe fuente detectable de throttling según
    lo descrito en la sección "Throttling no detectado", y aplicar las
    mitigaciones en caliente (clamping de límites, ajuste de hints de CPU) no
    produce los resultados de autoscaling deseados, **recomienda al cliente
    contactar con el Soporte de GCP**. Los ingenieros de soporte pueden
    aprovisionar ajustes profundos de backend y configuraciones de motor
    personalizadas para footprints de pipeline concretos.
