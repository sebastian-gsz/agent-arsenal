# Spec oficial de DESIGN.md — Resumen condensado

Fuente: https://github.com/google-labs-code/design.md (`docs/spec.md`). Formato de Google Labs (Apache-2.0, versión `alpha`): representación autocontenida de un sistema de diseño en texto plano, pensada como fuente compartida de verdad para humanos y agentes de código.

## Estructura de dos capas

1. **Tokens YAML** — en frontmatter (`---` … `---` exacto al inicio) y/o en bloques ` ```yaml ` intercalados dentro de las secciones Markdown. Ambos métodos pueden combinarse; todos los bloques se fusionan en un solo árbol de tokens. Cada clave top-level puede aparecer **una sola vez** en todo el documento: duplicarla rechaza el archivo.
2. **Cuerpo Markdown** — la racionalidad organizada en secciones `<h2>` que explican por qué existen los valores y cómo aplicarlos. Los tokens son los valores normativos; la prosa, el contexto.

## Schema top-level

| Campo | Tipo | Notas |
|-------|------|-------|
| `version` | string (opcional) | Actualmente `"alpha"` |
| `name` | string (**obligatorio**) | Nombre del sistema de diseño |
| `description` | string (opcional) | Resumen de la identidad |
| `omitted` | lista de nombres de sección o `{section, reason?}` | Declara ausencias intencionales y silencia warnings |
| `colors` | mapa nombre → Color | |
| `typography` | mapa nombre → Typography | |
| `rounded` | mapa nivel → Dimension | |
| `spacing` | mapa nivel → Dimension o número | |
| `components` | mapa nombre → mapa de sub-tokens | |

## Tipos de token

| Tipo | Formato |
|------|---------|
| Color | Cualquier color CSS válido: hex (`#RRGGBB` recomendado como default), nombrados, `rgb()/rgba()`, `hsl()/hsla()`, `hwb()`, gamut amplio (`oklch()`, `oklab()`, `lch()`, `lab()`), `color-mix()`. Se convierten a sRGB para el chequeo de contraste WCAG |
| Dimension | Número + unidad; **solo `px`, `em`, `rem`** (negativos permitidos) |
| Referencia | `{path.to.token}`, ej. `{colors.primary}`. La mayoría de los grupos solo pueden referenciar primitivos; **solo dentro de `components`** las referencias pueden apuntar a valores compuestos (ej. `{typography.label-md}`) |
| Typography | Objeto: `fontFamily`, `fontSize` (Dimension), `fontWeight` (número), `lineHeight` (Dimension o multiplicador unitless — recomendado), `letterSpacing` (Dimension), `fontFeature`, `fontVariation` |

## Sub-tokens de componentes

Propiedades válidas: `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`.

Variantes de estado como entradas separadas con claves relacionadas: `button-primary`, `button-primary-hover`, `button-primary-active`, `button-primary-pressed`. Propiedad desconocida (ej. `borderColor`) se acepta con warning — las reglas de borde van mejor en prosa.

## Orden canónico de secciones (todas `<h2>`; un `<h1>` de título opcional no se parsea)

Las secciones son opcionales, pero las presentes deben seguir este orden relativo:

1. **Overview** (alias: *Brand & Style*) — personalidad de marca, audiencia, tono emocional; guía cuando no hay regla explícita.
2. **Colors** — al menos una paleta `primary`; orden convencional `primary`, `secondary`, `tertiary`, `neutral`. La prosa puede usar nombres descriptivos ("Midnight Forest Green"); los tokens son normativos.
3. **Typography** — típicamente 9–15 niveles; nombres semánticos (`headline`, `body`, `label`, `caption`) con sufijos de tamaño.
4. **Layout** (alias: *Layout & Spacing*) — modelo de grid + tokens de espaciado (escala base, gutters, márgenes, columnas).
5. **Elevation & Depth** — sombras o alternativas planas (bordes, contraste tonal). No existe grupo de tokens `elevation`/`shadows`: se documentan aquí en prosa y como sub-tokens de componentes.
6. **Shapes** — lenguaje de esquinas; aquí viven los tokens `rounded`.
7. **Components** — átomos comunes: botones, chips, listas, tooltips, checkboxes, radios, inputs. Área "en evolución activa" del spec.
8. **Do's and Don'ts** — guardarramales concretos: reservar el color primario para una acción clave por pantalla, contraste WCAG AA (4.5:1 en texto normal), limitar pesos tipográficos por pantalla.

## Nombres recomendados (no normativos)

- **Colores:** `primary`, `secondary`, `tertiary`, `neutral`, `surface`, `on-surface`, `error`
- **Tipografía:** `headline-display/lg/md`, `body-lg/md/sm`, `label-lg/md/sm`
- **Radios:** `none`, `sm`, `md`, `lg`, `xl`, `full`

## CLI de validación (`npx @google/design.md`)

| Comando | Función |
|---------|---------|
| `lint <archivo>` | Estructura, referencias rotas, contraste WCAG AA; exit 1 en errores. También como librería: `import { lint } from '@google/design.md/linter'` |
| `diff <a> <b>` | Compara tokens entre dos archivos y regresiones de lint; exit 1 si el "después" empeora |
| `export <archivo>` | Convierte tokens a Tailwind v3 JSON, Tailwind v4 `@theme` CSS o W3C DTCG `tokens.json` |
| `spec` | Imprime el spec (opcionalmente las reglas de lint) para inyectarlo en prompts de agentes |

Nota Windows: usar alias `designmd` (el bin `design.md` colisiona con la asociación de archivos Markdown).

### Reglas de lint

- `missing-primary` — falta paleta primaria
- `broken-ref` (**error**) — referencia `{path}` que no resuelve
- `contrast-ratio` — pares texto/fondo bajo WCAG AA 4.5:1
- `section-order` — secciones fuera del orden canónico
- `orphaned-tokens` — tokens definidos sin uso
- `missing-typography` — falta grupo typography
- `unknown-key` (warning) — typos de claves del schema (ej. `colours:`)
- Resúmenes adicionales a nivel info

## Comportamiento del validador

| Situación | Resultado |
|-----------|-----------|
| Duplicado de `<h2>` o de clave top-level YAML | **Rechazo del archivo** |
| Sección desconocida (ej. `## Iconography`) | Se preserva, sin error |
| Nombre de token desconocido | Se acepta si el valor es válido |
| Valor de espaciado sin tipo | Se acepta como string |
| Propiedad de componente desconocida | Se acepta con warning |
| Sección faltante | Warning de lint, salvo que esté en `omitted` |

## Qué hace bueno a un DESIGN.md

- Prosa para la racionalidad; tokens para los valores normativos.
- Mapear consistentemente las descripciones de paleta de la prosa a los nombres de tokens.
- Preferir nombres semánticos (`body-md`, `button-primary`) y niveles de escala descriptivos.
- Documentar omisiones intencionales con `omitted` y su razón, en vez de dejar huecos silenciosos.
- Do's and Don'ts concretos y accionables: reglas observables ("las tablas usan todo el ancho disponible"), no frases subjetivas ("mantenlo limpio"); nombrar explícitamente los patrones generativos de IA a evitar.
