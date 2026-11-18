---
name: design-md
description: Extrae el sistema de diseño de una base de código y genera un archivo DESIGN.md autocontenido con tokens en YAML y especificación visual (formato Design MD de Google Stitch, spec github.com/google-labs-code/design.md). Úsala cuando el usuario pida documentar el diseño, extraer tokens de color/typografía/espaciado, crear un DESIGN.md, o replicar el look & feel de una UI — incluso si solo dice "documéntame el diseño" o "dame los tokens".
---

# Design MD — Extracción de sistema de diseño

Analiza el sistema de diseño del codebase y genera un archivo `DESIGN.md` en la raíz del proyecto, listo para copiar y pegar y consumible por agentes de código.

## Referencias del formato

- Spec condensado con reglas de lint y cheatsheet del CLI: `references/spec.md` — léelo antes de generar el archivo si no tienes el spec fresco.
- Spec oficial: https://github.com/google-labs-code/design.md (docs/spec.md)
- Docs de Stitch: https://stitch.withgoogle.com/docs/design-md/format/
- Ejemplo largo real: `examples/example.md` — nivel de detalle esperado para componentes, elevaciones y responsive.

## Requisitos del archivo generado

1. Empieza con YAML frontmatter (`---` … `---`) con los metadatos del schema: `version: alpha`, `name` (obligatorio), `description`, y los grupos de tokens `colors`, `typography`, `rounded`, `spacing`, `components`.
   - **No** inventes claves top-level como `elevation`, `motion` ni `shadows`: no existen en el schema y generan warning `unknown-key`. Elevación y sombras se describen en la sección de prosa "Elevation & Depth" y como sub-tokens de componentes.
   - `omitted` (lista de secciones o `{section, reason}`) declara secciones ausentes intencionalmente y silencia sus warnings.
2. Continúa con Markdown libre en el orden canónico de `<h2>`: **Overview** (alias Brand & Style) → **Colors** → **Typography** → **Layout** (alias Layout & Spacing) → **Elevation & Depth** → **Shapes** → **Components** → **Do's and Don'ts**. Puedes añadir secciones extra (Responsive, Iconografía…): el validador las preserva sin error.
3. El archivo debe ser completamente autocontenido: sin referencias a archivos, variables ni rutas del codebase.
4. Todos los tokens usan YAML válido del spec: colores en CSS (hex `#RRGGBB` recomendado), dimensiones solo con unidades `px`/`em`/`rem`, tipografía como objeto (`fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`). Referencias con `{colors.primary}`; solo dentro de `components` pueden apuntar a valores compuestos como `{typography.label-md}`.
5. Puedes intercalar bloques ` ```yaml ` dentro de cada sección en lugar de (o además del) frontmatter; cada clave top-level puede aparecer **una sola vez** en todo el documento — duplicarla rechaza el archivo.

## Procedimiento

1. **Extrae de las fuentes reales:** archivos de configuración de Tailwind/CSS variables/styles, y los componentes con más uso visual (botones, inputs, cards, nav).
2. **Deduce las escalas:** espaciado base (¿4px u 8px?), jerarquía tipográfica, radios y sombras — no inventes valores que no existen en el código.
3. **Escribe el DESIGN.md** cumpliendo los 5 requisitos de arriba. Nombres recomendados: colores `primary`/`secondary`/`neutral`/`surface`/`on-surface`/`error`; tipografía `headline`/`body`/`label` con sufijos `lg/md/sm` (9–15 niveles); radios `none`/`sm`/`md`/`lg`/`xl`/`full`. Componentes con sub-tokens `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width` y variantes de estado como claves relacionadas (`button-primary-hover`, `button-primary-active`).
4. **Valida con el CLI oficial:** corre `npx @google/design.md lint DESIGN.md` y corrige hasta que no haya errores. Reglas clave: `missing-primary`, `broken-ref`, `contrast-ratio` (WCAG AA 4.5:1), `section-order`, `orphaned-tokens`, `unknown-key`, `missing-typography`.
5. **Verifica contra el render:** si hay un servidor local corriendo o acceso a la UI, usa la skill `browser-use:control-browser` para capturar pantallas reales y compara tu DESIGN.md contra el render. Revísalo hasta que los tokens y la descripción reflejen fielmente la identidad visual.

## Reglas observables, no subjetivas

- Escribe reglas medibles: "las tablas usan todo el ancho disponible" en vez de "mantenlo limpio". Las frases subjetivas producen resultados distintos en cada modelo.
- En Do's and Don'ts, nombra explícitamente los patrones recurrentes de IA a evitar (ej. "no uses degradados púrpura en heroes", "no combines cards con sombra grande y borde grueso").
- Mapea consistentemente los nombres descriptivos de la prosa (ej. "Deep Charcoal") a los nombres de tokens (`colors.primary`): el lector debe poder conectar ambos sin ambigüedad.

## Ejemplo de frontmatter (estructura esperada)

```yaml
---
version: alpha
name: DevFocus Dark
description: Sistema oscuro de productividad con acento azul eléctrico.
colors:
  primary: "#2665fd"
  secondary: "#475569"
  surface: "#0b1326"
  on-surface: "#dae2fd"
  error: "#ffb4ab"
typography:
  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
rounded:
  md: 8px
spacing:
  md: 16px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "#ffffff"
    rounded: "{rounded.md}"
    padding: "12px 24px"
  button-primary-hover:
    backgroundColor: "#1e53d6"
---
```

Tras el frontmatter: `## Overview` → `## Colors` (con el rol de cada color) → `## Typography` → `## Layout` → `## Elevation & Depth` → `## Shapes` → `## Components` → `## Do's and Don'ts`.
