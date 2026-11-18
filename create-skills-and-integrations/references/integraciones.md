# Integraciones: conectar MCP, plugins, commands, agentes y subagentes a una skill

Lee este archivo en Fase D, antes de conectar cualquier herramienta. La regla que gobierna
todo: **las herramientas sirven a la skill, no al revés.** Empieza con la integración mínima
que resuelve la necesidad real y escala solo con evidencia.

## 1. La escalera de integración (de más barata a más cara)

```text
1. Rule de routing      → cero coste nuevo; la skill ya existe, solo se hace descubrible
2. Command              → ergonómico; se paga solo al invocarlo
3. MCP server           → datos/acciones externas; coste de contexto por tool definition
4. Plugin               → capacidad empaquetada oficial (skills incluidas)
5. Subagente / agente   → otro perfil de trabajador; máximo coste, usar último
```

Escala al siguiente peldaño solo cuando el actual no resuelva la necesidad.

## 2. Rule de routing (hacer la skill descubrible en el flujo)

Patrón: reglas cortas que enrutan hacia skills, manteniendo el prompt siempre-activo pequeño.

```markdown
### Skills (en AGENTS.md)
- Al cambiar componentes UI, carga el skill `ui-change`.
- Al depurar errores de producción, carga el skill `incident-triage`.
```

Prueba: "¿quiero que el agente cargue esta skill aunque no esté pensando en ella?" — no;
por eso es routing y no una regla de comportamiento completo.

## 3. Command como atajo que dispara skills

El command es determinista (lo invocas, el tool inyecta el prompt); la skill es la política
revisable. Sepáralos: actualizas la skill → cambia el comportamiento sin cambiar la
invocación; actualizas el command → cambiaste la invocación misma.

Ejemplos de la fuente:

- `/release` → "Carga el skill `release` y sigue su checklist."
- `/refactor` → "Carga los skills `tanstack` y `panda-css` y refactoriza este componente de
  Next.js. Usa el MCP Context7 si tienes dudas de documentación que las skills no cubren."
- Con parámetros posicionales: `/pr 123 bot-integration` →
  "Carga el skill `pr`. Crea un PR que arregle el issue #$1 y apunte a la rama $2."

Reglas: lista de commands corta y memorable; la variable va explícita en la llamada; la
lógica dura (convenciones, qué significa "done") vive en la skill.

## 4. MCP servers

Cuándo: la skill necesita datos o acciones fuera del repo — navegador (Playwright), base de
datos (Supabase/Postgres), issues (Linear), errores de producción (Sentry), docs actualizadas
(Context7), despliegue (Netlify/Vercel), Figma.

Configuración por scopes: `local` (solo tu máquina), `project` (`.mcp.json` compartido en el
repo), `user` (todas tus sesiones). Ejemplo `.mcp.json` con stdio:

```json
{
  "mcpServers": {
    "mi-servidor": {
      "command": "npx",
      "args": ["-y", "@org/mi-servidor@VERSION_FIJA"],
      "env": { "MI_API_KEY": "${MI_API_KEY}" }
    }
  }
}
```

Reglas de higiene:

1. **Secretos solo como `${VAR}`** — jamás valores literales en el JSON; el valor va en el
   entorno o en un gestor de secretos.
2. **Empezar read-only** y con blast radius acotado; mínimos privilegios por servidor.
3. **Pocos servers a la vez**: cada tool definition ocupa contexto. El Tool Search de
   Claude Code mitiga (~85% de reducción con multi-server), pero el principio se mantiene.
4. Preferir servers **oficiales** del proveedor sobre community; debuggear con MCP Inspector.
5. **Fijar la versión** (`@x.y.z`, no `@latest`) para servers críticos.
6. En Windows, envolver `npx` con `cmd /c` si el cliente no resuelve el shim.
7. Tras añadir un server, verificar con `claude mcp list` (o el equivalente del host) y
   reiniciar la sesión si hace falta.

## 5. Plugins

Cuándo: existe una skill/capacidad ya empaquetada y mantenida por un proveedor oficial —
usarla antes de reinventarla (p. ej. `skill-creator` para redactar skills, `pdf` para
documentos, `browser-use` para automatización de navegador).

Reglas:

1. Antes de crear, pregunta si ya existe un plugin oficial que lo cubra.
2. **Instalar, habilitar o conectar un plugin SOLO con confirmación explícita del usuario.**
   Una referencia o mención del plugin no es permiso.
3. Si el plugin trae skills propias, evalúalas con el mismo checklist de las 6 preguntas —
   "empaquetada" no significa "bien escrita".

## 6. Subagentes y agentes

Skills cambian lo que el agente **sabe**; agentes cambian **quién** hace el trabajo y qué
puede **acceder** (otro system prompt, otro acceso a tools, otro modelo/temperatura).

Perfiles típicos:

- **plan**: acceso read-only, prompt orientado a preguntas de clarificación y plan detallado.
- **build**: modelo rápido/barato, enfocado a ejecutar planes.
- **review**: modelo lento/caro, para revisar diffs.

Subagentes: el agente padre delega en paralelo. Los "explore" integrados (ripgrep) son el
caso común de búsqueda de contexto. Patrón útil: un subagente especializado de investigación
web que ahorra contexto al agente principal.

**Cuándo subir de skill a agente** (solo si aparece):

- La tarea necesita otro LLM (p. ej. revisión con modelo caro, ejecución con modelo barato).
- Problemas de scoping de permisos (aislamiento de acceso a tools).
- Contaminación visible del contexto del agente principal.

**Si el trabajo es paralelo**: límites de archivos no solapados por agente, un coordinador
que integra, evidencia por subtarea, y git worktree por agente para aislar el working tree.

## 7. Tabla de decisión rápida

| Necesidad | Vehículo | Ejemplo |
|---|---|---|
| Que el agente la use cuando toque UI | Rule de routing | "Al cambiar UI, carga skill `ui-change`" |
| Ejecutar un ritual largo sin retipearlo | Command → skill | `/release` |
| Leer/escribir datos externos | MCP server | Supabase, Linear, Sentry |
| Capacidad compleja ya empaquetada | Plugin | `skill-creator`, `pdf` |
| Otro nivel de acceso o de modelo | Subagente/agente | plan / build / review |
| Búsqueda de contexto veloz | Subagente explore (built-in) | "encuentra dónde vive el dark mode" |

## 8. Seguridad transversal

- Confirmación explícita del usuario para: instalar dependencias, instalar/habilitar plugins
  o MCP servers, `git push`, borrar archivos, builds completos.
- Permitido sin preguntar: leer/listar archivos, verificaciones por archivo (tsc/prettier/
  eslint/vitest de un solo archivo).
- Secretos: `${VAR}` en configs, `<REDACTED>` en documentación, nunca en la skill ni en logs.
- Cada integración añadida debe poder quitarse sin dejar la skill rota: referénciala como
  opcional con fallback ("si X no está disponible, continúa con Y y dilo").
