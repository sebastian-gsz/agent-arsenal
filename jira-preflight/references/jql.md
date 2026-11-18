# JQL — biblioteca y reglas

Fuentes: `claude-office-skills/skills#jira-automation`, `alirezarezvani/claude-skills#jira-expert`, `atlassian/atlassian-mcp-server` (triage-issue, jira-sprint-dashboard), `anthropics/claude-tag-plugins#jira-api` — ver `fuentes.md`.

## Reglas duras

- **Acotado siempre** (≥1 cláusula de filtro): un `ORDER BY` pelado es un `400`.
- **Probe:** valida queries costosas con `maxResults=1` antes de la ejecución completa.
- `fields` explícito en `/search/jql` (default = solo `id`).
- La búsqueda semántica (Rovo `search`) **no es JQL**: pásale lenguaje natural, nunca JQL/CQL — falla en silencio con resultados pobres.

## Biblioteca

| Objetivo | JQL |
|---|---|
| Mis issues activos | `assignee = currentUser() AND statusCategory != Done ORDER BY updated DESC` |
| Stale (>7d sin mover) | `project = PROJ AND status = "In Progress" AND updated < -7d` |
| Bloqueados | `project = PROJ AND statusCategory != Done AND (status = Blocked OR labels in (blocked, blocker))` |
| Review sin reviewer | `status = "Code Review" AND "Code Reviewer" is EMPTY` |
| Alcance de release | `fixVersion = "v2.5.0" AND resolution = Unresolved` |
| Velocity | `issuetype = Story AND sprint in closedSprints() AND resolved >= -90d` |
| Burndown del sprint | `sprint = 23 AND status changed TO "Done" DURING (startOfSprint(), endOfSprint())` |
| Capacidad del equipo | `assignee in (usuario1, usuario2) AND sprint in openSprints()` |
| Sprint activo | `project = PROJ AND sprint in openSprints() ORDER BY Rank ASC` |
| Snapshot (si no hay sprints) | `project = PROJ AND statusCategory != Done AND updated >= -60d ORDER BY priority DESC` |

**Triage de duplicados** (3 búsquedas dirigidas; incluye resueltos — puede haber reabiertos/regresiones):

1. Firma del error (frase clave/excepción, no el stack completo): `project = PROJ AND (text ~ "firma" OR summary ~ "firma") AND issuetype = Bug ORDER BY created DESC`
2. Componente: `project = PROJ AND text ~ "keywords de componente" AND issuetype = Bug ORDER BY updated DESC`
3. Síntoma: `project = PROJ AND summary ~ "síntoma" AND issuetype = Bug ORDER BY priority DESC, updated DESC`

**Hijos de un Epic en next-gen** (cuando `subtasks` viene vacío): `parent = EPIC-KEY`.

## Funciones útiles

`openSprints()` / `closedSprints()` · `membersOf("grupo")` · `issueHistory()` · `linkedIssues()` · `startOfSprint()` / `endOfSprint()` con `DURING`.

## Performance

- Evita wildcards iniciales (`text ~ "*err"`) y `~` sobre campos de texto grandes: caros.
- Queries repetidas → guárdalas como filtro con nombre en vez de reejecutarlas.
- `maxResults` 20-100 y pagina; no pidas todo de golpe.

## Honestidad de datos

- Story points, sprint y labels de equipo viven en **custom fields**: pídelos explícitamente (`customfield_*` o `fields=*all`); en Rovo vuelven bajo `fields.customFields`, no como keys top-level.
- No afirmes "sin blockers" si solo consultaste status/labels — di qué verificaste.
- Issues de hace >2 años y drafts: señálalos como posiblemente desactualizados.
