# Contexto de cuellos de botella y paralelismo

## 1. Escalabilidad, claves y paralelismo

Dataflow Streaming Engine opera con un **modelo de procesamiento por clave** para
escalar a decenas de millones de mensajes por segundo garantizando un
procesamiento exactly-once.

### Métricas relevantes

Métricas concretas que referenciar en
[dataflow_metrics_streaming_engine](dataflow_metrics_streaming_engine.md)

*   `job/processing_parallelism_keys` para el paralelismo
*   `job/bundle_user_processing_latencies` para la antigüedad del procesamiento
    de operaciones, indicando operaciones de procesamiento lentas o atascadas
*   `job/streaming_engine/stage_end_to_end_latencies` para el tiempo total de
    extremo a extremo, incluyendo todo el encolamiento, shuffling y procesado
    de usuario.

### Orquestación basada en claves

*   **Definición**: una clave es un identificador que vincula mensajes
    relacionados a lo largo del tiempo (p. ej., en `GroupByKey` para
    agregaciones). El estado se persiste por clave.
*   **Claves explícitas vs. implícitas**:
    *   *Explícitas*: claves definidas por el usuario resultantes del diseño del
        pipeline (p. ej., claves de agrupación).
    *   *Implícitas*: asignadas automáticamente por el sistema cuando no existe
        una clave semántica (p. ej., al leer de Pub/Sub) para rastrear metadatos
        del sistema y estado exactly-once.
*   **Procesamiento serial**: el procesamiento de mensajes está estrictamente
    **serializado por clave** dentro de una stage fusionada.
    *   *Mecanismo*: mientras se procesa un lote de mensajes para la clave
        activa $K$, los mensajes siguientes de $K$ quedan en buffer.
    *   *Justificación*: permite un caching de estado muy eficiente y
        "escrituras ciegas", evitando transacciones costosas o barreras de
        sincronización de grano fino.
    *   *Estado mínimo*: en el fondo, incluso las stages fusionadas puramente
        stateless mutan el estado de tracking exactly-once.

### Restricciones de paralelismo

*   **Límite superior**: el número de claves activas únicas representa el límite
    superior estricto de hilos de ejecución en paralelo.
*   **Cuellos de botella y ley de Amdahl**:
    *   *Baja cardinalidad de claves*: variedad insuficiente de claves
        restringe la ejecución en paralelo, provocando workers ociosos y
        procesamiento lento.
    *   *Hot keys*: distribución desigual del tráfico en la que un subconjunto
        minúsculo de claves recibe la mayoría de los registros. Esto serializa
        el procesamiento de esas claves concretas y crea un cuello de botella
        importante (alta antigüedad del watermark, backlog).
    *   *Alta cardinalidad de claves*: una cardinalidad extremadamente alta de
        claves junto con un retardo de procesamiento elevado puede causar un
        retardo de encolamiento excesivo.

## 2. Operaciones de larga duración o atascadas

La mayoría de las funciones definidas por el usuario (UDFs/`DoFn`s) se ejecutan
en milisegundos. Las llamadas RPC a servicios externos o las llamadas
síncronas y bloqueantes pueden detener la ejecución.

### La cascada de bloqueos

*   **Bloqueo de cabecera de línea a nivel de clave**: como el procesamiento es
    serial por clave, una operación atascada o de larga duración sobre la clave
    $K$ bloquea **todos los mensajes posteriores y futuros** asignados a $K$.
*   **Restricción exactly-once**: los datos en espera nunca se descartan para
    garantizar la seguridad exactly-once.
*   **Contrapresión a nivel de sistema**: a medida que las colas bloqueadas de
    claves concretas crecen y superan los umbrales de flow control/memoria, el
    pipeline aplica contrapresión hacia upstream. Esto acaba suspendiendo todo
    el procesamiento, incluidas las claves sanas.

### Remediación y buenas prácticas

*   **Conectores optimizados**: prefiere los sinks de I/O integrados y
    altamente escalados gestionados (`PubsubIO`, `BigQueryIO`) frente a
    implementaciones HTTP/RPC propias.
*   **I/O propio resiliente**: si las llamadas externas propias son necesarias,
    para garantizar rendimiento usa siempre:
    *   Timeouts estrictos de cliente/red.
    *   Reintentos acotados con backoff exponencial.
    *   Batching para agrupar múltiples elementos en llamadas RPC únicas.
    *   Durante el procesamiento del bundle, paraleliza las RPC externas en vez
        de emitirlas y esperarlas secuencialmente.

## 3. Cuellos de botella en colas y propagación del backlog

Los pipelines de streaming conectan componentes (Streaming Shuffle, hilos de
`DoFn` y checkpoints de estado) mediante **colas** secuenciales que fluyen de
upstream a downstream.

### Causa raíz vs. síntoma

*   **Propagación de la contrapresión**: un cuello de botella en un componente
    downstream hace crecer las colas hacia upstream. Una vez que la cola de
    buffer downstream alcanza su capacidad:
    1.  El downstream aplica contrapresión.
    2.  Las operaciones de shuffle/lectura upstream se pausan.
    3.  La alta latencia/backlog se propaga hasta el origen.
*   **Reto de depuración**: la contrapresión hace que todo el pipeline parezca
    degradado, ocultando el cuello de botella real. Apunta al componente exacto
    que bloquea la cola.

### Detección y umbrales

*   **Disparador**: la detección de cuellos de botella de Dataflow Streaming
    marca una stage como cuello de botella cuando el retardo de la cola del
    sistema supera los **5 minutos**.
*   **Evaluación**:
    *   Los retardos transitorios superiores a 5 minutos (p. ej., por picos de
        tráfico) pueden resolverse solos y quizá no requieran intervención.
    *   Consulta la métrica `job/is_bottleneck` (y los campos `likely_cause`,
        `bottleneck_kind`) para localizar causas raíz y remediar según la
        [Guía de troubleshooting de cuellos de botella de Dataflow](https://docs.cloud.google.com/dataflow/docs/guides/troubleshoot-bottlenecks).
