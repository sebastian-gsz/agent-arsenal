# Referencia de limitaciones de UI de Canvas Apps

Referencia compacta de elementos web/de diseño que Canvas Apps no puede renderizar de forma nativa,
con la alternativa de Canvas Apps más cercana para cada uno.

La skill lee este archivo durante la Fase 3 (Análisis) en **todos los modos** y usa la
etiqueta `Handle:` de cada entrada para decidir cómo proceder:

- **auto** — Aplica la alternativa de Canvas Apps en silencio. Regístrala en una lista "Sustituciones aplicadas" que se muestra al final de la salida.
- **ask** — Presenta el problema al usuario con opciones numeradas. Espera su elección antes de continuar.
- **skip** — Informa al usuario de que el elemento se omitirá. No hace falta decisión del usuario.

**Reglas globales (aplican a todas las entradas):**
- Sin PCF. Esta skill no puede generar componentes de Power Apps Component Framework.
- Sin embed de PowerBI Tile. Requiere configuración adicional del entorno fuera del alcance de esta skill.
- Sin controles hechos a medida. Usa el control nativo de Canvas Apps más cercano tal cual.

Formato por entrada:
> **[Elemento de diseño]**
> Handle: auto | ask | skip
> Limitación: [por qué Canvas Apps no puede hacerlo de forma nativa]
> Alternativa: [solución nativa de Canvas Apps — varias opciones listadas para entradas `ask`]

---

## GRÁFICOS Y VISUALIZACIÓN DE DATOS

**Gráfico de donut / anillo**
Handle: ask
Limitación: Canvas Apps no tiene un control nativo de gráfico de donut o anillo.
Alternativa A: Dos controles Ellipse superpuestos con un arco relleno por fórmula (solo estático).
Alternativa B: Path de arco SVG inline vía Image@2.2.3 (más preciso, sigue estático).
Alternativa C: Omitir.

**Gráfico multieje / de doble eje**
Handle: ask
Limitación: Los controles Chart integrados soportan un único eje Y.
Alternativa A: Simplificar a un control Chart de líneas o columnas de un solo eje.
Alternativa B: Apilar dos controles Chart separados que compartan las etiquetas del eje X.
Alternativa C: Omitir.

**Gráfico de dispersión / burbujas**
Handle: ask
Limitación: No hay control nativo de scatter o bubble chart.
Alternativa A: Gallery con controles Image de puntos posicionados en un contenedor ManualLayout (complejo pero posible).
Alternativa B: Simplificar a un control Chart de barras o columnas.
Alternativa C: Omitir.

**Gráfico animado / de actualización en vivo**
Handle: ask
Limitación: Los controles Chart son renders estáticos — sin animación ni streaming en tiempo real integrados.
Alternativa A: Timer + UpdateContext para disparar un refresco de datos a intervalos; el Chart se re-renderiza en cada cambio de datos.
Alternativa B: Solo gráfico estático (sin refresco).
Alternativa C: Omitir.

**Gráfico de gauge / velocímetro**
Handle: ask
Limitación: No hay control nativo de gauge.
Alternativa A: Dos formas de arco SVG vía Image@2.2.3 (SVG inline, representación estática).
Alternativa B: Indicador de barra horizontal simplificado usando controles Rectangle.
Alternativa C: Omitir.

**Gráfico treemap / sunburst / funnel**
Handle: ask
Limitación: No disponibles como tipos de gráfico nativos.
Alternativa A: Control Chart de barras o columnas simplificado.
Alternativa B: Visualización personalizada basada en Gallery usando controles Rectangle dimensionados proporcionalmente.
Alternativa C: Omitir.

**Sparklines (mini-gráficos inline)**
Handle: ask
Limitación: No hay control de sparkline; los controles Chart son elementos independientes de tamaño completo.
Alternativa A: Controles Rectangle apilados dentro de una plantilla de Gallery, dimensionados proporcionalmente a los valores.
Alternativa B: Indicador de tendencia basado en texto (p. ej., "+12%" o una flecha SVG arriba/abajo).
Alternativa C: Omitir.

---

## TIPOGRAFÍA Y FUENTES

**Fuentes web personalizadas (Google Fonts, Adobe Fonts, @font-face)**
Handle: auto
Limitación: Canvas Apps no puede cargar fuentes de URLs externas.
Alternativa: Usa la fuente integrada disponible más cercana. Entre las opciones: Lato, Lato Black, Calibri, Arial, Segoe UI, Open Sans, Raleway, Patrick Hand, Dancing Script, Georgia y otras. Nunca referencies una URL en una propiedad de fuente.

**Pesos de fuente variable (p. ej., font-weight: 300, 400, 600, 700)**
Handle: auto
Limitación: Solo hay valores discretos del enum FontWeight: Thin, Lighter, Light, Normal, Semibold, Bold, Bolder.
Alternativa: Usa el valor del enum FontWeight disponible más cercano.

**Degradados de texto / contorno de texto**
Handle: auto
Limitación: Sin fill degradado ni stroke en el texto.
Alternativa: Usa el color sólido dominante del tono visible del degradado.

**Estilos inline mixtos en una misma label (p. ej., negrita en una sola palabra)**
Handle: auto
Limitación: Classic/Label aplica un único estilo a todo su texto. No hay formato inline a nivel de span.
Alternativa: Usa el control HtmlText@0.0.45 — renderiza markup HTML y soporta spans inline de negrita, cursiva y color dentro de un mismo control. NOTA: HtmlText es de solo visualización — nunca lo uses para elementos interactivos (enlaces, botones, clics).

**Hipervínculo / enlace clicable**
Handle: auto
Limitación: HtmlText renderiza etiquetas <a> visualmente, pero las interacciones programadas sobre HtmlText pueden causar problemas — es un control de solo visualización.
Alternativa: Usa un Classic/Button@2.2.0 (o Modern/Button) con el texto del enlace como su propiedad Text y OnSelect: =Launch("<url>"). Estilízalo como botón de solo texto (sin fill/borde) para que parezca un enlace visual.

**Sombra de texto**
Handle: ask
Limitación: No hay propiedad nativa de sombra de texto en ningún control.
Alternativa A: Simúlala con una Label duplicada colocada ligeramente desplazada detrás de la label visible en un contenedor ManualLayout, con un color levemente distinto.
Alternativa B: Omite la sombra y usa la label simple.

---

## LAYOUT Y SCROLLING

**Flex-wrap / grid de tiles con wrap**
Handle: ask
Limitación: Los contenedores AutoLayout no soportan flex-wrap — los hijos no pueden refluir a una nueva fila cuando el contenedor se llena.
Alternativa A: Contenedores AutoLayout horizontales anidados (uno por fila) — filas fijas.
Alternativa B: Gallery con la propiedad WrapCount para grids de cards o tiles — layout dinámico.

**Scroll horizontal en un GroupContainer**
Handle: auto
Limitación: LayoutOverflowX (scroll horizontal) no está soportado en GroupContainers.
Alternativa: Usa una Gallery horizontal, que hace scroll horizontal de forma nativa.

**CSS Grid / layout masonry**
Handle: ask
Limitación: No hay motor nativo de grid o masonry.
Alternativa A: Contenedores AutoLayout anidados para simular un grid de columnas fijas.
Alternativa B: Gallery con WrapCount para un layout de tiles repetitivo más simple.

**Cabecera sticky / fija mientras el cuerpo hace scroll**
Handle: auto
Limitación: No hay equivalente a CSS position:sticky.
Alternativa: Coloca la cabecera fuera del contenedor desplazable. El contenedor desplazable con LayoutOverflowY queda debajo de la cabecera en un AutoLayout Vertical padre — es el patrón estándar usado en las plantillas de esta skill.

**Z-index de superposición dentro de AutoLayout**
Handle: auto
Limitación: Los contenedores AutoLayout no pueden renderizar hijos superpuestos.
Alternativa: Cambia ese contenedor concreto a ManualLayout y usa posicionamiento X/Y. O usa un Classic/Button transparente como overlay de capa superior dentro de ManualLayout.

**Paneles flotantes con posicionamiento absoluto/fijo (p. ej., toast, tooltip, modal)**
Handle: auto
Limitación: Los controles no pueden posicionarse respecto al viewport. Todo está en el flujo del documento.
Alternativa: Usa un GroupContainer ManualLayout de nivel superior como padre más externo, coloca ahí el panel flotante en coordenadas X/Y fijas, y alterna su propiedad Visible con una variable.

---

## ANIMACIÓN Y TRANSICIONES

**Transiciones CSS (animaciones hover, fade-ins, slide-ins)**
Handle: skip
Limitación: La animación CSS no está soportada en Canvas Apps. Las animaciones son fundamentalmente incompatibles con el modelo de renderizado de Canvas Apps.
Alternativa: Ninguna. Acepta un diseño estático. Los cambios de estado interactivos (color hover, estado pulsado) se gestionan vía propiedades de control, no con animación.

**SVG animado (SMIL o animación CSS dentro del SVG)**
Handle: ask
Limitación: El soporte de animación SVG en los controles Image de Canvas Apps es inconsistente — las animaciones SMIL pueden ejecutarse en algunos clientes y en otros no.
Alternativa A: Usa un SVG inline animado vía Image@2.2.3 para casos de uso simples (p. ej., spinner de carga, punto pulsante). Los resultados pueden variar entre dispositivos. Aceptable para animaciones decorativas no críticas.
Alternativa B: Usa un icono SVG estático.
Alternativa C: Omitir.

**Animaciones Lottie / ilustraciones animadas**
Handle: skip
Limitación: Sin reproductor Lottie nativo. No puede generarse como YAML plano de Canvas Apps.
Alternativa: Ninguna disponible en esta skill. El elemento se omitirá. Usa una ilustración SVG estática como reemplazo si hace falta.

**Animaciones de transición de pantalla / página**
Handle: auto
Limitación: Solo están disponibles las transiciones integradas en Navigate().
Alternativa: Usa Navigate() con el enum Transition disponible más cercano: None, Fade, Cover, CoverRight, UnCover, UnCoverRight, Slide, SlideRight.

**Pantallas de carga con skeleton**
Handle: auto
Limitación: No hay componente nativo de skeleton shimmer.
Alternativa: Usa un Rectangle con un fill neutro (RGBA(229,231,235,1)) como placeholder, Visible=true mientras cargan los datos, Visible=false cuando el control ligado a datos esté listo.

---

## ESTADOS INTERACTIVOS

**Estado hover en GroupContainer**
Handle: auto
Limitación: Los GroupContainers no tienen propiedad HoverFill.
Alternativa: Superpón un Classic/Button transparente (con el mismo Width/Height) encima del contenedor en ManualLayout. Asigna el HoverFill al botón. El OnSelect del botón dirige cualquier acción de clic.

**Anillo de focus / focus de teclado visible en GroupContainer**
Handle: auto
Limitación: Los GroupContainers no son enfocables de forma nativa.
Alternativa: El overlay de Classic/Button transparente usado para el hover también gestiona el focus de teclado y la interacción.

**Menús contextuales de clic derecho / pulsación larga**
Handle: ask
Limitación: No hay evento nativo de clic derecho ni de pulsación larga. Canvas Apps solo soporta tap/click.
Alternativa A: Un panel de menú contextual visible (GroupContainer + items) alternado por una variable, revelado con el tap de un Button estándar. El modelo de interacción cambia de clic derecho a tap.
Alternativa B: Omitir por completo el menú contextual.

**Tooltip en GroupContainer o control no interactivo**
Handle: skip
Limitación: El tooltip solo está disponible en controles que soportan la propiedad de forma nativa. Los GroupContainers y la mayoría de controles no interactivos, no. No se construirá ningún workaround.
Alternativa: Ninguna. Si el control objetivo no soporta Tooltip de forma nativa, el tooltip se omitirá del YAML generado.

**Drag and drop entre contenedores**
Handle: skip
Limitación: No hay drag-and-drop nativo entre controles o listas en Canvas Apps.
Alternativa: Ninguna generable. Sustitúyelo por botones de acción (mover arriba/abajo, transferir entre galerías) si la interacción es crítica — pero esto requiere un diseño explícito del usuario.

---

## MULTIMEDIA

**Vídeo en autoplay sin controles visibles**
Handle: ask
Limitación: El control Video siempre se renderiza con los controles de reproducción nativos del navegador. No hay opción para ocultarlos ni para arrancar en silencio.
Alternativa A: Usa el control Video y acepta la UI del player por defecto.
Alternativa B: Omite el elemento de vídeo.

**Control de audio con UI de player personalizada**
Handle: ask
Limitación: El control Audio renderiza el elemento de audio nativo del navegador. No puede estilizarse.
Alternativa A: Acepta el player de audio nativo tal cual.
Alternativa B: Omite el elemento de audio.

**Mapa interactivo embebido (pan, zoom, markers)**
Handle: ask
Limitación: No hay control nativo de mapa interactivo en las Canvas Apps estándar. El control Geospatial Map solo está disponible si el entorno tiene el add-on habilitado.
Alternativa A: Usa el control Geospatial Map de Power Apps (requiere habilitar el add-on en el entorno).
Alternativa B: Usa una imagen de mapa estática embebida vía un control Image.
Alternativa C: Omitir el mapa.

**Contenido 360 / VR / WebGL**
Handle: skip
Limitación: Sin soporte de contenido 3D, WebGL ni VR en Canvas Apps.
Alternativa: Ninguna. Omítelo. Sustitúyelo por una imagen estática plana si hace falta un placeholder visual.

---

## ICONOS Y GRÁFICOS

**Librerías de icon fonts (Font Awesome, Material Icons, Bootstrap Icons)**
Handle: auto
Limitación: Canvas Apps no puede cargar icon fonts externos.
Alternativa: Data URIs de SVG inline vía Image@2.2.3. Es el enfoque estándar usado en toda esta skill — Classic/Icon nunca se usa.

**Estilizado de SVG (degradados, fills multicolor)**
Handle: auto
Limitación: No es una limitación dura. Como el SVG es texto HTML/XML inline, tanto los degradados (linearGradient, radialGradient) como los paths multicolor son posibles vía Image@2.2.3.
Alternativa: Por defecto usa SVGs de color sólido plano por simplicidad y fiabilidad entre dispositivos. Usa degradados o fills multicolor cuando el mockup los muestre explícitamente o el usuario lo pida.

**Imagen de fondo con CSS object-fit / cover / contain**
Handle: auto
Limitación: Los controles Image tienen Scale (fit, fill, stretch, tile, center) pero el comportamiento difiere del object-fit de CSS. Las imágenes de fondo full-bleed detrás de contenido superpuesto requieren ManualLayout.
Alternativa: Control Image con ImagePosition=ImagePosition.Fill para estilo cover, colocado detrás de otros controles en ManualLayout con Width/Height coincidentes.

---

## FORMULARIOS Y CONTROLES DE ENTRADA

**Selector de fecha/hora con estilizado de calendario personalizado**
Handle: auto
Limitación: El popup de calendario del control DatePicker usa la apariencia por defecto del sistema/navegador y no puede restilizarse.
Alternativa: Usa Classic/DatePicker tal cual. No se construirá un selector de fecha personalizado.

**ComboBox multiselección que muestra las selecciones como chips**
Handle: auto
Limitación: Classic/ComboBox con SelectMultiple=true muestra los items seleccionados como texto separado por comas, no como chips de etiqueta eliminables.
Alternativa: Usa Classic/ComboBox con SelectMultiple=true y acepta la visualización por defecto.

**Subida de archivos por drag-and-drop**
Handle: auto
Limitación: No hay objetivo de subida por drag-and-drop en Canvas Apps.
Alternativa: Usa el control Attachment para seleccionar archivos vía el file picker del SO.

**Editor de texto enriquecido con toolbar personalizada**
Handle: auto
Limitación: Classic/RichTextEditor tiene una toolbar fija — negrita, cursiva, listas, enlaces. La toolbar no puede personalizarse.
Alternativa: Usa Classic/RichTextEditor y acepta su toolbar por defecto.

**Slider con track y thumb con estilos personalizados**
Handle: ask
Limitación: El control Slider solo expone RailFill, ValueFill, HandleFill y HandleActiveFill. La forma del track y del thumb es fija.
Alternativa A: Aplica overrides de color (RailFill/ValueFill/HandleFill) y acepta la forma por defecto.
Alternativa B: Omite el slider y usa otra entrada (p. ej., TextInput con validación numérica).

**Input con icono inline inicial/final dentro del límite del campo**
Handle: auto
Limitación: Los controles TextInput no tienen slot integrado para iconos embebidos dentro del borde del input.
Alternativa: GroupContainer ManualLayout como wrapper: coloca dentro el TextInput y un Image (icono) en posiciones X/Y superpuestas, asigna al TextInput un PaddingLeft que contemple la anchura del icono, y usa el estilizado de bordes del wrapper para simular un input unificado.

**Autocomplete / type-ahead más allá del ComboBox**
Handle: auto
Limitación: Classic/ComboBox con IsSearchable=true da un filtrado type-ahead básico. Autocompletado más avanzado (formato personalizado de resultados, búsqueda asíncrona, resultados agrupados) no está soportado de forma nativa.
Alternativa: Usa Classic/ComboBox con IsSearchable=true. Es la mejor opción nativa disponible. El autocompletado avanzado no puede generarse con esta skill.

---

## WORKAROUNDS Y PATRONES CONOCIDOS

**Simular un resaltado hover de card**
Usa un overlay de Classic/Button transparente (con la Width/Height de la card, colocado encima en ManualLayout) con HoverFill=RGBA(0,0,0,0.04). El propio GroupContainer de la card no puede recibir eventos de hover.

**Elementos superpuestos (diálogo modal, badge flotante, panel overlay)**
AutoLayout no soporta superposición. Usa un GroupContainer ManualLayout como contenedor más externo y coordenadas X/Y para todos los hijos superpuestos. Para modals a pantalla completa, usa un contenedor ManualLayout de nivel superior en X=0, Y=0, Width=Parent.Width, Height=Parent.Height.

**Sección colapsable / acordeón**
Usa una variable (p. ej., varSectionOpen) alternada por el OnSelect de un Button. La Visible del contenedor de la sección (o su Height asignada a 0 vs. su altura natural) responde a la variable. En AutoLayout, Visible=false en un hijo lo elimina del flujo del layout.

**Cuerpo desplazable con cabecera y pie fijos**
Coloca cabecera y pie como hijos directos de un padre AutoLayout Vertical con FillPortions=0. El cuerpo desplazable es un hijo GroupContainer con FillPortions=1 y LayoutOverflowY=LayoutOverflow.Scroll.

**Campo condicional que colapsa sin gap de layout**
En AutoLayout, Visible=false en un control lo oculta pero sigue ocupando espacio. Para colapsar de verdad: envuelve el control en un GroupContainer y asigna Visible=false al contenedor, lo que lo elimina del layout. Alternativamente, asigna Height=If(condition, naturalHeight, 0) directamente al control.
