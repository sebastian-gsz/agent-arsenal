# Integración de chat con Gemini Data Analytics

> [!CAUTION] Asegúrate de que las fuentes de datos existan como tablas de BigQuery.

> [!WARNING] **Error común: `400 REFERENCES_NOT_SET`** Ocurre cuando falta
> `datasource_references` en `inline_context`. La API **REQUIERE** al menos una
> referencia a una tabla de BigQuery. No uses tablas BigLake/Iceberg, no están
> soportadas.

## Requisitos de arquitectura

DEBES implementar el chat usando los siguientes patrones de última generación:

1.  **Server-Sent Events (SSE)**: la API del backend DEBE transmitir la respuesta
    chunk a chunk usando un generator (`text/event-stream`). No esperes a que la
    generación termine por completo.
2.  **Segregación de thoughts vs contenido**: el backend debe extraer
    `chunk.system_message.text_type` (o `chunk.system_message.text.text_type`
    según el anidado del SDK) y emitir bloques tipados como `"THOUGHT"` al
    instante, completamente separados del texto `"FINAL_RESPONSE"`. El backend
    también debería pasar los `"SUGGESTION"` (botones interactivos de
    seguimiento), que el frontend DEBE renderizar como prompts de seguimiento
    clicables.
3.  **Soporte multi-turno**: la interfaz de chat DEBE soportar conversaciones
    multi-turno, permitiendo a los usuarios hacer preguntas de seguimiento. Esto
    requiere mantener el historial y el contexto de la conversación.
4.  **Parseo estructurado de la UI**: el loop de la app React del frontend DEBE
    interceptar y buferizar el stream SSE, manteniendo propiedades de estado
    distintas (`m.thoughts` vs `m.content`).
5.  **Formato Markdown**: DEBES usar `react-markdown` + `remark-gfm` envueltos
    en un `<div>` localizado que contenga overrides de Tailwind Typography
    (`[&>p]:mb-2 [&>h3]:...`) en lugar de volcar blobs sin estructura en el
    chat. Evita pasar `className` estrictamente al componente raíz
    `ReactMarkdown` para prevenir crashes de renderizado en la versión 10+.
6.  **UX dinámica**: expón los thoughts del backend localmente como una línea de
    título dinámica e iterativa (p. ej., siguiendo la última cadena mapeada por
    `\n`) o una salida de cadena mapeada manualmente, en lugar de layouts
    básicos colapsables de Markdown.
7.  **Entrada multilínea**: el campo de entrada del chat en el frontend DEBE
    soportar entrada de texto multilínea, permitiendo a los usuarios escribir
    queries más largas antes de enviar.
8.  **Redimensionamiento dinámico del textarea**: el `<textarea>` de entrada del
    chat debería redimensionar dinámicamente su altura para acomodar la entrada
    multilínea, mejorando la experiencia del usuario con mensajes más largos.
9.  **Soporte de temas**: la interfaz de chat DEBE soportar completamente los
    temas dark y light. Asegúrate de que todos los componentes dentro del panel
    de chat (p. ej., burbujas de mensajes, campos de entrada, fondo) tengan las
    variantes `dark:` apropiadas para las clases de Tailwind CSS, consistentes
    con el theming global de la aplicación.
10. **Manejo robusto de errores de API**: implementa bloques `try-catch`
    completos y mecanismos de manejo de errores para todas las llamadas a la
    API, especialmente alrededor de las conexiones de streaming, para gestionar
    y reportar las fallas de forma elegante.

## Implementaciones de referencia

La skill incluye varias implementaciones de referencia que DEBES revisar y
adaptar al construir la interfaz de chat:

### Configuración del backend (FastAPI o Node/Express)

Al configurar tu endpoint de chat con streaming, usa uno de los siguientes:

- **Python / FastAPI**: `examples/fastapi_chat.py`
- **Node.js / Express**: `examples/express_chat.ts`

### Implementación del frontend (React)

DEBES mantener localmente el array de contexto de la conversación para que las
preguntas de seguimiento funcionen. Revisa la implementación de referencia
completa del **streaming parser** en:

- `examples/react_chat_panel.jsx`

## Checklist de validación

> [!CAUTION]
> Si están disponibles, DEBES usar capacidades de testing de navegador (como
> `browser_subagent`, Puppeteer, Playwright o una herramienta equivalente
> disponible) para verificar visualmente que la aplicación frontend funciona
> correctamente *antes* de notificar al usuario que la tarea está completa.

> [!IMPORTANT]
> Debes incluir estos ítems en el formato que uses para seguir tu trabajo
> (p. ej., tu lista de tareas, el plan de implementación o un checklist interno).

-   [ ] ¿La interfaz de chat es funcional y funciona correctamente al hacerle una pregunta?
-   [ ] ¿Funciona con preguntas de seguimiento?
-   [ ] ¿Las fuentes de datos son correctas?
-   [ ] ¿La respuesta final se renderiza como markdown?
-   [ ] ¿Los thoughts están separados de la burbuja de chat de la respuesta final?
-   [ ] ¿Los thoughts se transmiten vía stream a medida que se generan y actualizan la UI para indicar el progreso?
