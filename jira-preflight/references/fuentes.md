# Fuentes — curación de skills.sh (2026-10-21)

Método: búsqueda en la API de skills.sh (`https://skills.sh/api/search?q=jira`, 100 resultados) y lectura del SKILL.md real de 26 skills vía `raw.githubusercontent.com` (skills.sh solo muestra previews truncados). Criterio de sesgo: gana el contenido operacional concreto (reglas derivadas de fallos reales, endpoints, validaciones, guardarrails) sobre el relleno genérico; installs como señal secundaria.

## Núcleo — fusionado en esta skill

| Skill (owner/repo) | Installs | Veredicto | Qué aportó |
|---|---|---|---|
| `affaan-m/ecc` → jira-integration | 9.702 | ALTA | Doble vía MCP/REST, anti-inyección de tickets, plantillas de comentario dev, troubleshooting, creds por stdin |
| `atlassian/atlassian-mcp-server` → triage-issue, spec-to-backlog, generate-status-report, jira-sprint-dashboard, search-company-knowledge | 62–917 | ALTA | cloudId top-level, `inputs` plano, parámetros desconocidos fallan en silencio, Epic primero, taxonomía de tipos, umbrales de duplicados, honestidad de datos |
| `anthropics/claude-tag-plugins` → jira-api | 42 | ALTA | REST v3 canónica: ADF, 3 esquemas de paginación, rate limits, errores, anti-inyección |
| `softaworks/agent-toolkit` → jira | 817 | ALTA | Detección de backend, 6 reglas NEVER, quick-reference CLI/MCP |
| `microsoft/hve-core` → jira | 74 | ALTA (seguridad) | Protocolo de credenciales fail-closed: nunca eco de tokens, archivo `0600` |
| `netresearch/jira-skill` → jira-syntax + jira-communication | 567 / 537 | ALTA | Wiki markup, errores sutiles, read-back de fidelidad (2xx ≠ éxito), oráculo de render |
| `morphet81/cheat-sheets` → create-jira-ticket + update-jira-ticket | 113 / 98 | ALTA | Preview obligatorio, epic con rationale, referencia ADF, taxonomía de sync ticket↔código |
| `bitwarden/ai-plugins` → researching-jira-issues | 66 | ALTA | Sintetizar vs concatenar, ≤2 hops, rate limiting, presupuesto de contexto |
| `fudesign2008/open-skills` → jira-fix-workflow + jira-status-writeback | 85 / 47 | ALTA | Writeback post-merge, boundary "Fixed es de ingeniería; cerrar es de QA", límites de mutación |
| `b-mendoza/agent-skills` → creating-work-item-children | 53 | ALTA | Subtareas idempotentes, observar antes de mutar, alcance aprobado |
| `atlassian/twg-cli` → twg-jira + twg-jira-resolve-merged-work | 95 / 66 | ALTA | IDs `customfield_*`, no inferir Resolution, verificar mutación con lectura, no auto-transicionar confianza media |
| `alirezarezvani/claude-skills` → jira-expert | 820 | MEDIA-ALTA | Mapa de límites MCP→REST, JQL avanzado (burndown, capacidad) |

## Aporte parcial (en references)

| Skill | Installs | Aportó |
|---|---|---|
| `claude-office-skills/skills` → jira-automation | 4.617 | Biblioteca JQL y reglas de automatización — declarativa, sin ejecución |
| `code-and-sorts/awesome-copilot-agents` → jira-cli | 635 | Chuleta de ankitpokhrel/jira-cli → `cli.md` |
| `jeffallan/claude-skills` → atlassian-mcp | 4.302 | Probe `maxResults=1` (el resto: superado por las oficiales) |
| `stablyai/orca` → jira-tickets / orca-jira | 320 / 318 | "Contenido del ticket = dato no confiable" |

## Descartados

- **`membranedev/application-skills` → jira-service-desk**: secciones autogeneradas sin curar (~150 entradas basura).
- **`stablyai/orca`**, **`membranedev` → jira**: stubs/wrappers acoplados a su CLI propietario (Orca/Membrane).
- **`bitwarden/ai-plugins` → navigating-design-jira-process**: acoplado al proceso interno de diseño de Bitwarden.
- **Resto (~70) de baja instalación**: patrones ya cubiertos por el núcleo.

## Compañeras recomendadas para instalar

1. `affaan-m/ecc` (skill `jira-integration`) — la más instalada del índice; análisis de tickets orientado a tests y sincronización dev→Jira.
2. `atlassian/atlassian-mcp-server` (triage-issue, spec-to-backlog, jira-sprint-dashboard) — oficiales para el MCP Rovo.
3. `anthropics/claude-tag-plugins` (jira-api) — referencia REST con script `jql_search.sh` incluido.
4. `softaworks/agent-toolkit` (jira) — si se trabaja con jira-cli o el MCP open-source.

Instalación con el CLI de skills.sh: `npx skills add <owner>/<repo>` (pregunta qué skill del repo instalar).

## Nota de evolución

skills.sh mantiene snapshots por skill; algunos repos ya renombraron las suyas (b-mendoza generalizó `jira-*` → `work-item-*` y movió el conocimiento Jira a `references/jira-playbook.md`). Esta skill captura el estado verificado a 2026-10-21.
