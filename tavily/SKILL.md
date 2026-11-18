---
name: tavily
description: Configura, autentica y enruta Tavily para datos web en vivo — search/extract/map/crawl/research por CLI, Agent Skills para coding agents, el plugin nativo de Claude o MCP remoto. Úsala cuando el usuario mencione Tavily, pida instalarla o autenticarla ("instala tavily", "conecta tavily"), necesite resultados de búsqueda web actuales o extracción de URLs durante la sesión ("búsqueda web actual", "busca en internet", "extrae el contenido de esta URL"), quiera mapear/crawlear un sitio, un report de research multi-fuente con citas, o añadir Tavily a código de aplicación — incluso si nunca dice "Tavily" pero necesita datos web en tiempo real en la sesión del agente.
---

# Tavily

Configura Tavily una vez y enruta cada trabajo de Tavily — búsqueda web en vivo, extracción de URLs, mapeo/crawleo de sitios y research multi-fuente con citas — por la superficie correcta: CLI, Agent Skills, plugin nativo o MCP remoto.

## Alcance

- Cubre: onboarding de Tavily (preflight → CLI → auth → Agent Skills → verificación), enrutar una tarea de sesión al comando `tvly` correcto, conectar Tavily a clientes (Claude Code, Cursor, Codex, …) y a código de aplicación.
- NO cubre: invariantes del repo (→ `AGENTS.md`/rules). Los playbooks por comando viven en el bundle oficial de skills (`tavily-search`, `tavily-extract`, `tavily-map`, `tavily-crawl`, `tavily-research`) cuando está instalado — esta skill enruta y configura; si el bundle no está, los comandos `tvly` de abajo bastan.

## Restricciones

1. **Confirmar antes de cualquier instalación.** Instalar el CLI (`curl … | bash`), Agent Skills (`npx skills add`), el plugin nativo (`claude plugin install`) o un MCP server (`claude mcp add`) exige una confirmación explícita del usuario — agrupa todas las instalaciones pendientes en una sola pregunta por sesión de setup. Las comprobaciones read-only (`command -v`, `tvly auth --json`, `tvly --version`) y reutilizar una credencial ya autenticada son siempre autónomos. Por qué: una skill que dispara sola no debe ejecutar código remoto ni alterar el entorno global sin permiso.
2. **Nunca exponer secretos.** No imprimir, hacer echo, loguear, commitear ni incluir el setup token o la API key en la salida del chat; los secretos viajan solo como `${VAR}`.
3. **Nunca cambiar el Node por defecto del usuario.** Si el runtime activo es incompatible pero existe un Node 20+ vía nvm/fnm/mise/volta/asdf, prepéndelo al `PATH` solo para el comando de setup de Tavily.
4. **No bypasear el flujo normal de OAuth por navegador** mientras esté disponible: no hacer background de `tvly login`, no inspeccionar el fuente del CLI, no reconstruir URLs de OAuth, no inyectar shims de shell, no usar automatización de navegador. Esperar el consentimiento humano en el navegador no es una condición de error.
5. **Una superficie primaria por cliente** salvo que el usuario pida varias (p. ej. no instalar a la vez el plugin nativo de Claude y las Agent Skills genéricas en Claude Code).

## Inputs

Recolecta de la conversación antes de empezar; confirma los huecos en el mismo mensaje que la confirmación de instalación:

- **Objetivo**: datos web para esta sesión (Path A), Tavily dentro de una app (Path B), dentro de un cliente (Path C), research citado (Path D), o sin CLI (Path E).
- **Agente(s) objetivo**: a qué coding agent(s) configurar; detecta los instalados si no se especifica.
- **Setup token** (opcional): llega por las instrucciones invocadoras, `TAVILY_SETUP_TOKEN` o la plataforma circundante; nunca imprimirlo.
- **Estado de la máquina**: instalación/autenticación existente de `tvly`, runtimes de Node disponibles.

## Steps

Trata el setup como una máquina de estados: **preflight → CLI → autenticación → Agent Skills → verificación**. No le hagas al usuario un paso que se puede completar desde el terminal.

### 1. Preflight (autónomo, read-only)

1. Detecta el/los coding agent objetivo. Configura solo esos targets.
2. `command -v tvly` — si existe, corre `tvly auth --json` inmediatamente y reutiliza la credencial que funcione; nunca reemplaces una credencial operativa innecesariamente.
3. Busca un setup token antes de cualquier auth por navegador (instrucciones invocadoras, `TAVILY_SETUP_TOKEN`, plataforma; nunca imprimirlo).
4. Verifica Node.js antes de cualquier ruta que use `npx` (`tvly login`, `npx skills`).

Si los únicos prerrequisitos que faltan requieren instalaciones, agrúpalas en la confirmación única del paso 2.

### 2. Instalar el Tavily CLI (requiere confirmación)

Pregunta una vez, p. ej.: `El CLI de Tavily no está instalado. ¿Lo instalo con curl … | bash desde cli.tavily.com? [agentes detectados: …]`. Tras el sí:

```bash
curl -fsSL https://cli.tavily.com/install.sh | bash
command -v tvly
```

Si el instalador tuvo éxito pero `tvly` no está en el `PATH` del proceso actual, antepón `~/.local/bin` (o donde haya caído el binario) al `PATH` y reintenta. No edites archivos de arranque de shell salvo necesidad.

### 3. Autenticar

Empieza siempre con:

```bash
tvly auth --json
```

- `authenticated: true` → reutiliza y continúa.
- No autenticado + setup token disponible → sigue [references/setup-token-redemption.md](references/setup-token-redemption.md). Si la redención falla (expirado, ya usado, endpoint caído) o no se puede verificar el formato de config del CLI, cae a OAuth por navegador.
- No autenticado + sin token → corre `tvly login` **en primer plano** y deja que el flujo normal de autorización por navegador termine. Nota de compatibilidad: en algunos Node 18 el puente OAuth (`npx -y mcp-remote https://mcp.tavily.com/mcp`, cuyo stdout/stderr el CLI suprime) puede fallar antes de que arranque OAuth y solo manifestarse como timeout de login; si hay un runtime Node 20+ instalado, prepéndelo al `PATH` para este comando. Si tras un reintento el flujo de navegador no puede completarse, reporta el puente de autenticación como el blocker; no caigas en pedir copiar/pegar una API key manual.

Nunca pidas al usuario buscar una API key ni añadir `TAVILY_API_KEY` a `.env` para el onboarding del CLI.

### 4. Instalar Agent Skills (requiere confirmación)

Para onboarding de coding agent, propone instalación **global** para que Tavily funcione en todos los proyectos (project-local solo si el usuario lo pide). Pregunta una vez, enumerando los targets detectados:

```bash
npx skills add tavily-ai/skills --skill '*' --agent claude-code --global --yes
```

Añade un `--agent <target>` por cada target adicional detectado. Si el runtime activo no puede ejecutar el Skills CLI, usa un runtime Node 20+ ya instalado para este comando sin cambiar los defaults.

### 5. Verificar cada capa

Verifica las capas por separado en vez de tratar una llamada exitosa como prueba de que todo está listo:

```bash
command -v tvly && tvly --version             # CLI
tvly auth --json                              # Autenticación
tvly search "Tavily Search API" --json        # Request real
npx skills list --global --agent claude-code  # Agent Skills (ajusta --agent)
```

Confirma que las skills de Tavily esperadas están instaladas para el agente objetivo; si la sesión actual expone un inventario de skills, confirma allí también que son visibles/activas.

## Superficies de instalación

Elige la mejor superficie para el objetivo; evita duplicados salvo petición expresa. Cada una es una instalación → confirmación explícita (agrupada con los pasos 2/4 cuando sea posible).

### Claude Code: plugin nativo

```bash
claude plugin install tavily@claude-plugins-official
```

El plugin espera credenciales en el entorno/settings de Claude Code. Si la redención del setup token produjo una key cruda, configúrala sin mostrarla; si solo hay OAuth por navegador, prefiere la ruta de MCP remoto de abajo en vez de pedir copiar una key a mano.

### Cliente MCP: Tavily MCP remoto

```bash
claude mcp add tavily-remote-mcp --transport http https://mcp.tavily.com/mcp/
# Search + Extract keyless:
claude mcp add tavily-remote-mcp --transport http https://mcp.tavily.com/mcp/ \
  --header "X-Tavily-Access-Mode: keyless"
```

## Elige tu camino

- Necesitas datos web en esta sesión → Path A
- Añadir Tavily a código de aplicación → Path B
- Tavily dentro de Claude Code / Cursor / Codex / … → Path C
- Report de research multi-fuente con citas → Path D
- No quieres instalar el CLI → Path E

### Path A: CLI skills / herramientas web en vivo (datos de sesión)

Escalada por defecto: **Search** (descubrimiento) → **Extract** (URLs ya conocidas) → **Map** (sitio conocido, páginas no) → **Crawl** (muchas páginas de una sección) → **Research** (síntesis citada).

```bash
tvly search "latest AI agent releases" --json
tvly extract "https://example.com/article" --json
tvly map "https://docs.example.com" --json
tvly crawl "https://docs.example.com" --output-dir ./docs/
tvly research "competitive landscape for AI coding agents" --json
```

Haz hand-off a la skill oficial instalada que corresponda (`tavily-search`, `tavily-extract`, `tavily-map`, `tavily-crawl`, `tavily-research`) cuando exista. Search y Extract funcionan keyless; Map, Crawl y Research requieren autenticación.

### Path B: build skills / integrar Tavily en una app

Para código que seguirá corriendo cuando acabe la sesión: usa el SDK oficial o la REST API, nunca el CLI como integración de runtime.

```bash
pip install tavily-python       # Python
npm install @tavily/core        # JS/TS
```

Coloca `TAVILY_API_KEY` en el mecanismo de secretos existente del proyecto (`.env` gitignoreado / secret manager); nunca commitear ni imprimir. Usa `tavily-best-practices` (si está instalada) como guía de implementación. Corre una request real por la superficie de integración real de la app antes de afirmar que funciona. Modificar el proyecto del usuario exige la confirmación única descrita en [references/post-setup-handoff.md](references/post-setup-handoff.md).

### Path C: conectar Tavily a un agente o cliente

Elige la superficie nativa del cliente; verifica después una búsqueda real y que las skills quedaron instaladas. Opciones Claude Code: plugin nativo, Agent Skills globales, MCP remoto (todas arriba). Otros agentes: Agent Skills globales con `--agent` explícito; MCP cuando el cliente sea principalmente un host MCP.

### Path D: research

```bash
tvly research "topic" --model pro
tvly research "topic" --json
tvly research "topic" --no-wait --json
tvly research status <request_id> --json
tvly research poll <request_id> --json
```

Research requiere autenticación. No existe hoy un bundle separado "Blueprints": no afirmes que existe; enruta los entregables citados por `tavily-research` / `tvly research`.

### Path E: sin instalar el CLI

REST: `https://api.tavily.com` (`/search`, `/extract`, `/crawl`, `/map`, `/research`). Search/Extract keyless: envía `X-Tavily-Access-Mode: keyless` y omite la key; con rate limit. Crawl/Map/Research requieren auth.

## Superficie actual del CLI

```text
tvly
├── login
├── logout
├── auth
├── search
├── extract
├── crawl
├── map
└── research
    ├── run
    ├── status
    └── poll
```

Ante una opción desconocida, corre `tvly --help` / `tvly <command> --help` antes de usarla.

## Checks (criterios de éxito)

Reporta **Tavily ready** solo cuando todas las líneas aplicables sean verdad:

```text
Tavily CLI installed
Authentication verified
Live Tavily Search verified
Agent Skills installed
Agent Skills active in current session
```

Si las skills están instaladas pero el cliente necesita restart/rescan, di **"installed; restart/rescan required"** — nunca afirmes que están activas prematuramente.

## Stop conditions

Pausa y pide al humano cuando:

- Cualquier instalación (CLI, skills, plugin, MCP) está sin confirmar — propón y espera, agrupando todas las instalaciones en una pregunta.
- El usuario declina una instalación — continúa solo con las superficies ya disponibles (p. ej. Path E keyless) y dilo.
- El OAuth por navegador no puede completarse tras el reintento con Node 20+ — reporta el puente de autenticación como blocker; no pidas API keys manuales.
- Se propone un cambio de código de proyecto (Path B) — exactamente una confirmación concisa antes de modificar; tras el sí, continúa sin más menús ni segundas confirmaciones.

## Recovery

- **Instalador OK, `tvly` fuera del PATH** → antepón el directorio de instalación (habitualmente `~/.local/bin`) al `PATH` del proceso actual; reintenta `command -v tvly`.
- **`tvly login` en timeout en Node 18** → reintenta una vez con un runtime Node 20+ instalado prepuesto al `PATH`; luego reporta blocker.
- **Redención de token fallida** → cae automáticamente a OAuth por navegador.
- **Node incompatible para `npx skills`** → usa un runtime Node 20+ instalado solo para ese comando.
- **Script de bootstrap ausente** → usa la redención por `curl` documentada en [references/setup-token-redemption.md](references/setup-token-redemption.md); si no puedes verificar el formato de config del CLI, usa OAuth por navegador.
- Tras cada corrección, vuelve a la verificación del paso 5 con los mismos checks.

## Tras la verificación: convertir el setup en trabajo útil

Antes de proponer siguientes pasos tras un setup exitoso, lee [references/post-setup-handoff.md](references/post-setup-handoff.md) — define cómo clasificar el contexto de la sesión, seleccionar proyecto y hacer el handoff con como máximo una pregunta.

## Referencias (cargar solo cuando se necesiten)

- [references/setup-token-redemption.md](references/setup-token-redemption.md) — lee cuando haya un setup token presente y la máquina no esté autenticada.
- [references/post-setup-handoff.md](references/post-setup-handoff.md) — lee tras una verificación exitosa, antes de proponer siguientes pasos.
