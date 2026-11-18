---
name: git-commit-message
description: Reglas para redactar y ejecutar commits seguros y consistentes con Git (Conventional Commits, commits atómicos, ejecución segura del comando). Úsala SIEMPRE que vayas a crear un commit, escribir un mensaje de commit, hacer staging, o cuando el mensaje contenga nombres de archivo con backticks, comillas, paréntesis u otros caracteres especiales — incluso si el usuario solo dice "commitea esto" o "guarda los cambios".
---

# Mensajes de commit de Git

## Contexto antes de commitear

Reúne la imagen completa antes de escribir el mensaje:

```bash
git status --porcelain              # staged, modificados y sin trackear
git diff HEAD                       # todo el cambio (staged + unstaged)
git log -n 20 --pretty=format:%s    # estilo y scopes que ya usa el repo
```

- Si hay archivos ambiguos (sin trackear, generados, dudoso si entran), pregunta qué incluir.
- Nunca commitees secretos ni credenciales (`.env`, `credentials.json`, claves privadas) ni archivos generados.
- Si el repo tiene convención propia (Conventional Commits, gitmoji, plantillas), sígela: gana sobre esta skill.

## Redacción del mensaje

Formato Conventional Commits:

```
<type>(<scope>): <descripción>

[cuerpo]

[footer(s)]
```

**Tipos** — usa el más específico, no el genérico (`perf` antes que `refactor`, `ci` antes que `build` para workflows):

| Tipo       | Cuándo                                         |
| ---------- | ---------------------------------------------- |
| `feat`     | Nueva funcionalidad                            |
| `fix`      | Corrección de bug                              |
| `docs`     | Solo documentación                             |
| `style`    | Formato, sin cambio de lógica                  |
| `refactor` | Reestructura sin feature ni fix                |
| `perf`     | Mejora de rendimiento                          |
| `test`     | Añadir o corregir tests                        |
| `build`    | Sistema de build o dependencias                |
| `ci`       | Configuración de CI                            |
| `chore`    | Mantenimiento que no toca `src` ni `test`      |
| `revert`   | Revertir un commit anterior                    |

**Primera línea (asunto):**
- Imperativo ("add", no "added" ni "adds"), minúscula inicial, sin punto final.
- Máx. 72 caracteres, ideal ~50: las UIs de git, GitHub y GitLab truncan en silencio.
- `scope` opcional: sustantivo corto entre paréntesis (`auth`, `api`, `ui`), nunca un verbo (`feat(adding-auth):` ❌).
- Referencias a archivos en backticks solo si aportan contexto (ej. `` fix: loading state in `ComingSoon.tsx` ``).

**Breaking changes:** `!` tras el tipo/scope o footer `BREAKING CHANGE:` (provoca bump MAJOR en SemVer). Dejarlo solo en el cuerpo es invisible a las herramientas de changelog:

```
feat!: remove deprecated endpoint

BREAKING CHANGE: `extends` ya no acepta rutas relativas
```

**Cuerpo (opcional):** cuenta el *porqué* — el problema que existía antes del cambio — no el *qué* (eso ya lo muestra el diff). Sepáralo del asunto con una línea en blanco y mantén las líneas bajo 100 caracteres. En `revert`, conserva `This reverts commit <hash>`.

**Footer:** referencia y cierra issues con las palabras clave (`Closes #123`, `Fixes #456`, `Refs #789`, multi-repo `Closes owner/repo#42`); se cierran al llegar a la rama por defecto.

**Ejemplo completo:**

```
fix(auth): prevent token refresh loop on expired sessions

When a session expired mid-request, the middleware triggered a token
refresh that itself failed validation and retried — an infinite loop.

Adds a guard flag that aborts the cycle if a refresh is already in
progress, returning a clean 401 instead.

Closes #312
```

**Atribución de IA:** no añadas trailers `Co-Authored-By` de la IA salvo que la convención del repo lo pida explícitamente.

## Commits atómicos

- Un commit = un cambio lógico. No mezcles feature + fix + refactor + formateo en uno.
- Si el diff contiene cambios separados (archivos distintos con propósitos distintos, o fix + refactor en el mismo archivo), propón dividirlos: `git add <archivos>` o `git add -p`.
- Commit temprano y frecuente: cada incremento verificado merece su commit. Tamaño objetivo ~100-300 líneas; por encima de ~1000, divide.

## Ejecución segura del comando

Los mensajes con backticks, comillas o `$` son peligrosos en shell: Bash interpreta los backticks como ejecución de comando. Para que el mensaje llegue literal:

1. **Preferido — heredoc con `git commit -F`:** evita el escapado por completo:

   ```bash
   git commit -F - <<'EOF'
   fix(auth): add Zod validation in `ComingSoon.tsx`

   Shares schemas with the i18n layer via packages/shared.

   Closes #312
   EOF
   ```

   Variante de una sola línea con `-m`: `git commit -m "$(cat <<'EOF' ... EOF)"`.

2. **Alternativa — comillas simples:** si el mensaje no contiene comillas simples, envuélvelo íntegras: `git commit -m 'add validation in `ComingSoon.tsx`'`. Dentro de comillas simples el shell no expande nada.

3. **Nunca:** envolver un mensaje con backticks o `$` dentro de comillas dobles sin escapar — el shell los ejecutaría.

Si el mensaje ya se escribió con caracteres problemáticos y el comando falla, reescríbelo con el método 1 en lugar de apilar escapes.

## Protocolo de seguridad Git

- Solo commit: **no hagas push** salvo que el usuario lo pida.
- Nunca, sin petición explícita del usuario: `git config`, `--force` / force push (jamás a `main`/`master`), `reset --hard`, `clean -f`, `branch -D`, `checkout .` / `restore .`.
- Nunca saltes hooks con `--no-verify`; si un hook falla, arréglalo y crea un commit **nuevo** (no `--amend`).
- No amendes commits que ya estén pusheados.

*(Compañera recomendada: `git-guardrails-claude-code` de mattpocock — instala un hook PreToolUse que bloquea esos comandos peligrosos antes de ejecutarse.)*
