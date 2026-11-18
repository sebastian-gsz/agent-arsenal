---
name: canvas-apps-ui-gen
description: Genera YAML listo para pegar de Power Apps Canvas Apps. Invócala cuando el usuario quiera replicar un mockup de UI, mejorar una pantalla existente de una Canvas app o crear una pantalla nueva a partir de una descripción de texto. También invócala cuando el usuario pida "mejorar", "rediseñar" o "generar YAML" para una pantalla de una Canvas app.
argument-hint: path/to/screenshot.png o descripción
allowed-tools: Read, Glob, Write, WebFetch, Agent
---

Eres un desarrollador y orquestador experto en Power Apps Canvas Apps. Cuando se invoque esta skill, sigue el proceso de abajo exactamente.

---

## FASE 1 — DETECCIÓN DE MODO

Al invocarse, determina si hay una imagen disponible:

**Imagen disponible** (`$ARGUMENTS` contiene una ruta de archivo, O hay una imagen pegada/arrastrada en la conversación):
- Carga la imagen ahora (sigue la Fase 2 exactamente)
- Procede directamente a la Fase 3 — **NO** hagas la pregunta de modo
- El modo (Replicar vs. Mejorar) se determinará como parte de la Fase 3

**Sin imagen disponible** (`$ARGUMENTS` está vacío o contiene solo texto que no es una ruta, y no hay imagen pegada):
- Pregunta al usuario exactamente esto y espera su respuesta:

"¿En qué modo te gustaría trabajar?

**1. Replicar** — Tienes un mockup, un wireframe o una captura de diseño. Lo replicaré en YAML de Power Apps, manteniendo fielmente el layout, los colores y las proporciones.

**2. Mejora** — Tienes una captura de una pantalla existente de Power Apps que quieres modernizar, rediseñar o pulir.

**3. Construir desde cero** — No hay imagen. Describe lo que quieres y generaré el YAML a partir de tu descripción."

- Modo 1 o 2 → pide la imagen (Fase 2) y luego procede a la Fase 3
- Modo 3 → salta la Fase 2 y procede directamente a la Fase 3 en modo de solo texto

---

## FASE 2 — CARGAR LA IMAGEN

*(Omite esta fase por completo si se seleccionó el Modo 3 en la Fase 1.)*

Determina cómo se proporcionó la imagen:

**Método A — Ruta de archivo en `$ARGUMENTS`:**
Si `$ARGUMENTS` no está vacío y parece una ruta de archivo (contiene `/` o `\`, o termina en `.png`, `.jpg`, `.jpeg` o `.webp`), usa la herramienta `Read` para cargarla. Trata toda la cadena de `$ARGUMENTS` como la ruta — NO la dividas por espacios (las rutas pueden contener espacios, como `C:\Users\My Documents\screen.png`).

Si no se puede leer el archivo, dile al usuario: "No pude cargar el archivo en `[path]`. Verifica que la ruta sea correcta y que el archivo sea PNG, JPG o WEBP — o pega la imagen directamente en el chat." Luego detente.

**Método B — Imagen en la conversación:**
Si `$ARGUMENTS` está vacío (o contiene solo texto que no es una ruta), busca una imagen pegada o arrastrada en la conversación actual. Si no se encuentra ninguna imagen, pregunta: "Pega tu imagen en el chat (una captura de una Canvas app, un mockup de UI, un wireframe o cualquier diseño que quieras convertir a YAML), o proporciona una ruta de archivo como argumento: `/canvas-apps-ui-gen C:\path\to\image.png`"

**Si se proporcionan tanto una ruta de archivo como una imagen:** prioriza el argumento de la ruta de archivo — es más explícito.

---

## FASE 3 — ANÁLISIS Y PREGUNTAS COMBINADOS

Esta fase produce la descripción estructural **y** hace todas las preguntas previas a la generación en una **única respuesta**. No la dividas en dos mensajes — el análisis y las preguntas deben aparecer juntos para que el usuario pueda responder todo de una vez.

### Paso A — Descripción estructural

Imprime una descripción que cubra:
1. **Secciones del layout**: cada región distintiva de la UI (p. ej., "Una fila horizontal de 4 tarjetas de métricas KPI en la parte superior", "Una data table que cubre el 65 % inferior de la pantalla")
2. **Tipos de componentes identificados**: tarjetas, table/grid, campos de form, barra de navegación, encabezado, botones, insignias de estado, gráficos, etiquetas de texto, etc.
3. **Esquema de colores aproximado**: fondo oscuro/claro, colores de acento visibles
4. **Proporciones aproximadas de las secciones**: qué ocupa la mayor parte de la pantalla

Formato de ejemplo:
> "Puedo ver lo siguiente:
> - Una barra lateral izquierda (~130px de ancho) con un logo, una galería de navegación vertical y una fila de perfil en la parte inferior
> - Una barra superior con título de página y breadcrumb (~64px de alto)
> - Una barra de pestañas horizontal con 6 pestañas
> - Una tarjeta de form centrada (~440px de ancho) con campos de título de producto, categoría, marca, variación y botón de acción
>
> Esquema de colores: fondo claro (gris cálido), barra lateral y tarjetas blancas, acento naranja para los estados activos."

*(En Modo 3 / solo texto: imprime tu comprensión de la descripción que proporcionó el usuario en lugar de observaciones visuales.)*

### Paso B — Verificación de compatibilidad

Lee `reference/canvas-apps-limitations.md`. Contrasta tu descripción estructural (o la descripción en texto del usuario en el Modo 3) con cada entrada. Agrupa cualquier coincidencia por su etiqueta `Handle:`:

**Handle: auto** — Aplica la alternativa de Canvas Apps silenciosamente. Registra cada sustitución en una lista interna (se usará más adelante en el Paso 4 de la Fase 4). No las menciones aquí.

**Handle: ask** — Incluye este bloque en la respuesta actual (el usuario responderá junto con las demás preguntas):

> **Compatibilidad con Canvas Apps — necesitamos tu opinión:**
> Estos elementos no se pueden implementar de forma nativa en Canvas Apps.
> Elige una alternativa para cada uno:
>
> 1. [Elemento detectado] — [razón en una línea]
>    (a) [Alternativa A]
>    (b) [Alternativa B]
>    (c) Omitir este elemento
>
> *(repetir por cada coincidencia con `ask`)*

**Handle: skip** — Incluye este bloque y continúa (no se necesita decisión del usuario):

> **Compatibilidad con Canvas Apps — elementos omitidos:**
> Lo siguiente no se puede generar como YAML de Canvas Apps y se excluirá de la salida:
> - [Elemento]: [razón breve]

Si no se detectan limitaciones, omite por completo todos los bloques de compatibilidad.

### Paso C — Preguntas previas a la generación

Inmediatamente después de la descripción y de cualquier bloque de compatibilidad, en la **misma respuesta**, muestra el bloque de preguntas.

---

**Si se proporcionó una imagen (modo aún sin confirmar):**

> **Listo para generar — preguntas rápidas:**
>
> **1. ¿Qué quieres hacer con este diseño?**
> (a) **Replicar** — recrear este diseño fielmente como YAML de PA
> (b) **Mejorar** — modernizarlo y pulirlo (estandarizar espaciado/tipografía, actualizar controles, añadir estados hover/focus adecuados)
>
> **2. ¿Dónde pegarás este YAML?**
> (a) En una pantalla existente — generaré los controles sin wrapper de Screen
> (b) Como pantalla nueva — generaré un bloque `Screens:` con el nombre de la pantalla y todos los children
> (c) Solo una sección/componente específico de una pantalla
>
> **3. ¿Controles Classic o Modern (Fluent)?**
> - **Classic** — estilo base mínimo, totalmente personalizable; cada color, borde, fuente y estado de interacción es ajustable individualmente
> - **Modern / Fluent** — diseño Microsoft Fluent 2 integrado; apariencia pulida, animaciones fluidas y valores predeterminados accesibles
>
> *Extraeré la paleta de colores y usaré los nombres de campos visibles directamente desde la imagen. Para diseño responsivo, la respuesta por defecto es No — menciónalo en tu respuesta si necesitas adaptación a móvil/tableta.*
>
> **Opcional — cambios respecto a la imagen:** ¿Algo que quieras que difiera de lo que se muestra? (p. ej., "añadir un textarea de Notas en la parte inferior", "reemplazar el campo Marca por un dropdown de Proveedor", "quitar la barra lateral"). Déjalo en blanco para proceder exactamente como se muestra.

Espera la única respuesta del usuario y luego procede a la Fase 4.

---

**Si es Modo 3 (sin imagen — solo texto):**

> **Listo para generar — algunas preguntas rápidas:**
>
> **1. ¿Dónde pegarás este YAML?**
> (a) En una pantalla existente — sin wrapper de Screen
> (b) Como pantalla nueva — bloque `Screens:` completo
> (c) Solo una sección/componente específico
>
> **2. ¿Controles Classic o Modern (Fluent)?**
> - **Classic** — totalmente personalizable; cada propiedad ajustable individualmente
> - **Modern / Fluent** — diseño Fluent 2 integrado; valores predeterminados pulidos
>
> **3. ¿Qué paleta de colores debo usar?**
> - Tema oscuro con acentos azules (fondo RGBA(35,36,47,1), acento RGBA(0,142,210,1))
> - Tema claro/blanco
> - Describe la tuya (p. ej., "azul marino y dorado", "azul corporativo y blanco")
>
> **4. ¿Cuál es el patrón general del layout?**
> (p. ej., barra lateral + contenido principal, columna única a todo lo ancho, form tipo wizard/pasos, panel dividido, dashboard con tarjetas)
>
> **5. ¿Cuál es el propósito principal de esta pantalla?**
> (p. ej., data table/lista, form de captura de datos, dashboard con KPIs, vista de detalle de registro, página de configuración)

Espera la única respuesta del usuario y luego procede a la Fase 4.

---

## FASE 4 — GENERACIÓN DE YAML MULTI-AGENTE

### Paso 0: Resolver rutas

Antes de hacer cualquier otra cosa, resuelve dos rutas raíz que se usan a lo largo de la Fase 4.

**A. Directorio de la skill (`SKILL_DIR`)**
Determina la ruta absoluta del directorio de esta skill (la carpeta que contiene este archivo SKILL.md).
Usa `SKILL_DIR` solo para leer los assets del plugin: `agents/`, `reference/` y `templates/`. Nunca escribas aquí archivos de salida.

**B. Directorio de salida (`OUTPUT_DIR`)**
Ejecuta este comando de Bash para obtener el directorio de trabajo del usuario:

```bash
pwd
```

Guarda el resultado como `USER_CWD`. Luego construye:
`OUTPUT_DIR = {USER_CWD}/canvas-apps-output`

Créalo si no existe:

```bash
mkdir -p "{USER_CWD}/canvas-apps-output"
```

**Caso especial:** si `USER_CWD` contiene `.claude/plugins`, advierte al usuario:
> "Nota: tu terminal parece estar dentro del directorio de plugins de Claude. La salida se guardará en `{OUTPUT_DIR}`. Navega primero a la carpeta de tu proyecto si tu intención era otra."

Continúa de todas formas — no te detengas.

Usa `SKILL_DIR` para todas las operaciones de lectura sobre los assets del plugin. Usa `OUTPUT_DIR` para todas las operaciones de escritura, relectura y borrado sobre los archivos generados a lo largo de la Fase 4.

### Paso 1: Generar esqueleto + spec de diseño

Con base en el análisis de la Fase 3 y las respuestas del usuario, produce dos artefactos y escríbelos en archivos temporales.

**Antes de escribir, resuelve el modo a partir de la respuesta del usuario:**
- El usuario respondió (a) Replicar → procede en modo de replicación
- El usuario respondió (b) Mejorar → procede en modo de mejora
- Invocación en Modo 3 → procede en modo de solo texto

**Si el usuario especificó modificaciones (campo de cambios opcionales):** incorpóralas tanto al esqueleto como a la spec de diseño antes de escribir. Añade una sección `MODIFICATIONS` a la spec de diseño que liste cada cambio solicitado. Construye el esqueleto reflejando esas modificaciones — no el mockup original.

#### Artefacto 1: Esqueleto estructural → escríbelo en `{OUTPUT_DIR}/temp-skeleton.md` usando directamente la herramienta Write (nunca Python ni Bash)

Un árbol de texto indentado y compacto que muestre nombres de controles, tipos, jerarquía y dirección general del layout. Usa este formato exacto:

```
Screen: [ScreenName]
Paste target: [a / b / c]

  screenRoot [GroupContainer, AutoLayout, fills screen]
  ├── sidebarContainer [GroupContainer, Vertical, fixed-width]
  │   ├── logoArea [GroupContainer, AutoLayout]
  │   │   ├── logoIcon [Image, icon]
  │   │   └── logoText [Label]
  │   ├── navGallery [Gallery, Vertical, items-cols: NavLabel|NavIcon|NavItemID, active-var: CurrentNavID, active-col: NavItemID]
  │   │   └── navItemRow [GroupContainer, ManualLayout]
  │   │       ├── navItemIcon [Image, icon, gallery-child]
  │   │       ├── navItemLabel [Label, gallery-child]
  │   │       └── navItemOverlay [Classic/Button, transparent overlay]
  │   └── profileRow [GroupContainer, AutoLayout]
  └── mainContent [GroupContainer, Vertical, fills remaining]
      ├── topBar [GroupContainer, Vertical]
      │   ├── pageTitle [Label]
      │   └── breadcrumb [Label]
      └── scrollableBody [GroupContainer, Vertical, scrollable]
          └── formCard [GroupContainer, Vertical, centered, card]
              ├── inputTitle [Classic/TextInput]
              └── actionRow [GroupContainer, AutoLayout]
                  ├── btnCancel [Classic/Button]
                  └── btnSave [Classic/Button]
```

Incluye TODOS los controles. Marca los roles especiales entre corchetes: `transparent overlay`, `icon`, `gallery-child`, `scrollable`, `centered`, `card`.

**Contrato de datos de la gallery (obligatorio para cada control Gallery):** además de las etiquetas de rol, cada línea Gallery debe incluir tres anotaciones extra que todos los agentes especialistas leerán para mantenerse coordinados:

- `items-cols: A|B|C` — los nombres exactos de columnas que el controls-agent debe usar en la `Table()`. Deriva los nombres del propósito semántico de la galería:
  - Galería de navegación/barra lateral → `NavLabel|NavIcon|NavItemID`
  - Galería de pestañas → `TabLabel|TabID`
  - Galería de filas de data table → nombres de columna que coincidan con los campos visibles en el diseño (p. ej., `OrderName|Status|Amount|OrderID`)
  - Galería de lista simple → `ItemLabel|ItemID`
- `active-var: CurrentX` — el nombre de la variable global que registra el elemento seleccionado. Usa siempre el prefijo `Current` (nunca `var`), seguido de un sustantivo semántico: `CurrentNavID`, `CurrentTabID`, `CurrentOrderID`. Esta variable se inicializa en `OnVisible` y se establece en el `OnSelect` del botón overlay.
- `active-col: X` — la columna de `items-cols` que se compara contra `active-var` para determinar el estado activo/seleccionado. Siempre es la columna de ID/clave.

Ejemplo: `navGallery [Gallery, Vertical, items-cols: NavLabel|NavIcon|NavItemID, active-var: CurrentNavID, active-col: NavItemID]`

#### Artefacto 2: Spec de diseño → escríbela en `{OUTPUT_DIR}/temp-design-spec.md` usando directamente la herramienta Write (nunca Python ni Bash)

Lee ahora `reference/design-spec-formats.md`. Usa la sección que corresponda al modo resuelto:
- Modo de replicación → sección **## Replicate Mode**
- Modo de mejora → sección **## Improvement Mode**
- Modo de solo texto → sección **## Text-Only Mode**

**Para el modo de mejora y el modo de solo texto — sintetiza primero el bloque DESIGN TOKENS, antes de completar cualquier otra sección:**

Usa las siguientes reglas para completar cada campo. Todos los agentes especialistas tratan este bloque como la autoridad principal para sus dominios respectivos.

- **Palette**: define exactamente 7 valores RGBA. Derívalos del propósito descrito por el usuario o del contexto de la pantalla (p. ej., "portal de RR. HH." → grises fríos + azul; "app de servicio en campo" → tonos tierra + naranja; "dashboard de analítica" → oscuro + turquesa). Background y Surface deben diferir en al menos 4 puntos de luminosidad. Deja Accent-Secondary vacío si solo se necesita un acento. Cada fill en la salida debe usar uno de estos 7 valores o una versión transparente — no se inventan colores independientes.

- **Typography**: elige UNA familia tipográfica para toda la pantalla. No mezcles fuentes. Escala: Heading debe ser al menos 2 unidades de tamaño de PA mayor que Body; Body al menos 1 unidad mayor que Caption. Predeterminado (controles Classic, nada que sugiera lo contrario): Heading=14, Body=11, Caption=9. Para controles Modern, aplica la conversión de tamaños Classic→Modern de `reference/sizing-reference.md`.

- **Density**: infiere del tipo de layout:
  - Data tables / listas → compacto: LayoutGap=8, PaddingH=12, PaddingV=8
  - Forms / vistas de detalle de registro → equilibrado: LayoutGap=12, PaddingH=16, PaddingV=12
  - Dashboards / landing pages → generoso: LayoutGap=16, PaddingH=20, PaddingV=16

- **Radius**: elige un valor para los contenedores (0=recto, 4=ligeramente redondeado, 6=estándar, 8=redondeado, 12=suave). Los controles interactivos reciben el mismo valor o la mitad — nunca más. Predeterminado: Container=6, Interactive=4.

- **State Colors**: todos se derivan de Accent-Primary:
  - HoverFill: RGBA(accent_r, accent_g, accent_b, 0.08)
  - PressedFill: RGBA(accent_r, accent_g, accent_b, 0.15)
  - FocusedBorder: Accent-Primary a opacidad total
  - DisabledFill: cercano al fondo (contraste muy bajo)
  - DisabledColor: Text-Secondary a opacidad 0.38

Completa el formato elegido con los valores extraídos de la captura / de las respuestas del usuario. Escribe el resultado en `{OUTPUT_DIR}/temp-design-spec.md`.

Después de escribir ambos archivos, procede de inmediato al Paso 2 — **no** vuelvas a leer `reference/controls-index.md` aquí (el controls-agent lo lee directamente).

---

### Paso 2: Lanzar agentes especialistas en paralelo

Lee los tres archivos de instrucciones de agentes del subdirectorio `agents/` de esta skill:
- `agents/layout-sizing-agent.md`
- `agents/controls-agent.md`
- `agents/styling-agent.md`

Lanza los tres agentes **simultáneamente en un solo mensaje** usando la herramienta Agent. Cada agente recibe un prompt construido a partir del contenido de su archivo de instrucciones más estos datos específicos:

**Para cada agente, pasa:**
- El contenido completo de su archivo de instrucciones (como descripción de la tarea)
- La ruta absoluta de `{OUTPUT_DIR}/temp-skeleton.md`
- La ruta absoluta de `{OUTPUT_DIR}/temp-design-spec.md`
- `SKILL_DIR` (la ruta absoluta del directorio de la skill, para leer los documentos de referencia en `reference/`)
- La preferencia de estilo de controles (classic / modern) de la respuesta de la Fase 3 del usuario — pásasela explícitamente al controls-agent para que seleccione las variantes correctas
- La ruta absoluta de su archivo de salida:
  - Layout+Sizing → `{OUTPUT_DIR}/temp-layout-annotations.yaml`
  - Controls → `{OUTPUT_DIR}/temp-controls-annotations.yaml`
  - Styling → `{OUTPUT_DIR}/temp-styling-annotations.yaml`

Espera a que los tres agentes terminen antes de proceder al Paso 3.

---

### Paso 3: Ensamblado + QA

Lee `agents/assembly-agent.md`. Lanza el agente de ensamblado usando la herramienta Agent con un prompt que incluya:
- El contenido completo de `agents/assembly-agent.md`
- Rutas absolutas de: `{OUTPUT_DIR}/temp-skeleton.md`, `{OUTPUT_DIR}/temp-layout-annotations.yaml`, `{OUTPUT_DIR}/temp-controls-annotations.yaml`, `{OUTPUT_DIR}/temp-styling-annotations.yaml`
- Ruta absoluta de `{OUTPUT_DIR}/temp-design-spec.md` (necesaria para las verificaciones de fidelidad de QA)
- `SKILL_DIR` (la ruta absoluta del directorio de la skill, para leer `reference/controls-reference.md` durante el QA)
- El destino de pegado (a, b o c) y el nombre de la pantalla
- El nombre del archivo de salida: determina primero un nombre único usando `Glob` sobre `{OUTPUT_DIR}/[ScreenName]-*-generated.yaml`. Si existe `{OUTPUT_DIR}/[ScreenName]-1-generated.yaml`, usa `-2-`, y así sucesivamente.
- Instrucciones de data binding (en línea para el destino de pegado b; bloque separado para a y c)
- Flag de diseño responsivo (si el usuario mencionó responsivo en su respuesta de la Fase 3) con el nombre de la pantalla para las fórmulas `ScreenName.Size`

El agente de ensamblado se autovalida y autocorrige todos los problemas de QA antes de escribir el archivo de salida. Espera a que termine y confirma que el archivo de salida fue escrito.

---

### Paso 4: Limpieza y salida

Elimina los archivos temporales usando la herramienta Bash (nunca Python):
- `{OUTPUT_DIR}/temp-skeleton.md`
- `{OUTPUT_DIR}/temp-design-spec.md`
- `{OUTPUT_DIR}/temp-layout-annotations.yaml`
- `{OUTPUT_DIR}/temp-controls-annotations.yaml`
- `{OUTPUT_DIR}/temp-styling-annotations.yaml`

Luego muestra al usuario:

**1. Enlace destacado al archivo (colócalo PRIMERO):**
> **Tu YAML está listo:** tu archivo se ha guardado en `{OUTPUT_DIR}/[ScreenName-N-generated.yaml]`. Ábrelo en tu editor, presiona **Ctrl+A** y luego **Ctrl+C** para copiar todo, y pégalo en Power Apps Studio.

**2. Instrucción de pegado según el destino de pegado:**
- **(a) o (c)**: "En PA Studio: abre la vista de árbol, haz clic derecho en la pantalla o contenedor destino y selecciona **Paste code**."
- **(b) pantalla nueva**: "En PA Studio: haz clic derecho en el panel de lista de pantallas y elige **Paste code**. Si no está disponible, crea una pantalla en blanco (Insert → New Screen → Blank), haz clic derecho sobre ella en la vista de árbol, selecciona **Paste code** y pega solo el contenido bajo `Children:`. Luego copia la fórmula de `OnVisible` en la propiedad `OnVisible` de la pantalla."

**3. Nota sobre el tamaño del canvas:**
"Está dimensionado para un canvas de tableta (1366px de ancho). Si tu app apunta a layout de teléfono, reduce los valores de `LayoutMinWidth` y establece los anchos de los contenedores a `=Parent.Width`."
(Se pidió responsivo: "Este YAML usa `[ScreenName].Size` para la responsividad. Asegúrate de que tu app sea de tipo de canvas **Tablet** y, en **Settings → Display**, desactiva **Scale to fit** y **Lock Orientation**.")

**4. Para los destinos de pegado (a)/(c) — bloque de data binding:**
Si el mensaje de finalización del agente de ensamblado contiene un campo `ONVISIBLE_BLOCK`, extrae esa fórmula y muéstrala aquí en el chat. NO la leas del archivo de salida — el archivo contiene solo YAML. Formátala así:

> **Pega también esto en la propiedad `OnVisible` de tu pantalla:**
> ```
> [formula from ONVISIBLE_BLOCK]
> ```

**5. Si es modo de mejora — resumen de cambios:**
Una lista breve de viñetas con las mejoras clave realizadas (paleta de colores aplicada, controles actualizados, espaciado estandarizado, estados añadidos, etc.).

**6. Visualización del YAML en línea (con límite de tamaño):**
Cuenta las líneas del archivo de salida. Si tiene **400 líneas o menos**, muestra el YAML completo en un bloque de código cercado `yaml`. Si tiene **más de 400 líneas**, muestra: "El YAML tiene [N] líneas — es demasiado grande para mostrarlo en línea. Abre el archivo de arriba y presiona **Ctrl+A** → **Ctrl+C** para copiarlo."

**7. Advertencias de QA (si las hay):**
Si el agente de ensamblado reportó advertencias de versión PA2105 u otras preocupaciones no bloqueantes, lístalas aquí:

> **Advertencias de QA (no bloqueantes):**
> - [texto de la advertencia]

Omite este bloque por completo si el agente de ensamblado no reportó advertencias.

**8. Autosustituciones aplicadas (si las hay):**
Si se encontraron limitaciones con `Handle: auto` durante la Fase 3 y se sustituyeron silenciosamente, lístalas aquí:

> **Sustituciones aplicadas automáticamente:**
> - [Elemento original] → [alternativa de Canvas Apps usada]

Omite este bloque por completo si no ocurrieron sustituciones `auto`.

**En refinamientos posteriores:** si el usuario pide cambiar algo, aplica los cambios directamente sobre el mismo archivo de salida usando la herramienta Edit. En seguimientos de replicación, cambia SOLO lo que se pidió — no ajustes otras partes.

---

## PLANTILLAS DE CRECIMIENTO AUTOMÁTICO

Después de generar YAML para un tipo de componente que no tiene una plantilla correspondiente en `templates/`, ofrece:
"Generé [tipo de componente] sin una plantilla de referencia. ¿Quieres que guarde esto como una plantilla nueva para que las generaciones futuras puedan usarla como referencia?"

Si el usuario está interesado, sigue el proceso de staging descrito en **PROTECTED FILE CHANGES** más abajo.

---

## CAMBIOS DE ARCHIVOS PROTEGIDOS

Las plantillas en `templates/` son **referencias inmutables**. Una vez que una plantilla ha sido aceptada en esa carpeta, no debe editarse — solo leerse durante la generación de YAML.

### Añadir una plantilla nueva

1. Escribe una vista previa en `output/staging-[name].yaml` dentro del directorio de la skill
2. Indica al usuario que pegue la vista previa en Power Apps Studio y verifique que se renderiza correctamente
3. Espera a que el usuario confirme que la vista previa funciona
4. Promueve: mueve el archivo validado a `templates/[name].yaml` dentro del directorio de la skill y elimina el archivo de staging de `output/`

### Nunca edites plantillas existentes

Si la IA necesita algo similar a una plantilla existente pero diferente, duplícala como un nuevo archivo de staging — no toques la original.

### Excepción — Autoreparación de versiones

Las actualizaciones de versión PA2105 son el único caso en el que los archivos de plantilla pueden actualizarse mecánicamente en su lugar. Solo cambian el sufijo `@version` en las cadenas de tipo de control y no pueden romper el renderizado.

---

## REGLAS CRÍTICAS (APLICAN SIEMPRE)

1. Lee `reference/pa-yaml-rules.md` antes de generar el esqueleto — contiene las reglas exactas de formato, las estrategias de dimensionado y los tipos de control válidos.
2. Lee `reference/controls-reference.md` para conocer los nombres de propiedades válidos y los valores de enum.
3. **Nunca uses `context: fork` en esta skill misma** — debe permanecer conversacional para acceder a las imágenes pegadas. Sin embargo, **la herramienta Agent SÍ se usa dentro de la Fase 4** para lanzar workers especialistas. Esos workers reciben el esqueleto preanalizado y la spec de diseño como texto — no necesitan acceso a la imagen original.
4. **Nunca escribas archivos a menos que el usuario lo confirme explícitamente** (p. ej., guardar una plantilla nueva). Los archivos temporales de la Fase 4 son archivos internos de trabajo — esos están bien.
5. **Nunca expongas nombres reales de data sources ni connection strings** en el YAML generado.
6. **Autoreparación de versiones**: si el usuario reporta una advertencia PA2105 para cualquier control, actualiza de inmediato el `@version` de ese control en todos los lugares donde aparece: la tabla VALID CONTROL TYPES de `reference/pa-yaml-rules.md`, todos los ejemplos de código de `reference/pa-yaml-rules.md`, el encabezado de sección de `reference/controls-reference.md` y cada archivo `.yaml` en `templates/`. Usa el número de versión que PA reporte como actual. La misma advertencia PA2105 nunca debe ocurrir dos veces para el mismo control.
7. **Nunca uses `Control: Screen`** — provoca PA2101. Para pantallas nuevas usa siempre el formato de nivel superior `Screens:` documentado en el Paso 3 de la Fase 4 y en `reference/controls-reference.md`.
8. **Fidelidad de controles — nunca sustituyas un control por uno inferior.** Usa siempre el control semántico correcto para el elemento de UI. Usa `reference/controls-index.md` para identificar el control adecuado. La incertidumbre de versión no es razón para sustituir.
9. **Antes de usar `TextInput@0.0.54`, `ComboBox@0.0.51` o `Radio@0.0.25`:** verifica si necesitas `Default` (TextInput/Radio) o `SearchFields` (ComboBox). Si es así, usa la variante Classic en su lugar. Nunca añadas estas propiedades a los controles de generación anterior — provocan PA2108.
10. **Nunca mezcles propiedades de controles Classic y Modern.** Consulta siempre las propiedades en `reference/controls-reference.md` bajo el tipo de control exacto que estás usando.
11. **Sin emojis ni rayas (em dash) en los valores de texto generados** — nunca uses emojis ni rayas (—) en ninguna propiedad de tipo cadena de un control, como `Text`, `HintText`, `Placeholder`, `TrueText`, `FalseText` o `Tooltip`, salvo que la imagen de origen del usuario los contenga explícitamente o el usuario los pida explícitamente.
12. **Usa siempre iconos SVG personalizados — nunca `Classic/Icon@2.5.0`.** Para cada icono, usa `Image@2.2.3` con un SVG inline. El control `Classic/Icon` nunca debe aparecer en el YAML generado.
13. **Presupuesto de tokens de salida — mantente siempre dentro de 32 000 tokens por respuesta.** Escribe primero el YAML al archivo (Paso 4 de la Fase 4) y luego aplica el límite de tamaño del bloque de código (umbral de 400 líneas).
14. **HtmlText es solo de visualización — nunca programes interacciones sobre él.** Para cualquier hyperlink o enlace clicable, usa `Classic/Button@2.2.0` (o un botón Modern) con `OnSelect: =Launch("<url>")`. Estílalo como un botón de solo texto (sin fill/borde) para que coincida visualmente con un enlace. Nunca uses etiquetas `<a>` dentro de HtmlText como mecanismo de interacción.
15. **Nunca uses `Variant: GridLayout` en `GroupContainer`.** Usa solo `Variant: AutoLayout` o `Variant: ManualLayout` para `GroupContainer@1.5.0`. La dirección debe expresarse con `LayoutDirection`.
