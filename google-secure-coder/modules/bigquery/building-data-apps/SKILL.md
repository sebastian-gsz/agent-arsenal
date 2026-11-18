---
name: building-data-apps
description: |
  Construye data apps modernas, dashboards y reportes interactivos usando
  React + Vite o Streamlit. Incluye integración opcional de chat con Gemini
  Data Analytics para una experiencia "chat with your data" impulsada por IA.

  Relevante cuando alguna de las siguientes condiciones es verdadera:
    1. El usuario pide explícitamente construir un dashboard de datos, una aplicación de datos o una UI de visualización, y la UI obtiene datos de una base de datos de GCP (por defecto BigQuery, salvo indicación contraria).
    2. Necesitas generar una aplicación web frontend para interactuar con datos de fuentes de GCP, consultarlos y visualizarlos.
    3. El usuario quiere construir una experiencia "chat with your data" o integrar la API de chat de Gemini Data Analytics en una interfaz web.

  NO la uses cuando alguna de las siguientes condiciones sea verdadera:
    1. La petición es para construir servicios solo de backend.
    2. La petición es para scripts CLI simples o aplicaciones de línea de comandos.
    3. La aplicación web no está centrada en datos o no involucra visualizar/consultar datos de fuentes de GCP.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Construcción de aplicaciones de datos

Diseña dashboards de datos e informes interactivos de alta calidad. DEBES
seleccionar el framework apropiado antes de la implementación.

## Paso 0: Selección del framework

DEBES seleccionar el framework según los requisitos de mantenimiento del usuario
y su ecosistema de datos.

### Opción: Streamlit

-   **Perfil del usuario**: Data Scientists / usuarios de Python.
-   **Complejidad de la lógica**: alta dependencia de Python (Pandas, NumPy,
    procesamiento local de datos).
-   **Despliegue**: script de Python de un solo archivo.
-   **Personalización**: layout estándar (boilerplate rápido).

### Opción: React + Vite

-   **Perfil del usuario**: desarrolladores web / equipos full-stack.
-   **Complejidad de la lógica**: requisitos altos de UI e interactividad (p. ej.,
    drag-and-drop, mapas interactivos).
-   **Despliegue**: frontend independiente + API de backend.
-   **Personalización**: infinita (CSS propio, bibliotecas JS especializadas).

### Orientación:

-   **Revisa primero el stack existente**: prefiere SIEMPRE el framework que el
    usuario ya usa en su proyecto (p. ej., si ves un `package.json` con
    dependencias de React, usa React; si ves archivos Streamlit existentes, usa
    Streamlit).
-   **Por defecto, React + Vite** para aplicaciones de grado de producción que
    requieran estado complejo en el cliente, branding propio o integración en un
    ecosistema web más grande.
-   **Por defecto, Streamlit** si el usuario menciona específicamente "dashboard
    de Python", necesita iterar sobre procesamiento local complejo de datos en
    Python o requiere un despliegue de un solo script.

## Paso 1: Plan de implementación

DEBES proponerle al usuario un plan que especifique el framework elegido y
justifique la elección según los criterios anteriores.

--------------------------------------------------------------------------------

## Estándares de diseño compartidos

Independientemente del framework, DEBES seguir los principios de
`references/shared_design_system.md`.

-   **Estilo visual**: chrome mínimo, paleta de colores zinc y layouts basados
    en tarjetas.
-   **Tipografía**: `DM Sans` para contenido, `JetBrains Mono` para datos.

--------------------------------------------------------------------------------

## Implementación del framework

### Si usas Streamlit:

1.  Lee `references/streamlit_framework.md` para los patrones detallados de CSS
    y componentes.
2.  Sigue el "Checklist para nuevos dashboards" de ese archivo.

### Si usas React + Vite:

1.  Lee `references/react_framework.md` para la configuración de Tailwind y
    ECharts.
2.  Sigue las pautas detalladas de componentes para tarjetas KPI, tablas y
    paneles.

--------------------------------------------------------------------------------

## Interfaz de chat con IA (funcionalidad opcional)

```
> [!IMPORTANT]
>
> If the user does not explicitly request a chat interface, you SHOULD
> proactively ask them: "Would you like to include a Gemini-powered chat
> interface to enable natural language queries against your data?" OR if
> there is an implementation plan: "Would you like to include a
> Gemini-powered chat interface to enable natural language queries against
> your data? Let me know and I'll update the plan!".
```

Si el usuario pide o acepta la interfaz de chat:

```
> [!CAUTION]
>
> Adding the chat interface is a significant change. Implicit approval of
> the implementation plan for including the chat interface MUST never be
> assumed.
```

1.  **Recopilar los detalles técnicos**: DEBES leer `references/chat_integration.md`
    para conocer los requisitos técnicos.
2.  **Actualizar el plan de implementación**: si y solo si existe un plan de
    implementación, DEBES actualizar el plan de implementación. Es un cambio
    significativo, así que el usuario debe aprobar explícitamente el plan
    actualizado.
3.  **Verificar los prerrequisitos**: asegúrate de que el usuario tenga la API de
    Gemini Data Analytics habilitada y de que existan datos en BigQuery.
4.  **Ejemplos de referencia**: adapta los patrones de
    `examples/react_chat_panel.jsx` y de `examples/fastapi_chat.py` o
    `examples/express_chat.ts`.

## Criterios de aceptación

> [!CAUTION]
>
> Si están disponibles, DEBES usar capacidades de testing de navegador (como
> `browser_subagent`, Puppeteer, Playwright o una herramienta equivalente
> disponible) para verificar visualmente que la aplicación frontend funciona
> correctamente *antes* de notificar al usuario que la tarea está completa.

> [!IMPORTANT]
>
> El siguiente checklist representa los requisitos estrictos de esta tarea. Debes
> incluir estos ítems en el formato que uses para seguir tu trabajo (p. ej., tu
> lista de tareas, el plan de implementación o un checklist interno).

-   [ ] ¿Las transiciones hover de CSS son fluidas?
-   [ ] ¿Los campos de fecha tienen un formato legible? (p. ej., `MMM dd, yyyy`)
-   [ ] ¿Los z-index se apilan correctamente para que los dropdowns aparezcan
    por encima de los encabezados de tabla? (`relative z-30`)
-   [ ] ¿Todas las entradas interactivas de formularios/botones manejan estados
    de loading/disabled?
-   [ ] ¿La aplicación es responsive y el layout se adapta bien a distintos
    tamaños de pantalla?
-   [ ] ¿Las llamadas a la API para obtener datos son exitosas y hay un manejo
    de errores apropiado?
-   [ ] ¿El toggle de dark mode funciona correctamente y aplica los estilos de
    forma consistente?
-   [ ] ¿Todas las visualizaciones se renderizan correctamente y son interactivas
    donde se espera?
-   [ ] ¿El dashboard es visualmente atractivo?
