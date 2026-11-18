# CLI de Jira — acli, jira-cli (ankitpokhrel), twg

## Orden de detección

1. `which acli` (CLI oficial de Atlassian) / `which jira` (ankitpokhrel) / `which twg`.
2. Tools MCP en la sesión → SKILL.md §1.
3. REST con curl → `rest-api.md`.
4. Nada → informa y sugiere instalación (abajo); nunca operes a ciegas.

## acli (CLI oficial de Atlassian)

Setup: `brew tap atlassian/acli && brew install acli` → `acli auth status` (si falta auth: STOP con esa instrucción).

| Intención | Comando |
|---|---|
| Proyecto desde rama git | `git branch --show-current` → regex `[A-Z][A-Z0-9]+-[0-9]+` → prefijo = project key |
| Buscar | `acli jira workitem search --jql "project = PROJ AND issuetype = Epic" --json --limit 50` |
| Ver | `acli jira workitem view PROJ-123 --fields '*all' --json` |
| Crear | JSON (con ADF) → `acli jira workitem create --from-json /tmp/jira.json --json` |
| Editar | `acli jira workitem edit --from-json /tmp/jira.json --yes` |
| Comentar | `acli jira workitem comment create --key PROJ-123 --body "..."` |

- **Epic recomendado:** match temático de summary + descripción; preferir epics no Done/Closed en empates; escribir rationale de una frase.
- Si la API rechaza `parent` (proyectos clásicos usan el customfield Epic Link): inspecciona `--generate-json` y reintenta **una** vez.
- Trampa: `--description-file` = texto plano; rich text → `--from-json`.
- Figma (opcional): adjuntar diseños vía entity properties — `GET/PUT /rest/api/3/issue/KEY/properties/` con `Authorization: Bearer $(acli auth token)`.

## jira-cli (ankitpokhrel/jira-cli)

Setup: `brew install ankitpokhrel/jira-cli/jira-cli` → `jira init` (guiado; API token de Cloud).

| Intención | Comando |
|---|---|
| Ver | `jira issue view PROJ-123 --comments 10 --plain` |
| Listar mío | `jira issue list -a$(jira me) -s"In Progress" -yHigh --created week` |
| JQL crudo | `jira issue list -q "project = PROJ AND status != Done"` |
| Crear | `jira issue create -tBug -s"Título" -b"..." -yHigh -Cbackend --no-input` |
| Subtarea | añadir `-P"PROJ-123"` |
| Custom field | `--custom story-points=3` |
| Transicionar | `jira issue move PROJ-123 "In Progress"` |
| Cerrar con resolución | `jira issue move PROJ-123 Done -R"Fixed"` |
| Asignar / desasignar | `jira issue assign PROJ-123 $(jira me)` / `jira issue assign PROJ-123 x` |
| Sprint | `jira sprint list --state active` · `jira sprint add SPRINT-ID PROJ-123` |
| Worklog | `jira issue worklog add PROJ-123 "2h 30m" --comment "..."` |
| Link / clonar | `jira issue link PROJ-123 PROJ-456 Blocks` · `jira issue clone PROJ-123 -s"..."` |
| Epic | `jira epic create -n"Nombre"` · `jira epic add EPIC-1 PROJ-123` |
| Scripting | `--plain --no-headers --columns key,summary,status,assignee` · `--raw` (JSON) · `--csv` · `--paginate 10:50` |

**Multi-línea:** el CLI se atraganta con strings multi-línea → heredoc a archivo:

```bash
cat > /tmp/jira_body.md <<'EOF'
Descripción larga aquí...
EOF
jira issue create -tTask -s"Título" -b"$(cat /tmp/jira_body.md)" --no-input
```

## twg (Atlassian)

| Intención | Comando |
|---|---|
| Leer | `twg jira workitem get KEY --site SITE -o json` (`--comments`, `--remote-links`, `--full`) |
| Varias a la vez | `twg jira workitem bulk-get` con **todas las keys de una vez** — nunca loopear `get` |
| Query | `twg jira workitem query --jql <jql> --fields <fields>` (evita hidratación extra) |
| Descubrimiento fuzzy | `twg jira workitem search` · semántico: `twg rovo search --app jira` |
| Transición (descubrir) | `twg jira workitem transition --id KEY --site SITE -o json` **sin** `--transition-id` → lista transiciones y required fields |
| Transición (ejecutar) | mismo comando **con** `--transition-id` y los `requirements` |

- Antes de una mutación no familiar: `twg help describe "<path>"`.
- JSM (requests, queues, SLAs, aprobaciones) es otro dominio: los workitems subyacentes sí son Jira.

## Reglas comunes de CLI

- Nunca adivines comandos: `--help` primero.
- `--no-input` solo con required fields conocidos (SKILL.md §2).
- Multi-línea siempre por archivo/stdin, nunca inline con saltos.
- Bulk solo con aprobación explícita: cada edit notifica watchers.
