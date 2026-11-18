# Índice de controles de Canvas App

Tabla maestra de búsqueda para todos los controles de Power Apps Canvas App. Se usa en la Fase 4 Paso 1 para resolver el string correcto `Control: Type@version` para cualquier elemento de UI.

**Version strings obtenidas de exportes YAML reales de PA** — son las version strings confirmadas a inicios de 2026. Si PA reporta PA2105 para algún control, aplica la REGLA DE AUTO-REPARACIÓN DE VERSIONES de `pa-yaml-rules.md` para actualizar ambos archivos.

---

## Cómo usar este índice

1. Identifica el elemento de UI a renderizar (p. ej., "checkbox", "text input", "dropdown")
2. Busca la fila en la tabla relevante de abajo
3. Revisa la columna **Preferred** — usa ese tipo de control en el YAML
4. Consulta las propiedades únicas del control en `controls-reference.md`

**Para cambiar la variante preferida** de un control: actualiza `**Classic**` → `**Modern**` (o viceversa) en la columna Preferred. La skill usará la nueva preferencia en todas las generaciones futuras.

---

## Controles con variantes Classic y Modern

Ambas variantes son controles reales y separados — NO renombres ni reemplaces uno por otro. Usa la columna **Preferred** para elegir cuál generar.

| Elemento de UI | Tipo YAML Classic | Versión Classic | Tipo YAML Modern | Versión Modern | Preferido |
|---|---|---|---|---|---|
| Entrada de texto / campo de texto | `Classic/TextInput` | `@2.3.2` | `ModernTextInput` | `@1.0.0` | **Classic** |
| Dropdown (lista simple) | `Classic/DropDown` | `@2.3.1` | _(ninguno)_ | — | **Classic** |
| Combobox (con búsqueda) | `Classic/ComboBox` | `@2.4.0` | `ModernCombobox` | `@1.0.0` | **Classic** |

> **Prefiere `Classic/ComboBox` antes que `Classic/DropDown`.** ComboBox hace todo lo que hace DropDown (lista de selección única) más búsqueda y multi-selección. Por defecto usa ComboBox para todos los elementos de UI de tipo dropdown/select salvo que el usuario pida explícitamente un DropDown simple.
| Checkbox | `Classic/CheckBox` | `@2.1.0` | `CheckBox` | `@0.0.30` | **Classic** |
| Grupo de radio buttons | `Classic/Radio` | `@2.3.0` | `ModernRadio` | `@1.0.0` | **Classic** |
| Toggle / interruptor | `Classic/Toggle` | `@2.1.0` | `Toggle` | `@1.1.5` | **Classic** |
| Slider (entrada de rango) | `Classic/Slider` | `@2.1.0` | `Slider` | `@1.0.32` | **Classic** |
| Selector de fecha | `Classic/DatePicker` | `@2.6.0` | `ModernDatePicker` | `@1.0.0` | **Classic** |
| Botón | `Classic/Button` | `@2.2.0` | `Button` | `@0.0.45` | **Classic** |
| Etiqueta de texto | `Label` | `@2.5.1` | `ModernText` | `@1.0.0` | **Label** |
| Entrada numérica | `NumberInput` | `@2.9.12` | `ModernNumberInput` | `@1.1.0` | **NumberInput** |

> **Nota:** `Classic/Button@2.2.0` también se usa como overlay de clic transparente sobre contenedores con estilos personalizados. `Button@0.0.45` (Modern) es el botón de Fluent Design con propiedades `Appearance`, `Icon` y `Layout`.

---

## Controles comunes (sin división Classic/Modern)

Tipo de control único — úsalo tal cual.

| Elemento de UI | Tipo YAML | Versión | Notas |
|---|---|---|---|
| Contenedor de layout | `GroupContainer` | `@1.5.0` | Variantes: `AutoLayout`, `ManualLayout` |
| Lista desplazable / data grid | `Gallery` | `@2.15.0` | Variantes: `Vertical`, `Horizontal` |
| Imagen | `Image` | `@2.2.3` | Soporta `RadiusTopLeft/Right/BottomLeft/Right` |
| Icono (500+ SVG integrados) | `Classic/Icon` | `@2.5.0` | La propiedad `Icon` usa el enum `Icon.xxx` |
| Rectángulo / divisor | `Rectangle` | `@2.3.0` | **Sin soporte de radio de borde** — usa GroupContainer para formas redondeadas |
| Círculo / forma ovalada | `Circle` | `@2.3.0` | |
| Formulario de edición | `Form` | `@2.4.4` | Conectado a una fuente de datos |
| Formulario de visualización (solo lectura) | `FormViewer` | `@2.3.4` | Visualización de formulario de solo lectura |
| Valoración con estrellas | `Rating` | `@2.1.0` | |
| Lista multiselección | `ListBox` | `@2.2.0` | |
| Editor de texto enriquecido | `RichTextEditor` | `@2.7.0` | |
| Temporizador | `Timer` | `@2.1.0` | |
| Renderizador de texto HTML | `HtmlViewer` | `@2.1.0` | |
| Subida de imagen / medios | `AddMedia` | `@2.2.1` | |
| Entrada de lápiz / firma | `PenInput` | `@2.3.0` | |

---

## Controles solo Modern (sin equivalente Classic)

Estos controles existen solo en la familia Modern Fluent Design. Úsalos cuando la UI los pida específicamente.

| Elemento de UI | Tipo YAML | Versión | URL de Learn (relativa) |
|---|---|---|---|
| Barra de cabecera de la app | `Header` | `@0.0.44` | modern-controls-header |
| Lista de pestañas / tabs de navegación | `ModernTabList` | `@1.0.0` | modern-control-tabs-or-tabs-list |
| Botón de información con tooltip | `ModernInformationButton` | `@1.0.0` | modern-control-info-button |
| Avatar de usuario / entidad | `Avatar` | `@1.0.40` | modern-control-avatar |
| Badge de estado | `Badge` | `@0.0.24` | modern-controls-badge |
| Barra de progreso | `Progress` | `@1.1.34` | modern-control-progress-bar |
| Spinner de carga | `Spinner` | `@1.4.6` | modern-control-spinner |
| Hipervínculo | `ModernLink` | `@1.0.0` | modern-control-link |
| Contenedor de card | `ModernCard` | `@1.0.0` | modern-control-card |

---

## Capacidades de los controles Modern

Las variantes Modern más nuevas (`ModernTextInput@1.0.0`, `ModernCombobox@1.0.0`, `ModernRadio@1.0.0`, etc.) son **completas** y soportan propiedades que los controles de la generación anterior no tienen. Cuando el usuario elija controles Modern, prefierelos antes que los Classic — NO hay fallbacks Classic obligatorios para `Default`, `IsSearchable` ni `SearchFields`.

| Tipo YAML Modern | Versión | Capacidades clave |
|---|---|---|
| `ModernTextInput` | `@1.0.0` | Soporta `Default`, `Appearance`, `BasePaletteColor`, `BorderStyle`, `BorderThickness`, `RadiusXxx`, `PaddingXxx` |
| `ModernCombobox` | `@1.0.0` | Soporta `InputTextPlaceholder`, `IsSearchable`, `SelectMultiple`, `Items`, `ItemDisplayText` |
| `ModernRadio` | `@1.0.0` | Soporta `Default` (sintaxis de record: `={Value: "Option"}`), `Items` |
| `CheckBox` | `@0.0.30` | Checkbox Modern Fluent — sizing de tokens Fluent fijo, sin propiedad `CheckboxSize` |
| `Button` | `@0.0.45` | Soporta `Appearance`, `Icon`, `Layout` — NO soporta `Fill`, `Color`, `BorderColor` (Fluent gestiona todos los colores internamente) |

> **Cuando el usuario elige Modern:** usa `ModernTextInput`, `ModernCombobox`, `ModernRadio`, `CheckBox@0.0.30`, `Button@0.0.45` para todos los controles aplicables. El ÚNICO fallback Classic obligatorio es `Classic/Button@2.2.0` para botones overlay transparentes (no hay equivalente Modern para el patrón overlay).

---

## Controles Modern de generación anterior

Son controles modern más antiguos. Son **controles separados** tanto de los Classic como de las variantes Modern más nuevas de arriba — no los confundas. Siguen siendo válidos pero tienen menos propiedades.

| Tipo YAML | Versión | Limitación clave |
|---|---|---|
| `TextInput` | `@0.0.54` | **Sin propiedad `Default`** — usa `ModernTextInput@1.0.0` o `Classic/TextInput@2.3.2` en su lugar |
| `ComboBox` | `@0.0.51` | **Sin propiedad `SearchField`** — usa `ModernCombobox@1.0.0` o `Classic/ComboBox@2.4.0` en su lugar |
| `Radio` | `@0.0.25` | **Sin propiedad `Default`** — usa `ModernRadio@1.0.0` o `Classic/Radio@2.3.0` en su lugar |
| `DatePicker` | `@0.0.46` | Sin soporte de radio de borde |
| `NumberInput` | `@2.9.12` | Generación anterior — `ModernNumberInput@1.1.0` es la variante más nueva |

---

## URLs de Microsoft Learn

URL base: `https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/`

| Control | Slug de la página |
|---|---|
| `Classic/Button` | `control-button` |
| `Classic/TextInput` | `control-text-input` |
| `Classic/DropDown` | `control-drop-down` |
| `Classic/ComboBox` | `control-combo-box` |
| `Classic/CheckBox` | `control-check-box` |
| `Classic/Radio` | `control-radio` |
| `Classic/Toggle` | `control-toggle` |
| `Classic/Slider` | `control-slider` |
| `Classic/DatePicker` | `control-date-picker` |
| `Classic/Icon` | `control-shapes-icons` |
| `Gallery` | `control-gallery` |
| `Label` | `control-text-box` |
| `Image` | `control-image` |
| `Rectangle` | `control-shapes-icons` |
| `Button` (Modern) | `modern-control-button` |
| `ModernTextInput` | `modern-control-text-input` |
| `ModernCombobox` | `modern-control-combobox` |
| `CheckBox` (Modern) | `modern-control-checkbox` |
| `ModernRadio` | `modern-controls-radio-group` |
| `Toggle` (Modern) | `modern-control-toggle` |
| `Slider` (Modern) | `modern-control-slider` |
| `ModernDatePicker` | `modern-controls-date-picker` |
| `Header` | `modern-controls-header` |
| `ModernTabList` | `modern-control-tabs-or-tabs-list` |
| `Avatar` | `modern-control-avatar` |
| `Badge` | `modern-controls-badge` |
| `Progress` | `modern-control-progress-bar` |
| `Spinner` | `modern-control-spinner` |
