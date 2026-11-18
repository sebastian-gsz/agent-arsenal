---
name: notebook-guidance
description: |-
  Esta skill guía el uso de Jupyter notebooks para análisis, exploración y visualización de datos, en particular con BigQuery. Describe buenas prácticas para la ejecución y validación de notebooks (soporta tanto la ejecución celda a celda como la generación del notebook completo, según la disponibilidad de herramientas), la instalación de librerías y la estructuración de notebooks con claridad. También cubre reglas específicas para la limpieza de datos, los gráficos y la integración con flujos de trabajo de BigQuery SQL y machine learning.
  Relevante cuando se cumple alguna de las siguientes condiciones:
    1. La solicitud del usuario implica una tarea de análisis, exploración, visualización o generación de insights de datos que requiere múltiples pasos, consultas o visualizaciones para responderse.
    2. El usuario pide explícitamente un notebook (.ipynb).
    3. Estás creando, editando o ejecutando celdas en un Jupyter notebook.
    4. Necesitas consultar BigQuery desde un notebook. NO uses la librería cliente de Python para BigQuery; en su lugar, DEBES usar los magics `%%bqsql` explicados en esta skill.
license: Apache-2.0
metadata:
  version: v5
  publisher: google
---

# Guía de notebooks

## Cuándo usar un notebook

Antes de decidir usar un notebook, evalúa la complejidad de la tarea con estas
heurísticas.

Usa un notebook si cumples al menos uno de estos criterios:

*   📈 **Data Insights & Storytelling**: usa un notebook para cualquier solicitud
    de "dar insights", "encontrar tendencias", "explorar datos" o "analizar
    datos". Estas tareas se benefician de usar visualizaciones para presentar
    los datos.
*   📊 **Se piden visualizaciones**: el usuario pide explícitamente gráficos o
    plots.
*   🔄 **Exploración con estado / iterativa**: necesitas ejecutar una consulta,
    inspeccionar los resultados y decidir la siguiente consulta en función de
    esos resultados, manteniendo el estado en memoria.

NO uses un notebook SOLO si:

*   📝 **Dato/estado simple**: la solicitud solo requiere un número único
    (p. ej., "cuántas filas") o una verificación de estado (p. ej., "cuándo se
    actualizó esta tabla").
*   🏃‍♂️ **Vista previa del esquema**: la solicitud trata solo del esquema o de
    los tipos de los campos.

**Regla de oro del data storytelling:** si hay cualquier insight analítico,
tendencia o comparación de por medio, favorece un notebook y una visualización.
El notebook es el entorno "estándar" de nuestro flujo de trabajo de desarrollo;
no lo evites por el "overhead".

## Buenas prácticas de notebooks

> [!IMPORTANT]
>
> **Reglas de ejecución del agente**: tu comportamiento DEBE depender de si la
> herramienta `notebook_execute_cell` está disponible en tu contexto actual: *
> **Si la herramienta `execute_cell` del notebook está disponible**: DEBES
> seguir el flujo incremental GENERAR CELDA -> EJECUTAR CELDA -> VALIDAR. * **Si
> la herramienta `execute_cell` del notebook NO está disponible**: DEBES
> generar el notebook completo y solicitar la ejecución al usuario.

1.  **FLUJO DE EJECUCIÓN CONDICIONAL**:
    *   **Si la herramienta `execute_cell` del notebook está disponible**: sigue
        el flujo **PASO A PASO GENERAR CELDA -> EJECUTAR CELDA -> VALIDAR
        SALIDA**. Genera UNA celda, ejecútala y luego verifica la salida. Si la
        salida son datos (p. ej. un dataframe), DEBES inspeccionarlos para
        confirmar que la lógica es correcta antes de generar el siguiente paso.
        La generación por lotes de un notebook entero está estrictamente
        prohibida, porque la propagación de errores en notebooks es costosa de
        corregir.
    *   **Si la herramienta `execute_cell` del notebook NO está disponible**:
        *   Crea el notebook completo de una vez.
        *   Indícale al usuario que ejecute el notebook.
        *   Pídele al usuario que te avise cuando termine la ejecución del
            notebook, para que puedas revisar las salidas, verificar que es
            correcto y corregir cualquier error.
2.  **IDENTIFICA LOS DATOS TEMPRANO**: usa `@skill:discovering-gcp-data-assets`
    o las herramientas de listado de BigQuery para encontrar el
    `project.dataset.table` correcto antes de escribir NINGÚN código. Si falta
    el ID de la tabla, pregúntale al usuario.
3.  **ESTADO FINAL LIMPIO**: el notebook final NO DEBE tener celdas fallidas.
    Si una celda falla, DEBES corregirla. Si probaste varias versiones, elimina
    los intentos fallidos antes de presentar el notebook al usuario.
4.  **FIDELIDAD DE LOS BLOQUES LÓGICOS**: mantén las celdas pequeñas. Una
    transformación lógica o visualización por celda. Agrupa las celdas
    relacionadas en unidades lógicas (p. ej., una celda magic `%%bqsql` de
    BigQuery seguida inmediatamente de una celda de visualización en Python
    para esos resultados). Usa **celdas de markdown** descriptivas para separar
    y documentar las distintas secciones lógicas.
5.  **GENERA VISUALIZACIONES**: acompaña siempre los insights de datos con
    visualizaciones; los gráficos suelen ser más efectivos que los números
    crudos para comunicar tendencias y comparaciones.

## Gestión del kernel y del entorno

Los notebooks se ejecutan en **Kernels** específicos (backends de ejecución).
Debes asegurarte de que el entorno Python del kernel contenga las librerías
necesarias (`bigframes`, `ipykernel`, etc.).

### Tipos de kernel

1.  **Local Python**: kernel estándar de Python 3 que se ejecuta en el host del
    notebook (instancia administrada, máquina local).
2.  **Cloud Spark Remote (Dataproc Serverless)**: entorno Spark transitorio
    administrado por GCP. Úsalo para procesamiento de datos a gran escala.
3.  **Cloud Spark Remote (Dataproc Cluster)**: clústeres Spark persistentes para
    configuraciones compartidas o personalizadas.
4.  **Colab (administrado)**: runtimes efímeros administrados por Google.

### Sin kernel activo / verificación de la configuración

1.  **Infiere o pregunta las preferencias de kernel**:
    -   **Infiere del contexto**:
        -   Si la tarea menciona "Spark", "PySpark" o "distributed compute", o
            si el workspace activo ya es un clúster de Spark, inclínate hacia
            **Remote Spark**.
        -   Si la tarea se centra en "BigQuery", "BigFrames" o llamadas estándar
            a la API, inclínate hacia **Local Python**.
    -   **Pregunta cuando sea ambiguo**: si encajan varias opciones, pregunta
        si prefieren un **Local Python** o un **kernel en la nube/remoto**
        (p. ej., Colab, Spark).
2.  **Para configuración local**: usa `@skill:managing-python-dependencies`
    para verificar si existe un entorno virtual. Si no existe, crea uno.
    Asegúrate de que `ipykernel` esté instalado en ese entorno. Instala
    cualquier otra librería relevante.
3.  **Para configuración remota**: recomienda al usuario usar la UI para
    seleccionar el kernel remoto apropiado.

> [!IMPORTANT]
>
> **ALTO TOTAL ante fallos del kernel**: si la ejecución de una celda devuelve
> "no active kernel" o cualquier error de kernel no encontrado, DEBES
> **detenerte de inmediato**. NO crees estructuras, ni generes, ni insertes más
> celdas. Informa al usuario qué kernel se necesita (p. ej., PySpark / Dataproc
> Serverless) y espera la confirmación explícita de que hay un kernel activo
> antes de continuar con la ejecución del notebook.

### Instalación correcta de librerías

#### 1. Kernels locales

Antes de instalar cualquier librería de Python, DEBES usar
`@skill:managing-python-dependencies` para detectar cómo se gestionan las
dependencies de Python en el proyecto.

#### 2. Kernels remotos (Spark/Colab)

Como suelen ser efímeros o estar administrados por GCP:

*   **Verifica primero (OBLIGATORIO)**: antes de escribir cualquier celda
    `%pip install`, ejecuta `%pip list` o `import <package>` para confirmar que
    el paquete no está ya presente. Los runtimes administrados (Dataproc
    Serverless, Colab) preinstalan muchos paquetes comunes. Instala solo lo que
    se confirme como faltante.
*   Usa `%pip install <package>` en la primera celda si se confirma que falta
    un paquete y es la única forma de modificar el runtime.

Ante la duda sobre el tipo de kernel o el método de instalación preferido,
pide una aclaración al usuario.

## Reglas de análisis y visualización de datos

Directrices para realizar análisis exploratorio de datos, limpieza de datos y
visualización en notebooks.

### Estructura del notebook

El notebook debe leerse como una historia. Aunque tienes flexibilidad (p. ej.,
múltiples visualizaciones para una celda de datos, o celdas de datos que se
construyen unas sobre otras), apunta a este flujo general:

1.  **Título y objetivo** (celda de markdown)
    *   ¿Para qué es este notebook? (p. ej., `# Retention Analysis`)
2.  **Encabezado de sección** (celda de markdown)
    *   ¿Qué estamos viendo ahora? (p. ej., `## Exploring User Retention`)
3.  **Adquisición/transformación de datos** (celda de Python, puede contener
    magics `%%bqsql`)
    *   Consulta BigQuery o transforma los datos.
4.  **Verificación (opcional pero recomendada)** (celda de Python)
    *   `df.head()` o verificaciones de sanidad con assert.
5.  **Visualización (el objetivo)** (celda de Python)
    *   Grafica el insight (p. ej., `df.plot()`).

*Repite los pasos 2-5 para cada nuevo subtema o insight. Puedes tener varias
celdas de datos antes de una visualización, o varias visualizaciones a partir
de una celda de datos. La clave es mantenerlas agrupadas lógicamente y
separadas por encabezados de markdown.*

1.  **Resumen final** (celda de markdown)

    *   Al final del notebook, añade una celda de markdown que contenga un
        párrafo de resumen que sintetice los hallazgos para el usuario. El
        resumen DEBE seguir estas directrices:
    *   NO DEBE añadir código Python al resumen.
    *   El resumen NO DEBE empezar con un bloque de código.
    *   El resumen DEBE basarse estrictamente en los datos numéricos verificados
        en el notebook.
    *   El resumen DEBE contener únicamente las siguientes tres secciones:
        *   ### Q&A Si la tarea de análisis de datos contiene preguntas
            (implícitas o explícitas), DEBES responderlas basándote en el
            proceso de resolución. Omite esta sección si no hay preguntas que
            responder.
        *   ### Hallazgos clave del análisis de datos Resume los hallazgos
            clave del análisis en viñetas; citar los números de los pasos
            anteriores es un plus. Reporta solo los hallazgos de alto valor y
            omite los obvios.
        *   ### Insights o siguientes pasos Aporta 1 o 2 insights o siguientes
            pasos concisos en viñetas.

2.  **Siguientes pasos**: después de que el notebook se haya ejecutado y
    verificado con éxito, y el resumen esté completo, notifica al usuario y
    propón sugerencias de siguientes pasos.

### Reglas de graficación

1.  DEBES usar colores diferentes para las distintas features para asegurar que
    los plots sean legibles para las personas.
2.  Al crear un plot, DEBES ajustar el tamaño de la figura según el número de
    features. Las etiquetas y las leyendas NO DEBEN superponerse.
3.  DEBERÍAS organizar el layout con criterio. Usar subplots PUEDE ayudar a
    colocar los distintos plots de forma efectiva.
4.  DEBES usar figuras inline para presentar las figuras y los plots junto con
    el código y el texto en el notebook.
5.  Para clustering, usa PCA para reducir a 2D antes de hacer el scatter plot.
6.  Usa **gráficos de líneas** SOLO para datos continuos (p. ej., series
    temporales) donde la interpolación entre puntos tenga sentido.

### Reglas de limpieza de datos

1.  DEBES tener cuidado con los valores faltantes y los valores duplicados.
2.  NO DEBES eliminar columnas salvo que sea absolutamente necesario. Eliminar
    columnas es irreversible.
3.  DEBERÍAS centrarte en las columnas directamente relacionadas con completar
    la tarea; no todas las columnas NECESITAN limpiarse.

## Guía especializada de notebooks

Consulta los siguientes recursos como guía sobre temas específicos de
notebooks:

### 1. BigQuery en notebooks

Usa los magics de BigFrames `%%bqsql` para consultas SQL de BigQuery. Estas
celdas soportan la ejecución nativa de SQL de BigQuery y la exportación de
datos a dataframes de BigFrames.

> [!IMPORTANT]
>
> *   Salvo que el usuario lo especifique, **usa siempre SQL para consultar
>     BigQuery.**
> *   NO uses la librería cliente estándar de Python para BigQuery
>     (`google.cloud.bigquery`) ni `pandas.read_gbq`.
> *   **Exportación obligatoria a dataframe**: indica siempre un nombre de
>     dataframe, p. ej. `%%bqsql <df_name>`. Esto facilita usar los resultados
>     en celdas de Python posteriores.
> *   Verifica que esté instalada la versión `2.38.0` o superior de `bigframes`
>     en el entorno de ejecución del notebook. Si falta, pregúntale al usuario
>     si quiere que la actualices por él.

**Ejemplo de uso del magic %%bqsql:**

```python
# Initialize BigFrames and load %%bqsql magics
import bigframes
import bigframes.pandas as bpd
%load_ext bigframes
```

> [!CAUTION]
>
> Usa siempre `%load_ext bigframes` exactamente como se muestra. No cargues
> submódulos; por ejemplo, `%load_ext bigframes.magics` o
> `%load_ext bigframes.bigquery` no son válidos y no deben usarse.

> [!IMPORTANT]
>
> La librería `bigframes` debe estar instalada. Determina si es necesario
> instalar bigframes siguiendo @skill:managing-python-dependencies.

```python
%%bqsql df_sample
SELECT * FROM `project.dataset.table` LIMIT 10
```

#### Anti-patrones (NUNCA HAGAS ESTO)

> [!CAUTION]
>
> 1.  **NO uses el SDK de Python para consultas**: no cambies a
>     `client.query(sql).to_dataframe()` si el SQL falla. Corrige la sintaxis
>     SQL en su lugar.
> 2.  **NO mezcles lógica**: no pongas código Python en la misma celda que los
>     magics `%%bqsql`.

#### Trabajar con resultados SQL en Python

Las celdas magic con `%%bqsql <df_name>` producen un **BigQuery DataFrame**. En
las celdas siguientes puedes usar `<df_name>` directamente.

> [!IMPORTANT]
>
> DEBES usar BigFrames para exploración, manipulación, división de datos, etc.
> DEBES usar SQL de BQML o bigframes.ml para las tareas de machine learning. NO
> DEBES usar pandas ni Scikit-learn.

##### Consejos para BigQuery DataFrame

*   **Evita `.to_pandas()`**: NO DEBES usar `.to_pandas()` para descargar el
    dataset completo a memoria. Hay algunas excepciones:
    *   Un mensaje de error te pide explícitamente usar `to_pandas()`
    *   Vas a visualizar los datos, **y** la librería de visualización no
        acepta instancias de BigFrames Dataframe/Series. En este caso, reduce
        la cantidad de datos que vas a descargar antes de llamar a
        `.to_pandas()`
*   **Evita `read_gbq()` para SQL**: no escribas consultas SQL y las ejecutes
    con `read_gbq()`. Usa en su lugar los métodos de BigFrames
    Dataframe/Series.
*   **Usa el paquete BigFrames ML para tareas de machine learning**: no uses
    Scikit-learn ni otras librerías de ML con dataframes de BigFrames. Importa
    tus herramientas/clases desde `bigframes.ml`.
*   **Quédate en la nube**: realiza la limpieza, transformación y análisis de
    datos con métodos de BigFrames para aprovechar la escala de BigQuery.
*   **Accessors en lugar de UDFs/lambdas**:
    *   Prefiere los accessors integrados (p. ej., `df.col.str.*`,
        `df.col.dt.*`) antes que las UDFs remotas.
    *   **No uses lambdas** con `Series.map()` ni `DataFrame.apply()`.
*   **Verificación del esquema**: no asumas el esquema de las salidas
    intermedias. Revisa `.dtypes` después de cargar, y usa `display()` con
    `.head()` o `.peek()`.
*   **Visualización**: el BigFrames Dataframe funciona directamente en su
    mayoría con Matplotlib, Seaborn y otras librerías de graficación. Si tu
    intento no funcionó, prueba con el accessor "plot". Si eso tampoco
    funcionó, DEBES muestrear o agregar tus datos para reducirlos lo
    suficiente antes de llamar a "to_pandas()".
*   **Persistencia de modelos**: para persistir un modelo, usa
    `model.to_gbq()`. Para cargar un modelo persistido, usa
    `bpd.read_gbq_model()`.

### 2. Machine learning en notebooks

Integración con los flujos de trabajo y buenas prácticas de machine learning. -
**Guía**: usa `@skill:ml-best-practices`. - **LECTURA OBLIGATORIA CUANDO**: la
tarea implica machine learning, entrenar un modelo, clustering, clasificación,
regresión o forecasting de series temporales.

Si se cumple alguna condición de "LECTURA OBLIGATORIA CUANDO", DEBES leer la
guía correspondiente antes de continuar.
