# Referencia de controles de Canvas App

Referencia enfocada a los nombres de propiedades y valores de enum válidos usados en la generación de YAML.
Usa `controls-index.md` para resolver el string correcto de tipo de control para cualquier elemento de UI.

**Estructura:**
- Sección A: Propiedades comunes (aplican a la mayoría de controles — listadas una vez)
- Sección B: Propiedades únicas por control (solo lo que difiere de la Sección A)
- Sección C: Tabla de trampas PA2108 (propiedades inválidas conocidas — nunca usar)

---

## SECCIÓN A — Propiedades comunes

Aplican a la mayoría de controles. No las repitas en las secciones por control.

### Layout y dimensionado

| Propiedad | Tipo | Notas |
|---|---|---|
| `Height` | Number/Formula | Distancia entre los bordes superior e inferior |
| `Width` | Number/Formula | Distancia entre los bordes izquierdo y derecho |
| `X` | Number/Formula | Distancia del borde izquierdo al padre (solo ManualLayout) |
| `Y` | Number/Formula | Distancia del borde superior al padre (solo ManualLayout) |
| `Visible` | Boolean | `=true` o `=false` o fórmula |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `=DisplayMode.View`, `=DisplayMode.Disabled` |

### Layout de hijos de contenedor (se asigna en el HIJO, no en el contenedor)

| Propiedad | Tipo | Notas |
|---|---|---|
| `AlignInContainer` | Enum | `=AlignInContainer.Start`, `.Center`, `.End`, `.Stretch`, `.SetByContainer`. **Asígnalo siempre explícitamente en cada hijo — nunca confíes en el `LayoutAlignItems` del padre.** `Stretch` anula el tamaño fijo del eje cruzado: `Height` en contenedores horizontales, `Width` en contenedores verticales. Usa `Center`/`Start`/`End` para preservar un tamaño fijo (p. ej., avatares circulares, iconos, badges). |
| `FillPortions` | Number | `=0` (tamaño fijo), `=1`, `=2`, etc. (fill flex proporcional) |
| `LayoutMinWidth` | Number | Suelo para la anchura flex; asigna `=0` cuando uses `AlignInContainer.Stretch` |
| `LayoutMinHeight` | Number | Suelo para la altura flex; normalmente `=0` a `=30` |
| `LayoutMaxWidth` | Number | Tope para la anchura flex |

### Color y borde

| Propiedad | Tipo | Notas |
|---|---|---|
| `Fill` | Color | `=RGBA(r,g,b,a)` o `=Color.White`, etc. |
| `Color` | Color | Color del texto/primer plano |
| `BorderColor` | Color | `=RGBA(r,g,b,a)` |
| `BorderStyle` | Enum | `=BorderStyle.None`, `.Solid`, `.Dashed`, `.Dotted` |
| `BorderThickness` | Number | Píxeles |

### Estados de interacción

| Propiedad | Tipo | Notas |
|---|---|---|
| `HoverFill` | Color | Fondo al hacer hover |
| `HoverColor` | Color | Color del texto al hacer hover |
| `HoverBorderColor` | Color | Borde al hacer hover |
| `PressedFill` | Color | Fondo al pulsar/hacer clic |
| `PressedColor` | Color | Color del texto al pulsar |
| `PressedBorderColor` | Color | Borde al pulsar |
| `DisabledFill` | Color | Fondo cuando está deshabilitado |
| `DisabledColor` | Color | Texto cuando está deshabilitado |
| `DisabledBorderColor` | Color | Borde cuando está deshabilitado |
| `FocusedBorderColor` | Color | Borde cuando tiene focus (navegación por teclado) |
| `FocusedBorderThickness` | Number | Píxeles |

### Interacción y accesibilidad

| Propiedad | Tipo | Notas |
|---|---|---|
| `OnSelect` | Behavior | Fórmula a ejecutar al hacer tap/clic |
| `Tooltip` | String | Texto del tooltip |
| `TabIndex` | Number | Orden de tabulación; `=0` incluido, `=-1` excluido |
| `AccessibleLabel` | String | Etiqueta para lectores de pantalla |

### Padding

| Propiedad | Tipo | Notas |
|---|---|---|
| `PaddingTop` | Number | Píxeles |
| `PaddingBottom` | Number | Píxeles |
| `PaddingLeft` | Number | Píxeles |
| `PaddingRight` | Number | Píxeles |

---

## SECCIÓN B — Propiedades únicas por control

Solo las propiedades únicas del control, no presentes en la Sección A de arriba.

---

## GroupContainer@1.5.0

Contenedor de layout principal. Variantes: `AutoLayout` (hijos posicionados por reglas de layout) y `ManualLayout` (hijos posicionados por X/Y).

**Diferencia clave con otros controles:** el default de `DropShadow` NO es None — asígnalo siempre explícitamente.

| Propiedad | Tipo | Valores válidos |
|---|---|---|
| `LayoutDirection` | Enum | `=LayoutDirection.Horizontal`, `=LayoutDirection.Vertical` |
| `LayoutAlignItems` | Enum | `=LayoutAlignItems.Start`, `.Center`, `.End`, `.Stretch` |
| `LayoutJustifyContent` | Enum | `=LayoutJustifyContent.Start`, `.End`, `.Center`, `.SpaceBetween`, `.SpaceAround`, `.SpaceEvenly` |
| `LayoutGap` | Number | Píxeles entre hijos |
| `LayoutWrap` | Boolean | `=true` hace wrap de los hijos a nueva fila/columna |
| `LayoutOverflowX` | Enum | `=LayoutOverflow.Scroll`, `=LayoutOverflow.Hidden` |
| `LayoutOverflowY` | Enum | `=LayoutOverflow.Scroll`, `=LayoutOverflow.Hidden` |
| `LayoutMinWidth` | Number | **El default es 250 — sobrescríbelo a `=0` cuando uses Stretch** |
| `LayoutMinHeight` | Number | **El default es 100 — sobrescríbelo a `=0` cuando haga falta** |
| `DropShadow` | Enum | `=DropShadow.None`, `.Light`, `.Regular`, `.Bold`, `.ExtraBold` — **asígnalo siempre explícitamente** |
| `RadiusTopLeft` | Number | Píxeles — radio de borde de la esquina superior izquierda |
| `RadiusTopRight` | Number | Píxeles — radio de borde de la esquina superior derecha |
| `RadiusBottomLeft` | Number | Píxeles — radio de borde de la esquina inferior izquierda |
| `RadiusBottomRight` | Number | Píxeles — radio de borde de la esquina inferior derecha |

> **Patrón de badge de estado (GroupContainer redondeado):** usa un GroupContainer ManualLayout con `Fill: =ColorFade(Self.BorderColor, 80%)` — el fill se deriva automáticamente del color del borde con un fade del 80%. Asigna `BorderColor` vía Switch/If según el valor de estado. **Nunca incrustes Fill y BorderColor por separado** — la derivación con ColorFade los mantiene visualmente consistentes:
> ```yaml
> Fill: =ColorFade(badgeName.BorderColor, 80%)
> BorderColor: =Switch(ThisItem.Status, "Active", RGBA(0,168,120,1), "Pending", RGBA(255,140,0,1), RGBA(160,160,180,1))
> BorderThickness: =1
> RadiusBottomLeft: =12
> RadiusTopLeft: =12
> RadiusBottomRight: =12
> RadiusTopRight: =12
> ```

---

## Gallery@2.15.0

Lista desplazable. Variantes: `Vertical`, `Horizontal`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Items` | Table/Collection | Fuente de datos o `Table({...},{...})` inline |
| `Default` | Record/Formula | Item o record seleccionado inicialmente desde la fuente de datos — **esto SÍ es válido en Gallery** |
| `TemplateSize` | Number | **Solo escritura** — asigna la altura del item (galería Vertical) o la anchura (galería Horizontal). No puede leerse vía `Self.TemplateSize` — usa `Self.TemplateHeight` o `Self.TemplateWidth` en su lugar |
| `TemplateHeight` | Number (lectura) | Altura efectiva del item — úsala en fórmulas: `Height: =Self.AllItemsCount * Self.TemplateHeight` |
| `TemplateWidth` | Number (lectura) | Anchura efectiva del item — úsala en fórmulas para galerías horizontales |
| `TemplatePadding` | Number | Gap entre plantillas — normalmente `=0` |
| `TemplateFill` | Color | Color de fondo de cada item de plantilla de la galería |
| `WrapCount` | Number | Items por fila (Vertical) o por columna (Horizontal) |
| `LayoutMinWidth` | Number | **Hay que sobrescribirlo — el default 250 causa overflow** |
| `LayoutMinHeight` | Number | **Hay que sobrescribirlo — el default 100 causa overflow** |
| `DelayItemLoading` | Boolean | `=false` para carga inmediata |
| `Transition` | Enum | `=Transition.None`, `.Pop`, `.Push`, `.Fade` |
| `ShowScrollbar` | Boolean | |
| `ShowNavigation` | Boolean | `=true` muestra flechas de navegación en cada extremo |
| `Selectable` | Boolean | `=true` permite seleccionar items de la galería (por defecto true) |

Dentro de la plantilla de la galería: `Parent.TemplateHeight` / `Parent.TemplateWidth` para dimensiones; `ThisItem.FieldName` para datos. **Nunca uses `Parent.TemplateSize`** — es write-only y no puede leerse.

---

## Label@2.5.1

**Sin soporte de radio de borde** — envuélvelo en un GroupContainer para conseguir esquinas redondeadas.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Text` | String/Formula | `="Hello"` o `=ThisItem.Name` |
| `Font` | Enum | `=Font.Lato`, `=Font.'Lato Black'`, `=Font.'Lato Light'`, `=Font.'Open Sans'`, `=Font.Arial` |
| `Size` | Number | Tamaño de fuente en puntos (8–28 habitual) |
| `FontWeight` | Enum | `=FontWeight.Normal`, `.Semibold`, `.Bold`, `.Lighter` |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `AutoHeight` | Boolean | `=true` para hacer crecer la altura automáticamente hasta encajar el texto. **Requerido en toda Label dentro de un contenedor de grupo de campos AutoLayout** — sin él, PA Studio muestra las labels a la altura por defecto de 200px y el contenedor padre no puede calcular correctamente su Height a partir de los hijos. |
| `Wrap` | Boolean | `=true` permite que el texto haga wrap a varias líneas. Úsalo junto a `AutoHeight: =true` en labels cuyo texto pueda ocupar más de una línea. |

---

## Image@2.2.3

Soporta radio de borde directamente (a diferencia de Label y Rectangle).

| Propiedad | Tipo | Notas |
|---|---|---|
| `Image` | Image/Formula | `=SomeImage`, `=Blank()` (usa Fill para el color), o fórmula SVG. **Asigna siempre `=Blank()` cuando uses el control Image como rectángulo/fondo de color — omitirlo muestra el feo placeholder "SampleImage".** |
| `ImagePosition` | Enum | `=ImagePosition.Fill`, `.Fit`, `.Stretch`, `.Center`, `.Tile` |
| `RadiusTopLeft` | Number | Soportado directamente |
| `RadiusTopRight` | Number | Soportado directamente |
| `RadiusBottomLeft` | Number | Soportado directamente |
| `RadiusBottomRight` | Number | Soportado directamente |

Patrón SVG:
```yaml
Image: |-
  ="data:image/svg+xml;utf8, " & EncodeUrl(
      Substitute(ThisItem.Icon, "currentColor", "RGBA(246,246,250,1)")
  )
```

---

## Rectangle@2.3.0

Usado solo para divisores y separadores. **NO soporta radio de borde.**

| Propiedad | Tipo | Notas |
|---|---|---|
| `Height` | Number | `=1` para un divisor fino |
| `Width` | Formula | Normalmente `=Parent.TemplateWidth` o `=Parent.Width` |

> **Regla de dimensionado de divisores:**
> - **Divisor horizontal** (dentro de un contenedor AutoLayout vertical): `Height: =1`, `Width: =Parent.Width`, `AlignInContainer: =AlignInContainer.Stretch`. Nunca asignes `Height: =Parent.Height` a un divisor horizontal — llena todo el contenedor y se renderiza como un bloque sólido.
> - **Divisor vertical** (dentro de un contenedor AutoLayout horizontal): `Width: =1`, `Height: =Parent.Height`, `AlignInContainer: =AlignInContainer.Stretch`.
> - El grosor es siempre 1 o 2 — nunca más.
>
> **⚠️ Sin soporte de radio.** `RadiusTopLeft/Right/BottomLeft/Right` provocan PA2108. Para separadores redondeados, usa un GroupContainer con `Fill` y propiedades de radio.

---

## Circle@2.3.0

| Propiedad | Tipo | Notas |
|---|---|---|
| `Height` | Number | Diámetro |
| `Width` | Number | Diámetro (igual a Height para un círculo real) |

---

## Classic/Button@2.2.0

Botón classic completo. También usado como overlay de clic transparente.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Text` | String | Etiqueta del botón; `=""` para overlay invisible |
| `FontWeight` | Enum | `=FontWeight.Bold`, `.Semibold`, `.Normal`, `.Lighter` — usa esto, no `Bold` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Font` | Enum | Ver enum de fuentes de Label |
| `Size` | Number | Tamaño de fuente |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `AutoDisableOnSelect` | Boolean | `=true` auto-deshabilita el botón mientras `OnSelect` está en ejecución |
| `RadiusTopLeft` | Number | Soportado |
| `RadiusTopRight` | Number | Soportado |
| `RadiusBottomLeft` | Number | Soportado |
| `RadiusBottomRight` | Number | Soportado |

Para el patrón de overlay transparente:
```yaml
Fill: =RGBA(0,0,0,0)
HoverFill: =RGBA(0,0,0,0.08)
PressedFill: =RGBA(0,0,0,0.16)
BorderStyle: =BorderStyle.None
BorderColor: =RGBA(0,0,0,0)
HoverBorderColor: =RGBA(0,0,0,0)
PressedBorderColor: =RGBA(0,0,0,0)
FocusedBorderColor: =ColorFade(Self.Fill, 75%)
FocusedBorderThickness: =2
DisabledBorderColor: =RGBA(0,0,0,0)
Width: =Parent.Width
Height: =Parent.Height
```

> **Patrón de `FocusedBorderColor`:** usa `=ColorFade(Self.Fill, 75%)` para derivar el anillo de focus del propio color de fill del botón. Así, un botón overlay transparente recibe un anillo de focus transparente (invisible), mientras que un botón de color recibe un anillo tintado suave — sin colores hardcodeados.

---

## Button@0.0.45 (Botón Modern)

Botón Fluent Design con soporte de iconos.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Text` | String | Etiqueta del botón |
| `Appearance` | Enum | `='ButtonCanvas.Appearance'.Transparent`, `.Subtle`, `.Outline`, `.Secondary`, `.Primary` |
| `Icon` | String | Nombre del icono, p. ej., `="Save"`, `="ArrowReset"`, `="Add"` |
| `Layout` | Enum | `='ButtonCanvas.Layout'.IconAfter`, `.IconBefore`, `.IconOnly` |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente en puntos |
| `FontColor` | Color | Color de la fuente |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## ModernTextInput@1.0.0

Input de texto Fluent de una línea o multilínea.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | String/Formula | Valor inicial al cargar el control |
| `Text` | String (salida) | Texto actual introducido por el usuario |
| `Placeholder` | String | Texto de hint cuando está vacío |
| `Type` | Enum | `=TextInputType.SingleLine`, `.Multiline`, `.Password`, `.Search` |
| `TriggerOutput` | Enum | `=TriggerOutput.Keypress`, `FocusOut`, `Delayed` |
| `MaxLength` | Number | `=0` significa ilimitado |
| `Required` | Boolean | Habilita el estilizado de validación obligatoria |
| `ValidationState` | Enum | `=ValidationState.None` o `.Error` |
| `Appearance` | Enum | `=Appearance.Filled`, `=Appearance.FilledDarker`, `=Appearance.Outline` |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right` |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente en puntos |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Radio de esquina |
| `RadiusTopRight` | Number | Radio de esquina |
| `RadiusBottomLeft` | Number | Radio de esquina |
| `RadiusBottomRight` | Number | Radio de esquina |

---

## ModernCombobox@1.0.0

Combobox Fluent con búsqueda (multiselección opcional).

| Propiedad | Tipo | Notas |
|---|---|---|
| `Items` | Table/Collection | Fuente de datos de las opciones |
| `ItemDisplayText` | Formula | Texto mostrado por item (usa `ThisItem`) |
| `DefaultSelectedItems` | Table | Selección inicial (debe ser una tabla de records) |
| `InputTextPlaceholder` | String | Placeholder cuando no hay nada seleccionado |
| `SelectMultiple` | Boolean | `=false` para selección única |
| `IsSearchable` | Boolean | `=true` habilita la búsqueda |
| `DelayOutput` | Boolean | Retrasa `OnChange` hasta que el usuario hace clic fuera/pulsa Enter |
| `SelectedItems` | Table (salida) | Tabla de items seleccionados |
| `Selected` | Record (salida) | Record seleccionado cuando `SelectMultiple=false` |
| `SearchText` | String (salida) | Texto de búsqueda actual |
| `MultiValueDelimiter` | String | Separador para la visualización de multiselección |
| `Required` | Boolean | Habilita el estilizado de validación obligatoria |
| `ValidationState` | Enum | `ValidationState.Error` o `ValidationState.None` |
| `Appearance` | Enum | `=Appearance.Filled`, `.FilledDarker`, `.Outline` |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente en puntos |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Radio de esquina |
| `RadiusTopRight` | Number | Radio de esquina |
| `RadiusBottomLeft` | Number | Radio de esquina |
| `RadiusBottomRight` | Number | Radio de esquina |

---

## ModernRadio@1.0.0

Grupo de radios Fluent mutuamente excluyentes.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Items` | Table/Collection | Opciones de los radio buttons |
| `ItemDisplayText` | Formula | Texto mostrado por item (usa `ThisItem`) |
| `Default` | Record/Formula | Valor seleccionado inicial (debe coincidir con un item) |
| `Selected` | Record (salida) | Item seleccionado actualmente |
| `Layout` | Enum | `=Layout.Vertical` o `=Layout.Horizontal` |
| `TriggerOutput` | Enum | Fuerza la actualización de la salida |
| `Required` | Boolean | Habilita el estilizado de validación obligatoria |
| `RadioSize` | Number | Tamaño de los círculos de radio |
| `RadioBackgroundFill` | Color | Fondo del círculo de radio |
| `RadioBorderColor` | Color | Color del borde del círculo de radio |
| `RadioSelectionFill` | Color | Fill del círculo de radio seleccionado |
| `HoverColor` | Color | Color del texto al hacer hover |
| `PressedColor` | Color | Color del texto al pulsar/hacer clic |
| `DisabledColor` | Color | Color del texto en modo Disabled |
| `RadiusTopLeft` | Number | Radio de esquina |
| `RadiusTopRight` | Number | Radio de esquina |
| `RadiusBottomLeft` | Number | Radio de esquina |
| `RadiusBottomRight` | Number | Radio de esquina |

---

## CheckBox@0.0.30 (Checkbox Modern)

Control de checkbox Fluent.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Checked` | Boolean | Estado marcado actual/inicial |
| `Label` | String | Etiqueta de texto |
| `OnCheck` | Behavior | Acciones al marcar |
| `OnSelect` | Behavior | Acciones al seleccionar |
| `OnUncheck` | Behavior | Acciones al desmarcar |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente en puntos |
| `FontColor` | Color | Color del texto de la label |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## ModernDatePicker@1.0.0

Control de selector de fecha Fluent.

| Propiedad | Tipo | Notas |
|---|---|---|
| `PlaceHolder` | String | Placeholder cuando no hay fecha seleccionada |
| `Format` | Enum/String | `Short`, `LongAbbreviated`, `YearMonth`, o string personalizado |
| `Required` | Boolean | Habilita el estilizado de validación obligatoria |
| `EndDate` | Date | Fecha más tardía seleccionable |
| `StartDate` | Date | Fecha más temprana seleccionable |
| `SelectedDate` | Date (salida) | Fecha seleccionada actualmente (hora local) |
| `StartOfWeek` | Enum | `=StartOfWeek.Sunday`, `.Monday` |
| `IsEditable` | Boolean | Si es false, el usuario solo puede elegir vía calendario |
| `ValidationState` | Enum | `ValidationState.Error` o `ValidationState.None` |
| `OnChange` | Behavior | Acciones cuando cambia el valor |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente en puntos |
| `FontColor` | Color | Color del texto |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Slider@1.0.32

Slider Fluent.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Value` | Number | Valor actual del slider |
| `Min` | Number | Valor mínimo |
| `Max` | Number | Valor máximo |
| `OnChange` | Behavior | Acciones cuando cambia el valor |
| `Layout` | Enum | `=Layout.Vertical` o `=Layout.Horizontal` |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `AccessibleLabel` | String | Etiqueta para lectores de pantalla |

---

## ModernNumberInput@1.1.0

Input numérico Fluent con validación y precisión.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | Number | Valor inicial |
| `Value` | Number (salida) | Valor actual |
| `HintText` | String | Texto de placeholder/hint |
| `Min` | Number | Valor mínimo permitido |
| `Max` | Number | Valor máximo permitido |
| `Step` | Number | Paso de incremento/decremento |
| `Precision` | Enum | `=DecimalPrecision.Auto`, `'0'`, `'1'`, `'2'`, `'3'` |
| `ValidationState` | Enum | `ValidationState.None` o `ValidationState.Error` |
| `Appearance` | Enum | `=Appearance.Filled`, `.FilledDarker`, `.Outline` |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente en puntos |
| `Color` | Color | Color del texto |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Radio de esquina |
| `RadiusTopRight` | Number | Radio de esquina |
| `RadiusBottomLeft` | Number | Radio de esquina |
| `RadiusBottomRight` | Number | Radio de esquina |

---

## ModernText@1.0.0

Texto estático Fluent.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Text` | String/Formula | Contenido de texto |
| `OnSelect` | Behavior | Acciones al hacer tap/clic |
| `Wrap` | Boolean | Si el texto hace wrap |
| `AutoHeight` | Boolean | Crecimiento automático de la altura cuando el texto desborda |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente en puntos |
| `Color` | Color | Color del texto |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `RadiusTopLeft` | Number | Radio de esquina |
| `RadiusTopRight` | Number | Radio de esquina |
| `RadiusBottomLeft` | Number | Radio de esquina |
| `RadiusBottomRight` | Number | Radio de esquina |

---

## Header@0.0.44

Cabecera de app Modern.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Title` | String | Título de la cabecera |
| `IsTitleVisible` | Boolean | |
| `Logo` | Image/Formula | Logo de la cabecera |
| `IsLogoVisible` | Boolean | |
| `IsProfilePictureVisible` | Boolean | |
| `OnSelectLogo` | Behavior | Acciones al seleccionar el logo |
| `LogoMaxHeight` | Number | Altura máxima del logo |
| `LogoToolTip` | String | Tooltip del logo |
| `BasePaletteColor` | Color | Override de la paleta base del tema |
| `Style` | Enum | Variante de color de la cabecera |
| `Font` | Enum | Nombre de la familia de fuente |
| `TitleFontSize` | Number | Tamaño de fuente del título |
| `FontColor` | Color | Color del texto del título |

---

## ModernTabList@1.0.0

Navegación por pestañas.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Items` | Table/Collection | Fuente de los tabs |
| `Default` | String/Formula | Tab seleccionado al cargar |
| `Selected` | Record (salida) | Tab seleccionado actualmente |
| `OnChange` | Behavior | Se dispara cuando cambia el tab seleccionado |
| `OnSelect` | Behavior | Se dispara al seleccionar cualquier tab |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `Alignment` | Enum | `=TabListAlignment.Start`, `.Center`, `.End` |
| `Appearance` | Enum | `=TabListAppearance.Transparent`, `.Subtle`, `.Underline`, `.Filled` |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente en puntos |
| `Color` | Color | Color del texto del tab |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |
| `BorderColor` | Color | Color del borde del tab |
| `BorderStyle` | Enum | `=BorderStyle.Solid`, `.Dashed`, `.Dotted`, `.None` |
| `BorderThickness` | Number | Píxeles |
| `RadiusTopLeft` | Number | Radio de esquina |
| `RadiusTopRight` | Number | Radio de esquina |
| `RadiusBottomLeft` | Number | Radio de esquina |
| `RadiusBottomRight` | Number | Radio de esquina |

---

## Avatar@1.0.40

Avatar de usuario/entidad.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Name` | String | Usado para las iniciales y accesibilidad |
| `Image` | Image/Formula | Imagen del avatar |
| `Badge` | String | Badge de estado opcional (si el runtime lo soporta) |
| `Shape` | Enum | Circular o cuadrado |
| `Appearance` | Enum | `Colorful`, `Neutral`, `Brand` |
| `BasePaletteColor` | Color | Override de la paleta base del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente en puntos |
| `FontColor` | Color | Color del texto |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Badge@0.0.24

Badge de estado decorativo.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Content` | String/Formula | Contenido del badge |
| `Shape` | Enum | `Circular`, `Square`, `Rounded` |
| `Appearance` | Enum | Variante de apariencia del badge |
| `ThemeColor` | Enum | `Brand`, `Danger`, `Important`, `Informative`, `Severe`, `Subtle`, `Success`, `Warning` |
| `BasePaletteColor` | Color | Override de la paleta base del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente en puntos |
| `FontColor` | Color | Color del texto |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Progress@1.1.34

Barra de progreso.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Value` | Number | Valor de progreso actual |
| `Max` | Number | Valor máximo para el estado determinado |
| `Indeterminate` | Boolean | Si es true, se ignoran `Value` y `Max` determinados |
| `ProgressColor` | Enum | `Brand`, `Error`, `Warning`, `Success` |
| `Thickness` | Enum | `Medium`, `Large` |
| `Shape` | Enum | Variante de forma de la barra y el track |
| `OnChange` | Behavior | Acciones cuando cambia el valor |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `View`, `Disabled` |
| `BasePaletteColor` | Color | Override de la paleta base del tema |

---

## Spinner@1.4.6

Spinner de carga.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Label` | String | Texto de la label |
| `LabelPosition` | Enum | Posición de la label respecto al spinner |
| `AccessibleLabel` | String | Etiqueta para lectores de pantalla |
| `SpinnerSize` | Enum | Tamaño del spinner |
| `Appearance` | Enum | `Primary` o `Inverted` |
| `OnChange` | Behavior | Acciones cuando cambia el spinner (si aplica) |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente |
| `FontColor` | Color | Color del texto |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## ModernLink@1.0.0

Hipervínculo clicable.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Text` | String/Formula | Texto mostrado |
| `Url` | String | URL de destino (protocolo seguro obligatorio) |
| `Type` | Enum | `=LinkAppearance.Default` o `=LinkAppearance.Subtle` |
| `Wrap` | Boolean | Si el texto hace wrap |
| `AutoHeight` | Boolean | Crecimiento automático de la altura |
| `Align` | Enum | `=Align.Left`, `.Center`, `.Right`, `.Justify` |
| `VerticalAlign` | Enum | `=VerticalAlign.Top`, `.Middle`, `.Bottom` |
| `BasePaletteColor` | Color | Override de la paleta base del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente en puntos |
| `Color` | Color | Color del texto del enlace |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |

---

## ModernCard@1.0.0

Contenedor de card.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Direction` | Enum | Layout de card vertical u horizontal |
| `Title` | String/Formula | Título de la card |
| `Subtitle` | String/Formula | Texto complementario |
| `Description` | String/Formula | Descripción del cuerpo |
| `Image` | Image/Formula | Imagen de vista previa |
| `ImageAltText` | String/Formula | Texto alt de la imagen de vista previa |
| `HeaderImage` | Image/Formula | Avatar/icono pequeño en la cabecera |
| `HeaderImageAltText` | String/Formula | Texto alt de la imagen de cabecera |
| `ImagePosition` | Enum | Imagen antes/después de la cabecera |
| `OnSelect` | Behavior | Acción al seleccionar la card |
| `Radius` | Number | Redondez de las esquinas de la card |
| `DropShadow` | Enum | Estilo de sombra |
| `Visible` | Boolean | Muestra u oculta la card |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `View`, `Disabled` |

---

## ModernInformationButton@1.0.0

Icono de información con contenido de flyout.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Content` | String/Formula | Contenido del flyout |
| `IconColor` | Color | Color del icono de información |
| `OnSelect` | Behavior | Acciones además de abrir el flyout |
| `DisplayMode` | Enum | `=DisplayMode.Edit`, `View`, `Disabled` |
| `BasePaletteColor` | Color | Override de la paleta base del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `Size` | Number | Tamaño de fuente del flyout |
| `Color` | Color | Color del texto del flyout |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `Italic` | Boolean | |
| `Underline` | Boolean | |
| `Strikethrough` | Boolean | |

---

## Classic/TextInput@2.3.2

Input de texto completo. Soporta `Default` y `Mode`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | String/Formula | Valor pre-rellenado — **este control SÍ soporta Default** |
| `HintText` | String | Texto de placeholder/hint |
| `Mode` | Enum | `=TextMode.SingleLine`, `.MultiLine`, `.Password` |
| `Format` | Enum | `=TextInputFormat.Text`, `.Number` |
| `MaxLength` | Number | Recuento máximo de caracteres (`=0` = ilimitado) |
| `Clear` | Boolean | `=true` muestra un botón X para limpiar el input |
| `DelayOutput` | Boolean | `=true` retrasa `OnChange` hasta que el usuario deja de escribir |
| `VirtualKeyboardMode` | Enum | `=VirtualKeyboardMode.Auto`, `.Numeric`, `.Text` |
| `RadiusTopLeft` | Number | Soportado |
| `RadiusTopRight` | Number | Soportado |
| `RadiusBottomLeft` | Number | Soportado |
| `RadiusBottomRight` | Number | Soportado |

---

## TextInput@0.0.54 (Modern de generación anterior)

**Sin propiedad `Default`** — usa `Classic/TextInput@2.3.2` cuando haga falta pre-rellenar.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Appearance` | Enum | `='TextInputCanvas.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Mode` | Enum | `='TextInputCanvas.Mode'.SingleLine`, `.Multiline`, `.Password` |
| `Placeholder` | String | Texto de placeholder |
| `Height` | Number | Normalmente `=40` (una línea) o `=80`+ (multilínea) |

---

## Classic/DropDown@2.3.1

Selección de lista simple (sin búsqueda).

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | String/Formula | Valor pre-seleccionado |
| `Items` | Table | Fuente de datos o `["Option1","Option2"]` |
| `Value` | String | Nombre de la columna a mostrar, p. ej., `="Value"` |
| `AllowEmptySelection` | Boolean | `=true` permite limpiar la selección |
| `SelectionColor` | Color | Color del texto del item seleccionado |
| `SelectionFill` | Color | Fondo del item seleccionado |
| `ChevronBackground` | Color | Fondo detrás del icono de chevron |
| `ChevronFill` | Color | Color del icono de chevron |

---

## Classic/ComboBox@2.4.0

Dropdown con búsqueda y multiselección opcional. Soporta `DefaultSelectedItems`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `DefaultSelectedItems` | Record/Table | Item(s) pre-seleccionados — **usa esto, no `Default` (deprecado)** |
| `Items` | Table | Fuente de datos |
| `SearchFields` | List | Columna(s) sobre las que buscar, p. ej., `["Value"]` — **Classic/ComboBox SÍ soporta SearchFields** |
| `IsSearchable` | Boolean | `=true` habilita la búsqueda (por defecto true) |
| `SelectMultiple` | Boolean | `=true` habilita la multiselección |
| `DisplayFields` | List | Columnas a mostrar en el dropdown, p. ej., `["MyFirstColumn", "MySecondColumn"]` |
| `InputTextPlaceholder` | String | Placeholder cuando no hay nada seleccionado |
| `SelectedItems` | Table (solo lectura) | Lista de items seleccionados por interacción del usuario |
| `Selected` | Record (solo lectura) | Último item seleccionado |

**Propiedades visuales / de chevron:**

| Propiedad | Tipo | Notas |
|---|---|---|
| `ChevronFill` | Color | Color del icono de flecha chevron |
| `ChevronHoverFill` | Color | Color del icono chevron al hacer hover |
| `ChevronBackground` | Color | Fondo detrás del chevron — usa `=RGBA(0,0,0,0)` para un look moderno |
| `ChevronHoverBackground` | Color | Fondo del chevron al hacer hover |
| `SelectionColor` | Color | Color del texto de un item seleccionado en la lista del dropdown |
| `SelectionFill` | Color | Fondo de un item seleccionado en la lista del dropdown |

> **`FocusedBorderThickness` en ComboBox** se comporta distinto que en otros controles — produce un borde punteado cuando el dropdown está abierto. Asigna `FocusedBorderThickness: =0` para quitarlo y lograr un look más limpio.

> `ComboBox@0.0.51` (generación anterior) NO soporta `SearchFields`. Solo `Classic/ComboBox@2.4.0` lo soporta.

> **`Default` está deprecado** en ComboBox — usa `DefaultSelectedItems` en su lugar. `SearchField` (singular) era el nombre antiguo; el nombre correcto de la propiedad es `SearchFields` (plural).

> **Sin soporte de radio de borde** — `RadiusTopLeft/Right/BottomLeft/Right` provocan PA2108. Aplica el **Radius Wrapper Pattern**: envuélvelo en un GroupContainer con `RadiusXxx`, `BorderStyle: Solid`, `BorderThickness` y `BorderColor`. Asigna al propio control `BorderStyle: =BorderStyle.None` y `BorderThickness: =0`. Las esquinas redondeadas del contenedor recortan visualmente al hijo. Ver "Radius Wrapper Pattern" en `pa-yaml-rules.md`.

---

## ComboBox@0.0.51 (Modern de generación anterior)

**Sin propiedad `SearchField`** — la búsqueda está integrada automáticamente.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Appearance` | Enum | `='ComboboxCanvas.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Items` | Table | Fuente de datos o tabla inline |
| `Height` | Number | Normalmente `=40` |

---

## Classic/CheckBox@2.1.0

Control de checkbox real. **Usa esto en lugar de Rectangle para checkboxes.**

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | Boolean | `=true` (marcado) o `=false` (desmarcado) |
| `Value` | Boolean (solo lectura) | Estado de marcado actual — `=checkName.Value` |
| `Text` | String | Texto de la label junto al checkbox |
| `CheckboxSize` | Number | Tamaño del checkbox en píxeles |
| `CheckmarkFill` | Color | Color de la marca de verificación |
| `CheckboxBackgroundFill` | Color | Fondo de la caja del checkbox. Usa `=If(checkboxName.Value, RGBA(42,43,71,1), RGBA(255,255,255,1))` para cambiar el color según el estado marcado. |
| `CheckboxBorderColor` | Color | Borde de la caja |

> **`OnCheckFill` NO existe** — provoca PA2108. Para estilizar el estado marcado, usa `CheckboxBackgroundFill` con una fórmula `If(checkboxName.Value, ...)`.

---

## Classic/Radio@2.3.0

Grupo de radio buttons. Soporta `Default`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | String/Formula | Valor pre-seleccionado, p. ej., `="Option1"` — **este control SÍ soporta Default** |
| `Items` | Table | Opciones — normalmente `=["Option1","Option2"]` o Table inline |
| `Layout` | Enum | `=Layout.Vertical`, `=Layout.Horizontal` |
| `Value` | String (solo lectura) | Valor seleccionado actualmente |
| `RadioBackgroundFill` | Color | Fondo del círculo de cada radio button |
| `RadioBorderColor` | Color | Borde de cada círculo |
| `RadioSelectionFill` | Color | Fill del círculo de radio seleccionado |
| `RadioSize` | Number | Diámetro del círculo de radio |

> **`HoverBorderColor` NO existe** en Classic/Radio — provoca PA2108. El estilizado de hover en Radio se limita a `HoverFill` y `HoverColor`.

---

## Radio@0.0.25 (Modern de generación anterior)

**Sin propiedad `Default`** — usa `Classic/Radio@2.3.0` cuando haga falta pre-selección.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Layout` | Enum | `='RadioGroupCanvas.Layout'.Horizontal`, `.Vertical` |
| `Items` | Table | Opciones — `Table({Value:"Yes"},{Value:"No"})` |
| `Height` | Number | Normalmente `=40` |

---

## Classic/Toggle@2.1.0

Interruptor on/off. Soporta `Default`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | Boolean | `=true` (on) o `=false` (off) |
| `Value` | Boolean (solo lectura) | Estado actual del toggle |
| `TrueText` | String | Label cuando está on, p. ej., `="Yes"` |
| `FalseText` | String | Label cuando está off, p. ej., `="No"` |
| `ShowLabel` | Boolean | `=true` muestra la label `TrueText`/`FalseText` |
| `TextPosition` | Enum | `=TextPosition.Left`, `=TextPosition.Right` |
| `TrueFill` | Color | Fondo del rail cuando está on |
| `FalseFill` | Color | Fondo del rail cuando está off |
| `TrueHoverFill` | Color | Rail al hacer hover cuando está on |
| `FalseHoverFill` | Color | Rail al hacer hover cuando está off |
| `HandleFill` | Color | Color del handle deslizante |
| `RailFill` | Color | Fill del rail (estado base) |
| `RailHoverFill` | Color | Fill del rail al hacer hover |

---

## Toggle@1.1.5 (Toggle Modern)

| Propiedad | Tipo | Notas |
|---|---|---|
| `Checked` | Boolean | Estado marcado inicial |
| `Label` | String | Etiqueta de texto |
| `LabelPosition` | Enum | `='ToggleCanvas.LabelPosition'.Before`, `.After` |
| `OnCheck` | Behavior | Acciones al activar |
| `OnSelect` | Behavior | Acciones al seleccionar |
| `OnUncheck` | Behavior | Acciones al desactivar |
| `BasePaletteColor` | Color | Override del color base de paleta del tema |
| `Font` | Enum | Nombre de la familia de fuente |
| `FontSize` | Number | Tamaño de fuente en puntos |
| `FontColor` | Color | Color del texto de la label |
| `FontWeight` | Enum | `=FontWeight.Bold`, `Semibold`, `Normal`, `Lighter` |
| `FontItalic` | Boolean | |
| `FontUnderline` | Boolean | |
| `FontStrikethrough` | Boolean | |

---

## Classic/Slider@2.1.0

Slider de rango numérico. Soporta `Default`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Default` | Number | Valor inicial |
| `Min` | Number | Valor mínimo |
| `Max` | Number | Valor máximo |
| `Value` | Number (solo lectura) | Valor actual |
| `Layout` | Enum | `=Layout.Horizontal`, `=Layout.Vertical` |
| `ShowValue` | Boolean | `=true` muestra la label del valor actual |
| `ReadOnly` | Boolean | `=true` impide la interacción |
| `HandleFill` | Color | Color del círculo del handle |
| `HandleHoverFill` | Color | Handle al hacer hover |
| `HandleActiveFill` | Color | Handle al arrastrar |
| `HandleSize` | Number | Diámetro del círculo del handle |
| `RailFill` | Color | Color del rail sin rellenar |
| `RailHoverFill` | Color | Rail al hacer hover |
| `ValueFill` | Color | Porción rellenada del rail |
| `ValueHoverFill` | Color | Rail rellenado al hacer hover |

---

## Classic/DatePicker@2.6.0

Selector de fecha con popup de calendario. Soporta `DefaultDate`.

| Propiedad | Tipo | Notas |
|---|---|---|
| `DefaultDate` | Date/Formula | Fecha pre-seleccionada, p. ej., `=Today()` |
| `SelectedDate` | Date (solo lectura) | Fecha seleccionada actualmente |
| `Format` | Enum/String | `="ShortDate"` (por defecto), `="LongDate"`, o string personalizado como `="yyyy/mm/dd"` |
| `Language` | String | String de locale, p. ej., `="en-US"` |
| `StartOfWeek` | Enum | `=StartOfWeek.Sunday`, `.Monday` |
| `IsEditable` | Boolean | `=false` deshabilita la entrada manual de texto — el usuario debe usar el popup del calendario |
| `DateTimeZone` | Enum | `="UTC"` o `="Local"` — si la fecha se muestra en UTC o en hora local |
| `StartYear` | Number | Año más temprano seleccionable en el picker |
| `EndYear` | Number | Año más tardío seleccionable en el picker |
| `InputTextPlaceholder` | String | Texto instructivo cuando no se ha introducido fecha |
| `IconFill` | Color | Color de primer plano del icono de calendario |
| `IconBackground` | Color | Color de fondo del icono de calendario |

> **`IsReadOnly` NO existe en Classic/DatePicker** — la propiedad correcta es `IsEditable` (asigna `=false` para hacerlo de solo lectura). `IsReadOnly` provoca PA2108.

---

## DatePicker@0.0.46 (Modern de generación anterior)

**Sin soporte de radio de borde.**

| Propiedad | Tipo | Notas |
|---|---|---|
| `Appearance` | Enum | `='DatePickerCanvas.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Fill` | Color | `=Color.Transparent` para que se vea el fondo del contenedor |
| `Height` | Number | Normalmente `=40` |

---

## NumberInput@2.9.12 (Modern de generación anterior)

**Sin soporte de radio de borde.**

| Propiedad | Tipo | Notas |
|---|---|---|
| `Appearance` | Enum | `='NumberInput.Appearance'.FilledLighter`, `.FilledDarker`, `.Outline` |
| `Height` | Number | Normalmente `=40` |

---

## Classic/Icon@2.5.0

> **⛔ No lo uses en el YAML generado.** `Classic/Icon` produce iconos planos y anticuados que chocan con la estética moderna a la que apunta esta skill. Usa siempre `Image@2.2.3` con un SVG inline de Bootstrap Icons en su lugar (ver el patrón SVG Inline Icon en `pa-yaml-rules.md`). Este control se documenta aquí solo por referencia — no debe aparecer nunca en la salida de la skill.

500+ iconos SVG integrados. Distinto de `Image` — los iconos son vectoriales, no bitmap.

| Propiedad | Tipo | Notas |
|---|---|---|
| `Icon` | Enum | `=Icon.ChevronRight`, `=Icon.Search`, `=Icon.Add`, `=Icon.Cancel`, etc. |
| `Rotation` | Number | Grados (0–360) |
| `Color` | Color | Color del icono — asigna `Color`, no `Fill`, para la tinta del icono |

Valores de icono habituales: `Icon.Add`, `Icon.Cancel`, `Icon.Check`, `Icon.ChevronLeft`, `Icon.ChevronRight`, `Icon.ChevronDown`, `Icon.ChevronUp`, `Icon.Search`, `Icon.Edit`, `Icon.Trash`, `Icon.Filter`, `Icon.Sort`, `Icon.Settings`, `Icon.People`, `Icon.Person`, `Icon.Upload`, `Icon.Download`, `Icon.Save`, `Icon.Back`, `Icon.Forward`, `Icon.Refresh`, `Icon.Home`, `Icon.Star`, `Icon.StarFilled`, `Icon.Bell`, `Icon.Lock`, `Icon.Unlock`, `Icon.Eye`, `Icon.EyeOff`, `Icon.Info`, `Icon.Warning`, `Icon.Error`

---

## Screen

| Propiedad | Tipo | Notas |
|---|---|---|
| `Fill` | Color | Color de fondo de la pantalla |
| `OnVisible` | Behavior | Se ejecuta cuando la pantalla se vuelve visible — úsalo para `ClearCollect` |
| `OnHidden` | Behavior | Se ejecuta al navegar fuera de la pantalla |
| `Size` | Number (solo lectura) | Nivel responsive: 1=Small, 2=Medium, 3=Large, 4=ExtraLarge |

Breakpoints de `Size` (por defecto): Small <600px, Medium 600–900px, Large 900–1200px, ExtraLarge >1200px.

Referéncialo como `ScreenName.Size` en las fórmulas.

Formato YAML de la raíz de pantalla (para el paste target b — nueva pantalla):
```yaml
Screens:
  ScreenName:
    Properties:
      Fill: =RGBA(248,249,250,1)
      OnVisible: |-
        =ClearCollect(colSampleData, ...)
    Children:
      - mainContainer:
          Control: GroupContainer@1.5.0
          Variant: AutoLayout
          Properties:
            Height: =Parent.Height
            Width: =Parent.Width
```

> **AVISO: `Control: Screen` NO es válido — provoca PA2101. Usa siempre el formato de clave top-level `Screens:` de arriba.**

---

## SECCIÓN C — ⚠️ Tabla de trampas PA2108

Usar cualquier propiedad de esta tabla provoca un error PA2108 "Unknown property" en PA Studio.

| Control | Propiedad inválida | Error de PA | Enfoque correcto |
|---|---|---|---|
| `TextInput@0.0.54` | `Default` | PA2108 | Omítelo, o usa `Classic/TextInput@2.3.2`, que soporta `Default` |
| `ComboBox@0.0.51` | `SearchField` | PA2108 | Omítelo — la búsqueda está integrada, sin configuración |
| `Classic/ComboBox@2.4.0` | `SearchField` | PA2108 | Nombre singular antiguo — la propiedad correcta es `SearchFields` (plural) |
| `Classic/ComboBox@2.4.0` | `Default` | PA2108 | Deprecado — usa `DefaultSelectedItems` en su lugar |
| `Classic/ComboBox@2.4.0` | `NoSelectionText` | PA2108 | No existe — usa `InputTextPlaceholder` para el texto de placeholder |
| `Radio@0.0.25` | `Default` | PA2108 | Omítelo, o usa `Classic/Radio@2.3.0`, que soporta `Default` |
| `DatePicker@0.0.46` | `Default` | PA2108 | Omítelo, o usa `Classic/DatePicker@2.6.0`, que usa `DefaultDate` |
| `Classic/DatePicker@2.6.0` | `IsReadOnly` | PA2108 | No existe — usa `IsEditable: =false` para impedir la entrada manual de texto |
| `Rectangle@2.3.0` | `RadiusTopLeft` | PA2108 | Rectangle NO soporta radio — usa GroupContainer con Fill |
| `Rectangle@2.3.0` | `RadiusTopRight` | PA2108 | Igual — usa GroupContainer |
| `Rectangle@2.3.0` | `RadiusBottomLeft` | PA2108 | Igual — usa GroupContainer |
| `Rectangle@2.3.0` | `RadiusBottomRight` | PA2108 | Igual — usa GroupContainer |
| `Label@2.5.1` | `RadiusTopLeft/Right/BottomLeft/Right` | PA2108 | Label NO soporta radio — envuélvelo en un GroupContainer |
| `Classic/CheckBox@2.1.0` | `OnCheckFill` | PA2108 | No existe. Usa `CheckboxBackgroundFill: =If(checkName.Value, checkedColor, uncheckedColor)` en su lugar |
| `Classic/Button@2.2.0` | `Bold` | PA2108 | No existe como propiedad independiente. Usa `FontWeight: =FontWeight.Bold` en su lugar |
| `Classic/ComboBox@2.4.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | PA2108 | Classic/ComboBox NO soporta radio de borde. Usa el Radius Wrapper Pattern (ver `pa-yaml-rules.md`): envuélvelo en un GroupContainer con `RadiusXxx`, borde y fill; asigna al control `BorderStyle: None` y `BorderThickness: 0`. |
| `Classic/DatePicker@2.6.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | PA2108 | Classic/DatePicker NO soporta radio de borde. Usa el Radius Wrapper Pattern. |
| `Classic/Radio@2.3.0` | `HoverBorderColor` | PA2108 | Classic/Radio NO soporta `HoverBorderColor`. Elimínalo — el hover del radio se estiliza solo vía `HoverFill` y `HoverColor`. |
| `Classic/Radio@2.3.0` | `PressedBorderColor` | PA2108 | Classic/Radio NO soporta `PressedBorderColor`. Elimínalo — el estado pulsado del radio se estiliza solo vía `PressedFill` y `PressedColor`. |
| `ModernRadio@1.0.0` | `HoverBorderColor` | PA2108 | ModernRadio NO soporta `HoverBorderColor`. Elimínalo — el hover del radio se estiliza solo vía `HoverFill` y `HoverColor`. |
| `ModernRadio@1.0.0` | `PressedBorderColor` | PA2108 | ModernRadio NO soporta `PressedBorderColor`. Elimínalo — el estado pulsado del radio se estiliza solo vía `PressedFill` y `PressedColor`. |
| `Button@0.0.45` | `Fill` | PA2108 | El botón Modern NO soporta `Fill`. Usa `Appearance` (`Primary`, `Secondary`, `Outline`) en su lugar — Fluent gestiona todos los colores internamente. |
| `Button@0.0.45` | `Color` | PA2108 | El botón Modern NO soporta `Color`. Usa `Appearance` en su lugar — Fluent gestiona el color del texto internamente según la apariencia. |
| `Button@0.0.45` | `BorderColor` | PA2108 | El botón Modern NO soporta `BorderColor`. Usa `Appearance` en su lugar — Fluent gestiona los bordes internamente. |
| `Classic/CheckBox@2.1.0` | `RadiusTopLeft`, `RadiusTopRight`, `RadiusBottomLeft`, `RadiusBottomRight` | PA2108 | Classic/CheckBox NO soporta radio de borde. Usa el Radius Wrapper Pattern si se necesitan esquinas redondeadas. |
| `Gallery@2.15.0` | `Self.TemplateSize` en fórmulas | Valor erróneo | `TemplateSize` es write-only — no puede leerse. Usa `Self.TemplateHeight` (vertical) o `Self.TemplateWidth` (horizontal) en cualquier fórmula que referencie el tamaño del item de la galería. |
| `Gallery@2.15.0` | `DropShadow` | PA2108 | Gallery NO soporta `DropShadow`. Elimínalo por completo — las galerías no tienen propiedad de sombra. |

---

## Referencia rápida de enums

| Enum | Valores |
|---|---|
| `LayoutDirection` | `Horizontal`, `Vertical` |
| `LayoutAlignItems` | `Start`, `Center`, `End`, `Stretch` |
| `LayoutJustifyContent` | `Start`, `End`, `Center`, `SpaceBetween`, `SpaceAround`, `SpaceEvenly` |
| `AlignInContainer` | `Start`, `Center`, `End`, `Stretch`, `SetByContainer` |
| `LayoutOverflow` | `Scroll`, `Hidden` |
| `DropShadow` | `None`, `Light`, `Regular`, `Bold`, `ExtraBold` |
| `BorderStyle` | `None`, `Solid`, `Dashed`, `Dotted` |
| `Align` (texto) | `Left`, `Center`, `Right`, `Justify` |
| `VerticalAlign` | `Top`, `Middle`, `Bottom` |
| `FontWeight` | `Normal`, `Semibold`, `Bold`, `Lighter` |
| `ImagePosition` | `Fill`, `Fit`, `Stretch`, `Center`, `Tile` |
| `ScreenSize` | `Small` (1), `Medium` (2), `Large` (3), `ExtraLarge` (4) |
| `Transition` | `None`, `Pop`, `Push`, `Fade` |
| `Font` | `Lato`, `'Lato Black'`, `'Lato Light'`, `'Open Sans'`, `Arial`, `Calibri` |
| `DisplayMode` | `Edit`, `View`, `Disabled` |
| `TextMode` | `SingleLine`, `MultiLine`, `Password` |
| `TextInputFormat` | `Text`, `Number` |
| `Layout` (Radio/Slider Classic) | `Horizontal`, `Vertical` |
