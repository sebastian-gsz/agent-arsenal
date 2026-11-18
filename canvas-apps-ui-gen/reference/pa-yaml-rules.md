# Reglas YAML de Power Apps Canvas App

> **Nota de formato de salida para la skill canvas-ui-gen**: a diferencia de la app web original que emitía YAML puro, esta skill DEBE envolver toda la salida en un bloque de código ` ```yaml ` para un copy-paste limpio en el chat de Claude Code.

---

## REGLAS CRÍTICAS DE FORMATO

- El elemento raíz empieza con `- ` (guión + espacio)
- Todo valor de propiedad empieza con el prefijo `=`: `Height: =205`, `Text: ="Hello"`
- Los arrays de hijos usan `- ` (guión + espacio) para cada control hijo
- Las expresiones multilínea usan `|-` con `=` en la siguiente línea indentada
- Usa Galleries con `Table()` para layouts repetitivos — NUNCA hardcodees controles duplicados
- Incluye las propiedades `DropShadow`, `RadiusBottomLeft/Right`, `RadiusTopLeft/Right` en todos los contenedores

### La regla de dos puntos/hash — el error PA1001 más común

**NUNCA** uses `:` (dos puntos) ni `#` (hash) dentro de expresiones de una sola línea. Rompen el parseo de YAML. CUALQUIER expresión que contenga dos puntos o hash DEBE usar el formato multilínea `|-`. Esto incluye:
- Strings de texto como `"Goal: 5"` o `"Item #3"`
- Interpolación de strings como `$"Goal: {value}"`
- Strings SVG que contengan `xmlns='http://...'`
- Cualquier expresión con URLs

**CORRECTO** (dos puntos dentro del string → multilínea):
```yaml
Text: |-
  =$"Goal: {ThisItem.Goal}"
```

**INCORRECTO** (provoca PA1001):
```yaml
Text: =$"Goal: {ThisItem.Goal}"
```

Antes de emitir cualquier expresión de una línea: ¿contiene `:` o `#`? Si SÍ → hazla multilínea con `|-`.

---

## GALLERY WRAP COUNT

Las galerías tienen la propiedad `WrapCount` para layouts de varias filas/columnas. NUNCA crees varias galerías cuando una sola galería con WrapCount puede hacer el mismo trabajo.

### Wrap de galería horizontal:
- Los items llenan COLUMNA A COLUMNA (arriba → abajo), luego la siguiente columna (izquierda → derecha)
- `WrapCount` = cuántos items se apilan verticalmente antes de saltar a la siguiente columna
- `TemplateSize` asigna la ANCHURA de cada item (independiente de WrapCount)
- Ejemplo: 10 items, WrapCount 2: las columnas son `[1,2]`, `[3,4]`, `[5,6]`, `[7,8]`, `[9,10]`

### Wrap de galería vertical:
- Los items llenan FILA A FILA (izquierda → derecha), luego la siguiente fila (arriba → abajo)
- `WrapCount` = cuántos items van en horizontal antes de hacer wrap
- `TemplateSize` asigna la ALTURA de cada item
- Ejemplo: 10 items, WrapCount 2: las filas son `[1,2]`, `[3,4]`, `[5,6]`, `[7,8]`, `[9,10]`

> **`TemplateSize` es write-only.** Úsalo para asignar el tamaño del item, pero nunca lo referencies en fórmulas como `Self.TemplateSize` — no es legible. Para leer el tamaño efectivo en una fórmula, usa `Self.TemplateHeight` (galería vertical) o `Self.TemplateWidth` (galería horizontal). Ejemplo: `Height: =Self.AllItemsCount * Self.TemplateHeight`.

### Regla clave:
Si un diseño muestra items en patrón de grid (varias filas Y columnas), usa UNA galería con el WrapCount apropiado. Si las filas tienen la misma estructura de card, DEBEN ser una única galería — los textos/colores/valores distintos son solo records distintos en `Table()`, no una razón para galerías separadas.

---

## GALLERY EN SIDEBARS Y NAVEGACIÓN

### TemplateSize — solo escritura, no puede leerse

`TemplateSize` asigna el tamaño del item en el control Gallery. No puede referenciarse como `Self.TemplateSize` en fórmulas — es write-only. Para leer el tamaño efectivo en una fórmula, usa siempre:
- `Self.TemplateHeight` — para galerías verticales (sidebars de nav)
- `Self.TemplateWidth` — para galerías horizontales

```yaml
# CORRECT
Height: =Self.AllItemsCount * Self.TemplateHeight

# WRONG — Self.TemplateSize does not exist as a readable property
Height: =Self.AllItemsCount * Self.TemplateSize
```

### FillPortions y Height son mutuamente excluyentes en Gallery

**Nunca asignes tanto `FillPortions: =1` como una fórmula de `Height` estática a la misma galería.** Se contradicen entre sí:
- `FillPortions: =1` → el sistema de layout del padre controla la altura de la galería — NO asignes además `Height`
- Fórmula de `Height` estática → asigna `FillPortions: =0` (tamaño fijo, sin fill flex)

### Tres patrones de galería de nav en sidebar

**Patrón A — La galería llena la altura restante de la sidebar (úsalo por defecto)**

El navGallery toma todo el espacio restante tras el área del logo. No hace falta spacer.
```yaml
- navGallery:
    Control: Gallery@2.15.0
    Variant: Vertical
    Properties:
      FillPortions: =1        # fills remaining height — do NOT also set Height
      TemplateSize: =48
      TemplatePadding: =0
      ShowScrollbar: =false
      LayoutMinHeight: =0
      LayoutMinWidth: =0
```
Área del logo y footer (divisor + perfil): `FillPortions: =0` con alturas estáticas.

**Patrón B — La galería mide exactamente lo que ocupan sus items (compacta, sin scroll)**

Úsalo cuando el diseño tiene tan pocos items de nav que la galería no necesita llenar el espacio restante, y quieres que los elementos de debajo sigan inmediatamente después.
```yaml
- navGallery:
    Control: Gallery@2.15.0
    Variant: Vertical
    Properties:
      FillPortions: =0        # fixed size
      Height: =Self.AllItemsCount * (Self.TemplateHeight + Self.TemplatePadding)
      TemplateSize: =48
      TemplatePadding: =0
      ShowScrollbar: =false
      LayoutMinHeight: =0
      LayoutMinWidth: =0
```

**Patrón C — Items de nav centrados verticalmente en la sidebar (diseño especial)**

Úsalo solo cuando el diseño muestra explícitamente iconos de nav agrupados y centrados en mitad de la altura de la sidebar. Envuelve la galería en un contenedor centrador.
```yaml
- navCenterWrapper:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Vertical
      LayoutJustifyContent: =LayoutJustifyContent.Center
      LayoutMinHeight: =0
      LayoutMinWidth: =0
      DropShadow: =DropShadow.None
      RadiusBottomLeft: =0
      RadiusBottomRight: =0
      RadiusTopLeft: =0
      RadiusTopRight: =0
    Children:
      - navGallery:
          Control: Gallery@2.15.0
          Variant: Vertical
          Properties:
            FillPortions: =0
            Height: =Self.AllItemsCount * Self.TemplateHeight
            TemplateSize: =48
            TemplatePadding: =0
            ShowScrollbar: =false
            LayoutMinHeight: =0
            LayoutMinWidth: =0
```

### ⚠️ Nunca añadas un `sidebarSpacer` entre navGallery y footer

Un `sidebarSpacer` con `FillPortions: =1` junto a un `navGallery` con `FillPortions: =1` reparte el espacio restante 50/50 — la galería solo recibe la mitad de la sidebar. Usa en su lugar el Patrón A: `navGallery` toma `FillPortions: =1` solo, y no hay spacer.

---

## DIMENSIONADO Y VIEWPORT — CRÍTICO

Anchura de viewport por defecto de Power Apps: **1366px**. El contenido nunca debe cortarse salvo que haya una barra de scroll explícita.

### Usa dimensionado flexible (Stretch + FillPortions + valores LayoutMin pequeños) para:
- Labels con texto dinámico (títulos, valores, descripciones)
- Contenedores de contenido que llenan el espacio disponible
- Plantillas de cards dentro de galerías
- Cualquier elemento que deba adaptarse al tamaño de pantalla

### Usa dimensionado estático (Width/Height fija, FillPortions: =0) para:
- Iconos y contenedores de iconos (p. ej., `Width: =35, Height: =35`)
- Botones personalizados con dimensiones concretas
- Pills de estado y badges
- Barras de progreso y divisores/separadores
- `TemplateSize` de la galería

### LayoutMinWidth / LayoutMinHeight — LA TRAMPA DE DEFAULTS MÁS CRÍTICA

**Los valores DEFAULT de Power Apps son `LayoutMinWidth: =250` y `LayoutMinHeight: =100`.**

Si NO asignas estos valores explícitamente, Power Apps aplica estos defaults automáticamente, lo que CAUSARÁ overflow y contenido cortado en la mayoría de layouts.

**DEBES asignar `LayoutMinWidth` y `LayoutMinHeight` explícitamente en TODO contenedor y control.**

Valores típicos:
- Label `LayoutMinHeight`: `=20` a `=30` (según tamaño de fuente)
- Label `LayoutMinWidth`: `=50` a `=100` (según el número de caracteres esperado)
- Contenedor de icono: `LayoutMinWidth/Height` coincide con su tamaño fijo
- Hijo con Stretch (`AlignInContainer.Stretch`): `LayoutMinWidth: =0` — toma la anchura del padre

**CRÍTICO**: cuando un hijo tiene `AlignInContainer: =AlignInContainer.Stretch`, su `LayoutMinWidth` debe ser `=0`. Un `LayoutMinWidth` grande en un item estirado causa overflow.

### Reglas:
- NUNCA asignes a un hijo una Width fija que pueda desbordar a su padre
- Por defecto usa Stretch + FillPortions antes que tamaños fijos
- Si el contenido se corta: reduce `LayoutMinWidth` a `=0` y usa Stretch o FillPortions
- Nada debe cortarse sin un `LayoutOverflowY: =LayoutOverflow.Scroll` explícito

---

## LÓGICA DE DIRECCIÓN DE CONTENEDORES

Para cada grupo de hijos pregúntate: "¿van LADO A LADO, o están APILADOS?"

- Lado a lado (izquierda a derecha) → `LayoutDirection: =LayoutDirection.Horizontal`
- Apilados (arriba a abajo) → `LayoutDirection: =LayoutDirection.Vertical`
- Se necesitan ambas direcciones → ANIDA contenedores

Ejemplo (card con fila de icono arriba, título debajo, valor debajo de ese):
```
Root = Vertical container (rows stack top to bottom)
  ├── Child 1: Horizontal container (icon left, percentage right — side by side)
  ├── Child 2: Title label
  ├── Child 3: Value label
  └── Child 4: Comparison label
```

Los layouts complejos requieren varios niveles de anidamiento — un horizontal dentro de un vertical dentro de un horizontal es normal.

### Proceso de descomposición del layout (para pantallas complejas)

Cuando un layout tiene varias secciones (cabecera, sidebar, áreas de contenido), determina la dirección de la raíz con esta regla:

**Pregúntate: "¿Cómo puedo descomponer este layout a lo largo de UN SOLO eje de modo que ninguna sección individual quede partida?"**

- Prueba primero el eje horizontal (líneas horizontales cortando el layout). Si alguna sección abarca varias porciones horizontales, el corte horizontal no funcionará.
- Prueba el eje vertical (líneas verticales cortando el layout). Si cada sección encaja limpiamente en una columna, usa un contenedor raíz horizontal.
- El eje que funciona limpiamente = la dirección del contenedor raíz.

Luego recurse: aplica la misma pregunta a cada contenedor hijo.

Ejemplo — Un layout con sidebar + (cabecera, overview, tareas apiladas verticalmente):
1. Una línea vertical lo parte limpiamente en 2 columnas (sidebar a la izquierda, contenido a la derecha) → la raíz es Horizontal
2. Dentro de la columna de contenido, líneas horizontales lo parten en cabecera/overview/tareas → el contenedor de contenido es Vertical
3. Dentro de la fila de tareas, una línea vertical la parte en tareas/inspecciones → el contenedor de tareas es Horizontal

---

## ALIGNINCONTAINER — ASÍGNALO SIEMPRE EXPLÍCITAMENTE EN CADA HIJO

Todo hijo dentro de un contenedor AutoLayout **DEBE** tener `AlignInContainer` asignado explícitamente.
Nunca confíes en el `LayoutAlignItems` del padre como default.

**Por qué:** si el padre tiene `LayoutAlignItems: Stretch` (default habitual), cualquier hijo sin su propio `AlignInContainer` explícito se estira en silencio por el eje cruzado, anulando su tamaño fijo y rompiendo círculos, iconos y badges.

### Regla de override del eje cruzado

| Dirección del padre | Eje cruzado | Qué anula `Stretch` |
|---|---|---|
| `Horizontal` | Height | La `Height` fija del hijo |
| `Vertical` | Width | La `Width` fija del hijo |

### Guía de decisión — qué asignar a cada hijo

Identifica primero la `LayoutDirection` del padre — determina qué eje controla `AlignInContainer`:
- **Padre horizontal** → AlignInContainer controla la asignación de **altura**
- **Padre vertical** → AlignInContainer controla la asignación de **anchura**

| Lo que el hijo debe hacer | Asignar al hijo |
|---|---|
| Llenar todo el eje cruzado (labels, áreas de contenido, inputs) | `AlignInContainer: =AlignInContainer.Stretch` |
| Labels que necesitan centrado vertical en un padre **horizontal** | `AlignInContainer: Stretch` (llena la altura) + `VerticalAlign: Middle` en la label — **NO Center** |
| Labels que necesitan centrado horizontal en un padre **vertical** | `AlignInContainer: Stretch` (llena la anchura) — **NO Center** salvo que la label tenga una `Width` explícita intencionalmente menor |
| Mantener su propio tamaño fijo, centrado en el eje cruzado | `AlignInContainer: =AlignInContainer.Center` |
| Mantener su propio tamaño fijo, pegado al inicio | `AlignInContainer: =AlignInContainer.Start` |
| Mantener su propio tamaño fijo, pegado al final | `AlignInContainer: =AlignInContainer.End` |

> **ANTIPATRÓN**: NO uses `AlignInContainer: Center` en una label para conseguir el centrado vertical. Una label centrada solo ocupa su altura natural (~27px para Size 11) y crea huecos arriba/abajo en filas más altas. Usa en su lugar `AlignInContainer: Stretch` + `VerticalAlign: Middle`. `AlignInContainer` controla cuánto **espacio** ocupa el control — no es una propiedad de alineación de texto.

### Patrón de avatar circular (fotos de perfil, thumbnails)

```yaml
- profileAvatar:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      AlignInContainer: =AlignInContainer.Center   # explicit — preserves fixed Height in horizontal row
      FillPortions: =0
      Height: =28
      Width: =28                                   # Height = Width for a circle
      RadiusBottomLeft: =14                        # = Height ÷ 2
      RadiusBottomRight: =14
      RadiusTopLeft: =14
      RadiusTopRight: =14
      DropShadow: =DropShadow.None
```

Reglas para círculos:
- `Height = Width` (mismo valor — base cuadrada)
- Los cuatro valores de radio = `Height ÷ 2` (produce un círculo real)
- `AlignInContainer: Center` — siempre explícito, nunca Stretch

---

## ALTURA DEL CONTENEDOR DE GRUPO DE CAMPOS — CALCÚLALA SIEMPRE A PARTIR DE LOS HIJOS

Los contenedores AutoLayout que agrupan una label con un input (grupos de campos) DEBEN tener una
fórmula de `Height` explícita que sume a sus hijos. Nunca dejes la Height sin asignar en un contenedor
con `FillPortions: =0` — PA Studio le asignará 200px por defecto en lugar de dimensionarlo al contenido.

**Las labels deben tener `AutoHeight: =true`** para que su `.Height` pueda referenciarse en la fórmula del padre.

### Fórmula completa de altura (contenedor vertical)

```
Height = PaddingTop + PaddingBottom
       + sum of each child's effective height
       + (number_of_gaps × LayoutGap)    ← gaps = number of children - 1

Where each child's effective height =
    child.Height + (2 × Max(child.BorderThickness, child.FocusedBorderThickness))

PA IMPORTANT: Power Apps Height/Width properties do NOT include border rendering.
Borders render OUTSIDE the allocated size. A container's own BorderThickness does NOT
affect its own Height property — but when a PARENT sums this container as a child, it
must add 2 × Max(BorderThickness, FocusedBorderThickness) to account for the border
that renders outside the allocated Height.
```

### Patrón: label (sin borde) + input de texto (borde=1, focusedBorder=2)
```yaml
- fieldGroup:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =0
      LayoutDirection: =LayoutDirection.Vertical
      LayoutGap: =6
      Height: =lblField.Height + (inputField.Height + 2 * 2) + 6
      #                                                  ↑↑↑
      #              2 × FocusedBorderThickness(=2) adds 4px outside the input's Height
    Children:
      - lblField:
          Control: Label@2.5.1
          Properties:
            AutoHeight: =true          # required — lets Height be referenced in parent formula
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
      - inputField:
          Control: Classic/TextInput@2.3.2
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            FillPortions: =0
            Height: =40
            BorderThickness: =1
            FocusedBorderThickness: =2
```

### Patrón: contenedor CON padding (sin borde en el propio contenedor)
```yaml
# 3 children, gap=8, padding top/bottom=16
# If any child has a border, add 2 × Max(child.BorderThickness, child.FocusedBorderThickness) per bordered child
Height: =16 + 16 + child1.Height + child2.Height + child3.Height + (2 * 8)
```

### Patrón: label + varios sub-items (p. ej., lista de categorías recomendadas, gap=4)
```yaml
# 4 children → 3 gaps at 4px each
Height: =lblTitle.Height + lbl1.Height + lbl2.Height + lbl3.Height + (3 * 4)
```

### Filas horizontales (hijos lado a lado, no apilados)
Para contenedores horizontales, Height = la altura del hijo más alto + padding arriba/abajo + 2×borde.
Asigna una Height fija igual al hijo más alto conocido:
```yaml
Height: =32   # matches the radio/checkbox/button height in the row (no border/padding on this row)
```

### FillPortions vs. Height explícita — mutuamente excluyentes en el mismo eje

| Situación | Qué hacer |
|---|---|
| Dimensionar según los hijos (grupo de campos, sección fija) | `FillPortions: =0` + fórmula de `Height` explícita |
| Llenar el espacio restante proporcionalmente | `FillPortions: =1` — la `Height` explícita se **ignora** |
| Llenar con restricción de tamaño mínimo | `FillPortions: =1` + `LayoutMinHeight: =N` |
| Llenar con restricción de tamaño máximo | `FillPortions: =1` + `LayoutMaxHeight: =N` |

**Regla:** una vez que `FillPortions > 0`, cualquier Height/Width estática en ese eje queda anulada por el fill.
Usa `LayoutMinHeight`/`LayoutMaxHeight` para constreñir el comportamiento del fill.
- AutoLayout vertical → `FillPortions` controla la **Height**
- AutoLayout horizontal → `FillPortions` controla la **Width**

### Qué NO hacer
- NO omitas la Height en contenedores con `FillPortions: =0` — PA Studio les asigna 200px por defecto
- NO asignes tanto `FillPortions: =1` como una Height calculada — gana FillPortions, la Height se ignora
- NO incrustes una estimación estática — calcula a partir de hijos + gaps + padding + borde

---

## RECORTES DE BORDES — EL PADRE DEBE AÑADIR PADDING PARA LOS HIJOS CON BORDE

Cuando un hijo dentro de un contenedor AutoLayout vertical tiene un borde, el borde se renderiza FUERA de
la propiedad Width del hijo, en los lados izquierdo y derecho. Sin padding en el padre, este borde
queda visualmente recortado en el borde del contenedor.

**Regla:** el padre directo de un hijo con borde DEBE tener:
- `PaddingLeft` y `PaddingRight` ≥ `Max(child.BorderThickness, child.FocusedBorderThickness)`

Aplica en ambas direcciones:
- Contenedor vertical → los hijos se apilan verticalmente → los bordes izquierdo/derecho pueden recortarse → añade PaddingLeft/Right
- Contenedor horizontal → los hijos van lado a lado → los bordes superior/inferior pueden recortarse → añade PaddingTop/Bottom

### Patrón
```yaml
- fieldGroup:       # vertical container: must add left/right padding
    Properties:
      PaddingLeft: =2    # = Max(inputField.BorderThickness=1, inputField.FocusedBorderThickness=2)
      PaddingRight: =2   # same
    Children:
      - inputField:
          Properties:
            BorderThickness: =1
            FocusedBorderThickness: =2    # ← this drives the padding requirement
```

### Qué NO hacer
- NO dejes un contenedor vertical de grupo de campos con `PaddingLeft: =0` si contiene un input con borde
- Es un bug recurrente: el borde focused aparece recortado por los lados izquierdo y derecho en la app

---

## LAYOUT OVERFLOW — SCROLL VS. HIDE

Cuando el contenido hijo puede superar la altura del contenedor, asigna `LayoutOverflowY` al contenedor:

| Valor de propiedad | Efecto |
|---|---|
| `=LayoutOverflow.Scroll` | El contenedor se vuelve desplazable verticalmente cuando los hijos superan su altura |
| `=LayoutOverflow.Hide` | El contenido se recorta en el límite (por defecto — no hace falta asignarlo explícitamente) |

Usa `LayoutOverflowY: =LayoutOverflow.Scroll` en cualquier contenedor que:
- Tenga una Height acotada (FillPortions o fija)
- Pueda contener más contenido del que cabe en runtime (cuerpos de formulario, paneles de listas, etc.)

```yaml
- scrollableBody:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =1
      LayoutDirection: =LayoutDirection.Vertical
      LayoutOverflowY: =LayoutOverflow.Scroll
```

Nota: asígnalo solo a contenedores acotados. Un contenedor que auto-se dimensiona según sus hijos
(`Height = suma de hijos`) nunca puede desbordarse a sí mismo — no hace falta ajuste de overflow.
El scroll horizontal (`LayoutOverflowX`) existe pero se usa poco.

---

## REGLAS DE DIMENSIONADO DE HIJOS — LA FUENTE DE ERRORES MÁS COMÚN

Todo hijo en un contenedor AutoLayout DEBE tener una estrategia de tamaño explícita en el EJE PRIMARIO. Sin ella, los hijos se inflan o colapsan de forma impredecible.

### En un padre AutoLayout VERTICAL (eje primario = vertical):
Todo hijo necesita una estrategia de HEIGHT explícita:
- `FillPortions: =0` + `Height: =valor` estática → altura exacta
- `FillPortions: =1` (o superior) → altura flexible a partir del espacio restante; usa `LayoutMinHeight` como suelo

### En un padre AutoLayout HORIZONTAL (eje primario = horizontal):
Todo hijo necesita una estrategia de WIDTH explícita:
- `FillPortions: =0` + `Width: =valor` estática → anchura exacta
- `FillPortions: =1` (o superior) → anchura flexible a partir del espacio restante; usa `LayoutMinWidth` como suelo

### Cómo funciona FillPortions:
1. El contenedor asigna espacio primero a todos los hijos de tamaño estático (`FillPortions: =0`)
2. El espacio restante se reparte entre los hijos flexibles proporcionalmente
3. `LayoutMinWidth/Height` actúa como suelo — el hijo no se encoge por debajo de esto

Ejemplo (contenedor horizontal, anchura 1000):
- Hijo A: `FillPortions: =0, Width: =200` → recibe exactamente 200
- Hijo B: `FillPortions: =2` → recibe 2/3 de los 800 restantes = ~533
- Hijo C: `FillPortions: =1` → recibe 1/3 de los 800 restantes = ~267

**NUNCA** dejes un hijo sin estrategia de tamaño. Todo hijo debe tener o una Height/Width estática + `FillPortions: =0`, O `FillPortions: =1+` con una `LayoutMinHeight/Width`.

---

## ALIGNINCONTAINER — POSICIONAMIENTO EN EL EJE SECUNDARIO

`AlignInContainer` posiciona a un hijo en el eje SECUNDARIO (perpendicular a la dirección del padre).

### En un padre HORIZONTAL (eje secundario = vertical):
- `AlignInContainer.Start` → hijo ARRIBA, usa Height estática
- `AlignInContainer.Center` → hijo CENTRADO verticalmente, usa Height estática
- `AlignInContainer.End` → hijo ABAJO, usa Height estática
- `AlignInContainer.Stretch` → hijo ESTIRADO para llenar la altura del padre, usa `LayoutMinHeight` como suelo
- `AlignInContainer.SetByContainer` → hereda del `LayoutAlignItems` del padre

### En un padre VERTICAL (eje secundario = horizontal):
- `AlignInContainer.Start` → hijo a la IZQUIERDA, usa Width estática
- `AlignInContainer.Center` → hijo CENTRADO horizontalmente, usa Width estática
- `AlignInContainer.End` → hijo a la DERECHA, usa Width estática
- `AlignInContainer.Stretch` → hijo ESTIRADO para llenar la anchura del padre, usa `LayoutMinWidth` como suelo

---

## ASIGNA SIEMPRE TODAS LAS PROPIEDADES DE LAYOUT EXPLÍCITAMENTE

Todo `GroupContainer` con `Variant: AutoLayout` debe tener TODAS las siguientes propiedades asignadas explícitamente — nunca omitas ninguna, ni siquiera cuando el default coincida con la intención. Power Apps asigna por defecto `LayoutJustifyContent` a `Start`, `LayoutAlignItems` a `Start` y `LayoutWrap` a `false` — pero omitirlos es un bug latente esperando a aflorar.

**Requerido en todo GroupContainer AutoLayout:**
- `LayoutDirection` — `=LayoutDirection.Vertical` o `=LayoutDirection.Horizontal`
- `LayoutJustifyContent` — controla el espaciado del eje principal (ver tabla abajo)
- `LayoutAlignItems` — controla la alineación del eje secundario (`Start`, `Center`, `End`, `Stretch`)
- `LayoutGap` — valor px explícito; usa `=0` si no hay gap
- `LayoutWrap` — `=false` salvo que el wrap sea intencional

> **Nota:** los contenedores `ManualLayout` NO usan `LayoutJustifyContent`, `LayoutAlignItems`, `LayoutGap` ni `LayoutWrap`. Asígnalos solo en `Variant: AutoLayout`.

### Referencia de LayoutJustifyContent

| Intención | Valor |
|---|---|
| Los items se apilan desde arriba/izquierda (la mayoría de contenedores) | `=LayoutJustifyContent.Start` |
| Items centrados en el eje principal | `=LayoutJustifyContent.Center` |
| Items empujados al borde inferior/derecho | `=LayoutJustifyContent.End` |
| Primer item al inicio, último al final | `=LayoutJustifyContent.SpaceBetween` |
| Espacio igual alrededor de cada item | `=LayoutJustifyContent.SpaceAround` |

### Patrón — Cancel a la izquierda, botones de acción a la derecha

```yaml
- actionRow:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      LayoutDirection: =LayoutDirection.Horizontal
      LayoutJustifyContent: =LayoutJustifyContent.SpaceBetween
      LayoutAlignItems: =LayoutAlignItems.Center
      LayoutGap: =0
      LayoutWrap: =false
  Children:
    - btnCancel: ...          # FillPortions: =0, fixed Width
    - rightButtonGroup:       # FillPortions: =1 — fills remaining space
        Control: GroupContainer@1.5.0
        Variant: AutoLayout
        Properties:
          FillPortions: =1           # responsive — takes all remaining space
          LayoutMinWidth: =200       # floor: never collapse below 200px
          LayoutDirection: =LayoutDirection.Horizontal
          LayoutJustifyContent: =LayoutJustifyContent.End
          LayoutAlignItems: =LayoutAlignItems.Center
          LayoutGap: =10
          LayoutWrap: =false
      Children:
        - btnSave: ...
        - btnContinue: ...
```

---

## FILLPORTIONS — DIMENSIONADO ESTÁTICO VS. RESPONSIVO

En cualquier contenedor AutoLayout, `FillPortions` controla cómo un hijo se dimensiona en el **eje primario** (anchura para horizontal, altura para vertical):

| `FillPortions` | Comportamiento |
|---|---|
| `=0` (o omitido) | **Estático** — equivalente a una `Width`/`Height` dura. Debe tener una `Width`/`Height` explícita asignada, o el item colapsa a cero y recorta a sus hijos. |
| `=1` o cualquier valor positivo | **Responsivo/flexible** — toma una porción proporcional del espacio restante tras el reparto de todos los hermanos con `FillPortions: =0`. |

**FillPortions vale cero si se omite** — asígnalo siempre explícitamente.

**Cuándo usar `FillPortions: =1`:**
- Cualquier hijo que deba expandirse para llenar el espacio restante de su padre
- El grupo de botones en una fila `SpaceBetween` (Cancel fijo a la izquierda, grupo flexible a la derecha)
- Cualquier sección que deba crecer/encogerse con el contenedor

**Matemática de ratios:** varios hermanos flexibles se reparten el espacio restante proporcionalmente. P. ej., `2:1:1` = 50% : 25% : 25%.

**`LayoutMinWidth` / `LayoutMinHeight`** — asigna el suelo de un item flexible. Al alcanzar el suelo, el sistema trata ese item como estático en su valor mínimo y redistribuye el espacio flexible restante entre los demás items.

---

## ALINEACIÓN DEL EJE SECUNDARIO — ESTÁTICO VS. STRETCH

En el eje **perpendicular** a `LayoutDirection`, cada hijo puede tener tamaño estático o estirarse:

| Valor de `AlignInContainer` | Tamaño del hijo en el eje secundario | Qué asignar |
|---|---|---|
| `Start`, `Center`, `End` | Estático | `Width` (en padre vertical) o `Height` (en padre horizontal) |
| `Stretch` | Se expande para llenar el contenedor | Usa `LayoutMinWidth`/`LayoutMinHeight` como suelo — NO asignes tamaño estático |

**Override por hijo:** asigna `AlignInContainer` a un hijo individual para anular el `LayoutAlignItems` del padre solo para ese item.

**Regla:** cuando el padre tiene `LayoutAlignItems: =LayoutAlignItems.Stretch`, NO asignes además una `Width`/`Height` estática al hijo — entra en conflicto con el stretch.

---

## OVERFLOW Y CONTENEDORES DESPLAZABLES

Define el comportamiento de overflow explícitamente en cualquier contenedor cuyo contenido pueda superar sus límites:

| Propiedad | Aplica cuando |
|---|---|
| `LayoutOverflowVertical` | El contenido supera la altura del contenedor |
| `LayoutOverflowHorizontal` | El contenido supera la anchura del contenedor |

| Valor | Comportamiento |
|---|---|
| `=LayoutOverflow.Hide` | El contenido extra se recorta (por defecto) |
| `=LayoutOverflow.Scroll` | Se añade scrollbar automáticamente |

**Asigna siempre `LayoutOverflow.Scroll`** a:
- El panel de contenido principal / cuerpo del formulario (scroll vertical)
- Cualquier contenedor que envuelva una galería o lista que pueda superar la altura disponible

Asígnalo al **wrapper desplazable más interno**, no a un contenedor padre por encima de él.

```yaml
- scrollableContent:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      LayoutDirection: =LayoutDirection.Vertical
      LayoutOverflowVertical: =LayoutOverflow.Scroll
      LayoutOverflowHorizontal: =LayoutOverflow.Hide
```

---

## WRAP — REQUISITOS Y GOTCHAS

`LayoutWrap: =true` habilita el salto de línea cuando los hijos desbordan el eje primario. Deben cumplirse tres condiciones a la vez:

1. **Anchura del item** asignada (estática o por fórmula)
2. **Altura del item** asignada (estática o altura mínima)
3. **Altura del padre** asignada para acomodar varias filas

**Fórmula de altura del padre al hacer wrap:**
```
(rows × item_height) + ((rows - 1) × LayoutGap) + PaddingTop + PaddingBottom
```

**Gotcha — los items ocultos siguen consumiendo gap:** con `Visible: =false`, el gap junto al item oculto se sigue aplicando. Anula el gap condicionalmente:
```yaml
LayoutGap: =If(IsBlank(someValue), 0, 10)
```

**Gotcha — Stretch + Wrap:** cuando los items tienen `AlignInContainer: Stretch` Y el wrap está habilitado, Power Apps usa `LayoutMinHeight` (no la altura flexible) como la altura real del item. Asigna siempre `LayoutMinHeight` explícitamente a los items dentro de un contenedor con wrap.

---

## PROCESO DE DESCOMPOSICIÓN DEL LAYOUT

Al convertir un diseño en contenedores, usa el **test de descomposición de un solo eje** en cada nivel:

1. **Identifica las secciones mayores** — trata el diseño como un wireframe, ignora los detalles internos.
2. **Test del eje horizontal:** ¿partir con una línea horizontal parte alguna sección?
   - No → usa un contenedor **AutoLayout Vertical** para este nivel.
   - Sí → prueba el eje vertical en su lugar.
3. **Test del eje vertical:** ¿partir con una línea vertical parte alguna sección?
   - No → usa un contenedor **AutoLayout Horizontal**.
4. **Recurse** — repite para cada contenedor hijo.
5. **Regla de la hoja:** si una sección tiene un solo control, o varios controles todos en la misma dirección que su padre, añádelos directamente — no hace falta wrapper extra.

**Usa ManualLayout solo cuando:**
- Los items deban **superponerse** (p. ej., imagen de fondo + botón overlay + label encima)
- El layout sea genuinamente no estructurado, sin descomposición de un solo eje
- Nota: ManualLayout sacrifica la facilidad de responsividad — prefiere AutoLayout siempre que sea posible

---

## DIMENSIONADO RESPONSIVO DE GALERÍAS

**Anchura de plantilla — resta 1 para evitar scrollbars por redondeo:**
```
TemplateWidth: =(Parent.Width / columnCount) - 1
```
La división en coma flotante puede producir una anchura 0.5px mayor que el contenedor, disparando una scrollbar no deseada. Resta 1 para evitarlo.

**El contenedor raíz de la galería debe llenar la plantilla exactamente:**
```yaml
X: =0
Y: =0
Width: =Parent.TemplateWidth
Height: =Parent.TemplateHeight
```

**Recuento de columnas responsivo vía WrapCount:**
```yaml
WrapCount: =If(App.ActiveScreen.Size <= 2, 2, 4)
```

**Padding de plantilla:** usa la propiedad `TemplatePadding` de la galería solo para la alineación centrada. Para posicionamiento personalizado, asigna `TemplatePadding: =0` y posiciona el contenedor raíz manualmente.

---

## DROP SHADOW — LÍMPIALO SIEMPRE SALVO QUE SE NECESITE EXPLÍCITAMENTE

Power Apps añade `DropShadow` a todo nuevo `GroupContainer` por defecto. **Límpialo en todos los contenedores** salvo que el diseño pida explícitamente una sombra (p. ej., un modal flotante o una card).

```yaml
DropShadow: =DropShadow.None    # required on every container
```

---

## CAPAS EN LA PLANTILLA DE GALERÍA

Los hijos dentro de una plantilla de galería se comportan como layout manual — los hermanos pueden SUPERPONERSE entre sí.

### Galerías de estilo card (espaciado visible, fondos, esquinas redondeadas)

Usa anidamiento de 3 niveles dentro de la plantilla de la galería:

**Nivel 1 — Hijos de la galería (hermanos que se superponen):**
- `imgTemplateBG`: control Image como fondo
- `rootContainer`: AutoLayout transparente, llena toda la plantilla de la galería, aporta el padding entre cards

**Nivel 2 — Dentro de rootContainer:**
- `galCardTemplate`: el contenedor de card visible con radio de borde, su propio padding y color de fondo

**Nivel 3 — Dentro de galCardTemplate:**
- El contenido real (título, valor, iconos, etc.)

El tamaño y la posición de la imagen de fondo provienen de `galCardTemplate` (no de `rootContainer`) para que coincida exactamente con la card visible. El radio de borde también debe coincidir.

**Patrón de card CORRECTO:**
```yaml
Children:
  - imgTemplateBG:
      Control: Image@2.2.3
      Properties:
        Image: =Blank()
        Fill: =ThisItem.BG_Color
        Height: =galCardTemplate.Height
        Width: =galCardTemplate.Width
        X: =galCardTemplate.X
        Y: =galCardTemplate.Y
        RadiusBottomLeft: =20
        RadiusBottomRight: =20
        RadiusTopLeft: =20
        RadiusTopRight: =20
  - rootContainer:
      Control: GroupContainer@1.5.0
      Variant: AutoLayout
      Properties:
        DropShadow: =DropShadow.None
        Height: =Parent.TemplateHeight
        LayoutDirection: =LayoutDirection.Vertical
        PaddingBottom: =15
        PaddingLeft: =15
        PaddingRight: =15
        PaddingTop: =15
        Width: =Parent.TemplateWidth
      Children:
        - galCardTemplate:
            Control: GroupContainer@1.5.0
            Variant: AutoLayout
            Properties:
              Height: =Parent.Height
              Width: =Parent.Width
              LayoutDirection: =LayoutDirection.Vertical
              PaddingBottom: =20
              PaddingLeft: =20
              PaddingRight: =20
              PaddingTop: =20
              RadiusBottomLeft: =20
              RadiusBottomRight: =20
              RadiusTopLeft: =20
              RadiusTopRight: =20
            Children:
              # actual content here
```

### Galerías de estilo tabla/lista (filas continuas, sin gaps)
El contenido va directamente en el contenedor raíz de la plantilla. Cada fila tiene su propio padding interno. No hace falta capa de imagen de fondo salvo que el diseño la pida.

---

## CONTROLES DE IMAGEN DE FONDO

Al usar un control Image como capa de fondo:
- Asigna SIEMPRE `Image: =Blank()` para evitar el placeholder SampleImage por defecto
- Asigna `Fill: =ThisItem.BG_Color` (o la expresión de color) para el fondo
- El control Image actúa puramente como un rectángulo de color con esquinas redondeadas

---

## DEFAULT DE DROP SHADOW

Todo contenedor en Power Apps tiene la drop shadow ACTIVADA por defecto.

**Asigna SIEMPRE `DropShadow: =DropShadow.None` a todos los contenedores** salvo que quieras una sombra específicamente. Olvidarlo hace que la UI se vea recargada.

---

## PATRÓN DE BOTÓN PERSONALIZADO

Para crear un área clicable en Power Apps, usa un contenedor ManualLayout con:
1. Una `Label` para el texto
2. Un `Image` para el icono
3. Un `Classic/Button@2.2.0` transparente superpuesto encima:
   ```yaml
   Fill: =RGBA(0, 0, 0, 0)
   HoverFill: =RGBA(0,0,0,0.1)
   PressedFill: =RGBA(0,0,0,0.2)
   Text: =""
   Width: =Parent.Width
   Height: =Parent.Height
   ```

El botón captura los clics; la label y el icono aportan la apariencia visual.

---

## WORKAROUND DE RADIO DE BORDE

Controles que NO tienen propiedades de radio de borde (asignar RadiusBottomLeft/Right/TopLeft/Right directamente provocará PA2108):
- `Label@2.5.1`
- `ComboBox@0.0.51`
- `DatePicker@0.0.46`
- `NumberInput@2.9.12`
- `Radio@0.0.25`
- `TextInput@0.0.54`

**Para dar esquinas redondeadas a estos controles:**
1. Envuelve el control dentro de un `GroupContainer`
2. Aplica `RadiusBottomLeft/Right` y `RadiusTopLeft/Right` al CONTENEDOR
3. Asigna `Fill` al contenedor para el color de fondo

---

## GESTIÓN DE COLOR DE ICONOS SVG — CRÍTICO

Los SVG son strings de texto HTML. Los colores dentro de los SVG deben ser STRINGS DE TEXTO, no llamadas a la función RGBA de Power Apps.

### Patrón 1: Color hardcodeado
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(
          ThisItem.Icon, "currentColor", "RGBA(246,246,250,1)"
      )
  )
```
Nota: `"RGBA(246,246,250,1)"` es un string de texto entre comillas — NO una llamada a función.

### Patrón 2: Color dinámico desde un campo o variable

Cuando el color proviene de un campo de datos (p. ej., `ThisItem.KPIColor`) o de una variable, usa `JSON()` para serializarlo a string y luego quita las comillas circundantes:

```yaml
Substitute(JSON(ThisItem.KPIColor), """", "")
```

Esto produce un string como `#22c55e` o `rgba(0, 142, 210, 1)` — un valor de texto que el SVG puede usar.

Para un valor de color hardcodeado: `Substitute(JSON(RGBA(34,197,94,1)), """", "")` produce `#22c55e`.

**INCORRECTO — `Text()` sobre un tipo Color devuelve string vacío o error:**
```yaml
Text(ThisItem.KPIColor)      # WRONG — Color is not a text-convertible type
```

**INCORRECTO — pasar un valor Color directamente:**
```yaml
RGBA(34,197,94,1)   # This is a Color value, not a string — SVG cannot use it
```

**Regla clave**: dentro de `EncodeUrl(Substitute(...))`, el color de reemplazo debe ser SIEMPRE un string de texto. Usa `Substitute(JSON(colorValue), """", "")` para cualquier color dinámico/de campo. Usa un literal entre comillas como `"RGBA(246,246,250,1)"` para colores hardcodeados.

---

## ESTÁNDARES DE CALIDAD DE DISEÑO

### Espaciado y padding
- Contenedores de contenido de cards: `PaddingTop/Bottom/Left/Right: =15` a `=25` — NUNCA omitas el padding
- Contenedores wrapper externos: `PaddingTop/Bottom/Left/Right: =15` a `=20`
- Gallery `TemplatePadding: =0` — controla el espaciado vía el padding del contenedor interno
- `LayoutGap: =8` a `=16` entre elementos hijos en contenedores AutoLayout
- Gallery `TemplateSize: =Self.Width/NumberOfCards` para cards de anchura igual
- Comprueba siempre: "¿El texto tocaría el borde?" Si es así, añade más padding

### Padding de plantilla de galería
El contenedor de card visible dentro de una galería DEBE tener padding:
```yaml
PaddingBottom: =20
PaddingLeft: =20
PaddingRight: =20
PaddingTop: =20
```

### Paleta de colores
- Fondos oscuros: `RGBA(35,36,47,1)` o `RGBA(42,43,71,1)`
- Acentos azules: `RGBA(0,142,210,1)` o `RGBA(14,165,233,1)`
- Texto claro sobre oscuro: `RGBA(246,246,250,1)` a opacidad total
- Texto sutil sobre oscuro: `RGBA(246,246,250,0.7)` para texto secundario
- Texto casi negro: `RGBA(15,23,42,1)` o `RGBA(35,36,47,1)`
- Texto gris: `RGBA(100,116,139,1)` o `RGBA(131,141,157,1)`
- Evita el negro puro `RGBA(0,0,0,1)` y el blanco puro `RGBA(255,255,255,1)`
- Los fills de las cards deben alternar entre 2 colores para dar variedad visual

### Jerarquía de fuentes (debe ser claramente distinguible)

**Tope general: el tamaño de fuente nunca debe superar 13 para texto de UI ordinario.** Las Canvas Apps se renderizan a ~96 DPI en un canvas de 1366px, así que tamaños que en pantalla se ven razonables aparecen muy grandes en la app renderizada. La única excepción son los valores numéricos grandes de KPI/estadísticas donde el diseño pide explícitamente una cifra prominente.

| Elemento | Rango de tamaño | Fuente |
|---|---|---|
| Título de página/pantalla | `=13` a `=14` máximo | `Font.'Lato Black'`, Semibold |
| Título de sección/card | `=12` | `Font.Lato`, Semibold |
| Labels de campos, texto de botones, labels de tabs | `=11` | `Font.Lato`, Semibold o Normal |
| Texto de cuerpo, placeholder de input | `=10` a `=11` | `Font.Lato`, Normal |
| Texto secundario/atenuado, breadcrumbs | `=9` a `=10` | `Font.Lato`, Normal, color más claro |
| Cabeceras de tabla | `=8` | `Font.'Lato Light'`, Semibold, gris |
| Contenido de tabla, texto de badges | `=9` | `Font.Lato`, Normal |
| Valor principal de KPI/estadística (pedido explícito de diseño) | `=20` a `=28` | `Font.'Lato Black'`, Bold |

**Nunca uses Size: =13 o superior por defecto para todas las labels** — produce una UI que se ve sobredimensionada en pantalla. Usa la jerarquía de arriba para mantener una escala visual clara.

### Nunca dejes controles con la apariencia por defecto

Todo control que emitas debe estar completamente estilizado para coincidir con el diseño objetivo. La apariencia por defecto de los controles de Power Apps es plana, gris y sin estilo — nunca es aceptable en la salida generada.

**Para cada control, asigna siempre explícitamente:**
- `Font`, `Size`, `Color` / `Fill` — que coincidan con la tipografía y la paleta del diseño
- Estados hover y pulsado (`HoverFill`, `HoverColor`, `PressedFill`, `PressedColor`) — que las interacciones se sientan pulidas
- `BorderColor`, `BorderStyle`, `BorderThickness` — que coincidan con el diseño, o bordes transparentes para elementos fantasma
- `RadiusTopLeft/Right/BottomLeft/Right` — que coincidan con el estilo de esquinas del diseño; nunca dejes esquinas vivas en inputs o botones si el diseño es redondeado
- `Font.Lato` como fuente por defecto — nunca dejes `Font.OpenSansRegular` (el default de PA)

Aplica a todo `Classic/TextInput`, `Classic/ComboBox`, `Classic/CheckBox`, `Classic/Radio`, `Classic/Button`, `Label`, `Gallery` y cualquier otro control. Si un control está en la salida, está estilizado. Sin excepciones.

### Radius Wrapper Pattern

Algunos controles **no** soportan `RadiusTopLeft/Right/BottomLeft/Right` — añadir estas propiedades provoca PA2108. Controles confirmados sin soporte de radio:

- `Classic/ComboBox@2.4.0`
- `Classic/DatePicker@2.6.0`
- `Classic/Radio@2.3.0`
- `Classic/CheckBox@2.1.0`
- `Label@2.5.1`
- `Rectangle@2.3.0`

**Regla:** cuando necesites esquinas redondeadas en un control que no soporta radio, envuélvelo en un `GroupContainer` con el radio y el borde asignados al contenedor. Las esquinas redondeadas del GroupContainer recortan visualmente al control hijo. El control interno debe tener su propio borde eliminado (`BorderStyle: =BorderStyle.None`, `BorderThickness: =0`) para evitar un borde doble.

**Patrón:**
```yaml
- controlNameFrame:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      BorderColor: =RGBA(210,210,220,1)
      BorderStyle: =BorderStyle.Solid
      BorderThickness: =1
      DropShadow: =DropShadow.None
      Fill: =RGBA(255,255,255,1)
      FillPortions: =0
      Height: =40                        # same as the inner control
      LayoutAlignItems: =LayoutAlignItems.Stretch
      LayoutDirection: =LayoutDirection.Vertical
      LayoutMinHeight: =0
      LayoutMinWidth: =0
      PaddingBottom: =2                  # >= inner control's FocusedBorderThickness
      PaddingLeft: =2
      PaddingRight: =2
      PaddingTop: =2
      RadiusBottomLeft: =8
      RadiusBottomRight: =8
      RadiusTopLeft: =8
      RadiusTopRight: =8
    Children:
      - controlName:
          Control: Classic/ComboBox@2.4.0   # or any non-radius control
          Properties:
            AlignInContainer: =AlignInContainer.Stretch
            BorderStyle: =BorderStyle.None    # ← no own border; frame provides it
            BorderThickness: =0
            Fill: =RGBA(0,0,0,0)             # ← transparent; frame provides background
            FocusedBorderColor: =RGBA(160,160,175,1)
            FocusedBorderThickness: =2
            # ... other control properties
```

**Convención de nombres:** llama al wrapper `[controlName]Frame` (p. ej., `inputCategoryFrame` envuelve a `inputCategory`).

**Fórmula de altura:** cualquier contenedor que referencie la altura del control interno debe referenciar el frame: `=lblField.Height + inputCategoryFrame.Height + 6`.

**Nota:** los estados de fill hover/pulsado del control interno siguen funcionando con normalidad (se aplican al interior del control). Los estados de hover relacionados con bordes (p. ej., `HoverBorderColor`) se pierden, ya que el control interno no tiene borde — acepta esta limitación, o usa fórmulas `OnHover` para el color dinámico del borde del frame si es necesario.

---

### Recorte de bordes dentro de contenedores con padding cero

Cuando un control hijo tiene un borde visible y su contenedor padre tiene padding cero en el lado donde cae el borde, el borde queda recortado por el borde del contenedor. Esto también afecta al borde **focused**: al enfocar un control, su borde se expande de `BorderThickness` a `FocusedBorderThickness`. Si el padding del contenedor solo iguala la `BorderThickness` base, el borde focused desborda y se recorta.

**Regla:** el padding mínimo del contenedor debe igualar el **`FocusedBorderThickness`** del hijo (NO solo `BorderThickness`). Es la anchura de borde en el peor caso y garantiza que no haya recorte en ningún estado.

- Si el hijo tiene `FocusedBorderThickness: =2` → asigna al contenedor `PaddingTop/Bottom/Left/Right: =2`
- Si el hijo tiene `FocusedBorderThickness: =1` → `PaddingTop/Bottom/Left/Right: =1` es suficiente
- Esto es SOLO para evitar recortes — NO añadas padding a todos los contenedores por defecto. Aplica solo cuando un hijo con borde quedaría pegado al borde del contenedor.

Ejemplo:
```yaml
- inputWrapper:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      PaddingLeft: =2    # >= child's FocusedBorderThickness (2) — prevents clipping in focused state
      PaddingRight: =2
      PaddingTop: =2
      PaddingBottom: =2
```

Si el contenedor ya tiene un padding significativo (p. ej., `PaddingLeft: =16`), no hace falta padding extra — el borde no se recortará.

### Tope de FocusedBorderThickness

**Regla:** `FocusedBorderThickness` debe ser **≤ 2** en TODOS los controles — inputs, botones, checkboxes, radios, overlays, galerías — sin excepciones. Aplica tanto si el anillo de focus es visible como si es invisible.

Asigna siempre `FocusedBorderThickness` explícitamente. Nunca lo omitas — Power Apps usa 4 por defecto, que es demasiado grueso.

Patrón estándar:
```yaml
BorderThickness: =1          # base border
FocusedBorderColor: =RGBA(r,g,b,1)   # same or slightly darker than HoverBorderColor
FocusedBorderThickness: =2   # focus state — 1 step thicker, never more
```

Nunca uses valores de 3 o 4, ni siquiera en botones overlay transparentes donde el anillo sería invisible. La regla es incondicional.

### Las Heights explícitas de contenedores deben incluir padding y gap

Cuando un `GroupContainer` en modo `AutoLayout` usa una fórmula de **`Height` explícita** que suma alturas de hijos, la fórmula debe incluir siempre el padding y el gap propios del contenedor. Omitir el padding hace que los hijos se recorten — la altura lógica del contenedor es menor que su área visual de contenido.

**Patrón de fórmula (contenedor AutoLayout vertical — 2 hijos):**
```
Height: =child1.Height + child2.Height + Self.LayoutGap + Self.PaddingTop + Self.PaddingBottom
```

**Patrón de fórmula (n hijos):**
```
Height: =child1.Height + ... + childN.Height + (N - 1) * Self.LayoutGap + Self.PaddingTop + Self.PaddingBottom
```

**Patrón de fórmula (contenedor AutoLayout horizontal — dirigido por anchura):**
```
Width: =child1.Width + ... + childN.Width + (N - 1) * Self.LayoutGap + Self.PaddingLeft + Self.PaddingRight
```

**Reglas:**
- Usa siempre `Self.PaddingTop` / `Self.PaddingBottom` (no números hardcodeados) — si el padding cambia, la fórmula sigue siendo correcta automáticamente.
- Usa siempre `Self.LayoutGap` (no un número hardcodeado), por la misma razón.
- Si `LayoutGap` es 0 o solo hay un hijo, el término del gap es 0 — omítelo.
- Si el contenedor usa altura dirigida por `FillPortions` o `Height: =Parent.Height`, esta regla no aplica — Power Apps gestiona el espacio automáticamente.
- Cuando un hijo es un frame del Radius Wrapper (GroupContainer que envuelve un control sin radio), la altura del frame ya contempla su propio padding interno — usa `frame.Height` directamente, no la altura del control interno.

**Incorrecto — falta el padding (provoca recorte inferior):**
```yaml
Height: =lblTitle.Height + inputName.Height + 6
PaddingTop: =2
PaddingBottom: =2
LayoutGap: =6
```

**Correcto:**
```yaml
Height: =lblTitle.Height + inputName.Height + Self.LayoutGap + Self.PaddingTop + Self.PaddingBottom
PaddingTop: =2
PaddingBottom: =2
LayoutGap: =6
```

### Asigna siempre valores explícitos a las propiedades visuales

Power Apps aplica su propio estilizado por defecto cuando se omite una propiedad. Esos defaults son genéricos y no coincidirán con el diseño. Para cada control en la salida, **asigna explícitamente TODAS las propiedades** de las listas aplicables de abajo. Nunca las omitas.

**Controles interactivos** — `Classic/Button`, `Classic/TextInput`, `Classic/ComboBox`, `Classic/CheckBox`, `Classic/Radio`, `Gallery`:

Cada instancia debe incluir TODOS:
- `HoverFill`, `HoverColor`, `HoverBorderColor`
- `PressedFill`, `PressedColor`, `PressedBorderColor`
- `DisabledFill`, `DisabledColor`, `DisabledBorderColor`
- `BorderStyle`, `BorderThickness`
- `FontWeight`
- `FocusedBorderColor`, `FocusedBorderThickness`
- `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight`

Además:
- `Classic/CheckBox`: debe tener también `CheckboxSize`, `CheckboxBackgroundFill`, `CheckmarkFill`, `CheckboxBorderColor` (ver la sección dedicada abajo)
- `Classic/Radio`: debe tener `Height: =39` como mínimo, más `RadioSize`, `RadioBackgroundFill`, `RadioBorderColor`, `RadioSelectionFill` (ver la sección dedicada abajo)
- `Classic/ComboBox`: debe tener también `ChevronFill`, `ChevronHoverFill`, `ChevronBackground`, `ChevronHoverBackground`, `SelectionColor`, `SelectionFill`, y `FocusedBorderThickness: =0` (ver la sección dedicada abajo)

> **Importante:** no todos los controles soportan cada propiedad de arriba. Cruza siempre con `controls-reference.md` para el tipo de control exacto. Asigna solo las propiedades que aparezcan allí. Nunca lo adivines. Las propiedades no listadas provocan PA2108.

**Controles de visualización** — `Label@2.5.1`:

Las labels no son interactivas — NO añadas HoverColor, PressedColor ni estados similares. Asigna siempre:
- `FontWeight` — `=FontWeight.Normal`, `.Semibold` o `.Bold` (nunca lo omitas)
- `Align` — `=Align.Left`, `.Center` o `.Right`
- `VerticalAlign` — solo en labels de altura fija; omítelo cuando `AutoHeight: =true`

**Controles estructurales** — `GroupContainer`, `Rectangle`, `Image`:
- `GroupContainer`: asigna siempre `Fill` (usa `=RGBA(0,0,0,0)` para transparente), y `DropShadow: =DropShadow.None` salvo que la sombra sea intencional.
- `Rectangle`: asigna siempre `BorderStyle`, `BorderThickness`, `BorderColor`.
- `Image`: asigna siempre `BorderStyle: =BorderStyle.None`, `BorderThickness: =0`, `BorderColor: =RGBA(0,0,0,0)` para imágenes decorativas.

**Valores por defecto de referencia rápida cuando el diseño no da una guía concreta:**

| Propiedad | Valor por defecto |
|---|---|
| `HoverFill` en inputs | `=RGBA(248,249,250,1)` |
| `HoverFill` en botones fantasma/transparentes | `=RGBA(0,0,0,0.04)` |
| `PressedFill` en botones fantasma/transparentes | `=RGBA(0,0,0,0.08)` |
| `DisabledFill` en inputs y botones sólidos | `=RGBA(240,240,245,1)` |
| `DisabledColor` en inputs y botones sólidos | `=RGBA(180,180,195,1)` |
| `DisabledBorderColor` en controles con borde | `=RGBA(220,220,230,1)` |
| `FontWeight` cuando no es bold ni semibold | `=FontWeight.Normal` |
| `Align` cuando no es centrado | `=Align.Left` |
| `BorderStyle` cuando no hay borde | `=BorderStyle.None` con `BorderThickness: =0` |
| `HoverBorderColor` en inputs | `=Self.BorderColor` |
| `PressedBorderColor` en inputs | `=Self.BorderColor` |

### Pulido visual
- Radio de borde: 15–20 para cards, 10–15 para botones/pills, 30+ para elementos circulares
- Contenedores de iconos: `Fill: =RGBA(255,255,255,0.5)`, tamaño 35×35, `RadiusBottomLeft/Right/TopLeft/Right: =10`
- Pills de estado: `BorderColor` con el color de acento + `Fill: =ColorFade(Self.BorderColor, 80%)`

---

## CLASSIC/CHECKBOX — PROPIEDADES DE TAMAÑO Y ESTILO REQUERIDAS

**`CheckboxSize` — asígnalo siempre explícitamente.** Power Apps asigna a `CheckboxSize` un default de 40, que es demasiado grande para formularios compactos. Sin esta propiedad, todo checkbox se renderiza como un cuadrado de 40px que choca con el diseño.

Asigna siempre `CheckboxSize` para que coincida con el diseño. Para formularios compactos (tamaño de fuente 10–12), valores típicos:
- `=16` — formularios pequeños y apretados
- `=18` — formularios compactos estándar (default recomendado)
- `=20` — algo más grande, sigue compacto

**Dos propiedades de borde separadas en `Classic/CheckBox`:**
- `CheckboxBorderColor` — borde solo del **cuadrado de la casilla**
- `BorderColor` — borde alrededor de **todo el control** (casilla + texto de label combinados)

Son distintos. `BorderColor` debe asignarse transparente (`=RGBA(0,0,0,0)`), ya que el borde exterior del control casi nunca es visible en los diseños.

**Propiedades requeridas en todo `Classic/CheckBox@2.1.0`:**
```yaml
CheckboxSize: =18               # REQUIRED — default 40 is too large
CheckboxBackgroundFill: =If(Self.Value, RGBA(42,43,71,1), RGBA(255,255,255,1))
CheckmarkFill: =RGBA(255,255,255,1)
CheckboxBorderColor: =RGBA(180,180,195,1)  # tick box square border
Color: =RGBA(60,60,80,1)        # text label color
Fill: =RGBA(255,255,255,1)      # control background
BorderColor: =RGBA(0,0,0,0)     # outer control border — set transparent
BorderStyle: =BorderStyle.None
BorderThickness: =0
FocusedBorderColor: =RGBA(100,100,120,1)
FocusedBorderThickness: =2
HoverBorderColor: =RGBA(100,100,120,1)
HoverFill: =RGBA(0,0,0,0)
PressedBorderColor: =RGBA(100,100,120,1)
PressedColor: =RGBA(42,43,71,1)
PressedFill: =RGBA(0,0,0,0.04)
FontWeight: =FontWeight.Normal
```
> `Classic/CheckBox` NO soporta `DisabledBorderColor`, `DisabledColor` ni `DisabledFill` — omítelos.

---

## CLASSIC/RADIO — ALTURA MÍNIMA PARA EVITAR SCROLLBAR

**`Classic/Radio@2.3.0` (layout horizontal) tiene una altura mínima que depende tanto de `RadioSize` como del `Size` de la fuente.** Por debajo de ese mínimo aparece una scrollbar. El mínimo NO es un valor plano — escala con ambas propiedades.

**Fórmula (solo layout horizontal):**
```
min_height = 39 + floor(max(0, RadioSize - 16) / 2) + max(0, fontSize - 10)
```

**Tabla de referencia:**

| RadioSize | Font Size | Min Height |
|---|---|---|
| ≤16 | ≤10 | 39 |
| ≤16 | 11 | 40 |
| ≤16 | 12 | 41 |
| 18 | ≤10 | 40 |
| 18 | 11 | 41 |
| 18 | 12 | 42 |
| 18 | 13 | 43 |
| 20 | ≤10 | 41 |
| 20 | 11 | 42 |

**Regla:** calcula siempre la altura mínima correcta con la fórmula de arriba antes de asignar `Height` a cualquier `Classic/Radio@2.3.0` con `Layout: =Layout.Horizontal`. Todo contenedor padre con una `Height` fija explícita debe ser también ≥ este mínimo.

**Propiedades requeridas en todo `Classic/Radio@2.3.0`:**
```yaml
Height: =41                     # EXAMPLE for RadioSize=18, fontSize=11 — compute per formula above
RadioSize: =18                  # size of the radio circle
RadioBackgroundFill: =RGBA(255,255,255,1)
RadioBorderColor: =RGBA(160,160,180,1)
RadioSelectionFill: =RGBA(42,43,71,1)
Color: =RGBA(42,43,71,1)
Fill: =RGBA(255,255,255,1)
BorderStyle: =BorderStyle.None
BorderThickness: =0
BorderColor: =RGBA(180,180,195,1)
FocusedBorderColor: =RGBA(100,100,120,1)
FocusedBorderThickness: =2
HoverFill: =RGBA(0,0,0,0)
FontWeight: =FontWeight.Normal
```

> `Classic/Radio` NO soporta `HoverBorderColor` — omítelo. Cruza con `controls-reference.md` antes de añadir `PressedFill`, `PressedBorderColor`, `PressedColor`, `DisabledBorderColor`, `DisabledColor` o `DisabledFill`.

### Estilizado de Classic/ComboBox

Todo `Classic/ComboBox@2.4.0` debe tener las siguientes propiedades de chevron y selección asignadas explícitamente, además de las propiedades estándar de control interactivo listadas arriba.

```yaml
ChevronFill: =RGBA(100,100,120,1)
ChevronHoverFill: =RGBA(100,100,120,1)
ChevronBackground: =RGBA(0,0,0,0)
ChevronHoverBackground: =RGBA(0,0,0,0.04)
SelectionColor: =RGBA(42,43,71,1)
SelectionFill: =RGBA(0,0,0,0.05)
FocusedBorderThickness: =0
```

- `ChevronBackground: =RGBA(0,0,0,0)` hace transparente el área del chevron para un look moderno.
- `ChevronHoverFill` debe coincidir con `ChevronFill` si no se desea cambio de color al hover.
- `FocusedBorderThickness: =0` elimina el borde punteado que aparece al abrir el dropdown de un ComboBox. En otros controles esta propiedad controla el grosor del anillo de focus, pero en ComboBox produce un contorno punteado no deseado.
- Envuélvelo siempre en el **Radius Wrapper Pattern** para esquinas redondeadas — ComboBox no soporta propiedades `RadiusXxx`.

### Conversión de Dropdown a Radio/Checkbox

Al generar una pantalla que contenga controles ComboBox o DropDown con un conjunto pequeño y fijo de opciones, recomienda convertirlos para reducir los clics del usuario:

- **2–3 opciones (selección única):** recomienda `Classic/Radio@2.3.0` en lugar de un ComboBox/DropDown. Un Radio permite al usuario seleccionar en un clic en lugar de dos (abrir dropdown + seleccionar). Preséntalo al usuario: "Este campo tiene solo N opciones — ¿quieres que use un Radio en lugar de un ComboBox para menos clics?"
- **2–4 opciones (multiselección):** recomienda controles `Classic/CheckBox@2.1.0` individuales en lugar de un ComboBox multiselección. Cada checkbox es directamente visible y clicable.

Esto es **solo una recomendación** — no conviertas automáticamente sin preguntar al usuario. Si el usuario lo rechaza, usa el ComboBox tal como estaba diseñado.

---

## PATRONES DE DISEÑO RESPONSIVO

### Sistema de tamaño de pantalla

Power Apps ofrece una propiedad `Size` en cada pantalla, calculada a partir de la anchura de la pantalla y los breakpoints de la app.

**Breakpoints por defecto:**
| Size | Enum ScreenSize | Rango de anchura |
|---|---|---|
| 1 | `ScreenSize.Small` | < 600px (móvil) |
| 2 | `ScreenSize.Medium` | 600–900px (tablet vertical) |
| 3 | `ScreenSize.Large` | 900–1200px (tablet horizontal) |
| 4 | `ScreenSize.ExtraLarge` | > 1200px (escritorio) |

Referéncialo en fórmulas como `ScreenName.Size` (sustituye `ScreenName` por el nombre real de la pantalla). Usa `ScreenSize.Large`, etc., para comparaciones legibles.

> **`ScreenName.Size` vs `App.ActiveScreen.Size`:**
> - **Usa `ScreenName.Size`** siempre que el usuario pida diseño responsivo o la salida sea una pantalla completa. Esto se ata directamente al viewport y habilita el escalado responsivo real. Pide al usuario el nombre de la pantalla si no lo conoces.
> - **`App.ActiveScreen.Size` solo es aceptable** al generar un componente o sección parcial (paste target a o c) Y el usuario no ha mencionado responsividad. En ese caso el nombre de la pantalla es desconocido y `App.ActiveScreen.Size` es un fallback razonable.
> - **Nunca uses `App.ActiveScreen.Size` en una salida de pantalla completa (paste target b)** — usa siempre el nombre real de la pantalla ahí.
> - Si ves `App.ActiveScreen.Size` en código de referencia/plantilla, ten en cuenta esta distinción y sustituye `ScreenName.Size` en la salida generada siempre que se conozca el nombre de la pantalla.

### Patrón de dimensionado condicional

Para alternar entre altura/anchura flexible y estática según el tamaño de pantalla:

```yaml
# Flexible height on large screens, static on small:
FillPortions: =If(HomeScreen.Size >= ScreenSize.Large, 1, 0)
Height: =If(HomeScreen.Size >= ScreenSize.Large, 0, galleryContainer.Height + 20)
```

- `FillPortions: =1` = flexible (llena el espacio restante)
- `FillPortions: =0` = estático (usa la propiedad Height/Width explícita)

### Alternancia de FillPortions para altura responsiva

Para secciones que deben hacer flex en escritorio pero tener altura fija en móvil:
```yaml
FillPortions: =If(HomeScreen.Size >= ScreenSize.Large, 1, 0)
LayoutMinHeight: =350
Height: =If(HomeScreen.Size < ScreenSize.Large, taskSection.Height + inspectionSection.Height, 0)
```

### Visibilidad condicional
```yaml
Visible: =HomeScreen.Size >= ScreenSize.Large   # show only on tablet/desktop
Visible: =HomeScreen.Size < ScreenSize.Large    # show only on mobile/small
```

### Padding / espaciado condicional
```yaml
PaddingLeft: =If(HomeScreen.Size >= ScreenSize.Large, 32, 24)
PaddingRight: =If(HomeScreen.Size >= ScreenSize.Large, 32, 24)
```

### Patrones responsivos de galería

**Altura dinámica de galería** (la galería se expande para mostrar todos sus items):
```yaml
Height: =Self.AllItemsCount * Self.TemplateHeight
```

**WrapCount según tamaño de pantalla** (2 columnas en móvil, 4 en escritorio):
```yaml
WrapCount: =If(HomeScreen.Size < ScreenSize.Large, 2, 4)
TemplateSize: =Self.Width / Self.WrapCount - 1
```
Nota: resta 1 al TemplateSize para evitar scroll por redondeo decimal.

**Reducir items visibles en pantallas pequeñas:**
```yaml
Items: =FirstN(colSampleData, If(HomeScreen.Size < ScreenSize.Large, 7, 14))
```

### Propiedad Wrap — 3 criterios

Al usar `LayoutWrap: =true` en un contenedor, las TRES condiciones deben estar asignadas o el layout se romperá:

1. **Anchura de cada item** — debe estar asignada para que el wrap sepa cuándo saltar a la siguiente línea
2. **Altura de cada item** — debe estar asignada o los items colapsan
3. **Altura del padre** — estática (añade `LayoutOverflowY: =LayoutOverflow.Scroll`) o dinámica (suma de las alturas de todos los items hijos + padding + gaps)

### Patrón de contenido desplazable

Para pantallas que deben hacer scroll solo en el área de contenido (cabecera fija + cuerpo desplazable):
```yaml
- staticHeaderContainer:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      Height: =75
      FillPortions: =0
      # ... header content
- scrollableContent:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =1
      LayoutOverflowY: =LayoutOverflow.Scroll
      # ... scrollable content
```

### Principio de diseño de pantallas

No existe el requisito de que todo el contenido deba estar en una sola pantalla. Si un formulario tiene muchos campos, usa un formulario multi-pantalla. Si un dashboard es complejo, muestra solo los KPIs más importantes en la pantalla principal. Los layouts bien espaciados, con padding y modernos importan más que meter todo en una vista.

---

## PATRONES DE COMPONENTES

Son patrones probados y reutilizables extraídos de YAML real de Canvas Apps en producción. Úsalos directamente — no reinventes la rueda.

---

### Patrón de botón personalizado

El patrón estándar de botón en Canvas Apps NO es el control Button nativo. Es un GroupContainer `ManualLayout` estilizado como botón, con una Image de icono SVG y un overlay transparente de `Classic/Button` para la interacción. Esto da control total sobre forma, color, icono y estados hover/pulsado.

```yaml
- customButtonName:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      DropShadow: =DropShadow.None
      Fill: =RGBA(42,43,71,1)          # button background color
      Height: =35
      Width: =150
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
    Children:
      - btnLabel:
          Control: Label@2.5.1
          Properties:
            Color: =Color.White
            Font: =Font.Lato
            FontWeight: =FontWeight.Semibold
            Height: =Parent.Height
            Width: =Parent.Width
            Size: =10
            PaddingLeft: =5 + btnIcon.X + btnIcon.Width   # offsets text past the icon
            Text: ="Button Label"
      - btnIcon:
          Control: Image@2.2.3
          Properties:
            Height: =25
            Width: =25
            X: =10
            Y: =(Parent.Height - Self.Height)/2
            Image: |-
              ="data:image/svg+xml;utf8, " & EncodeUrl(
                  Substitute(
                      "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' fill='currentColor' viewBox='0 0 16 16'>
                <path d='M8 4a.5.5 0 0 1 .5.5v3h3a.5.5 0 0 1 0 1h-3v3a.5.5 0 0 1-1 0v-3h-3a.5.5 0 0 1 0-1h3v-3A.5.5 0 0 1 8 4z'/>
              </svg>",
                      "currentColor", "RGBA(246,246,250,1)"
                  )
              )
      - btnClickOverlay:
          Control: Classic/Button@2.2.0
          Properties:
            Text: =""
            Fill: =RGBA(0,0,0,0)
            HoverFill: =RGBA(0,0,0,0.08)
            PressedFill: =RGBA(0,0,0,0.16)
            HoverColor: =RGBA(255,255,255,1)
            PressedColor: =RGBA(255,255,255,1)
            BorderStyle: =BorderStyle.None
            BorderColor: =RGBA(0,0,0,0)
            HoverBorderColor: =RGBA(0,0,0,0)
            PressedBorderColor: =ColorFade(RGBA(0,188,242,1), 50%)
            FocusedBorderColor: =ColorFade(Self.Fill, 75%)
            DisabledBorderColor: =RGBA(0,0,0,0)
            Height: =Parent.Height
            Width: =Parent.Width
            OnSelect: =false   # replace with actual action
```

**Botón pequeño de solo icono** (círculo de 25×25, usado para acciones de fila y nav):
```yaml
- iconButton:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(42,43,71,1)
      Height: =25
      Width: =25
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
    Children:
      - iconImg:
          Control: Image@2.2.3
          Properties:
            Height: =Parent.Height
            Width: =Parent.Width
            PaddingTop: =3
            PaddingBottom: =3
            Image: |-
              ="data:image/svg+xml;utf8, " & EncodeUrl(
                  Substitute("<svg ...>...</svg>", "currentColor", "RGBA(246,246,250,1)")
              )
      - iconOverlay:
          Control: Classic/Button@2.2.0
          Properties:
            Text: =""
            Fill: =RGBA(0,0,0,0)
            HoverFill: =RGBA(0,0,0,0.1)
            PressedFill: =RGBA(0,0,0,0.2)
            BorderStyle: =BorderStyle.None
            Height: =Parent.Height
            Width: =Parent.Width
            OnSelect: =false
```

---

### Patrón de icono SVG inline

**Es el ÚNICO método de iconos aprobado.** Nunca uses `Classic/Icon@2.5.0` — su set de iconos se ve anticuado y socava la estética moderna que esta skill existe para producir. Todos los iconos deben usar `Image@2.2.3` con un data URI de SVG inline.

Preferencia de fuente: Bootstrap Icons (`bi-*`) para iconos de UI comunes; construye paths SVG personalizados para lo que no esté en Bootstrap Icons. La llamada `Substitute` reemplaza `currentColor` por un string de color para que puedas tintar cualquier SVG.

```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(
          "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16' fill='currentColor' viewBox='0 0 16 16'>
    <path d='...'/>
  </svg>",
          "currentColor", "RGBA(42,43,71,1)"
      )
  )
```

**SVG desde un campo de datos** (p. ej., un item de galería guarda el string SVG):
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(ThisItem.Icon, "currentColor", "RGBA(42,43,71,1)")
  )
```

**Tintado condicional del icono** (estado seleccionado vs. no seleccionado):
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(
          If(CurrentMenuID = ThisItem.MenuID, ThisItem.MenuIcon_Selected, ThisItem.MenuIcon),
          "currentColor",
          If(CurrentMenuID = ThisItem.MenuID, "RGBA(42,43,71,1)", "RGBA(192,194,197,1)")
      )
  )
```

> **Regla SVG:** los strings SVG que contienen `xmlns='http://...'` usan siempre el formato multilínea `|-` por los dos puntos de la URL (ver la Regla de dos puntos/hash arriba).

---

### Patrón de sidebar colapsable

Gobierna toda la sidebar con una única variable booleana `varMenuExpanded`. Establece el estado inicial en el `OnVisible` de la pantalla.

**OnVisible de la pantalla:**
```yaml
OnVisible: |-
  =Set(varMenuExpanded, false);
  Set(CurrentMenuID, 1)
```

**Contenedor de la sidebar** (anchura fija, alterna entre colapsado/expandido):
```yaml
- sidebarContainer:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      FillPortions: =0
      Width: =If(varMenuExpanded, 200, 60)
      Height: =Parent.Height
      LayoutDirection: =LayoutDirection.Vertical
      DropShadow: =DropShadow.None
      Fill: =Color.White
```

**Botón toggle de hamburguesa** (colapsa/expande al hacer clic):
```yaml
OnSelect: =Set(varMenuExpanded, !varMenuExpanded)
```

**Elementos que reaccionan al estado de la sidebar:**
```yaml
# Label: visible only when expanded
Visible: =varMenuExpanded

# Icon: centered when collapsed, left-aligned when expanded
X: =If(varMenuExpanded, 10, (Parent.Width - Self.Width)/2)

# Container alignment
LayoutJustifyContent: =If(varMenuExpanded, LayoutJustifyContent.Start, LayoutJustifyContent.Center)
PaddingLeft: =If(varMenuExpanded, 10, 0)

# Logo: swap between full logo and icon-only logo
Image: =If(varMenuExpanded, 'AppLogoFull', 'AppLogoIcon')
```

**Gallery como menú de nav** — guarda todos los items de nav en una `Table()` inline dentro del `Items` de la galería:

```yaml
Items: |-
  =Table(
      {
          MenuIcon_Selected: "<svg ...filled version...>",
          MenuIcon: "<svg ...outline version...>",
          MenuName: "Home",
          MenuID: 1,
          PageNavigation: HomeScreen
      },
      {
          MenuIcon_Selected: "<svg ...>",
          MenuIcon: "<svg ...>",
          MenuName: "Tasks",
          MenuID: 2,
          PageNavigation: TasksScreen
      }
  )
TemplateSize: =60
TemplatePadding: =0
Height: =Self.AllItemsCount * (Self.TemplateHeight + Self.TemplatePadding)
```

**Resaltado del item de menú activo** — dentro de la plantilla de la galería:
```yaml
# Background: highlighted when active
Fill: =If(CurrentMenuID = ThisItem.MenuID, RGBA(240,239,248,1), RGBA(0,0,0,0))

# Label color
Color: =If(CurrentMenuID = ThisItem.MenuID, RGBA(42,43,71,1), RGBA(192,194,197,1))

# Navigation on click
OnSelect: =Navigate(ThisItem.PageNavigation); Set(CurrentMenuID, ThisItem.MenuID)
```

**Barra de acento izquierda** (indicador pill de 5px, solo radio derecho, visible solo en el item activo):
```yaml
- activeAccentBar:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(42,43,71,1)
      Width: =5
      Height: =Parent.Height * 80%
      Y: =(Parent.Height - Self.Height)/2
      RadiusTopRight: =20
      RadiusBottomRight: =20
      RadiusTopLeft: =0
      RadiusBottomLeft: =0
      Visible: =CurrentMenuID = ThisItem.MenuID
      DropShadow: =DropShadow.None
```

---

### Patrón de barra de tabs (tabs horizontales con subrayado)

Una barra de tabs horizontal que alterna secciones de contenido en la misma pantalla. Construida con una `Gallery@2.15.0 (Horizontal)` dentro de un contenedor AutoLayout vertical. Cada item de tab tiene una label, un subrayado de estado activo y un overlay de botón transparente para gestionar el tap. Un divisor de 1px queda debajo de la galería.

**Plantilla:** `templates/tab-bar.yaml`

**Variable de estado:** `varActiveTab` (Integer) — guarda el `TabID` del tab activo actualmente.

**OnVisible de la pantalla — inicializar estado y datos de tabs:**
```yaml
OnVisible: |-
  =Set(varActiveTab, 1);
  ClearCollect(colTabs,
      {TabName: "General", TabID: 1},
      {TabName: "Details", TabID: 2},
      {TabName: "Settings", TabID: 3}
  )
```

**Galería de tabs — propiedades clave:**
```yaml
Items: =colTabs
TemplateSize: =180
```
Usa `TemplateSize: =Self.Width / Self.AllItemsCount - 1` para tabs de anchura igual auto-ajustada. Usa un `TemplateSize` fijo (p. ej., 180) cuando las labels de los tabs varíen en longitud — dimensiona a la label MÁS ancha más padding. NO asignes un valor demasiado pequeño para el nombre del tab más largo o se cortará.

**CRÍTICO — anchura del contenedor del item de tab dentro de la plantilla de galería:**
Dentro de una galería horizontal, el contenedor del item de plantilla DEBE usar `Width: =Parent.TemplateWidth` — NO `Width: =Parent.Width`. `Parent.Width` da la anchura completa de la galería (p. ej., 1236px), haciendo cada item de tab tan ancho como toda la galería. `Parent.TemplateWidth` da la anchura correcta por item (p. ej., 180px).

```yaml
- tabItemContainer:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Width: =Parent.TemplateWidth   # CORRECT — per-template width
      Height: =44
      # Width: =Parent.Width        # WRONG — full gallery width, not template width
```

Aplica a cualquier contenedor raíz de plantilla de galería. Usa siempre `Parent.TemplateWidth` / `Parent.TemplateHeight` para las dimensiones propias del item de plantilla.

**Dentro de cada item de plantilla de la galería:**

- `tabLabel` — `Label@2.5.1` que muestra `ThisItem.TabName`. Color y FontWeight alternan según el estado activo:
  ```yaml
  Color: =If(varActiveTab = ThisItem.TabID, RGBA(30,30,35,1), RGBA(140,140,150,1))
  FontWeight: =If(varActiveTab = ThisItem.TabID, FontWeight.Semibold, FontWeight.Normal)
  ```

- `tabUnderline` — `Rectangle@2.3.0`, de 2px de alto, visible solo en el tab activo:
  ```yaml
  Visible: =varActiveTab = ThisItem.TabID
  Height: =2
  Y: =Parent.TemplateHeight - 2
  ```

- `tabOverlay` — `Classic/Button@2.2.0` transparente que cubre toda la celda de la plantilla. Fija el tab activo al hacer clic:
  ```yaml
  OnSelect: =Set(varActiveTab, ThisItem.TabID)
  Fill: =RGBA(0,0,0,0)
  HoverFill: =RGBA(0,0,0,0.03)
  PressedFill: =RGBA(0,0,0,0.06)
  Text: =""
  ```

**Conectar la visibilidad del contenido de los tabs:**

Cada sección de contenido (contenedor, grupo de formulario, galería, etc.) que pertenezca a un tab concreto asigna su propiedad `Visible` a una comparación contra `varActiveTab`:

```yaml
# Content for Tab 1 (General)
Visible: =varActiveTab = 1

# Content for Tab 2 (Details)
Visible: =varActiveTab = 2

# Content for Tab 3 (Settings)
Visible: =varActiveTab = 3
```

Funciona en cualquier control o contenedor. Cuando `varActiveTab` cambia (vía el `OnSelect` del overlay del tab), solo se muestra la sección correspondiente. Todas las demás secciones se ocultan al instante — sin navegación.

**Reglas:**
- Inicializa siempre `varActiveTab` en el `OnVisible` de la pantalla para que el primer tab quede seleccionado al cargar.
- Usa valores de `TabID` enteros (1, 2, 3...) — no strings — para comprobaciones de igualdad rápidas.
- Coloca todas las secciones de contenido de tabs como hermanas dentro del mismo contenedor padre (normalmente el área de contenido vertical principal). Solo una es visible a la vez, así que efectivamente ocupan el mismo espacio.
- Si los colores de la barra de tabs deben coincidir con un tema personalizado, actualiza en consecuencia el Color de `tabLabel`, el Fill de `tabUnderline` y los fills hover/pulsado de `tabOverlay`.
- El divisor de tabs (`tabDivider`) es un `Rectangle@2.3.0` de 1px debajo de la galería, que aporta una línea base visual. Inclúyelo siempre.

---

### Patrón de galería de KPI cards

Galería horizontal auto-ajustada donde las cards dividen equitativamente la anchura disponible sin importar el número.

```yaml
- kpiGallery:
    Control: Gallery@2.15.0
    Variant: Horizontal
    Properties:
      LayoutMinHeight: =0
      LayoutMinWidth: =0
      TemplatePadding: =0
      TemplateSize: =Self.Width / Self.AllItemsCount - 1
      Items: |-
        =Table(
            {Title: "General Safety", Value: 12, Total: 17, Color: RGBA(191,168,255,1), Icon: "<svg ...>"},
            {Title: "Fire Safety",    Value: 8,  Total: 10, Color: RGBA(254,230,194,1), Icon: "<svg ...>"},
            {Title: "Electrical",     Value: 5,  Total: 9,  Color: RGBA(237,250,84,1),  Icon: "<svg ...>"},
            {Title: "Ward",           Value: 3,  Total: 6,  Color: RGBA(82,222,229,1),  Icon: "<svg ...>"}
        )
    Children:
      - kpiCardRoot:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            Fill: =ThisItem.Color
            Width: =Parent.TemplateWidth * 90%
            Height: =Parent.TemplateHeight * 90%
            Y: =(Parent.TemplateHeight - Self.Height)/2
            LayoutDirection: =LayoutDirection.Vertical
            PaddingTop: =20
            PaddingBottom: =20
            PaddingLeft: =20
            PaddingRight: =20
            RadiusBottomLeft: =20
            RadiusBottomRight: =20
            RadiusTopLeft: =20
            RadiusTopRight: =20
            DropShadow: =DropShadow.None
```

> `TemplateSize: =Self.Width / Self.AllItemsCount - 1` — el `-1` evita el scroll horizontal por redondeo.

---

### Patrón de barra de progreso personalizada

Dos contenedores anidados. Sin necesidad del control Progress nativo.

```yaml
- progressTrack:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Fill: =RGBA(0,0,0,0.2)      # translucent dark track
      Height: =4
      LayoutMinWidth: =0
      DropShadow: =DropShadow.None
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
    Children:
      - progressFill:
          Control: GroupContainer@1.5.0
          Variant: ManualLayout
          Properties:
            Fill: =RGBA(42,43,71,1)   # filled portion color
            Height: =Parent.Height
            Width: =Parent.Width * (ThisItem.Completed / ThisItem.Total)
            DropShadow: =DropShadow.None
            RadiusBottomLeft: =10
            RadiusBottomRight: =10
            RadiusTopLeft: =10
            RadiusTopRight: =10
```

---

### Patrón de tabla de datos (cabecera + Gallery)

Una tabla personalizada construida con una fila de cabecera estática y una galería vertical. Usa valores de `FillPortions` coincidentes en ambas para mantener las columnas alineadas.

```yaml
- tableWrapper:
    Control: GroupContainer@1.5.0
    Variant: AutoLayout
    Properties:
      LayoutDirection: =LayoutDirection.Vertical
      BorderColor: =RGBA(131,141,157,1)
      BorderThickness: =1
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
      DropShadow: =DropShadow.None
    Children:
      - tableHeader:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            Fill: =RGBA(246,246,250,1)
            FillPortions: =0
            Height: =50
            LayoutDirection: =LayoutDirection.Horizontal
            PaddingLeft: =10
            PaddingRight: =10
            DropShadow: =DropShadow.None
            RadiusBottomLeft: =0
            RadiusBottomRight: =0
            RadiusTopLeft: =0
            RadiusTopRight: =0
          Children:
            - hdrCategory:
                Control: Label@2.5.1
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =1
                  LayoutMinHeight: =0
                  Color: =RGBA(131,141,157,1)
                  Font: =Font.'Lato Light'
                  FontWeight: =FontWeight.Semibold
                  Size: =8
                  Text: ="Category"
            - hdrTitle:
                Control: Label@2.5.1
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =2            # wider column
                  LayoutMinHeight: =0
                  Color: =RGBA(131,141,157,1)
                  Font: =Font.'Lato Light'
                  FontWeight: =FontWeight.Semibold
                  Size: =8
                  Text: ="Title"
            - hdrDate:
                Control: Label@2.5.1
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  FillPortions: =1
                  LayoutMinHeight: =0
                  LayoutMinWidth: =90
                  Color: =RGBA(131,141,157,1)
                  Font: =Font.'Lato Light'
                  FontWeight: =FontWeight.Semibold
                  Size: =8
                  Text: ="Date"
            - hdrActionSpacer:
                Control: Rectangle@2.3.0
                Properties:
                  AlignInContainer: =AlignInContainer.Stretch
                  Fill: =RGBA(0,0,0,0)
                  LayoutMinHeight: =0
                  Width: =50              # matches action button column width in rows
      - tableDataContainer:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            LayoutDirection: =LayoutDirection.Vertical
            PaddingLeft: =10
            PaddingRight: =10
            DropShadow: =DropShadow.None
            RadiusBottomLeft: =0
            RadiusBottomRight: =0
            RadiusTopLeft: =0
            RadiusTopRight: =0
          Children:
            - tableGallery:
                Control: Gallery@2.15.0
                Variant: Vertical
                Properties:
                  Items: =colSampleData
                  TemplateSize: =45
                  TemplatePadding: =0
                  LayoutMinHeight: =0
                  LayoutMinWidth: =0
                  DelayItemLoading: =false
                Children:
                  - rowContainer:
                      Control: GroupContainer@1.5.0
                      Variant: AutoLayout
                      Properties:
                        Height: =Parent.TemplateHeight
                        Width: =Parent.TemplateWidth
                        LayoutDirection: =LayoutDirection.Horizontal
                        DropShadow: =DropShadow.None
                        RadiusBottomLeft: =0
                        RadiusBottomRight: =0
                        RadiusTopLeft: =0
                        RadiusTopRight: =0
                      Children:
                        - cellCategory:
                            Control: Label@2.5.1
                            Properties:
                              AlignInContainer: =AlignInContainer.Stretch
                              FillPortions: =1     # MUST match header FillPortions
                              LayoutMinHeight: =0
                              Size: =9
                              Font: =Font.Lato
                              Text: =ThisItem.Category
                        - cellTitle:
                            Control: Label@2.5.1
                            Properties:
                              AlignInContainer: =AlignInContainer.Stretch
                              FillPortions: =2     # MUST match header FillPortions
                              LayoutMinHeight: =0
                              Size: =9
                              Font: =Font.Lato
                              Text: =ThisItem.Title
                        - cellDate:
                            Control: Label@2.5.1
                            Properties:
                              AlignInContainer: =AlignInContainer.Stretch
                              FillPortions: =1     # MUST match header FillPortions
                              LayoutMinHeight: =0
                              LayoutMinWidth: =90
                              Size: =9
                              Font: =Font.Lato
                              Text: =Text(ThisItem.Date, "mm/dd/yyyy")
                        - cellAction:
                            Control: GroupContainer@1.5.0
                            Variant: AutoLayout
                            Properties:
                              FillPortions: =0
                              Width: =50           # MUST match header spacer width
                              LayoutDirection: =LayoutDirection.Horizontal
                              LayoutAlignItems: =LayoutAlignItems.Center
                              LayoutMinHeight: =0
                              DropShadow: =DropShadow.None
                              RadiusBottomLeft: =0
                              RadiusBottomRight: =0
                              RadiusTopLeft: =0
                              RadiusTopRight: =0
                  - rowDivider:
                      Control: Rectangle@2.3.0
                      Properties:
                        Fill: =RGBA(246,246,250,1)
                        Height: =1
                        Width: =Parent.TemplateWidth
                        Y: =Parent.TemplateHeight - Self.Height
                        OnSelect: =Select(Parent)
```

> **Regla de alineación de columnas:** los valores de `FillPortions` en la fila de cabecera y en la plantilla de la galería de datos DEBEN ser idénticos para cada columna. Valores discrepantes hacen que las columnas no se alineen.
> **Divisor de fila:** `Y: =Parent.TemplateHeight - Self.Height` fija el Rectangle de 1px al fondo de cada fila de la galería.

---

### Patrón de badge de estado

Un contenedor de label con forma de pill donde el borde define el tema de color y el fill se deriva automáticamente.

```yaml
- statusBadge:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      BorderColor: |-
        =Switch(
            Text(ThisItem.Status),
            "Active",   RGBA(82,222,229,1),
            "Pending",  RGBA(254,230,194,1),
            "Closed",   RGBA(191,168,255,1),
                        RGBA(192,194,197,1)    # default
        )
      BorderThickness: =2
      Fill: =ColorFade(Self.BorderColor, 80%)   # auto-derives tinted fill from border
      Height: =25
      Width: =80
      RadiusBottomLeft: =10
      RadiusBottomRight: =10
      RadiusTopLeft: =10
      RadiusTopRight: =10
      DropShadow: =DropShadow.None
    Children:
      - badgeLabel:
          Control: Label@2.5.1
          Properties:
            Align: =Align.Center
            Color: =RGBA(35,36,47,1)
            Font: =Font.Lato
            FontWeight: =FontWeight.Semibold
            Height: =Parent.Height
            Width: =Parent.Width
            Size: =9
            Text: =ThisItem.Status
```

> `Fill: =ColorFade(Self.BorderColor, 80%)` — el fill es siempre una versión tintada suave del borde. Nunca los incrustes por separado; deriva uno del otro.

---

### Patrón de contenedor circular / avatar

Fuerza un círculo perfecto: asigna los cuatro valores de radio iguales a la mitad de la anchura.

```yaml
- avatarContainer:
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Width: =40
      Height: =Self.Width          # always square
      BorderColor: =RGBA(42,43,71,1)
      BorderThickness: =2
      RadiusTopLeft: =20           # = Width / 2
      RadiusTopRight: =20
      RadiusBottomLeft: =20
      RadiusBottomRight: =20
      DropShadow: =DropShadow.None
    Children:
      - avatarImage:
          Control: Image@2.2.3
          Properties:
            Image: =User().Image    # or ThisItem.Photo
            Width: =Parent.Width
            Height: =Parent.Height
            RadiusTopLeft: =20
            RadiusTopRight: =20
            RadiusBottomLeft: =20
            RadiusBottomRight: =20
            X: =(Parent.Width - Self.Width)/2
            Y: =(Parent.Height - Self.Height)/2
```

---

### Fórmulas de autocentrado (ManualLayout)

En contenedores `ManualLayout`, usa estas fórmulas para centrar a un hijo sin conocer sus dimensiones exactas:

```yaml
# Vertical center:
Y: =(Parent.Height - Self.Height)/2

# Horizontal center:
X: =(Parent.Width - Self.Width)/2

# Both centered:
X: =(Parent.Width - Self.Width)/2
Y: =(Parent.Height - Self.Height)/2
```

---

### Interpolación de strings

Usa `$"..."` para incrustar expresiones directamente en strings — sin necesidad de concatenar con `&`:

```yaml
Text: =$"You have {outstandingCount} tasks left this month!"
Text: =$"{ThisItem.Completed} of {ThisItem.Total} | {Text(ThisItem.Completed/ThisItem.Total * 100, '##')}%"
Text: =$"Welcome, {User().FullName}"
```

> **Regla:** si un string `$"..."` contiene `:` o `#` dentro del texto literal (no dentro de `{}`), usa el formato multilínea `|-` para evitar PA1001.

---

### Funciones User()

Funciones integradas de PA para obtener el usuario con la sesión iniciada. No requiere conexión a fuente de datos.

```yaml
Image: =User().Image      # profile photo (use in Image control)
Text: =User().FullName    # display name
Text: =User().Email       # email address
```

---

### Paletas de colores

**Paleta Dark Navy / Indigo** (de una app en producción):
| Rol | Valor |
|---|---|
| Acento primario / fill de botón | `RGBA(42,43,71,1)` |
| Fondo de app/pantalla | `RGBA(246,246,250,1)` |
| Fondo de card blanca | `RGBA(255,255,255,1)` |
| Fondo de item de menú activo | `RGBA(240,239,248,1)` |
| Icono inactivo / texto atenuado | `RGBA(192,194,197,1)` |
| Divisor / borde | `RGBA(214,221,224,1)` |
| Texto de cabecera de tabla | `RGBA(131,141,157,1)` |
| Texto de cuerpo oscuro | `RGBA(35,36,47,1)` |

**Colores de acento por categoría/estado** (pastel):
| Color | Valor |
|---|---|
| Morado | `RGBA(191,168,255,1)` |
| Naranja | `RGBA(254,230,194,1)` |
| Amarillo | `RGBA(237,250,84,1)` |
| Turquesa | `RGBA(82,222,229,1)` |

**Paleta estándar de la skill** (acento azul):
| Rol | Valor |
|---|---|
| Fondo | `RGBA(35,36,47,1)` |
| Acento | `RGBA(0,142,210,1)` |

---



Consulta `controls-index.md` para el inventario completo de controles (40+ controles). La tabla de abajo lista los controles generados con más frecuencia.

**Controles Classic** (preferidos por defecto — estables, muy personalizables):

| Control | Versión | Notas |
|---|---|---|
| `Classic/Button` | `@2.2.0` | Botón classic — úsalo para botones interactivos/estilizados |
| `Classic/TextInput` | `@2.3.2` | Input de texto classic — soporta `Default`, `Mode`, `HintText`, `MaxLength` |
| `Classic/DropDown` | `@2.3.1` | Dropdown classic — selección de lista simple, soporta `Default` |
| `Classic/ComboBox` | `@2.4.0` | Combobox classic — con búsqueda + multiselección, soporta `Default` |
| `Classic/CheckBox` | `@2.1.0` | Checkbox classic — úsalo para checkboxes (NO Rectangle) |
| `Classic/Radio` | `@2.3.0` | Grupo de radios classic — soporta `Default` |
| `Classic/Toggle` | `@2.1.0` | Toggle classic — soporta `Default`, `TrueFill`, `FalseFill` |
| `Classic/Slider` | `@2.1.0` | Slider classic — soporta `Default`, `Min`, `Max` |
| `Classic/DatePicker` | `@2.6.0` | Selector de fecha classic — soporta `DefaultDate` |
| `Classic/Icon` | `@2.5.0` | Icono classic — 500+ iconos vía el enum `Icon.xxx` |

**Controles comunes** (sin división Classic/Modern):

| Control | Versión | Notas |
|---|---|---|
| `GroupContainer` | `@1.5.0` | Variantes: `AutoLayout`, `ManualLayout` |
| `Gallery` | `@2.15.0` | Variantes: `Vertical`, `Horizontal` |
| `Label` | `@2.5.1` | Sin soporte de radio de borde |
| `Image` | `@2.2.3` | Soporta radio vía `RadiusTopLeft/Right/BottomLeft/Right` |
| `Rectangle` | `@2.3.0` | **Sin soporte de radio de borde** — usa GroupContainer para formas redondeadas |
| `Circle` | `@2.3.0` | Forma de círculo |

**Controles modern de generación anterior** (separados — NO los renombres a Classic o Modern):

| Control | Versión | Notas |
|---|---|---|
| `TextInput` | `@0.0.54` | Sin propiedad `Default` — usa `Classic/TextInput@2.3.2` si necesitas `Default` |
| `ComboBox` | `@0.0.51` | Sin propiedad `SearchField` — la búsqueda está integrada |
| `Radio` | `@0.0.25` | Sin propiedad `Default` — usa `Classic/Radio@2.3.0` si necesitas `Default` |
| `DatePicker` | `@0.0.46` | Sin soporte de radio de borde |
| `NumberInput` | `@2.9.12` | Sin soporte de radio de borde |
| `Button` | `@0.0.45` | Botón Modern Fluent |

**Usa siempre el nombre y número de versión exactos mostrados arriba.** Para la lista completa incluyendo los controles modern, consulta `controls-index.md`. Las versiones derivan según PA se actualiza — si salta un PA2105, aplica la REGLA DE AUTO-REPARACIÓN DE VERSIONES de abajo. Nota: PA2105 es una advertencia (no un error) y PA auto-actualiza a la versión actual, así que no es fatal, pero debe corregirse de inmediato para mantener la skill precisa.

---

## ⚠️ TABLA DE TRAMPAS PA2108 — PROPIEDADES INVÁLIDAS CONOCIDAS

Usar cualquier propiedad de esta tabla provocará un error PA2108. **Nunca las uses.**

| Control | Propiedad inválida — NUNCA USAR | Enfoque correcto |
|---|---|---|
| `TextInput@0.0.54` | `Default` | Omítelo, o cambia a `Classic/TextInput@2.3.2`, que soporta `Default` |
| `ComboBox@0.0.51` | `SearchField` | Omítelo — la búsqueda está integrada, sin configuración |
| `Radio@0.0.25` | `Default` | Omítelo, o cambia a `Classic/Radio@2.3.0`, que soporta `Default` |
| `DatePicker@0.0.46` | `Default` | Omítelo, o cambia a `Classic/DatePicker@2.6.0` (propiedad `DefaultDate`) |
| `Rectangle@2.3.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | Rectangle NO soporta radio de borde. Envuélvelo en un `GroupContainer` con Fill+radio para una forma redondeada |
| `Label@2.5.1` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | Label NO soporta radio de borde. Envuélvelo en un `GroupContainer` con Fill+radio |
| `Gallery@2.15.0` | `Default` | Las galerías no tienen Default — fija la preselección vía `UpdateContext` o `Set` |
| `Classic/TextInput@2.3.2` | `Mode: =TextInputMode.___` | Prefijo de enum incorrecto. El enum correcto es `TextMode`, no `TextInputMode`. Usa `=TextMode.SingleLine`, `=TextMode.MultiLine` o `=TextMode.Password` |

---

## REGLA DE AUTO-REPARACIÓN DE VERSIONES

Si Power Apps Studio reporta PA2105 para cualquier control (p. ej., "GroupContainer@1.5.0 has a version older than current version of @1.6.0"):

1. Anota el nombre del control y su nueva versión a partir del mensaje PA2105.
2. Actualiza la versión en la **TABLA DE TIPOS DE CONTROL VÁLIDOS** de arriba.
3. Reemplaza **TODAS las ocurrencias** del string de versión antiguo en:
   - Este archivo (`pa-yaml-rules.md`) — la entrada de la tabla y cada ejemplo de código inline
   - `controls-reference.md` — el encabezado de sección y cualquier ejemplo de código
   - Todos los archivos `.yaml` del directorio `templates/`
4. Usa `replace_all: true` en cada archivo por eficiencia.

La misma advertencia PA2105 nunca debe ocurrir dos veces para el mismo control. Esta regla aplica a todos los controles, no solo a `GroupContainer`.

---

## WRAP=FALSE EN LABELS DE UNA LÍNEA

Toda `Label@2.5.1` que deba mostrarse en **una línea** DEBE tener `Wrap: =false` asignado explícitamente.

Power Apps asigna a `Wrap` el default `true`. En un contenedor estrecho (área de logo, item de nav en sidebar, barra de tabs, badge de estado), una label sin `Wrap: =false` se parte en dos líneas cuando el texto cabría visualmente en una línea pero la anchura asignada es apenas demasiado estrecha.

**Añade siempre `Wrap: =false` a:**
- Labels de texto del logo
- Labels de items de navegación (sidebar y nav superior)
- Labels de items de la barra de tabs
- Cabeceras de columna en tablas y galerías
- Texto de badges de estado
- Valores de métricas KPI y títulos de cards
- Labels de texto de breadcrumbs
- Labels de texto junto a botones

**Omite `Wrap` (déjalo con el default `true`) solo para:**
- Campos de descripción multilínea
- Párrafos de cuerpo de texto
- Labels de mensajes de error con longitud variable y desconocida

**Regla de dimensionado para labels de una línea en contenedores horizontales:**
NO asignes una `Width` fija que encaje justo con el texto esperado — incluso 1px demasiado estrecha dispara el wrap. En su lugar usa `FillPortions: =1` (llena el espacio restante del contenedor) o `Width: =Parent.Width` con `Wrap: =false`. Usa una `Width` fija solo cuando la label es un elemento de tamaño fijo como una pill de estado o un badge.

---

## PATRÓN DE BOTÓN OVERLAY — MANUALLAYOUT OBLIGATORIO

Un **botón overlay transparente** es un `Classic/Button@2.2.0` con:
- `Fill: =RGBA(0,0,0,0)` — completamente transparente
- `BorderStyle: =BorderStyle.None` / `BorderThickness: =0`
- `Width: =Parent.Width` y `Height: =Parent.Height` (o coincidiendo exactamente con el padre)
- Posicionado en `X: =0`, `Y: =0` para interceptar taps en toda el área del padre

**REGLA: cualquier `GroupContainer` que contenga un botón overlay transparente como hijo DEBE ser `Variant: ManualLayout`.** Elimina `LayoutDirection` y todas las propiedades AutoLayout de ese contenedor.

**Por qué:** en un contenedor AutoLayout, los hijos se apilan secuencialmente en el eje de layout. Un botón overlay con `Height: =Parent.Height` (fija, altura completa del contenedor) se apila junto a sus hermanos y consume toda la altura del contenedor, dejando a los controles hermanos con 0px de altura. Se vuelven invisibles. El overlay parece ser lo único en el contenedor.

**Cómo convertir:**
1. Cambia `Variant: AutoLayout` → `Variant: ManualLayout`
2. Elimina: `LayoutDirection`, `LayoutAlignItems`, `LayoutJustifyContent`, `LayoutGap`, `LayoutWrap`, `LayoutMinWidth`, `LayoutMinHeight`, `PaddingLeft`, `PaddingRight`
3. Elimina `AlignInContainer` de todos los hijos (no aplica en ManualLayout)
4. Añade `X`, `Y`, `Width`, `Height` explícitos a cada hijo
5. Coloca el botón overlay **EL ÚLTIMO** en la lista `Children:` — los controles se renderizan en orden de árbol, así que el último = encima

**Patrón estándar de overlay en ManualLayout:**
```yaml
- itemContainer:          # ManualLayout — has overlay child
    Control: GroupContainer@1.5.0
    Variant: ManualLayout
    Properties:
      Width: =Parent.Width
      Height: =44
      Fill: =RGBA(0,0,0,0)
      DropShadow: =DropShadow.None
    Children:
      - itemLabel:        # visible content — explicit X/Y/W/H
          X: =12
          Y: =0
          Width: =Parent.Width - 24
          Height: =40
      - itemUnderline:    # decorative element — pinned to bottom
          X: =0
          Y: =42
          Width: =Parent.Width
          Height: =2
      - itemOverlay:      # LAST — renders on top of all siblings
          X: =0
          Y: =0
          Width: =Parent.Width
          Height: =44
```

**Este patrón aplica universalmente:** items de barra de tabs, items de nav en galerías, áreas de clic de cards, áreas de hit de filas de listas, y cualquier otro contenedor donde un clic deba capturarse sobre contenido visible.

---

## REGLA DE DIMENSIONADO DE LOS HIJOS DEL CONTENEDOR CON SCROLL

Cuando un `GroupContainer` tiene `LayoutOverflowY: =LayoutOverflow.Scroll`, sus **hijos directos que contienen el contenido desplazable** DEBEN usar `FillPortions: =0` (o no tener `FillPortions` asignado). **NUNCA** uses `FillPortions: =1` en contenido que deba desplazarse.

**Por qué `FillPortions: =1` anula el scroll:**
`FillPortions: =1` dimensiona al hijo para llenar exactamente la altura del viewport de scroll. El hijo no puede crecer más allá del viewport — el contenido queda fijado al tamaño del viewport y los desbordes se recortan. El viewport de scroll nunca se activa.

**Patrón de scroll correcto:**
```yaml
- scrollableBody:          # bounded container — has LayoutOverflowY
    Variant: AutoLayout
    FillPortions: =1       # bounded by parent (scroll viewport)
    LayoutOverflowY: =LayoutOverflow.Scroll
    Children:
      - contentWrapper:    # FillPortions: =0 — must have explicit Height
          FillPortions: =0
          Height: =formCard.Height + 48   # REQUIRED: explicit Height = child.Height + padding
          AlignInContainer: =AlignInContainer.Center  # horizontal centering if needed
          Children:
            - formCard:    # the actual content (taller than viewport = scroll activates)
```

**Patrón incorrecto (contenido recortado, el scroll nunca aparece):**
```yaml
- scrollableBody:
    FillPortions: =1
    LayoutOverflowY: =LayoutOverflow.Scroll
    Children:
      - contentWrapper:
          FillPortions: =1   # WRONG — pins child to viewport height. Scroll never activates.
```

**Los tres contratos de dimensionado (referencia):**

Todo control en un contenedor AutoLayout opera bajo un contrato:

| Contrato | Propiedades | Cuándo usarlo |
|----------|-----------|-------------|
| Fijo | `Height: =N` o `Width: =N` con `FillPortions: =0` | El control siempre mide esto |
| Flexible | `FillPortions: =N` (N > 0) | El control llena una porción proporcional del espacio restante |
| Manual | `X`, `Y`, `Width`, `Height` | El padre es ManualLayout; el control está posicionado absolutamente |

**Regla crítica:** los hijos fijos en el mismo eje AutoLayout se asignan primero. Los hijos flexibles reparten lo que queda. Si los hijos fijos suman ≥ el tamaño del padre, los hijos flexibles colapsan a 0px de altura/anchura y se vuelven invisibles. Verifica siempre que los hijos fijos no consuman por completo al padre antes de asignar `FillPortions: =1` a un hermano.

---

## FILLPORTIONS Y HEIGHT — LA REGLA COMPLETA DE DIMENSIONADO

Esta regla aplica a **todos los hijos de contenedores AutoLayout**. La misma lógica aplica a `Width` dentro de contenedores de dirección Horizontal.

| Valor de FillPortions | ¿Height requerida? | Comportamiento de PA |
|---|---|---|
| `FillPortions > 0` | **No — NO asignes Height** | PA calcula la altura proporcionalmente a partir del espacio disponible del padre |
| `FillPortions = 0` o sin asignar (default = 0) | **Sí — DEBES asignar una Height explícita** | PA usa altura estática; si está ausente, **usa 200px por defecto** |

**Los 200px nunca son correctos a propósito** para contenedores de layout. El default se dispara en silencio — sin error, sin advertencia — los contenedores simplemente se renderizan al tamaño equivocado.

### Patrón correcto (FillPortions = 0):
```yaml
topBarInner:
  FillPortions: =0
  Height: =pageTitle.Height + pageBreadcrumb.Height + 28   # padding 12+12 + gap 4
```

### Patrón correcto (FillPortions > 0):
```yaml
scrollableBody:
  FillPortions: =1
  # Height intentionally absent — PA computes it from FillPortions
  LayoutOverflowY: =LayoutOverflow.Scroll
```

### Patrón incorrecto (dispara el default de 200px):
```yaml
topBarInner:
  FillPortions: =0
  # Height missing → PA defaults to 200px
```

### Plantilla de fórmula para contenedores AutoLayout verticales:
```
Height: =PaddingTop + child1.Height + LayoutGap + child2.Height + ... + PaddingBottom
```

Para contenedores con muchos hijos, encadena las Heights de los grupos de campos:
```
Height: =fieldGroup1.Height + fieldGroup2.Height + fieldGroup3.Height + (PaddingTop + PaddingBottom + (n-1) * LayoutGap)
```

### Controles círculo/cuadrado:
Todo control con `Width` y `Height` iguales (avatar circular, icono cuadrado) DEBE tener `AlignInContainer: =AlignInContainer.Center`. El default de `AlignInContainer` es `Stretch`, que anula incluso un `LayoutAlignItems: Center` del padre — el control se estira en el eje cruzado, rompiendo su relación de aspecto (círculo → óvalo).
