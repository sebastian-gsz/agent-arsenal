# competences

Colección de **skills para agentes de código** (formato `SKILL.md`, compatibles con ZCode / Claude Code) junto con un proyecto de referencia sobre escaneo de vulnerabilidades.

## Contenido

| Skill | Descripción |
|---|---|
| [`arquitect-prime`](arquitect-prime/) | Arquitecto de software senior para proyectos TypeScript full-stack: monorepo, contratos OpenAPI, schemas y scaffolding inicial. |
| [`design-md`](design-md/) | Extrae el sistema de diseño de un codebase y genera un `DESIGN.md` con tokens en YAML (formato Design MD de Google Stitch). |
| [`git-commit-message`](git-commit-message/) | Reglas para commits seguros y consistentes: Conventional Commits, commits atómicos y manejo de caracteres especiales. |
| [`google-secure-coder`](google-secure-coder/) | Hub de la suite de Google para datos en GCP (BigQuery, Dataflow, Composer, catálogos) y seguridad de código, con enrutado a módulos. |
| [`create-skills-and-integrations`](create-skills-and-integrations/) | Crea skills siguiendo las reglas de autoría (routing, progressive disclosure, checklist de 6 preguntas) y las conecta con commands, MCP, plugins o subagentes bajo una escalada de integración. |
| [`jira-preflight`](jira-preflight/) | Operación segura de Jira de extremo a extremo: detección de canal (MCP/CLI/REST), preflight antes de escribir y JQL acotado. |
| [`standard-operating-procedure`](standard-operating-procedure/) | Redacta Procedimientos Operativos Estándar (SOP/POE) en 3 fases (Sistema → Proceso → SOP) con dueño, trigger y escalación; crea o actualiza archivos en `sops/` del proyecto. |
| [`orquestador-seed`](orquestador-seed/) | Índice de la semilla orquestadora: harness documental multiagente (contratos `ARCHITECTURE.md`, workboard con claims, protocolos) con enrutado a sus fases 0–7 en el repo hermano. |
| [`appwrite`](appwrite/) | Hub de las skills de Appwrite: enruta al SDK según lenguaje/plataforma (TS, Dart, Kotlin, Swift, Python, PHP, Go, Ruby, Rust, .NET) y a la CLI para despliegue y operación. |
| [`tavily`](tavily/) | Instala, autentica y enruta Tavily para datos web en vivo: CLI (search/extract/crawl), Agent Skills, plugin de Claude o MCP remoto. |
| [`canvas-apps-ui-gen`](canvas-apps-ui-gen/) | Genera YAML listo para pegar de Power Apps Canvas a partir de un mockup, una pantalla existente o una descripción textual. |
| [`mcp-cves`](mcp-cves/) | Escáner de vulnerabilidades CVE: descubrimiento, escaneo, enriquecimiento (NVD, KEV, EPSS), priorización y exportación CSV/PDF. |

## Uso

Cada carpeta (salvo `mcp-cves/`) es una skill autónoma con un `SKILL.md` que incluye el frontmatter de nombre y descripción para su descubrimiento automático por el agente. Copia la carpeta de la skill al directorio de skills de tu agente (p. ej. `.zcode/skills/` o `.claude/skills/`).
