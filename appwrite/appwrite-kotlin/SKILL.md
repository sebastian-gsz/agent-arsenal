---
name: appwrite-kotlin
description: Skill del SDK de Appwrite para Kotlin. Úsala al construir apps Android nativas o backends Kotlin/JVM server-side con Appwrite. Cubre auth de cliente (email, OAuth con integración de Activity), consultas de base de datos, uploads de archivos, suscripciones real-time con soporte de corutinas, y administración server-side vía API keys para gestión de usuarios, administración de bases de datos, storage y functions.
---


# Appwrite Kotlin SDK

## Instalación

```kotlin
// build.gradle.kts — Android
implementation("io.appwrite:sdk-for-android:+")

// build.gradle.kts — Server (Kotlin JVM)
implementation("io.appwrite:sdk-for-kotlin:+")
```

## Configuración del cliente

### Client-side (Android)

```kotlin
import io.appwrite.Client
import io.appwrite.ID
import io.appwrite.Query
import io.appwrite.enums.OAuthProvider
import io.appwrite.services.Account
import io.appwrite.services.Realtime
import io.appwrite.services.TablesDB
import io.appwrite.services.Storage
import io.appwrite.models.InputFile

val client = Client(context)
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject("[PROJECT_ID]")
```

### Server-side (Kotlin JVM)

```kotlin
import io.appwrite.Client
import io.appwrite.ID
import io.appwrite.Query
import io.appwrite.services.Users
import io.appwrite.services.TablesDB
import io.appwrite.services.Storage
import io.appwrite.services.Functions

val client = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject(System.getenv("APPWRITE_PROJECT_ID"))
    .setKey(System.getenv("APPWRITE_API_KEY"))
```

## Ejemplos de código

### Autenticación (client-side)

```kotlin
val account = Account(client)

// Signup
account.create(
    userId = ID.unique(),
    email = "user@example.com",
    password = "password123",
    name = "User Name"
)

// Login
val session = account.createEmailPasswordSession(
    email = "user@example.com",
    password = "password123"
)

// OAuth
account.createOAuth2Session(activity = activity, provider = OAuthProvider.GOOGLE)

// Get current user
val user = account.get()

// Logout
account.deleteSession(sessionId = "current")
```

### Gestión de usuarios (server-side)

```kotlin
val users = Users(client)

// Create user
val user = users.create(
    userId = ID.unique(),
    email = "user@example.com",
    password = "password123",
    name = "User Name"
)

// List users
val list = users.list()

// Get user
val fetched = users.get(userId = "[USER_ID]")

// Delete user
users.delete(userId = "[USER_ID]")
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere argumentos con nombre (p. ej., `databaseId = "..."`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```kotlin
val tablesDB = TablesDB(client)

// Create database (server-side only)
val db = tablesDB.create(databaseId = ID.unique(), name = "My Database")

// Create row
val doc = tablesDB.createRow(
    databaseId = "[DATABASE_ID]",
    tableId = "[TABLE_ID]",
    rowId = ID.unique(),
    data = mapOf("title" to "Hello", "done" to false)
)

// Query rows
val results = tablesDB.listRows(
    databaseId = "[DATABASE_ID]",
    tableId = "[TABLE_ID]",
    queries = listOf(Query.equal("done", false), Query.limit(10))
)

// Get row
val row = tablesDB.getRow(databaseId = "[DATABASE_ID]", tableId = "[TABLE_ID]", rowId = "[ROW_ID]")

// Update row
tablesDB.updateRow(
    databaseId = "[DATABASE_ID]",
    tableId = "[TABLE_ID]",
    rowId = "[ROW_ID]",
    data = mapOf("done" to true)
)

// Delete row
tablesDB.deleteRow(
    databaseId = "[DATABASE_ID]",
    tableId = "[TABLE_ID]",
    rowId = "[ROW_ID]"
)
```

#### Tipos de columnas string

> **Nota:** El tipo `string` heredado está deprecado. Usa tipos de columna explícitos en todas las columnas nuevas.

| Tipo | Caracteres máx. | Indexación | Almacenamiento |
|------|---------------|----------|---------|
| `varchar` | 16,383 | Índice completo (si size ≤ 768) | Inline en la fila |
| `text` | 16,383 | Solo prefijo | Fuera de página |
| `mediumtext` | 4,194,303 | Solo prefijo | Fuera de página |
| `longtext` | 1,073,741,823 | Solo prefijo | Fuera de página |

- `varchar` se almacena inline y cuenta dentro del límite de 64 KB por fila. Prefiérelo para campos cortos e indexados como nombres, slugs o identificadores.
- `text`, `mediumtext` y `longtext` se almacenan fuera de página (solo un puntero de 20 bytes vive en la fila), así que no consumen el presupuesto de tamaño de fila. `size` no es necesario para estos tipos.

```kotlin
// Create table with explicit string column types
tablesDB.createTable(
    databaseId = "[DATABASE_ID]",
    tableId = ID.unique(),
    name = "articles",
    columns = listOf(
        mapOf("key" to "title",    "type" to "varchar",    "size" to 255, "required" to true),
        mapOf("key" to "summary",  "type" to "text",                      "required" to false),
        mapOf("key" to "body",     "type" to "mediumtext",                "required" to false),
        mapOf("key" to "raw_data", "type" to "longtext",                  "required" to false),
    )
)
```

### Métodos de consulta

```kotlin
// Filtering
Query.equal("field", "value")             // == (or pass list for IN)
Query.notEqual("field", "value")          // !=
Query.lessThan("field", 100)              // <
Query.lessThanEqual("field", 100)         // <=
Query.greaterThan("field", 100)           // >
Query.greaterThanEqual("field", 100)      // >=
Query.between("field", 1, 100)            // 1 <= field <= 100
Query.isNull("field")                     // is null
Query.isNotNull("field")                  // is not null
Query.startsWith("field", "prefix")       // starts with
Query.endsWith("field", "suffix")         // ends with
Query.contains("field", "sub")            // contains
Query.search("field", "keywords")         // full-text search (requires index)

// Sorting
Query.orderAsc("field")
Query.orderDesc("field")

// Pagination
Query.limit(25)                           // max rows (default 25, max 100)
Query.offset(0)                           // skip N rows
Query.cursorAfter("[ROW_ID]")             // cursor pagination (preferred)
Query.cursorBefore("[ROW_ID]")

// Selection & Logic
Query.select(listOf("field1", "field2"))
Query.or(listOf(Query.equal("a", 1), Query.equal("b", 2)))   // OR
Query.and(listOf(Query.greaterThan("age", 18), Query.lessThan("age", 65)))  // AND (default)
Query.relationship("comments", listOf(Query.equal("approved", true), Query.limit(25)))  // related documents

// Aggregations (optional alias)
Query.count("*", "total")
Query.sum("amount", "revenue")
Query.avg("score", "avgScore")
Query.min("price", "minPrice")
Query.max("price", "maxPrice")

// Grouping
Query.groupBy(listOf("status", "year"))
Query.having(listOf(Query.greaterThan("total", 1)))
Query.distinct()

// Joins (table id, left key, right key, operator, alias)
Query.join("orders", "\$id", "customerId")
Query.leftJoin("orders", "\$id", "customerId", "=", "ord")
Query.leftJoin("orders", "ord", listOf(
    Query.on("\$id", "customerId"),
    Query.equal("ord.status", "paid"),
))

// Fluent builder. Pass build() into list methods.
val queries = Query.builder()
    .equal("status", "published")
    .leftJoin("orders", "\$id", "customerId", "=", "ord")
    .count("*", "total")
    .groupBy(listOf("status"))
    .limit(25)
    .build()

tablesDB.listRows(
    databaseId = "[DATABASE_ID]",
    tableId = "[TABLE_ID]",
    queries = queries,
)
```

### Storage de archivos

```kotlin
val storage = Storage(client)

// Upload file
val file = storage.createFile(
    bucketId = "[BUCKET_ID]",
    fileId = ID.unique(),
    file = InputFile.fromPath("/path/to/file.png")
)

// Get file preview
val preview = storage.getFilePreview(
    bucketId = "[BUCKET_ID]",
    fileId = "[FILE_ID]",
    width = 300,
    height = 300
)

// List files
val files = storage.listFiles(bucketId = "[BUCKET_ID]")

// Delete file
storage.deleteFile(bucketId = "[BUCKET_ID]", fileId = "[FILE_ID]")
```

#### Métodos de fábrica de InputFile

```kotlin
import io.appwrite.models.InputFile

InputFile.fromPath("/path/to/file.png")              // from filesystem path
InputFile.fromBytes(byteArray, "file.png")           // from ByteArray
```

### Teams

```kotlin
val teams = Teams(client)

// Create team
val team = teams.create(teamId = ID.unique(), name = "Engineering")

// List teams
val list = teams.list()

// Create membership (invite user by email)
val membership = teams.createMembership(
    teamId = "[TEAM_ID]",
    roles = listOf("editor"),
    email = "user@example.com"
)

// List memberships
val members = teams.listMemberships(teamId = "[TEAM_ID]")

// Update membership roles
teams.updateMembership(teamId = "[TEAM_ID]", membershipId = "[MEMBERSHIP_ID]", roles = listOf("admin"))

// Delete team
teams.delete(teamId = "[TEAM_ID]")
```

> **Acceso por roles:** Usa `Role.team("[TEAM_ID]")` para todos los miembros del team o `Role.team("[TEAM_ID]", "editor")` para un rol específico del team al asignar permisos.

### Suscripciones real-time (client-side)

```kotlin
import io.appwrite.Channel

val realtime = Realtime(client)

// Subscribe to row changes
val subscription = realtime.subscribe(
    Channel.tablesdb("[DATABASE_ID]").table("[TABLE_ID]").row()
) { response ->
    println(response.events)   // e.g. ["tablesdb.*.tables.*.rows.*.create"]
    println(response.payload)  // the affected resource
}

// Subscribe to multiple channels
val multi = realtime.subscribe(
    Channel.tablesdb("[DATABASE_ID]").table("[TABLE_ID]").row(),
    Channel.bucket("[BUCKET_ID]").file()
) { response -> /* ... */ }

// Cleanup
subscription.close()
```

**Canales disponibles:**

| Canal | Descripción |
|---------|-------------|
| `account` | Cambios en la cuenta del usuario autenticado |
| `tablesdb.[DB_ID].tables.[TABLE_ID].rows` | Todas las filas de una tabla |
| `tablesdb.[DB_ID].tables.[TABLE_ID].rows.[ROW_ID]` | Una fila específica |
| `buckets.[BUCKET_ID].files` | Todos los archivos de un bucket |
| `buckets.[BUCKET_ID].files.[FILE_ID]` | Un archivo específico |
| `teams` | Cambios en los teams a los que pertenece el usuario |
| `teams.[TEAM_ID]` | Un team específico |
| `memberships` | Las membresías de team del usuario |
| `functions.[FUNCTION_ID].executions` | Actualizaciones de ejecución de functions |

Campos de la respuesta: `events` (array), `payload` (recurso), `channels` (coinciden), `timestamp` (ISO 8601).

### Functions serverless (server-side)

```kotlin
val functions = Functions(client)

// Execute function
val execution = functions.createExecution(
    functionId = "[FUNCTION_ID]",
    body = """{"key": "value"}"""
)

// List executions
val executions = functions.listExecutions(functionId = "[FUNCTION_ID]")
```

#### Escribir un handler de function (runtime Kotlin)

```kotlin
// src/Main.kt — Appwrite Function entry point
import io.openruntimes.kotlin.RuntimeContext
import io.openruntimes.kotlin.RuntimeOutput

fun main(context: RuntimeContext): RuntimeOutput {
    // context.req.body        — raw body (String)
    // context.req.bodyJson    — parsed JSON (Map)
    // context.req.headers     — headers (Map)
    // context.req.method      — HTTP method
    // context.req.path        — URL path
    // context.req.query       — query params (Map)

    context.log("Processing: ${context.req.method} ${context.req.path}")

    if (context.req.method == "GET") {
        return context.res.json(mapOf("message" to "Hello from Appwrite Function!"))
    }

    return context.res.json(mapOf("success" to true))    // JSON
    // context.res.text("Hello")                          // plain text
    // context.res.empty()                                // 204
    // context.res.redirect("https://...")                 // 302
}
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR que usan frameworks de servidor Kotlin (Ktor, Spring Boot, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```kotlin
import io.appwrite.Client
import io.appwrite.services.Account
import io.appwrite.enums.OAuthProvider

// Admin client (reusable)
val adminClient = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject("[PROJECT_ID]")
    .setKey(System.getenv("APPWRITE_API_KEY"))

// Session client (create per-request)
val sessionClient = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject("[PROJECT_ID]")

val session = call.request.cookies["a_session_[PROJECT_ID]"]
if (session != null) {
    sessionClient.setSession(session)
}
```

#### Inicio de sesión con email/contraseña (Ktor)

```kotlin
post("/login") {
    val body = call.receive<LoginRequest>()
    val account = Account(adminClient)
    val session = account.createEmailPasswordSession(
        email = body.email,
        password = body.password,
    )

    // Cookie name must be a_session_<PROJECT_ID>
    call.response.cookies.append(Cookie(
        name = "a_session_[PROJECT_ID]",
        value = session.secret,
        httpOnly = true,
        secure = true,
        extensions = mapOf("SameSite" to "Strict"),
        path = "/",
    ))
    call.respond(mapOf("success" to true))
}
```

#### Peticiones autenticadas

```kotlin
get("/user") {
    val session = call.request.cookies["a_session_[PROJECT_ID]"]
        ?: return@get call.respond(HttpStatusCode.Unauthorized)

    val sessionClient = Client()
        .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
        .setProject("[PROJECT_ID]")
        .setSession(session)

    val account = Account(sessionClient)
    val user = account.get()
    call.respond(user)
}
```

#### Flujo OAuth2 para SSR

```kotlin
// Step 1: Redirect to OAuth provider
get("/oauth") {
    val account = Account(adminClient)
    val redirectUrl = account.createOAuth2Token(
        provider = OAuthProvider.GITHUB,
        success = "https://example.com/oauth/success",
        failure = "https://example.com/oauth/failure",
    )
    call.respondRedirect(redirectUrl)
}

// Step 2: Handle callback — exchange token for session
get("/oauth/success") {
    val account = Account(adminClient)
    val session = account.createSession(
        userId = call.parameters["userId"]!!,
        secret = call.parameters["secret"]!!,
    )

    call.response.cookies.append(Cookie(
        name = "a_session_[PROJECT_ID]", value = session.secret,
        httpOnly = true, secure = true,
        extensions = mapOf("SameSite" to "Strict"), path = "/",
    ))
    call.respond(mapOf("success" to true))
}
```

> **Seguridad de cookies:** Usa siempre `httpOnly`, `secure` y `SameSite=Strict` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `sessionClient.setForwardedUserAgent(call.request.headers["User-Agent"])` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```kotlin
import io.appwrite.AppwriteException

try {
    val row = tablesDB.getRow(databaseId = "[DATABASE_ID]", tableId = "[TABLE_ID]", rowId = "[ROW_ID]")
} catch (e: AppwriteException) {
    println(e.message)     // human-readable message
    println(e.code)        // HTTP status code (Int)
    println(e.type)        // error type (e.g. "document_not_found")
    println(e.response)    // full response body (Map)
}
```

**Códigos de error comunes:**

| Código | Significado |
|------|---------|
| `401` | No autorizado — falta la sesión/API key o no es válida |
| `403` | Prohibido — permisos insuficientes |
| `404` | No encontrado — el recurso no existe |
| `409` | Conflicto — ID duplicado o restricción unique |
| `429` | Limitado por tasa — demasiadas solicitudes |

## Permisos y roles (crítico)

Appwrite usa strings de permisos para controlar el acceso a los recursos. Cada permiso empareja una acción (`read`, `update`, `delete`, `create` o `write`, que otorga create + update + delete) con un rol destino. Por defecto, **ningún usuario tiene acceso** salvo que los permisos se establezcan explícitamente a nivel de fila/archivo o se hereden de la configuración de la tabla/bucket. Los permisos son arrays de strings construidos con los helpers `Permission` y `Role`.

```kotlin
import io.appwrite.Permission
import io.appwrite.Role
```

### Fila de base de datos con permisos

```kotlin
val doc = tablesDB.createRow(
    databaseId = "[DATABASE_ID]",
    tableId = "[TABLE_ID]",
    rowId = ID.unique(),
    data = mapOf("title" to "Hello World"),
    permissions = listOf(
        Permission.read(Role.user("[USER_ID]")),     // specific user can read
        Permission.update(Role.user("[USER_ID]")),   // specific user can update
        Permission.read(Role.team("[TEAM_ID]")),     // all team members can read
        Permission.read(Role.any()),                 // anyone (including guests) can read
    )
)
```

### Upload de archivos con permisos

```kotlin
val file = storage.createFile(
    bucketId = "[BUCKET_ID]",
    fileId = ID.unique(),
    file = InputFile.fromPath("/path/to/file.png"),
    permissions = listOf(
        Permission.read(Role.any()),
        Permission.update(Role.user("[USER_ID]")),
        Permission.delete(Role.user("[USER_ID]")),
    )
)
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.read(Role.any())` sobre datos sensibles** — hace el recurso legible públicamente

