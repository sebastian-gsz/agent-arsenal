---
version: alpha
name: Sovereign
description: Sistema premium y corporativo de contraste profundo — carbón y casi-negro anclados por acento dorado, tipografía arquitectónica y whitespace generoso.
colors:
  primary: "#BD7D0F"
  on-primary: "#16181D"
  primary-hover: "#CE9020"
  secondary: "#676F7E"
  neutral: "#2B303B"
  surface: "#FFFFFF"
  on-surface: "#16181D"
  surface-variant: "#E9EAED"
  error: "#B3261E"
typography:
  headline-display:
    fontFamily: Space Grotesk
    fontSize: 56px
    fontWeight: 700
    lineHeight: 1.05
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Space Grotesk
    fontSize: 36px
    fontWeight: 700
    lineHeight: 1.15
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Space Grotesk
    fontSize: 24px
    fontWeight: 700
    lineHeight: 1.25
  body-lg:
    fontFamily: Space Grotesk
    fontSize: 18px
    fontWeight: 400
    lineHeight: 1.5
  body-md:
    fontFamily: Space Grotesk
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
  body-sm:
    fontFamily: Space Grotesk
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.45
  label-lg:
    fontFamily: Space Grotesk
    fontSize: 16px
    fontWeight: 700
    lineHeight: 1.25
  label-md:
    fontFamily: Space Grotesk
    fontSize: 14px
    fontWeight: 700
    lineHeight: 1.2
    letterSpacing: 0.01em
  label-sm:
    fontFamily: Space Grotesk
    fontSize: 12px
    fontWeight: 700
    lineHeight: 1.2
    letterSpacing: 0.02em
  display-accent:
    fontFamily: Cormorant Garamond
    fontSize: 20px
    fontWeight: 700
    lineHeight: 1.3
    letterSpacing: 0.02em
rounded:
  none: 0px
  sm: 8px
  md: 12px
  lg: 16px
  xl: 24px
  full: 9999px
spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 32px
  2xl: 40px
  3xl: 48px
  4xl: 64px
  5xl: 80px
  6xl: 104px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label-md}"
    rounded: "{rounded.full}"
    padding: 12px 32px
    height: 48px
  button-primary-hover:
    backgroundColor: "{colors.primary-hover}"
  button-secondary:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.secondary}"
    typography: "{typography.label-md}"
    rounded: "{rounded.md}"
    padding: 10px 16px
    height: 44px
  button-secondary-hover:
    backgroundColor: "{colors.surface-variant}"
    textColor: "{colors.neutral}"
  button-ghost:
    textColor: "{colors.secondary}"
    typography: "{typography.label-md}"
    padding: 8px 12px
  button-ghost-hover:
    textColor: "{colors.neutral}"
  icon-button:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    rounded: "{rounded.full}"
    size: 48px
  card:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body-md}"
    rounded: "{rounded.lg}"
    padding: 32px
  badge:
    backgroundColor: "{colors.on-surface}"
    textColor: "{colors.surface}"
    typography: "{typography.label-sm}"
    rounded: "{rounded.md}"
    padding: 8px 16px
  input:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body-md}"
    rounded: "{rounded.md}"
    padding: 12px 16px
    height: 48px
  nav-bar:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.neutral}"
    typography: "{typography.body-md}"
    height: 64px
    padding: 16px 40px
  banner-error:
    backgroundColor: "{colors.error}"
    textColor: "{colors.surface}"
    typography: "{typography.body-sm}"
    rounded: "{rounded.sm}"
    padding: 12px 16px
---

# Sistema de diseño — Sovereign

## Overview

Sovereign encarna una presencia digital premium y corporativa definida por minimalismo refinado y contención autoritaria. El diseño comunica soberanía digital mediante contraste deliberado: bases de carbón profundo ancladas por acentos dorados que evocan lujo y poder al mismo tiempo. El lenguaje visual prioriza claridad y precisión, con whitespace generoso, tipografía arquitectónica y elementos geométricos sutiles que sugieren infraestructura y control. Cada componente se siente intencional y sustancial; se rechaza la ornamentación en favor de elegancia funcional.

**Características clave**
- Paleta sofisticada dominada por carbón y casi-negro con acentos dorados (`{colors.primary}`)
- Minimalismo ornamental con espaciado de precisión sobre base de 8px
- Serif premium (Cormorant Garamond) solo en acentos, sans-serif geométrica (Space Grotesk) para todo lo demás
- Sombras sutiles y superficies elevadas que sugieren profundidad sin ruido visual
- Layouts arquitectónicos basados en cuadrícula de 12 columnas
- Tipografía como protagonista, con contraste jerárquico audaz

## Colors

Los tokens son los valores normativos; los nombres descriptivos de la prosa se mapean así: **Deep Charcoal** → `{colors.neutral}`, **Near Black** → `{colors.on-surface}`, **Gold** → `{colors.primary}`, **Slate Gray** → `{colors.secondary}`, **White** → `{colors.surface}`, **Light Gray** → `{colors.surface-variant}`.

### Primario y acción
- **Gold** (`{colors.primary}`, `#BD7D0F`): único acento de la marca. Fondos de botones primarios, subrayados de enlaces, anillos de foco y puntos focales interactivos. Transmite calidad premium contra los neutros fríos.
- **Gold Hover** (`{colors.primary-hover}`, `#CE9020`): dorado aclarado para estados hover. Se aclara (no oscurece) para mantener contraste AA con el texto oscuro del botón.

### Texto
- **Near Black** (`{colors.on-surface}`, `#16181D`): texto de máximo contraste, títulos y cuerpo principal sobre superficies claras. También es el texto sobre dorado (`{colors.on-primary}`): 5.2:1 de contraste.
- **Deep Charcoal** (`{colors.neutral}`, `#2B303B`): texto estructural, bordes de cards contenidas y divisores con presencia.
- **Slate Gray** (`{colors.secondary}`, `#676F7E`): texto secundario, helper, metadata y estados deshabilitados (5.1:1 sobre blanco). Nunca como texto principal de párrafo.

### Superficies
- **White** (`{colors.surface}`, `#FFFFFF`): fondo principal, cards y contenedores; también texto invertido sobre superficies oscuras.
- **Light Gray** (`{colors.surface-variant}`, `#E9EAED`): fondo terciario, divisores sutiles, contenedores de formularios y la alternancia seccional.
- **Error** (`{colors.error}`, `#B3261E`): exclusivo de validación de formularios y banners de error destructivo.

## Typography

**Primaria:** Space Grotesk — sans-serif geométrica moderna. Fallback: `ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`.
**Acento display:** Cormorant Garamond — serif clásica para callouts premium. Fallback: `ui-serif, Georgia, "Palatino Linotype", serif`.

| Rol | Token | Notas de uso |
|-----|-------|--------------|
| Hero / H1 | `{typography.headline-display}` | Solo en heroes; centrado, acompañado de cuerpo corto |
| H2 | `{typography.headline-lg}` | Títulos de sección mayores |
| H3 | `{typography.headline-md}` | Subsecciones y títulos de card |
| Cuerpo destacado | `{typography.body-lg}` | Leads e introducciones |
| Cuerpo | `{typography.body-md}` | Todo el contenido descriptivo |
| Cuerpo pequeño | `{typography.body-sm}` | Contenido denso y metadata |
| Botón / CTA | `{typography.label-md}` | Texto de botones e interactivos |
| Etiqueta | `{typography.label-sm}` | Badges, labels de formulario, overlines |
| Label grande | `{typography.label-lg}` | Títulos de tarjeta enfatizados |
| Callout premium | `{typography.display-accent}` | Serif reservada a mensajes de marca; nunca en UI funcional |

**Principios**
- Jerarquía por contraste: peso y tamaño crean la distinción; el color solo la complementa.
- Line-height generoso (1.4–1.5× en cuerpo) para apariencia premium y legible.
- Letter-spacing ligeramente negativo en headlines (−0.01em a −0.02em) para densidad y autoridad.
- Nunca interpolar tamaños entre niveles: la escala es cerrada.

## Layout

### Sistema de espaciado
**Unidad base: 8px.** Todos los gaps, paddings y márgenes derivan de la escala `spacing`. Nunca valores arbitrarios.

- `{spacing.sm}` (8px) — micro-gaps, espaciado de iconos, items de lista ajustados
- `{spacing.md}` (16px) — padding estándar, separación interna de componentes
- `{spacing.lg}` (24px) — gaps medianos, grid gaps
- `{spacing.xl}` (32px) — padding de cards, separación de componentes
- `{spacing.2xl}` (40px) — padding horizontal de secciones
- `{spacing.3xl}` (48px) — separación entre bloques de contenido
- `{spacing.4xl}` (64px) — espaciado vertical de secciones
- `{spacing.5xl}` (80px) — padding de secciones destacadas
- `{spacing.6xl}` (104px) — padding máximo de secciones y heroes

**Contextos de uso:** botones e inputs usan padding interno `{spacing.md}`–`{spacing.xl}`; cards usan `{spacing.xl}`–`{spacing.4xl}`; márgenes verticales de sección van de `{spacing.4xl}` a `{spacing.6xl}`; grid gaps de `{spacing.lg}` a `{spacing.xl}`.

### Grid y contenedor
- **Ancho máximo:** 1440px para el contenedor principal, centrado.
- **Padding horizontal:** 40px mínimo por lado en desktop; 20px en tablet; 12px en móvil.
- **Columnas:** cuadrícula flexible de 12 columnas; secciones full-width con contenido interior contenido.
- **Patrón seccional:** alternancia de fondos blanco y `{colors.surface-variant}` para definir bloques de contenido.

### Filosofía de whitespace
Sovereign prioriza el whitespace agresivo para establecer posicionamiento premium. El espacio vacío es intencional y funcional: guía la mirada, enfatiza la jerarquía y refuerza la confianza de marca. Mínimo `{spacing.lg}` (24px) entre bloques de contenido distintos y `{spacing.4xl}` (64px) entre secciones mayores. El whitespace es material de diseño, no espacio desperdiciado.

## Elevation & Depth

| Nivel | Tratamiento | Uso |
|-------|-------------|-----|
| Plano | Sin sombra; borde `1px solid #2B303B` | Cards secundarias, inputs outline, enlaces |
| Sutil (sm) | `0px 1px 3px rgba(0, 0, 0, 0.1), 0px 1px 2px rgba(0, 0, 0, 0.1)` | Cards por defecto, elementos contenidos |
| Medio (md) | `0px 20px 25px rgba(0, 0, 0, 0.1), 0px 8px 10px rgba(0, 0, 0, 0.1)` | Botones, modales, estados hover interactivos |
| Fuerte (lg) | `0px 10px 15px rgba(0, 0, 0, 0.1), 0px 4px 6px rgba(0, 0, 0, 0.1)` | Elementos flotantes, modales secundarios |
| Extremo (xl) | `0px 25px 50px rgba(0, 0, 0, 0.25)` | Overlays full-screen, modales críticos |

**Filosofía:** las sombras son mínimas y contenidas (opacidad 0.1–0.25) para mantener claridad visual. Indican elevación e interactividad, no drama visual. El sistema prefiere bordes sobre sombras para separación; reserva sombras para elementos verdaderamente elevados (botones primarios, CTAs). Nunca apilar múltiples sombras: una sola elevación por elemento.

## Shapes

El lenguaje de esquinas usa la escala `rounded` de forma consistente:

- `{rounded.none}` (0px) — navegación full-width, secciones completas, divisores
- `{rounded.sm}` (8px) — badges pequeños, banners, acentos de input
- `{rounded.md}` (12px) — inputs, botones secundarios, contenedores pequeños
- `{rounded.lg}` (16px) — cards principales y contenedores de contenido
- `{rounded.xl}` (24px) — cards premium y contenedores destacados
- `{rounded.full}` (9999px) — CTAs primarios, botones de icono, chips

Reglas: un componente nunca mezcla dos radios distintos; los CTA primarios siempre pill; las secciones full-width siempre rectas.

## Components

### Botón primario (CTA)
Pill sólido dorado (`{components.button-primary}`): fondo `{colors.primary}`, texto `{colors.on-primary}` con `{typography.label-md}`, padding 12px 32px, altura 48px, radio `{rounded.full}`, sin borde. Hover: fondo a `{colors.primary-hover}` manteniendo texto oscuro. Foco: anillo `2px solid #BD7D0F` con offset 2px. Los botones de icono usan la misma paleta en círculo de 48×48px (`{components.icon-button}`).

### Botón secundario
Outline (`{components.button-secondary}`): fondo `{colors.surface}` (sobre superficies claras; nunca `transparent` — el validador lo computa como negro y falla el contraste), texto `{colors.secondary}`, borde `1px solid #2B303B`, radio `{rounded.md}`, altura 44px. Hover: fondo a `{colors.surface-variant}` y texto a `{colors.neutral}`.

### Botón ghost
Solo texto (`{components.button-ghost}`): texto `{colors.secondary}` sin fondo ni borde. Hover: texto a `{colors.neutral}` con subrayado.

### Enlaces
Texto en `{colors.neutral}` con subrayado dorado de 2px (`{colors.primary}`). El dorado nunca como color de texto de enlace: sobre blanco rinde 3.4:1 y falla AA en texto pequeño. Hover: engrosar subrayado y oscurecer texto a `{colors.on-surface}`.

### Cards y contenedores
Card elevada (`{components.card}`): fondo `{colors.surface}`, texto `{colors.on-surface}`, padding `{spacing.xl}`, radio `{rounded.lg}`, borde `1px solid #2B303B` + sombra sutil. Badge oscuro (`{components.badge}`): fondo `{colors.on-surface}`, texto `{colors.surface}`, radio `{rounded.md}`.

### Inputs y formularios
Text input (`{components.input}`): fondo `{colors.surface}`, borde `1px solid #E9EAED`, radio `{rounded.md}`, altura 48px. Placeholder en `rgba(43, 48, 59, 0.5)`. Label: `{typography.label-sm}` en `{colors.neutral}` con `{spacing.sm}` de margen inferior. Foco: borde a `{colors.primary}` + anillo dorado. Error: borde a `{colors.error}` con mensaje en `{colors.error}`; nunca indicar error solo con color — añade icono o texto.

### Navegación
Nav superior (`{components.nav-bar}`): fondo `{colors.surface}`, texto `{colors.neutral}`, altura 64px, borde inferior `1px solid #E9EAED`. Enlaces: por defecto `{colors.secondary}`; hover `{colors.primary}` (no subrayado); activo `{colors.neutral}` con `{typography.label-md}`. Enlaces secundarios detrás del menú hamburguesa en móvil.

## Do's and Don'ts

### Do
- Usa `{colors.primary}` como único acento: todos los CTAs primarios, enlaces (subrayado) y anillos de foco.
- Texto sobre dorado siempre en `{colors.on-primary}` (5.2:1). Nunca blanco sobre dorado (3.4:1, falla AA).
- Aplica padding interno de 12–32px a todo elemento clicable para áreas de toque premium de ≥44px.
- Contraste mínimo 4.5:1 en todo texto menor a 24px; reserva `{colors.secondary}` solo para texto secundario.
- Mantén padding vertical de `{spacing.4xl}`–`{spacing.6xl}` entre secciones mayores.
- Alterna fondos `{colors.surface}` y `{colors.surface-variant}` para definir bloques seccionales.
- Da indicador de foco visible (anillo dorado o cambio de fondo) a todo elemento enfocable.
- Centra los headlines de hero y empárejalos con cuerpo corto.

### Don't
- No uses degradados púrpura o azul-violeta en heroes ni fondos: es el patrón "AI slop" más recurrente y rompe la identidad carbón + dorado.
- No combines sombra grande y borde grueso en la misma card: elige uno — borde `1px` con sombra sutil, o sombra md sin borde.
- No mezcles acentos: un solo color focal por elemento; jamás azul, verde o rojo como acento (el rojo es exclusivo de error).
- No interpoles tamaños tipográficos: la escala es cerrada y por tokens.
- No uses padding menor a `{spacing.sm}` (8px) internamente ni márgenes menores a `{spacing.md}` (16px) entre componentes.
- No uses esquinas mixtas (dos radios distintos) en un mismo componente.
- No apliques sombras con opacidad mayor a 0.25 ni más de una sombra por elemento.
- No uses Cormorant Garamond en cuerpo, botones ni labels: exclusivo de `{typography.display-accent}`.

## Comportamiento responsive

### Breakpoints

| Nombre | Ancho | Cambios clave |
|--------|-------|---------------|
| Móvil | 320–479px | Una columna, padding horizontal 12px, H1 a 36px, H2 a 24px, botones apilados full-width, cards full-width, secciones a 32–48px verticales |
| Tablet | 480–1023px | Dos columnas, padding 20px, H1 a 44px, grid gaps de 16–24px, pares de botones envuelven |
| Desktop | 1024–1439px | 3–4 columnas, padding 40px, jerarquía completa, CTAs lado a lado |
| Desktop grande | 1440px+ | Contenedor máximo 1440px centrado, gaps de 24–32px |

### Áreas de toque
- Mínimo 44×44px en todo elemento interactivo; separación mínima de `{spacing.sm}` entre áreas de toque en móvil.
- Área de toque de enlaces extendida a 44px alrededor del texto en móvil.

### Estrategia de colapso
- **Tipografía:** H1 baja ~35% en móvil manteniendo jerarquía relativa; nunca romper la escala en pasos intermedios.
- **Espaciado:** el padding de sección baja ~30% de desktop a móvil (64px → 32px vertical; 40px → 12px horizontal).
- **Grid:** colapso a una columna en móvil, dos en tablet, multi-columna en desktop.
- **Componentes:** hero texto+imagen se apila en móvil; cards pasan de full-width con 12px de margen a grid de 2–4 columnas; padding interno de card baja de 32px a 16px.
- **Navegación:** enlaces secundarios tras el menú hamburguesa en móvil; menú horizontal completo en tablet+.

## Guía de prompt para agentes

### Referencia rápida de color
- **CTA / acento / foco:** Gold → `{colors.primary}`
- **Texto máximo contraste:** Near Black → `{colors.on-surface}`
- **Texto estructural / bordes:** Deep Charcoal → `{colors.neutral}`
- **Texto secundario / disabled:** Slate Gray → `{colors.secondary}`
- **Superficies:** White → `{colors.surface}`, Light Gray → `{colors.surface-variant}`
- **Error:** → `{colors.error}` (exclusivo)

### Guía de iteración
1. **Espaciado:** todo deriva de la escala `spacing` (4/8/16/24/32/40/48/64/80/104). Nunca valores arbitrarios.
2. **Tipografía:** aplica los tokens `typography` fijos; nunca interpole tamaños. Cormorant Garamond solo en `{typography.display-accent}`.
3. **Color:** `{colors.primary}` en exclusiva para interactivos y acentos. Hover del dorado siempre `{colors.primary-hover}` (aclara, no oscurece).
4. **Botones:** primario pill dorado con texto oscuro; secundario outline charcoal; ghost solo texto. Altura 48px primario, 44px secundario.
5. **Cards:** borde `1px solid #2B303B` + sombra sutil; padding `{spacing.xl}`; radio `{rounded.lg}`. Sin mezclar borde grueso con sombra grande.
6. **Contraste:** 4.5:1 mínimo en texto <24px. Dorado nunca como texto pequeño sobre blanco.
7. **Foco:** anillo dorado `2px` offset 2px en todo enfocable; los enlaces cambian a `{colors.on-surface}` en hover.
8. **Elevación:** un solo nivel de sombra por elemento; bordes para separación, sombras para elevación real.
9. **Responsive:** móvil una columna con 12px laterales; tablet dos columnas con 20px; desktop 40px y contenedor 1440px.
