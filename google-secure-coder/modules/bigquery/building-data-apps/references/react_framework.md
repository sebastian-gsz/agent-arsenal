# Construcción con React + Vite

Sigue estas instrucciones al construir aplicaciones de datos con el framework React y Vite.

## Stack tecnológico

-   **Framework:** React en Vite
-   **Estilizado:** Tailwind CSS (Dark Mode por defecto)
-   **Iconos:** `lucide-react`
-   **Formato de fechas:** `date-fns`
-   **Obtención de datos:** Axios (llamadas a API REST)
-   **Drag & Drop:** `@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/utilities`
    (usado para listas/swatches ordenables)
-   **Visualización de datos:** `echarts` y `echarts-for-react`

## Visuales globales y theming

-   **Fondos:** la app debe soportar tanto modo Light como Dark. Mapea siempre
    los colores a su variante `dark:` y provee un equivalente para modo light.
    -   Fondo de página: `bg-zinc-50 dark:bg-[#09090b]`
    -   Tarjetas / contenedores: `bg-white dark:bg-[#0c0c0f]` (oscuro sólido
        apenas más claro)
-   **Tipografía:** usa fuentes sans-serif por defecto.
    -   Texto primario (encabezados, datos importantes): `text-zinc-950
        dark:text-zinc-50`
    -   Texto secundario (labels, fechas, encabezados de tabla): `text-zinc-500
        dark:text-zinc-400`
    -   Bordes: separadores sutiles `border-zinc-200 dark:border-zinc-800`
    -   Esquinas redondeadas: los contenedores deben usar `rounded-xl`, las
        barras gráficas internas deben usar `rounded-sm` o `rounded-md`.
-   **Toggle de tema:** provee un botón de icono simple de Sol/Luna en el
    encabezado que active y desactive la clase HTML `dark`.

## Jerarquía del layout

La aplicación debe seguir un layout de dashboard limpio:

1. **Fila de encabezado:** un título principal alineado a la izquierda, toggle de
tema alineado a la derecha. `max-w-[1600px] mx-auto p-6`.
2. **Fila de tarjetas KPI:** una grilla horizontal (`grid-cols-4` o `grid-cols-1
md:grid-cols-4`) de estadísticas de resumen.
3. **División del contenido principal:** un contenedor flex que contiene un elemento primario prominente (p. ej., una tabla de datos) y un
panel secundario condicional (p. ej., un panel de investigación/detalles).

  - *Estado normal:* la tabla ocupa `w-full`.
  - *Estado seleccionado:* la tabla se reduce a `w-2/3`, el panel lateral entra deslizándose/fundido como `w-1/3`. Agrega `transition-all duration-300` para que el cambio sea suave.

## Componentes

### 1. Tarjetas KPI

-   Deben tener un borde sutil (`border-zinc-200 dark:border-zinc-800`).
-   Muestran un icono pequeño (p. ej., una flecha de tendencia o un icono de
    lucide) junto a un valor numérico grande.
-   Labels de texto pequeños para el título del valor
    (`text-zinc-500 dark:text-zinc-400`).
-   **Indicador de tendencia:** debe usar un fondo estilo píldora según el
    valor: `bg-emerald-100 text-emerald-700` y `dark:bg-emerald-900/30
    dark:text-emerald-400` para tendencias positivas. Las tendencias negativas
    deben usar los equivalentes `bg-rose-100 dark:bg-rose-900/30`. Incluye
    flechas de tendencia pequeñas embebidas.

### 2. Tabla de datos principal

-   **Contenedor:** el contenedor de la tabla debe ser `bg-white dark:bg-zinc-900
    border border-zinc-200 dark:border-zinc-800 rounded-[12px]
    overflow-hidden`.
-   **Toolbar:** sobre la tabla, coloca las acciones de filtrado (p. ej.,
    "Filtrar", "Fecha") como botones dropdown.
    -   *Lógica de filtrado:* no elimines los datos crudos al filtrar. Usa
        estado local del componente para mapear/filtrar la visibilidad según los
        criterios seleccionados en el dropdown.
-   **Encabezado:** encabezado de tabla sticky (`sticky top-0 bg-zinc-100
    dark:bg-zinc-900 z-10 text-zinc-500 dark:text-zinc-400`).
-   **Filas:** filas estándar (`text-zinc-900 dark:text-zinc-200 border-b
    border-zinc-100 dark:border-zinc-800`). Agrega efectos hover
    (`hover:bg-zinc-100 dark:hover:bg-zinc-800 cursor-pointer
    transition-colors`).
-   **Estado de selección:** la fila activa debe tener un resaltado distintivo
    (p. ej., `bg-blue-50 dark:bg-blue-900/40 hover:bg-blue-100
    dark:hover:bg-blue-900/60`).
-   **Indicadores visuales (sparklines):** para puntajes numéricos o valores de
    riesgo, usa barras de progreso horizontales en lugar de píldoras de texto
    crudo.
    -   Color del track: `bg-zinc-700` (gris oscuro)
    -   Colores de relleno: `bg-emerald-500` (riesgo bajo), `bg-orange-400`
        (medio), `bg-rose-500` (riesgo alto). Agrega lógica inline
        `style={{ width: score + "%" }}`.
-   **Controles de paginación:** colócalos en el área del borde inferior.
    -   Restringe opcionalmente la altura de la tabla, o fija los límites de
        paginación en unas 30 filas.
    -   Usa exclusivamente botones de solo icono para los labels de texto de
        paginación (`ChevronsLeft` para primera página, `ChevronLeft` para
        anterior, `ChevronRight` para siguiente, `ChevronsRight` para última).
    -   Estiliza visualmente los estados disabled (`disabled:opacity-50
        disabled:cursor-not-allowed`) para que los usuarios sepan claramente
        cuándo están en la primera o última página.

### 3. Panel lateral de investigación/detalles

-   Debe aparecer a la derecha de la tabla al hacer clic en una fila.
-   Incluye un botón de cierre "X" en la parte superior derecha para descartarlo.
-   **Sección de detalles:** usa labels pequeños apilados y bloques visualmente
    distintos (p. ej., info del merchant, monto, límites del cliente) con estilo
    `bg-zinc-800 border-zinc-700 rounded-md p-3`.
-   **Botones de acción:** abajo o al medio, coloca botones de acción primarios
    y secundarios (p. ej., "Aprobar" `bg-emerald-500`, "Rechazar" `bg-rose-500`
    con texto blanco). Actualiza el estado del padre para que la tabla refleje
    la acción al instante.

### 4. Tipografía y texto

-   Usa tags HTML semánticos y estilizado estandarizado.
-   **Encabezados:** `text-zinc-950 dark:text-zinc-50` con `tracking-tight`.
    `h1` es `text-4xl font-extrabold`, hasta `h6` que es `text-sm font-semibold
    uppercase tracking-wider text-zinc-500`.
-   **Cuerpo:** `text-zinc-500 dark:text-zinc-400`.
-   **Especialidad:** texto monoespaciado para IDs (`font-mono text-zinc-500
    bg-zinc-100 dark:bg-zinc-800 rounded px-1.5`).

### 5. Inputs y controles

-   **Botones:**
    -   Primario: `bg-[#059669] hover:bg-[#047857] text-[#000000] rounded-[8px]
        px-[16px] py-[8px] font-[500] transition-colors shadow-sm`.
    -   Secundario: `bg-[#ffffff] dark:bg-[#262626] hover:bg-[#f8fafc]
        dark:hover:bg-zinc-800 text-[#334155] dark:text-zinc-200 border
        border-zinc-200 dark:border-zinc-700 rounded-[8px] px-[16px] py-[8px]
        font-[500] transition-colors shadow-sm`.
    -   Destructivo: `bg-[#e11d48] hover:bg-[#be123c] text-[#ffffff]
        rounded-[8px] px-[16px] py-[8px] font-[500] transition-colors
        shadow-sm`.
    -   Ghost: `text-[#2563eb] dark:text-[#60a5fa] hover:bg-[#eff6ff]
        dark:hover:bg-[#1e3a8a] rounded-[8px] px-[16px] py-[8px] font-[500]
        transition-colors bg-transparent border border-transparent shadow-none`.
-   **Inputs:** `w-full bg-[#ffffff] dark:bg-[#09090b] border border-zinc-200
    dark:border-zinc-800 rounded-[8px] px-[12px] py-[8px] text-sm shadow-sm
    focus:outline-none focus:ring-2 focus:ring-blue-500/50 focus:border-blue-500
    text-zinc-950 dark:text-zinc-100 transition-all`. Provee un `label` con
    `text-xs font-medium text-zinc-500 mb-1.5`.
-   **Controles (checkboxes/radios/toggles):** estandariza el color de acento
    en `blue-500`.
-   **Controles segmentados:** usa un contenedor con forma de píldora
    `bg-zinc-100 dark:bg-[#0c0c0f] p-1 rounded-[8px]` con estados activos que
    tengan el estilo `bg-white dark:bg-zinc-800 text-[#2563eb]
    dark:text-[#60a5fa]` y sombra.

### 6. Navegación y overlays

-   **Tabs:** tabs con estilo de subrayado. Tab activo: `border-b-2
    border-[#3b82f6] text-[#2563eb] dark:text-[#60a5fa]`. Tab inactivo:
    `text-zinc-500 hover:text-[#334155] dark:hover:text-[#e2e8f0] transparent
    border-b-2 hover:border-[#3b82f6]`.
-   **Diálogos/modales:** overlay centrado con backdrop `bg-zinc-950/50
    backdrop-blur-sm`. Superficie del modal: `bg-[#ffffff] dark:bg-[#0c0c0f]
    rounded-[12px] border border-zinc-200 dark:border-zinc-800 shadow-xl
    w-full max-w-md p-6`.
-   **Menús dropdown:** menú flotante posicionado absoluto `bg-[#ffffff]
    dark:bg-[#0c0c0f] shadow-lg border border-zinc-200 dark:border-zinc-800
    rounded-[12px] py-1`.

### 7. Feedback y estado

-   **Alertas:** contenedores coloreados `bg-{color}-50 dark:bg-{color}-900/20 border
    border-{color}-200 dark:border-{color}-800/30 text-{color}-800
    dark:text-{color}-300 rounded-lg p-4 flex gap-3`. (Colores: emerald para
    éxito, rose para error, amber para advertencia, blue para info).
-   **Loaders:** loaders de skeleton con pulso `animate-pulse bg-zinc-200
    dark:bg-zinc-800 rounded`. Spinners SVG circulares para loading.

### 8. Visualización de datos (Apache ECharts)

-   NO intentes construir gráficos complejos desde cero con divs/Tailwind.
-   Usa `echarts` y `echarts-for-react`.
-   **Integración del tema base:** provee un objeto de configuración
    estandarizado: arrays `color`, `textStyle: { fontFamily: "Inter,
    sans-serif" }`, tooltips compatibles con dark mode `backgroundColor: "rgba(255,
    255, 255, 0.95)"`, `borderColor: "#e2e8f0"`, `textStyle: { color: "#0f172a" }`,
    y paddings de grilla sensatos.
-   **Estilizado del contenedor:** alojado dentro de `bg-white dark:bg-[#0c0c0f]
    rounded-xl border border-zinc-200 dark:border-zinc-800 shadow-sm`.
-   **Restricciones de layout:** los gráficos analíticos generalmente deben
    presentarse grandes para legibilidad, imponiendo un patrón de diseño estricto
    de **1 gráfico por fila** (`grid-cols-1`) cuando se colocan dentro de
    secciones de contenido principal o galerías. No los comprimas.
-   **Orientación al detalle:** los dashboards deben diseñarse para proveer
    insights en profundidad. Incluye múltiples gráficos relacionados, opciones de
    filtrado y capacidades de drill-down donde corresponda, para asegurar que el
    dashboard sea completo y no solo una vista de alto nivel. Ve más allá de los
    resultados superficiales para presentar insights significativos.

#### 9. Estado global y componentes ordenables (UI ordenable)

-   Para construir interfaces ordenables e interactivas (p. ej., swatches o
    listas reordenables), usa estrictamente `@dnd-kit/core` y `@dnd-kit/sortable`.
-   Usa un patrón de provider `Context` cuando el estado aplique a través de
    múltiples componentes hermanos (como personalizar los colores de los
    gráficos en toda la app y renderizar un componente de swatch ordenable
    interactivo con drag-and-drop).
-   Sigue la configuración estándar de DND Kit: envuelve en
    `<DndContext onDragEnd={...}>`, envuelve los ordenables en
    `<SortableContext items={items} strategy={...}>`, y vincula los elementos
    hijos con `useSortable({ id })`.
-   **Array de colores estándar para gráficos:** establece estas 10 variables
    estándar de nivel 500 de Tailwind al iniciar los conjuntos de gráficos:
    `["#3b82f6", "#10b981",
    "#f59e0b", "#ef4444", "#8b5cf6", "#ec4899", "#06b6d4", "#84cc16", "#f97316",
    "#6366f1"]`.
