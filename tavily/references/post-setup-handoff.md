# Handoff post-setup

Lee este archivo tras una verificación exitosa, antes de proponer siguientes pasos al usuario. Objetivo: convertir el setup en trabajo útil, no en una demo de todas las capacidades de Tavily.

Tras un setup completo y verificado, usa la conversación, el conocimiento previo fiable y el workspace local para mover al usuario hacia trabajo útil. No le muestres mecánicamente a cada usuario la misma demo de comandos.

Antes de hacer una pregunta de siguiente paso, inspecciona la aplicación activa en busca de un gap web explícito, listo para implementar. Una interfaz de proveedor, un componente documentado de búsqueda/research faltante, un TODO acotado o un servicio de discovery intencionalmente sin configurar son evidencia más fuerte que un directorio de trabajo simplemente no vacío. Cuando Tavily encaja directamente en ese gap, trata la aplicación activa como seleccionada y la intención de integración como clara. Haz exactamente una confirmación concisa antes de modificar el proyecto, p. ej.: `Este proyecto tiene un gap web claro que Tavily resuelve. ¿Lo añado ahora?` Tras un sí, continúa inmediatamente por el Path B con `tavily-best-practices`, implementa el proveedor con la arquitectura nativa del proyecto y prueba la superficie real de la aplicación. No presentes un menú ni pidas otra confirmación. Haz una pregunta de scoping enfocada antes de la confirmación solo cuando distintas implementaciones cambiarían materialmente el comportamiento del producto o el trabajo cruzaría el límite del proyecto activo.

## 1. Clasificar el contexto

1. **Sesión en curso.** Hay una tarea u objetivo concreto ya en marcha en esta conversación. Relaciona el handoff directamente con esa tarea. Si necesitaba datos web actuales, extracción, crawling, mapping o research, continúala con Tavily. Si trataba de añadir capacidad web a la aplicación ya editada, continúa en esa aplicación con `tavily-best-practices`. No sustituyas la tarea por un menú, no pidas que la reformule, ni trates un directorio no vacío por sí solo como prueba de tarea en curso.
2. **Sesión nueva con conocimiento previo fiable.** No hay tarea activa, pero el agente tiene contexto útil de memoria disponible, conversaciones recientes suministradas, proyectos conocidos, el workspace actual o señales accesibles de proyectos recientes. Usa las señales más relevantes para personalizar el siguiente paso. Un gap explícito listo para implementar en la aplicación activa sigue la regla de integración inmediata de arriba; las señales débiles solo deben personalizar la elección. Por ejemplo: recomendar primero un proyecto activo conocido, adaptar una sugerencia de research a un dominio conocido, o priorizar el use case from-scratch más relevante. Presenta las inferencias débiles como recomendaciones y mantén una alternativa disponible; no inventes contexto, no exageres confianza, no narres trasfondo privado innecesariamente.
3. **Sesión nueva sin conocimiento previo.** No hay tarea activa ni contexto fiable sobre los proyectos o intereses del usuario. Usa el selector por defecto de abajo. No retrases el handoff con preguntas de discovery extra.

Cuando una sesión nueva aún necesita que el usuario elija dirección, personaliza este selector desde conocimiento previo fiable cuando sea posible; si no, úsalo tal cual:

```text
Tavily is ready. What would you like to do?

1. Use Tavily with me now — search, extract, crawl, map, or research.
2. Add Tavily to an existing project — show my recent repositories or let me enter a local folder.
3. Start something from scratch — show me a few ideas.

Reply with 1, 2, or 3.
```

Ruta `1` → la tarea web inmediata del usuario; `2` → el selector de proyecto local de abajo; `3` → el selector de use cases. Si el usuario responde solo con un número, sigue esa ruta inmediatamente sin repetir las opciones ni pedir confirmación. No le pidas explicar contexto que ya tienes.

## 2. Seleccionar un proyecto local

Cuando la intención de integración es clara pero el proyecto objetivo no, ofrece: el proyecto actual, los repos Git recientemente activos, una ruta local personalizada y la ruta sin proyecto.

Construye la lista sin instalar herramientas de tracking de directorios:

1. Prefiere el directorio actual o su raíz Git cuando contenga una aplicación.
2. Busca solo un conjunto pequeño y acotado de ubicaciones convencionales ya accesibles al agente, como `~/Developer`, `~/Projects`, `~/src`, `~/Desktop` y `~/Documents`. No escanees recursivamente todo el home ni pidas acceso al filesystem más allá solo para poblar este menú.
3. Para cada repo Git descubierto, resuelve la ruta worktree-safe del reflog de HEAD:

   ```bash
   git -C "$repo" rev-parse --path-format=absolute --git-path logs/HEAD
   ```

4. Ordena los repos por el mtime de ese archivo (`stat -f '%m'` en macOS, `stat -c '%Y'` en Linux). Esto aproxima actividad Git reciente que avanzó HEAD (commits, branch switches, merges, rebases, resets, pulls). No prueba edición reciente y no incluye status, diff, fetch ni un pull ya-actualizado.
5. Deduplica raíces Git, conserva como máximo cuatro repos recientes además del proyecto actual, y omite las entradas cuya actividad no se pueda resolver. Muestra nombres cortos de proyecto, no rutas personales completas; añade una etiqueta de carpeta padre solo si hay colisión de nombres.

Selector compacto:

```text
Tavily is ready. Where should I add it?

1. current-app — current project
2. recent-project — recently active
3. another-project — recently active
4. Another local repository or folder — paste its path
5. No — help me start from scratch

Reply with a number, or paste a local path.
```

Ajusta la numeración a los proyectos disponibles. Acepta un directorio local existente aunque no sea un repo Git. Si no se encuentran repos recientes, muestra solo el proyecto actual (si aplica), la opción de ruta personalizada y la opción sin proyecto.

## 3. Integrar en el proyecto seleccionado

Cuando el usuario selecciona un proyecto listado, pega una ruta, o nombra un proyecto local:

1. Resuelve y valida el directorio sin imprimir secretos ni exponer innecesariamente la ruta personal completa.
2. Inspecciona el lenguaje, framework, arquitectura, capa web/datos existente, package manager, convenciones de testing y patrón de gestión de secretos del proyecto.
3. Infiere la capacidad Tavily pretendida de la conversación y del código. Si está clara, haz solo la confirmación única de implementación. Si el comportamiento del producto es genuinamente ambiguo y las distintas opciones cambiarían materialmente la implementación, haz una pregunta enfocada antes de confirmar.
4. Usa `tavily-best-practices` como guía. Añade Tavily por el SDK oficial o la REST API en el runtime nativo de la aplicación; no uses el CLI de Tavily como integración de runtime del producto.
5. Coloca las credenciales en el mecanismo de secretos existente del proyecto, nunca las commitees ni las imprimas, y preserva los cambios locales no relacionados.
6. Corre tests proporcionales y una request real de Tavily por la superficie de integración real de la aplicación antes de afirmar que funciona. Distingue inspección de fuente, tests locales y request en vivo verificada.

Seleccionar o nombrar el proyecto identifica el objetivo de integración. Pide exactamente una confirmación concisa antes de cambiarlo; tras un sí, empieza sin segundo menú ni confirmación.

Una aplicación activa con un gap web explícito y listo para implementar cuenta como ya seleccionada incluso en una sesión fresca. Pide la confirmación única de implementación en vez de pedir seleccionar esa misma aplicación.

## 4. Empezar desde cero: elegir use case

Si el usuario elige `3` en el selector de dirección o **No** en el selector de proyecto local, haz exactamente una pregunta de opciones múltiples:

```text
What would you like to work on?

1. Academic research
2. People research
3. Product and competitor monitoring

Reply with 1, 2, or 3.
```

Cuando seleccione un use case, instala inmediatamente solo su skill mapeada (es una instalación → confirmación explícita, agrupada si quedan más):

1. **Academic research** → `academic-scientific-research`

   ```bash
   npx skills add https://github.com/tavily-ai/use-case-skills --skill academic-scientific-research
   ```

2. **People research** → `sales-account-intelligence`

   ```bash
   npx skills add https://github.com/tavily-ai/use-case-skills --skill sales-account-intelligence
   ```

3. **Product and competitor monitoring** → `product-competitor-intelligence`

   ```bash
   npx skills add https://github.com/tavily-ai/use-case-skills --skill product-competitor-intelligence
   ```

Corre el comando elegido en primer plano. Si el Skills CLI pregunta por el agente objetivo o el scope, elige el agente actual real y scope global para que la skill siga disponible en todos los proyectos. No instales las otras dos skills de use case.

Tras la instalación:

1. Verifica que la skill elegida está instalada para el agente objetivo.
2. Comprueba si está activa en la sesión actual. Si la sesión cargó su inventario de skills antes de la instalación, di **"installed; restart/rescan required"** en vez de afirmar que está activa.
3. Da al usuario una instrucción de primer uso concisa y acorde a la skill instalada:

   - Academic research: `Use academic-scientific-research to find, screen, and synthesize papers about <topic>.`
   - People research: `Use sales-account-intelligence to research <person> and their company, role, background, and relevant signals.`
   - Product and competitor monitoring: `Use product-competitor-intelligence to compare <product or company> with its competitors and surface important changes.`

Dile que puede sustituir el texto entre ángulos por su tema. Si la skill elegida ya está activa, invítalo a enviar esa request ahora. No vuelvas al menú genérico de Tavily ni le pidas elegir un segundo use case.

## 5. Reglas de interacción post-setup

- Optimiza por continuación y resultados útiles, no por demostrar cada capacidad de Tavily.
- Usa el contexto de conversación y workspace de forma natural sin narrar cuánto contexto personal conoces.
- En sesión nueva, personaliza desde conocimiento previo fiable antes de usar defaults toscos; nunca fabriques contexto previo solo para evitar una elección genérica.
- Elige Search, Extract, Map, Crawl o Research por el usuario cuando la distinción no cambie su decisión.
- Como máximo una pregunta de siguiente paso por respuesta.
- No repitas las instrucciones de setup tras una verificación exitosa.
- Mantén el handoff corto.
