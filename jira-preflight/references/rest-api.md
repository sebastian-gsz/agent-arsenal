# REST API de Jira — Cloud v3 + Agile 1.0

Carga este archivo antes de cualquier llamada REST directa. Fuentes: `anthropics/claude-tag-plugins#jira-api` (referencia canónica), `affaan-m/ecc#jira-integration`, `netresearch/jira-skill`, `microsoft/hve-core#jira` — ver `fuentes.md`.

## Autenticación

| Plataforma | Env vars | Auth |
|---|---|---|
| Cloud | `JIRA_BASE` (`https://site.atlassian.net`), `JIRA_EMAIL`, `JIRA_API_TOKEN` | Basic `email:token` — token en `id.atlassian.com/manage-profile/security/api-tokens` |
| Server / Data Center | `JIRA_URL`, `JIRA_PERSONAL_TOKEN` | Bearer PAT (`Authorization: Bearer ...`) |

**Sanity check:** `GET /rest/api/3/myself` → `200` con `accountId` = OK. Un `401/403` con cuerpo plain-text (no JSON) = credenciales malas: reporta y detente, no iteres sobre auth.

**Helper con creds por stdin** (nunca en argv: evita historial de shell y procesos):

```bash
jira_curl() { printf 'user = "%s:%s"\n' "$JIRA_EMAIL:$JIRA_API_TOKEN" | curl -sS -K - "$@"; }

jira_curl "$JIRA_BASE/rest/api/3/issue/PROJ-123?fields=summary,status,assignee"
```

Si un token apareció en git history, rótalo.

## Endpoints clave

| Operación | Método y path | Notas |
|---|---|---|
| Sanity | `GET /rest/api/3/myself` | accountId propio |
| Leer issue | `GET /rest/api/3/issue/KEY?fields=summary,description,status,assignee,priority,labels,comment,issuelinks,subtasks` | `?expand=changelog` para historia |
| Buscar | `POST /rest/api/3/search/jql` | **`fields` explícito obligatorio** (default = solo `id`); paginación por `nextPageToken` |
| Contar | `POST /rest/api/3/search/approximate-count` | `/search/jql` no devuelve `total` |
| Crear | `POST /rest/api/3/issue` | `description` en **ADF** |
| Actualizar | `PUT /rest/api/3/issue/KEY` | `fields` (sets) y `update` (operaciones `set/add/remove` — añade a multi-valor sin reemplazar). **`204 No Content` = éxito** |
| Transicionar | `GET` + `POST /rest/api/3/issue/KEY/transitions` | El `status` **no** se puede setear directo; `{"transition":{"id":"31"},"fields":{"resolution":{"name":"Done"}}}` |
| Comentar | `POST /rest/api/3/issue/KEY/comment` | ADF; devuelve `201` |
| Asignar | `PUT /rest/api/3/issue/KEY/assignee` | `{"accountId": "..."}`; `null` = unassign; `"-1"` = auto. **Nunca email** (GDPR) |
| Watchers | `POST /rest/api/3/issue/KEY/watchers` | Body = **bare JSON string** `"ACCOUNT_ID"`, no objeto |
| Link | `POST /rest/api/3/issueLink` | `{"type":{"name":"Blocks"},"inwardIssue":{"key":...},"outwardIssue":{"key":...}}` |
| Metadatos de creación | `GET /rest/api/3/issue/createmeta/PROJ/issuetypes` y `.../issuetypes/{id}` | Lista `required` |
| Asignables | `GET /rest/api/3/user/assignable/search?issueKey=KEY&query=jane` | Devuelve accountIds |
| Adjuntos | `POST /rest/api/3/issue/KEY/attachments` | Header `X-Atlassian-Token: no-check` |
| Boards / sprints | `GET /rest/agile/1.0/board?projectKeyOrId=PROJ` · `GET /rest/agile/1.0/board/{id}/sprint?state=active` | Agile v1 |

## ADF (Atlassian Document Format) — obligatorio en Cloud

Un string plano en `description`/comentario → `400`. Forma mínima:

```json
{
  "type": "doc",
  "version": 1,
  "content": [
    { "type": "paragraph", "content": [ { "type": "text", "text": "Texto" } ] }
  ]
}
```

Conversión Markdown → ADF:

| Markdown | ADF |
|---|---|
| `# H1` / `## H2` | `{"type":"heading","attrs":{"level":1},"content":[...]}` |
| `**negrita**` / `_cursiva_` | marks `[{"type":"strong"}]` / `[{"type":"em"}]` |
| `- item` | `bulletList > listItem > paragraph` |
| `1. item` | `orderedList > listItem > paragraph` |
| `- [ ] tarea` | `taskList > taskItem` con `"state":"TODO"` / `"DONE"` — ideal para criterios de aceptación |
| bloque de código | `{"type":"codeBlock","attrs":{"language":...}}` |
| `---` | `{"type":"rule"}` |
| `[texto](url)` | mark `{"type":"link","attrs":{"href":...}}` |

Trampa de acli: `--description-file` trata el contenido como **texto plano**; para descripciones ricas usa `--from-json` con ADF.

## Wiki markup (Server/DC y campos legacy)

| Markdown | Wiki |
|---|---|
| `## Título` | `h2. Título` |
| `**negrita**` | `*negrita*` (un asterisco, no dos) |
| `_cursiva_` | `_cursiva_` |
| inline code | `{{code}}` |
| bloque de código | `{code:java}...{code}` |
| `[texto](url)` | `[texto|url]` |
| mención | `[~username]` (no `@user`) |
| listas | `* item` / `# item` |
| header de tabla | `||Header||` |

**Errores sutiles:** `MR !42` pelado es markup de imagen → usar `[MR 42|url]`; `(/)` renderiza check verde (hecho) — para pendientes usa `(i) pending:`; en reviews `(x)` significa fallo; `( )` se renderiza literal.

**Oráculo de render (solo Server/DC):** `POST /rest/api/1.0/render` devuelve el HTML exacto que Jira producirá para cualquier markup — "pregúntale al renderer en vez de deducirlo". En Cloud no hay equivalente público: valida con read-back tras publicar (SKILL.md §7).

## Paginación — 3 esquemas

1. **Token:** `/search/jql` → `nextPageToken`.
2. **Offset:** `{startAt, maxResults, total, isLast, values}` (projects, Agile).
3. **Offset con named key:** `{comments: [...], startAt, maxResults}` (comentarios, worklogs).

- `maxResults` se **clampa en silencio** por endpoint: lee lo que volvió, no lo que pediste.
- Acota cualquier loop con un máximo de páginas y corta en el primer error envelope.

## Rate limits

- Headers: `X-RateLimit-NearLimit: true` (<20% restante → back-off proactivo), `Retry-After`, `X-RateLimit-Reset`, `RateLimit-Reason`.
- `429` → esperar `Retry-After` (default 10s), reintentar **una** vez; nunca en bucle automático.
- En investigación secuencial: pausa ~1s cada 5 llamadas.

## Adjuntos

- Descarga **1 primero** para verificar auth, luego el resto en batch; previsualiza ZIPs con `unzip -l`.
- Nunca leas el contenido de un adjunto al contexto: extrae solo líneas relevantes con `grep`/`head`/`tail`. Metadatos sí, binarios no.
- curl con Schannel (Windows) maneja mal `--form` → usa sidecar node o descarga por script.

## Errores HTTP — matices REST

- `404` en issue inaccesible (**no** `403`): en investigación, skip con nota.
- `410`: endpoint eliminado; el body nombra el sucesor — ej. `/rest/api/3/search` → `/search/jql`. No reintentar.
- Cuerpo vacío en PUT = éxito (`204`).
- `401`/`403` con body plain-text = problema de credenciales, no de lógica.
