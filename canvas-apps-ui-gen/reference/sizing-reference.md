# Referencia de dimensionado — controles de Power Apps Canvas

Calibrado a partir de extracción CSS en vivo contra el player canvas publicado de PA.
Usa este archivo al asignar `Size`, `Height`, `CheckboxSize`, `LayoutGap` o `PaddingX/Y`
a cualquier control. **No** lo adivines — consulta aquí primero el valor.

---

## Regla crítica: las unidades de tamaño Classic y Modern NO son equivalentes

| Familia de control | PA `Size` → CSS px | Ejemplo |
|---|---|---|
| **Classic** (`Label@2.5.1`) | PA Size × **1.333** | Size 11 → 14.67px |
| **Modern** (`ModernText@1.0.0`) | PA Size × **1.0** (directo) | Size 11 → 11px |

**Un Classic Size 11 se renderiza casi 4px más grande que un Modern Size 11.**
Para conseguir un peso visual equivalente: **Modern Size ≈ Classic Size × 1.333**
(p. ej., Classic Size 11 ≈ Modern Size 15)

---

## 1. Tamaños de texto

### 1a. Classic — Label@2.5.1 (Font.Lato)

Elemento fuente: `.appmagic-label-text` (l1 en la extracción)
Elemento contenedor: `.appmagic-label` (l2) — tiene padding de 5px arriba/abajo/izquierda/derecha

| PA Size | CSS px  | Line-height | Carácter visual | Uso recomendado |
|---------|---------|-------------|------------------|-----------------|
| 8       | 10.67px | 12.8px      | Minúsculo — difícil de leer | Solo letra pequeña, texto legal |
| 9       | 12px    | 14.4px      | Muy pequeño     | Metadatos secundarios, captions, hints |
| 10      | 13.33px | 16px        | Pequeño-compacto | Tablas de datos densas, nav labels, breadcrumbs |
| 11      | 14.67px | 17.6px      | **Cuerpo estándar** ← por defecto | Form labels, cuerpo de texto, texto por defecto de inputs |
| 12      | 16px    | 19.2px      | Medio           | Títulos de cards, texto algo enfatizado |
| 13      | 17.33px | 20.8px      | Medio-grande    | Subtítulos de sección, cabeceras de cards |
| 14      | 18.67px | 22.4px      | Grande          | Cabeceras de sección de página |
| 16      | 21.33px | 25.6px      | Extra grande    | Títulos mayores de página |
| 18      | 24px    | 28.8px      | Display         | Cabeceras hero |
| 20      | 26.67px | 32px        | Hero            | Texto display/banner muy grande |

**Peso Bold (Font.'Lato Black'):** los mismos CSS px que el normal al mismo Size. Bold NO cambia el tamaño renderizado.

**Altura natural mínima del contenedor para una label de un Size dado:**
`altura mínima del contenedor = lineHeight + 10px` (5px de padding arriba + 5px abajo)

| Size | Altura mínima del contenedor |
|------|---------------------|
| 8    | 22.8px → usar Height=24 |
| 9    | 24.4px → usar Height=26 |
| 10   | 26px   → usar Height=28 |
| 11   | 27.6px → usar Height=30 |
| 12   | 29.2px → usar Height=32 |
| 13   | 30.8px → usar Height=32 |
| 14   | 32.4px → usar Height=36 |
| 16   | 35.6px → usar Height=40 |
| 18   | 38.8px → usar Height=40 |
| 20   | 42px   → usar Height=44 |

---

### 1b. Modern — ModernText@1.0.0 (Fluent UI)

Elemento fuente: `fui-Text` PRE (l1 en la extracción)
Ratio de line-height: fontSize × 1.4

| PA Size | CSS px | Line-height | Carácter visual | Uso recomendado |
|---------|--------|-------------|------------------|-----------------|
| 8       | 8px    | 11.2px      | Minúsculo — prácticamente ilegible | Evitar |
| 9       | 9px    | 12.6px      | Muy pequeño     | Solo para compacidad extrema |
| 10      | 10px   | 14px        | Pequeño         | Captions, hints |
| 11      | 11px   | 15.4px      | Cuerpo compacto | Labels secundarias |
| 12      | 12px   | 16.8px      | Estándar compacto | Hints, labels secundarias |
| 13      | 13px   | 18.2px      | **Cuerpo Modern estándar** ← por defecto | Form labels en apps Modern |
| 14      | 14px   | 19.6px      | Estándar-grande | Coincide con el default de Fluent UI (botones, checkboxes) |
| 16      | 16px   | 22.4px      | Medio-grande    | Cabeceras de cards, títulos de sección |
| 18      | 18px   | 25.2px      | Grande          | Cabeceras de página |
| 20      | 20px   | 28px        | Extra grande    | Cabeceras mayores |

**Peso Bold:** los mismos CSS px — el bold afecta solo a la apariencia, no al tamaño renderizado.

---

### 1c. Equivalencia de tamaño — Classic ↔ Modern

Para conseguir el mismo peso visual de texto en un control Modern que un tamaño Classic dado:

| Classic Size | Classic CSS px | Modern Size (más cercano) | Modern CSS px |
|-------------|---------------|----------------------|---------------|
| 9           | 12px          | **12**               | 12px ✓ exacto |
| 10          | 13.33px       | **13**               | 13px (cercano) |
| 11          | 14.67px       | **15**               | 15px (no está en la tabla de arriba, válido) |
| 12          | 16px          | **16**               | 16px ✓ exacto |
| 13          | 17.33px       | **17**               | 17px (cercano) |
| 14          | 18.67px       | **19**               | 19px (cercano) |
| 16          | 21.33px       | **21**               | 21px (cercano) |

**Regla práctica:** para formularios Modern, usa por defecto **Size 13** (= Classic Size 10) como texto de cuerpo y **Size 16** como título de sección. NO uses Modern Size 11 donde se usó Classic Size 11 — se ven muy diferentes.

---

## 2. Alturas de controles

### 2a. Botones

**Classic — Classic/Button@2.2.0**
- Fuente: siempre 14.67px (equivalente a PA Size 11), sin importar el valor de `Height`
- Altura CSS renderizada: PA Height + 0.8px (artefacto de borde — despreciable en la práctica)
- Padding: 5px arriba/abajo, 5px izquierda/derecha en la label interna

| PA Height | CSS renderizado | Sensación |
|-----------|-------------|------|
| 28        | 28.8px      | Compacta — apretada, uso en toolbar |
| 32        | 32.8px      | Pequeña — acciones secundarias |
| 36        | 36.8px      | Media |
| **40**    | **40.8px**  | **Estándar ← por defecto** |
| 44        | 44.8px      | Confortable — CTA primario |

**Modern — Button@0.0.45**
- Fuente: siempre 14px (default de Fluent UI), sin importar `Height`
- Altura CSS renderizada: exactamente PA Height (sin offset)
- Padding: 5px arriba/abajo, 12px izquierda/derecha (padding Fluent integrado)

| PA Height | CSS renderizado | Sensación |
|-----------|-------------|------|
| 28        | 28px        | Compacta |
| 32        | 32px        | Pequeña |
| 36        | 36px        | Media |
| **40**    | **40px**    | **Estándar ← por defecto** |
| 44        | 44px        | Confortable |

---

### 2b. Entradas de texto

**Classic — Classic/TextInput@2.3.2** y **Modern — ModernTextInput@1.0.0**

> La extracción del sentinel no capturó las alturas de los inputs (los valores de input son atributos, no nodos de texto del DOM).
> Los valores de abajo se basan en la escala de alturas de referencia y en el comportamiento conocido de PA.

| PA Height | Sensación visual | Recomendado para |
|-----------|-------------|-----------------|
| 28        | Muy compacta | Formularios densos, ediciones inline |
| 32        | Compacta     | Formularios apretados |
| 36        | Media      | Formularios compactos confortables |
| **40**    | **Estándar** | **Por defecto para la mayoría de formularios** |
| 44        | Confortable | Formularios amplios |
| 48        | Grande       | Inputs prominentes, barras de búsqueda |

Fuente interna por defecto de Classic: 14.67px (Size 11). Asigna `Size: =11` explícitamente.
Fuente interna por defecto de Modern: 14px (Fluent). Coincide con el tamaño de fuente del botón.

---

### 2c. Dropdowns / ComboBoxes

**Classic — Classic/ComboBox@2.4.0**
- El texto del valor seleccionado se renderiza por defecto a **17.33px** (equivalente a PA Size 13)
- Altura CSS: PA Height + 0.8px de offset

| PA Height | Altura CSS |
|-----------|-----------|
| 32        | 32.8px    |
| 36        | 36.8px    |
| **40**    | **40.8px** ← estándar |
| 44        | 44.8px    |

**Modern — ModernCombobox@1.0.0**
Datos no capturados (el sentinel está en la lista de Items, no en un nodo de texto del DOM). Usa PA Height = altura CSS (los controles Modern tienen altura exacta).

---

### 2d. Checkboxes

**Classic — Classic/CheckBox@2.1.0**
- Fuente: siempre 14.67px sin importar `CheckboxSize`
- `CheckboxSize` solo afecta al cuadrado del checkbox — NO cambia la fuente ni la altura del contenedor
- Altura natural renderizada del contenedor: **~50px** (no afectada por CheckboxSize 10–18)
- **Checkbox Classic en un contenedor de altura fija**: asigna `AlignInContainer: =AlignInContainer.Stretch` al hijo checkbox para que quede constreñido a la altura del padre. NO uses `AlignInContainer.Center` — la altura natural del checkbox (~50px) superará la mayoría de contenedores de fila (40–44px) y desbordará. Con `Stretch`, el checkbox se renderiza a la altura del padre y todo su contenido (caja + texto) se muestra correctamente dentro.

| CheckboxSize | Qué cambia | Qué no cambia |
|-------------|-------------|---------------------|
| 10          | La caja es 10×10px | Fuente 14.67px, altura 50px |
| 14          | La caja es 14×14px | Fuente 14.67px, altura 50px |
| 18          | La caja es 18×18px | Fuente 14.67px, altura 50px |

**Modern — CheckBox@0.0.30**
- Fuente: 14px (Fluent)
- Altura natural renderizada: **36px** (padding de 8px arriba/abajo dentro de la label)
- La propiedad `CheckboxSize` NO existe en el checkbox Modern — el tamaño lo fija el token de Fluent
- Fila contenedora: usa `Height: =40` para dar un espaciado confortable alrededor del control de 36px

---

### 2e. Radios, Date Pickers, Toggles

> No se capturaron datos del sentinel para estas secciones (datos truncados / sentinel en propiedades que no son nodos de texto).
> Defaults seguros basados en el comportamiento conocido de PA:

| Control | Height por defecto Classic | Height por defecto Modern |
|---------|----------------------|----------------------|
| Classic/Radio@2.3.0 | Fórmula: `=39 + Max(0, Floor((RadioSize-16)/2)) + Max(0, fontSize-10)` | — |
| ModernRadio@1.0.0 | — | 40 (altura de fila por defecto de Fluent) |
| Classic/DatePicker@2.6.0 | 40 (igual que TextInput) | — |
| ModernDatePicker@1.0.0 | — | 40 |
| Classic/Toggle@2.1.0 | 48 (altura natural del toggle) | — |
| Toggle@1.1.5 | — | 48 |

---

## 3. Espaciado — LayoutGap y Padding

> No se capturaron datos del sentinel para las filas de gap de la Sección 10 (el texto del gap-label no coincide con SENTINEL_RE).
> Los valores de abajo se basan en principios de calibración visual.

### Referencia de LayoutGap

| Valor | Sensación visual | Uso |
|-------|-------------|-----|
| 4     | Mínimo    | Entre elementos estrechamente relacionados (label + input inline) |
| 6     | Apretado       | Filas de formulario compactas |
| 8     | Cercano       | Gap por defecto entre filas de formulario (compacto) |
| 10    | Confortable | Espaciado por defecto en formularios estándar |
| 12    | Estándar    | Gap habitual para contenido de cards |
| 16    | Amplio    | Gaps de sección dentro de una card |
| 20    | Abierto        | Gap entre secciones mayores |
| 24    | Ligero        | De sección a sección en layouts amplios |
| 32    | Ancho        | Entre bloques de contenido distintos |

### PaddingX / PaddingY (padding interno del contenedor)

| Valor | Uso |
|-------|-----|
| 8     | Padding de card apretado |
| 12    | Padding de card compacto |
| 16    | Padding de card estándar |
| 20    | Padding de card/página confortable |
| 24    | Padding de página amplio |

### Margen de seguridad de bordes

Los contenedores AutoLayout de PA recortan a los hijos dentro de sus límites. Cuando un hijo tiene bordes visibles, el contenedor padre debe reservar espacio para ellos o el borde se cortará.

**Regla para contenedores AutoLayout verticales:**
- **Fórmula de altura**: añade `+ (child.FocusedBorderThickness * 2)` por cada hijo con `FocusedBorderThickness > 0` (cubre la expansión superior e inferior al hacer focus)
  Si el hijo no tiene FocusedBorderThickness pero sí `BorderThickness > 0`: añade `+ (child.BorderThickness * 2)`
- **PaddingLeft / PaddingRight**: asigna el `Max(child.BorderThickness, child.FocusedBorderThickness)` del hijo con borde más prominente

**Regla para contenedores AutoLayout horizontales:**
- Mismo principio en el otro eje: `PaddingTop / PaddingBottom = Max(BorderThickness, FocusedBorderThickness)`
- Si el contenedor tiene Height fija: añádele `(child.FocusedBorderThickness * 2)`

**Hijos GroupContainer con borde** (p. ej., un wrapper estilizado alrededor de un ComboBox):
- Los bordes del GroupContainer se renderizan DENTRO de los límites declarados; el padre solo necesita `PaddingLeft/Right = BorderThickness` para evitar que el píxel más externo quede justo en el borde del recorte
- Elimina la propiedad `Width` explícita del hijo y confía en `AlignInContainer.Stretch` — así el hijo auto-se ajusta a la anchura interna del padre con padding

**Controles de borde transparente** (botones con `FocusedBorderColor: =ColorFade(RGBA(0,0,0,0), ...)`, botones overlay):
- No hace falta corrección — el anillo focused es invisible, así que no hay recorte visual

**Tabla de referencia rápida:**

| Control hijo | Escenario | Ajuste de Height del padre | PaddingL/R del padre |
|---|---|---|---|
| TextInput | FocusedBorderThickness=2 | +4px | =2 |
| TextInput | FocusedBorderThickness=1 | +2px | =1 |
| Wrapper GroupContainer | BorderThickness=1 | +2px | =1 |
| Button (focused transparente) | cualquiera | 0 | 0 |

### Proporciones de filas horizontales

Cuando varios controles comparten una fila AutoLayout horizontal y TODOS deben verse por completo, asigna ratios de `FillPortions` — nunca dejes `FillPortions: =0` en un hijo visible sin una `Width` explícita.

**Regla: cada hijo visible necesita `FillPortions > 0` O una `Width` explícita. Nunca ambos ausentes.**

| Patrón de fila | Control izquierdo | Control derecho | Justificación |
|---|---|---|---|
| Checkbox + enlace de política | `FillPortions: =3` | `FillPortions: =2` | Checkbox+label necesita ~60%; el enlace ~40% |
| Label de pregunta + radio/toggle | `FillPortions: =2` | `FillPortions: =1` | La label recibe ~67%, el control ~33% |
| Hermanos iguales | `FillPortions: =1` | `FillPortions: =1` | Reparto 50/50 |
| Label + botón de anchura fija | `FillPortions: =1` | `FillPortions: =0` + `Width` explícita | El botón tiene tamaño fijo, la label llena el resto |

**Altura del botón en filas de acciones de altura fija (borde focused visible):**
```
Height: =Parent.Height - (Self.FocusedBorderThickness * 2)
```
El botón se auto-reduce para dejar sitio a su propio anillo de focus. Solo aplica cuando `FocusedBorderColor` NO es transparente y el padre tiene una `Height` fija.

---

## 4. Tabla de selección rápida

Usa estas combinaciones para escenarios de diseño comunes sin tener que consultar valores individuales.

| Escenario | Fuente (labels) | Fuente (cabeceras de sección) | Altura de control | Gap de fila | Padding de card |
|---|---|---|---|---|---|
| **Classic — formulario compacto** | Size 11 (14.67px) | Size 12–13 | Height 36 | LayoutGap 8 | Padding 12 |
| **Classic — formulario estándar** | Size 11 | Size 13 | Height 40 | LayoutGap 10–12 | Padding 16 |
| **Classic — formulario amplio** | Size 11–12 | Size 14 | Height 44 | LayoutGap 16 | Padding 20–24 |
| **Modern Fluent — compacto** | Size 13 | Size 16 | Height 36 | LayoutGap 8 | Padding 12 |
| **Modern Fluent — estándar** | Size 13 | Size 16 | Height 40 | LayoutGap 12 | Padding 16 |
| **Modern Fluent — amplio** | Size 14 | Size 18 | Height 44 | LayoutGap 16 | Padding 20 |
| **Fila de data table** | Size 10 (Classic) / Size 13 (Modern) | Size 11/14 | Height 36 | LayoutGap 0 (sin gap, usa fill de AlternateRow) | Padding 8 |

---

## 6. Dimensionado proporcional — regla del factor de escala

Los siguientes valores escalan con la anchura del canvas. Los valores de referencia de abajo están calibrados para un diseño de referencia de ~860px (anchura habitual de mockup/captura).
Fórmula de escala: `valor_objetivo = valor_referencia × (anchura_canvas_objetivo / anchura_canvas_referencia)`

| Anchura del canvas | Anchura de sidebar | Anchura de card de form/contenido | Wrapper de combo (card − 48px de padding) | Plantilla de tab (÷ 6 tabs) |
|---|---|---|---|---|
| 768px  | 120px | 360px | 312px | 108px |
| 1024px | 158px | 468px | 420px | 141px |
| 1280px | 194px | 584px | 536px | 180px |
| **1366px** | **200px** | **620px** | **572px** | **194px** |
| 1920px | 290px | 880px | 832px | 272px |

**Qué escala (proporcional — aplica el factor de escala):**
- Anchura del contenedor de la sidebar
- Anchura de la card de contenido / form card
- Anchuras de contenedores de columna (wrappers de combo, filas multicolumna)
- Tamaño de plantilla de la galería cuando la galería debe llenar la anchura disponible

**Qué NO escala (absoluto — mantén fijo sin importar la anchura del canvas):**
- Alturas de filas: nav items, top bar, tab bar, filas de inputs, filas de botones
- Altura de inputs, altura de botones
- Padding (`PaddingTop`, `PaddingBottom`, `PaddingLeft`, `PaddingRight`)
- Valores de `LayoutGap`
- Tamaños de fuente — **absolutos a propósito**: la legibilidad del texto no depende del viewport. Escalar labels de formulario de Size 10 ×1.585 produciría Size 16 — un título sobredimensionado, no una label de formulario. Los tamaños de fuente actuales son siempre correctos.
- Valores de radio de borde

**Cómo aplicarlo:**
1. Estima la anchura de la captura de referencia en píxeles (mira la anchura total del canvas en la imagen; la distancia de la sidebar al borde derecho es la anchura completa del canvas)
2. Calcula: `scale_factor = target_canvas_width / reference_canvas_width`
3. Consulta los valores precalculados de la tabla de arriba, o multiplica los valores de referencia por el scale_factor
4. Redondea al número par más cercano (p. ej., 206 → 200, 618 → 620)

---

## 5. Referencia de estructura del DOM (para futuras actualizaciones de extracción)

| Control | Nivel del sentinel con font-size real | Nivel con PA Height |
|---------|-----------------------------------|----------------------|
| Classic Label | l1 `.appmagic-label-text` | l2 `.appmagic-label` = l3 `.react-knockout-control` |
| Modern Text | l1 `fui-Text` PRE | l2 contenedor Fluent |
| Classic Button | l1 `.appmagic-button-label` (solo fuente) | l3 `BUTTON.appmagic-button-container` |
| Modern Button | l1 `SPAN` (texto) | l2 `BUTTON.fui-Button` |
| Classic ComboBox | l1 `SPAN` | l3 `.label_kohvda` |
| Classic Checkbox | l1 `.appmagic-checkbox-label` | l2 `LABEL.checkbox-label` |
| Modern Checkbox | l1 `LABEL.fui-Checkbox__label` | l2 `SPAN.fui-Checkbox` |
