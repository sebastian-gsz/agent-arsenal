# Análisis de salud de jobs de streaming

## 1. Clasificación de la salud del job

La salud de un job de streaming de Dataflow se determina principalmente por el
comportamiento y la latencia de su **data watermark**.

*   **Estado de salud: Healthy**

    *   **Criterios**: el data watermark es estable y está cerca del tiempo real
        ($\le$ 1 minuto de retraso).

*   **Estado de salud: Mostly Healthy**

    *   **Criterios**:
        *   El watermark es estable pero se sitúa en una línea base constante
            ligeramente más alta. ($\le$ 5 minutos de retraso).
        *   El job arrancó con un backlog masivo, pero el backlog decrece de forma
            constante. > [!NOTE] > El avance del watermark durante la limpieza del
            backlog suele ser irregular/espasmódico debido a los límites de
            procesamiento fuera de orden (habitual con fuentes de Pub/Sub).

*   **Estado de salud: Not Healthy**

    *   **Criterios**: el watermark presenta un retraso significativo (varios
        minutos) y/o muestra picos de latencia recurrentes o crecientes.

## 2. Directrices de telemetría y análisis

### Suficiencia del tiempo de ejecución

*   **Requisito**: un job necesita un mínimo de **5 minutos de telemetría activa**
    para acumular suficiente telemetría/tendencias que permitan una determinación
    fiable de la salud.

### Estrategia de diagnóstico para jobs unhealthy

Cuando un pipeline se marca como **Not Healthy**, investiga correlacionando las
fluctuaciones de métricas con otras métricas y con los logs de los workers:

1.  **Cuellos de botella**: analiza los queue delays usando `job/is_bottleneck`.
2.  **Restricciones de parallelism**: revisa si hay cardinalidad de claves
    insuficiente o hot keys (`job/backlogged_keys`).
3.  **Ejecución bloqueada**: audita los logs de los workers en busca de thread
    stack dumps, llamadas lentas de clientes HTTP/DB u operaciones de larga
    duración.

### Análisis temporal (segmentación de la línea de tiempo)

Si el comportamiento del pipeline cambia dentro de la ventana observada
(p. ej., pasa de healthy $\rightarrow$ unhealthy, o cambian las causas de los
cuellos de botella):

*   No agregues los tramos enteros de comportamientos contrastantes como un único
    estado.
*   **Segmenta la línea de tiempo** en fases distintas de comportamiento y analiza
    la telemetría/las causas raíz de forma independiente para cada una.
*   *Advertencia sobre oscilaciones*: si los cuellos de botella del stage oscilan
    repetidamente entre las mismas causas, resúmelo como una única fase de
    comportamiento recurrente.

### Corroboración y manejo de la incertidumbre

*   **Validación multi-métrica**: prevén proactivamente los falsos positivos.
    Corrobora todas las causas raíz hipotetizadas entre múltiples fuentes de
    telemetría (p. ej., valida un `likely_cause` de cuello de botella contra picos
    del watermark age, saturación de recursos y excepciones en los logs de los
    workers).
*   **Evita la especulación**: declara explícitamente la incertidumbre cuando los
    datos sean ambiguos o incompletos. Evita conclusiones altamente especulativas,
    ya que dirigir al usuario por un camino de resolución de problemas equivocado
    genera un gran esfuerzo adicional al desarrollador.
