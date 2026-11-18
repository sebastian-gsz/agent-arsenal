---
name: appwrite-cli
description: Cómo usar bien la CLI de Appwrite. Cubre cuándo usar appwrite.config.json vs `appwrite client`, pull/push vs llamadas de servicio one-shot, generación de SDKs y modelos type-safe, flags de consulta, auth en CI, endpoints regionales de Cloud, variables de functions/sites, y trampas que `--help` no va a mencionar. No cataloga comandos — ejecuta `appwrite <command> --help` para los flags. Úsala siempre que ejecutes `appwrite`, despliegues o hagas pull de functions/sites/tables, generes código de aplicación desde un esquema de tabla, inicialices o vincules un proyecto, hagas scripting de Appwrite, o cuando el usuario mencione la CLI de Appwrite, appwrite.config.json, `appwrite generate`, `appwrite types` o `appwrite client`.
---


# Appwrite CLI

`appwrite <command> --help` ya lista los flags. Léelo en lugar de adivinar.

## Config vs `client`

Un **proyecto real** (repo, esquema, functions, deploys, CI) vive en `appwrite.config.json`. Inicialízalo una vez. Después haz pull y push. No ejecutes `appwrite client --project-id` en cada comando — la config ya nombra el proyecto.

Una **acción puntual** (inspeccionar un usuario, parchear una fila, probar un remoto al que no volverás) usa `appwrite client`. No montes un proyecto para una sola llamada a la API.

```bash
# Good — project
appwrite login
appwrite list-projects --json
appwrite init project --project-id <ID>
appwrite pull table
# edit appwrite.config.json, then:
appwrite push table --force

# Bad — project treated as a pile of API calls
appwrite tablesdb create-table --database-id main --table-id songs ...
appwrite tablesdb create-varchar-column --database-id main --table-id songs ...
# the next clone has no record of this

# Good — one-off (already logged in)
appwrite client --project-id <ID>
appwrite users get --user-id <ID> --json

# Bad — one-off that litters a repo
cd ~/some-unrelated-app && appwrite init project
```

¿Sin sesión? `appwrite client --endpoint https://<REGION>.cloud.appwrite.io/v1 --key "$APPWRITE_API_KEY" --project-id <ID>`.

`appwrite client --project-id` escribe `appwrite.config.json` en este directorio (o actualiza uno que encuentre subiendo por el árbol). Así vinculas una carpeta rápidamente. No lo ejecutes en la raíz de un repo sin relación. `APPWRITE_PROJECT_ID` / `APPWRITE_ENDPOINT` sobreescriben el archivo sin tocarlo — úsalos cuando ya estés dentro de un proyecto y necesites una operación one-shot contra otra cosa.

## La auth no es el proyecto

| Capa | Dónde vive | Se define con |
|---|---|---|
| Quién eres (sesión o API key) | preferencias globales de la CLI | `appwrite login` o `appwrite client --key` |
| Qué proyecto | `appwrite.config.json` | `init project`, `pull` o `client --project-id` |

`init project` habla con la **consola**. Necesita una sesión de login. Una API key no puede listar organizaciones.

```bash
# Good — CI against a checked-in config
appwrite client --key "$APPWRITE_API_KEY"
appwrite push function --force

# Bad — CI that bakes the key into the repo
appwrite client --key 'standard_....'
# and commits appwrite.config.json with secrets inside
```

El entorno gana a la config: `APPWRITE_PROJECT_ID`, `APPWRITE_ENDPOINT`, `APPWRITE_ORGANIZATION_ID`. Haz commit de la config (sin secretos). Pon las keys en el entorno.

`--force` salta los prompts. Los agentes y la CI no tienen TTY, así que un push que pide confirmación falla sin él (`Pass --force instead`). Pregunta al usuario y luego pasa `--force`. No lo uses para saltarte una tabla de cambios cuya columna local está vacía.

`appwrite login --switch` rota las cuentas guardadas. `appwrite client --debug` imprime el endpoint/proyecto efectivo con las credenciales enmascaradas. `appwrite client --reset` cierra la sesión de todo.

## Endpoints de Cloud

Que `appwrite whoami` muestre `https://cloud.appwrite.io/v1` es el host de login de la **cuenta**. Déjalo como está. Las llamadas a la API del proyecto usan el host regional (`https://fra.cloud.appwrite.io/v1`, etc.). `init project` y `client --project-id` fijan esa región en la config. Self-hosted: un solo endpoint para ambos.

```bash
# Good — project work uses the config's regional endpoint
cat appwrite.config.json   # "endpoint": "https://fra.cloud.appwrite.io/v1"

# Bad — "fixing" whoami
appwrite client --endpoint https://fra.cloud.appwrite.io/v1
# now login/session calls go to a region host they do not belong on
```

## Esquema vs filas

| Pull / push (config) | Comandos de servicio (no están en la config) |
|---|---|
| settings, functions, sites, tablas y columnas, buckets, teams, webhooks, topics | filas, usuarios, archivos, executions, messages |

Si debe sobrevivir a un clone fresco, va en la config. Edita el archivo y luego haz push. Crear los mismos recursos con `tablesdb create-*` o `functions create` se salta el archivo que el siguiente `pull` / `push` espera.

`databases` está deprecado. Usa `tablesdb` y `push table`.

Si la tabla de cambios muestra valores remotos frente a una columna local vacía, a la config le faltan esos campos. Haz pull de ese recurso y luego push. No lo pases por la fuerza con `--force`, ni hagas pull de todo por ritual — `pull settings` es lento.

```text
   id            │ key             │ remote │ local
  ───────────────┼─────────────────┼────────┼───────
   Service       │ account         │ true   │
   Auth method   │ email-password  │ true   │
```

```bash
# Good — local column was empty, so sync first, then push
appwrite pull settings
appwrite push settings --force

# Bad — empty local, forced through
appwrite push settings --force
```

`push all --all --force` hace push de todos los recursos. Acótalo (`push function --function-id api`) salvo que quieras exactamente eso.

`unique()` sirve para una fila o usuario one-shot. Los recursos que vayas a hacer push necesitan un `$id` **estable** en la config para que el siguiente pull los empareje.

## Código de aplicación type-safe

Al escribir código de aplicación contra TablesDB, inspecciona `appwrite.config.json` antes de inventar interfaces, IDs de bases de datos, IDs de tablas o nombres de columnas. Los generadores leen la config local; no inspeccionan el esquema remoto. Una config desactualizada produce código desactualizado.

Si el esquema remoto es la fuente de verdad, actualiza primero la config con `appwrite pull table`. No hagas pull sobre ediciones locales de esquema sin hacer push. Si la config local va por delante, genera a partir de ella tal cual.

Para TypeScript, prefiere `appwrite generate`. Emite un wrapper completo de TablesDB: la elección de base de datos y tabla, los payloads de create/update, las filas devueltas, los campos de consulta y los valores de consulta quedan verificados por TypeScript. Esto es más fuerte que interfaces de modelos generadas por sí solas.

```bash
appwrite generate
```

Usa la API generada en lugar de volver a IDs de string crudos y tipos de payload escritos a mano:

```typescript
import { databases, type Songs } from "./generated/appwrite/index.js";

// Database lookup uses its ID; table lookup uses its name from the config.
const songs = databases.use("main").use("Songs");

const page = await songs.list({
  queries: (query) => [
    query.equal("published", true),
    query.orderDesc("createdAt"),
    query.limit(20),
  ],
});

const first: Songs | undefined = page.rows[0];
```

No adivines los literales de ese ejemplo: lee los tipos generados o la config. En particular, `.use()` para una tabla recibe su `name`, que puede diferir de su `$id`. Deja que la compilación exponga una tabla o columna mal escrita o un valor de consulta erróneo. Ejecuta el type checker del proyecto después de generar.

`generate` actualmente provee el wrapper completo para TypeScript. Detecta la salida cliente vs servidor a partir del paquete de Appwrite instalado. La salida de servidor lee `APPWRITE_API_KEY`; nunca pongas esa key en constantes generadas ni la hagas commit. Sobreescribe lenguaje, origen de los imports, extensión de módulo o modo cliente/servidor solo cuando la detección falle — lee `appwrite generate --help` para esos flags.

Para otro lenguaje soportado, o cuando el proyecto use deliberadamente el SDK regular directamente, usa `appwrite types <output-directory>`. Soporta TypeScript (`ts`), JavaScript (`js`), PHP (`php`), Kotlin (`kotlin`), Swift (`swift`), Java (`java`), Dart (`dart`) y C# (`cs`). Pasa estos valores cortos a `--language`; nombres como `typescript` y `csharp` no son valores aceptados por el comando. Los archivos generados tipan los datos de fila, pero **no** verifican tipos de IDs crudos de base de datos/tabla ni de llamadas `Query`:

Para TypeScript, pasa un destino `.ts` cuando el resultado se vaya a importar como módulo. Pasar un directorio en su lugar crea `appwrite.d.ts` dentro de él.

```bash
appwrite types ./src/appwrite-types.ts --language ts
```

```typescript
import { Client, TablesDB } from "appwrite";
import type { Songs } from "./appwrite-types.js";

// Read these values from appwrite.config.json and expose them through the
// application's existing configuration mechanism.
const client = new Client()
  .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
  .setProject("<PROJECT_ID>");
const tablesDB = new TablesDB(client);

const page = await tablesDB.listRows<Songs>({
  databaseId: "main", // still an unchecked string
  tableId: "songs",  // still an unchecked string
});
```

No describas `types --strict` como "type safety total". Solo convierte los nombres de los campos a las convenciones de nomenclatura del lenguaje destino. Usa `generate` cuando el objetivo sea una API de tablas TypeScript completamente tipada.

El código generado es salida derivada. No lo parchees para arreglar un problema de esquema o de nombres; corrige `appwrite.config.json` o las entradas del generador y vuelve a generar. Tras cada cambio de tabla o columna, reejecuta el mismo comando de generación que usa el repositorio y ejecuta su formateador/type checker. Conserva la ruta de salida existente del repositorio y su convención de check-in.

## Functions y sites

Las variables viven en `<path>/.env`, no en `appwrite.config.json`. `--with-variables` **reemplaza** el conjunto remoto a partir de ese archivo. Omítelo salvo que pretendas sincronizar secretos.

```bash
# Good — ship code, leave remote vars alone
appwrite push function --function-id api --activate --force

# Bad — remote vars you did not list locally are gone
appwrite push function --function-id api --with-variables --force

# Good — deploy without switching live traffic
appwrite push function --function-id api --activate=false --force
```

`--async` devuelve antes de que termine el build. En local: `appwrite run function`.

## Consultas y salida

Prefiere `--filter`, `--sort-asc` / `--sort-desc`, `--limit`, `--select`, `--cursor-after`. `--where` está deprecado. `--queries` solo para JSON de Query que los flags no puedan expresar.

```bash
# Good
appwrite users list \
  --filter 'emailVerification=true' \
  --sort-desc '$createdAt' \
  --limit 20 \
  --json

# Bad
appwrite users list --queries '[{"method":"equal","attribute":"emailVerification","values":[true]}]'
```

`--filter` parsea `true` / `false` / `null`, números y arrays JSON. Repite el flag para hacer AND. Las páginas de los listados se capan a 100; prefiere `--cursor-after <lastId>` antes que un `--offset` grande.

`--json` para scripts (los campos vacíos se omiten). `--raw` para el payload sin filtrar. Los secretos quedan redactados salvo con `--show-secrets`.

## El archivo de config en sí

La CLI busca hacia **arriba** desde el cwd hasta encontrar `appwrite.config.json` (o el legacy `appwrite.json`). Hacer push desde `functions/api/` sigue llegando a la raíz del proyecto. Una config perdida en un directorio padre te capturará.

Divide los proyectos grandes con `includes` — cada valor es un array de `.json` relativos. Un recurso no puede estar a la vez inline e incluido. Los valores de `path` de functions y sites se resuelven relativos al archivo include, no a la raíz del repo.

No inventes el archivo de memoria. `init project` o `pull` escribe uno válido. Luego edita.
