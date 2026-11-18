---
name: appwrite-swift
description: Skill del SDK de Appwrite para Swift. Úsala al construir apps nativas de iOS, macOS, watchOS o tvOS, o aplicaciones Swift server-side con Appwrite. Cubre auth de cliente (email, OAuth), consultas de base de datos, uploads de archivos, suscripciones real-time con async/await, y administración server-side vía API keys para gestión de usuarios, administración de bases de datos, storage y functions.
---


# Appwrite Swift SDK

## Instalación

```swift
// Swift Package Manager — Package.swift
.package(url: "https://github.com/appwrite/sdk-for-swift", branch: "main")
```

## Configuración del cliente

### Client-side (plataformas Apple)

```swift
import Appwrite

let client = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject("[PROJECT_ID]")
```

### Server-side (Swift)

```swift
import Appwrite

let client = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject(ProcessInfo.processInfo.environment["APPWRITE_PROJECT_ID"]!)
    .setKey(ProcessInfo.processInfo.environment["APPWRITE_API_KEY"]!)
```

## Ejemplos de código

### Autenticación (client-side)

```swift
let account = Account(client)

// Signup
let user = try await account.create(userId: ID.unique(), email: "user@example.com", password: "password123", name: "User Name")

// Login
let session = try await account.createEmailPasswordSession(email: "user@example.com", password: "password123")

// OAuth
try await account.createOAuth2Session(provider: .google)

// Get current user
let me = try await account.get()

// Logout
try await account.deleteSession(sessionId: "current")
```

### Gestión de usuarios (server-side)

```swift
let users = Users(client)

// Create user
let user = try await users.create(userId: ID.unique(), email: "user@example.com", password: "password123", name: "User Name")

// List users
let list = try await users.list(queries: [Query.limit(25)])

// Get user
let fetched = try await users.get(userId: "[USER_ID]")

// Delete user
try await users.delete(userId: "[USER_ID]")
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere parámetros con nombre (p. ej., `databaseId: "..."`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```swift
let tablesDB = TablesDB(client)

// Create database (server-side only)
let db = try await tablesDB.create(databaseId: ID.unique(), name: "My Database")

// Create row
let doc = try await tablesDB.createRow(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", rowId: ID.unique(), data: [
    "title": "Hello",
    "done": false
])

// Query rows
let results = try await tablesDB.listRows(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", queries: [
    Query.equal("done", value: false),
    Query.limit(10)
])

// Get row
let row = try await tablesDB.getRow(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", rowId: "[ROW_ID]")

// Update row
try await tablesDB.updateRow(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", rowId: "[ROW_ID]", data: ["done": true])

// Delete row
try await tablesDB.deleteRow(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", rowId: "[ROW_ID]")
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

```swift
// Create table with explicit string column types
try await tablesDB.createTable(
    databaseId: "[DATABASE_ID]",
    tableId: ID.unique(),
    name: "articles",
    columns: [
        ["key": "title",    "type": "varchar",    "size": 255, "required": true],
        ["key": "summary",  "type": "text",                    "required": false],
        ["key": "body",     "type": "mediumtext",              "required": false],
        ["key": "raw_data", "type": "longtext",                "required": false],
    ]
)
```

### Métodos de consulta

```swift
// Filtering
Query.equal("field", value: "value")          // == (or pass array for IN)
Query.notEqual("field", value: "value")       // !=
Query.lessThan("field", value: 100)           // <
Query.lessThanEqual("field", value: 100)      // <=
Query.greaterThan("field", value: 100)        // >
Query.greaterThanEqual("field", value: 100)   // >=
Query.between("field", start: 1, end: 100)    // 1 <= field <= 100
Query.isNull("field")                         // is null
Query.isNotNull("field")                      // is not null
Query.startsWith("field", value: "prefix")    // starts with
Query.endsWith("field", value: "suffix")      // ends with
Query.contains("field", value: "sub")         // contains
Query.search("field", value: "keywords")      // full-text search (requires index)

// Sorting
Query.orderAsc("field")
Query.orderDesc("field")

// Pagination
Query.limit(25)                               // max rows (default 25, max 100)
Query.offset(0)                               // skip N rows
Query.cursorAfter("[ROW_ID]")                 // cursor pagination (preferred)
Query.cursorBefore("[ROW_ID]")

// Selection & Logic
Query.select(["field1", "field2"])
Query.or([Query.equal("a", value: 1), Query.equal("b", value: 2)])   // OR
Query.and([Query.greaterThan("age", value: 18), Query.lessThan("age", value: 65)])  // AND (default)
Query.relationship("comments", queries: [Query.equal("approved", value: true), Query.limit(25)])  // related documents

// Aggregations (optional alias)
Query.count("*", alias: "total")
Query.sum("amount", alias: "revenue")
Query.avg("score", alias: "avgScore")
Query.min("price", alias: "minPrice")
Query.max("price", alias: "maxPrice")

// Grouping
Query.groupBy(["status", "year"])
Query.having([Query.greaterThan("total", value: 1)])
Query.distinct()

// Joins (table id, left key, right key, operator, alias)
Query.join("orders", left: "$id", right: "customerId")
Query.leftJoin("orders", left: "$id", right: "customerId", operator: "=", alias: "ord")
Query.leftJoin("orders", alias: "ord", on: [
    Query.on("$id", "customerId"),
    Query.equal("ord.status", value: "paid"),
])

// Fluent builder. Pass build() into list methods.
let queries = Query.builder()
    .equal("status", value: "published")
    .leftJoin("orders", left: "$id", right: "customerId", operator: "=", alias: "ord")
    .count("*", alias: "total")
    .groupBy(["status"])
    .limit(25)
    .build()

_ = try await tablesDB.listRows(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", queries: queries)
```

### Storage de archivos

```swift
let storage = Storage(client)

// Upload file
let file = try await storage.createFile(bucketId: "[BUCKET_ID]", fileId: ID.unique(), file: InputFile.fromPath("/path/to/file.png"))

// List files
let files = try await storage.listFiles(bucketId: "[BUCKET_ID]")

// Delete file
try await storage.deleteFile(bucketId: "[BUCKET_ID]", fileId: "[FILE_ID]")
```

#### Métodos de fábrica de InputFile

```swift
InputFile.fromPath("/path/to/file.png")                    // from filesystem path
InputFile.fromData(data, filename: "file.png", mimeType: "image/png")  // from Data
```

### Teams

```swift
let teams = Teams(client)

// Create team
let team = try await teams.create(teamId: ID.unique(), name: "Engineering")

// List teams
let list = try await teams.list()

// Create membership (invite user by email)
let membership = try await teams.createMembership(
    teamId: "[TEAM_ID]",
    roles: ["editor"],
    email: "user@example.com"
)

// List memberships
let members = try await teams.listMemberships(teamId: "[TEAM_ID]")

// Update membership roles
try await teams.updateMembership(teamId: "[TEAM_ID]", membershipId: "[MEMBERSHIP_ID]", roles: ["admin"])

// Delete team
try await teams.delete(teamId: "[TEAM_ID]")
```

> **Acceso por roles:** Usa `Role.team("[TEAM_ID]")` para todos los miembros del team o `Role.team("[TEAM_ID]", "editor")` para un rol específico del team al asignar permisos.

### Suscripciones real-time (client-side)

```swift
let realtime = Realtime(client)

// Subscribe to row changes
let subscription = try await realtime.subscribe(channels: [
    Channel.tablesdb("[DATABASE_ID]").table("[TABLE_ID]").row()
]) { response in
    print(response.events)   // e.g. ["tablesdb.*.tables.*.rows.*.create"]
    print(response.payload)  // the affected resource
}

// Subscribe to multiple channels
let multi = try await realtime.subscribe(channels: [
    Channel.tablesdb("[DATABASE_ID]").table("[TABLE_ID]").row(),
    Channel.files(),
]) { response in /* ... */ }

// Cleanup
try await subscription.close()
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

```swift
let functions = Functions(client)

// Execute function
let execution = try await functions.createExecution(functionId: "[FUNCTION_ID]", body: "{\"key\": \"value\"}")

// List executions
let executions = try await functions.listExecutions(functionId: "[FUNCTION_ID]")
```

#### Escribir un handler de function (runtime Swift)

```swift
// Sources/main.swift — Appwrite Function entry point
func main(context: RuntimeContext) async throws -> RuntimeOutput {
    // context.req.body        — raw body (String)
    // context.req.bodyJson    — parsed JSON ([String: Any]?)
    // context.req.headers     — headers ([String: String])
    // context.req.method      — HTTP method
    // context.req.path        — URL path
    // context.req.query       — query params ([String: String])

    context.log("Processing: \(context.req.method) \(context.req.path)")

    if context.req.method == "GET" {
        return context.res.json(["message": "Hello from Appwrite Function!"])
    }

    return context.res.json(["success": true])       // JSON
    // context.res.text("Hello")                     // plain text
    // context.res.empty()                           // 204
    // context.res.redirect("https://...")            // 302
}
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR que usan Swift server-side (Vapor, Hummingbird, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```swift
import Appwrite

// Admin client (reusable)
let adminClient = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject("[PROJECT_ID]")
    .setKey(Environment.get("APPWRITE_API_KEY")!)

// Session client (create per-request)
let sessionClient = Client()
    .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .setProject("[PROJECT_ID]")

if let session = req.cookies["a_session_[PROJECT_ID]"]?.string {
    sessionClient.setSession(session)
}
```

#### Inicio de sesión con email/contraseña (Vapor)

```swift
app.post("login") { req async throws -> Response in
    let body = try req.content.decode(LoginRequest.self)
    let account = Account(adminClient)
    let session = try await account.createEmailPasswordSession(
        email: body.email,
        password: body.password
    )

    // Cookie name must be a_session_<PROJECT_ID>
    let response = Response(status: .ok, body: .init(string: "{\"success\": true}"))
    response.cookies["a_session_[PROJECT_ID]"] = HTTPCookies.Value(
        string: session.secret,
        isHTTPOnly: true,
        isSecure: true,
        sameSite: .strict,
        path: "/"
    )
    return response
}
```

#### Peticiones autenticadas

```swift
app.get("user") { req async throws -> Response in
    guard let session = req.cookies["a_session_[PROJECT_ID]"]?.string else {
        throw Abort(.unauthorized)
    }

    let sessionClient = Client()
        .setEndpoint("https://<REGION>.cloud.appwrite.io/v1")
        .setProject("[PROJECT_ID]")
        .setSession(session)

    let account = Account(sessionClient)
    let user = try await account.get()
    // Return user as JSON
}
```

#### Flujo OAuth2 para SSR

```swift
// Step 1: Redirect to OAuth provider
app.get("oauth") { req async throws -> Response in
    let account = Account(adminClient)
    let redirectUrl = try await account.createOAuth2Token(
        provider: .github,
        success: "https://example.com/oauth/success",
        failure: "https://example.com/oauth/failure"
    )
    return req.redirect(to: redirectUrl)
}

// Step 2: Handle callback — exchange token for session
app.get("oauth", "success") { req async throws -> Response in
    let userId = try req.query.get(String.self, at: "userId")
    let secret = try req.query.get(String.self, at: "secret")

    let account = Account(adminClient)
    let session = try await account.createSession(userId: userId, secret: secret)

    let response = Response(status: .ok, body: .init(string: "{\"success\": true}"))
    response.cookies["a_session_[PROJECT_ID]"] = HTTPCookies.Value(
        string: session.secret,
        isHTTPOnly: true, isSecure: true, sameSite: .strict, path: "/"
    )
    return response
}
```

> **Seguridad de cookies:** Usa siempre `isHTTPOnly`, `isSecure` y `sameSite: .strict` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `sessionClient.setForwardedUserAgent(req.headers.first(name: .userAgent) ?? "")` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```swift
import Appwrite
// AppwriteException is included in the main module

do {
    let row = try await tablesDB.getRow(databaseId: "[DATABASE_ID]", tableId: "[TABLE_ID]", rowId: "[ROW_ID]")
} catch let error as AppwriteException {
    print(error.message)     // human-readable message
    print(error.code)        // HTTP status code (Int)
    print(error.type)        // error type (e.g. "document_not_found")
    print(error.response)    // full response body
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

```swift
import Appwrite
// Permission and Role are included in the main module import
```

### Fila de base de datos con permisos

```swift
let doc = try await tablesDB.createRow(
    databaseId: "[DATABASE_ID]",
    tableId: "[TABLE_ID]",
    rowId: ID.unique(),
    data: ["title": "Hello World"],
    permissions: [
        Permission.read(Role.user("[USER_ID]")),     // specific user can read
        Permission.update(Role.user("[USER_ID]")),   // specific user can update
        Permission.read(Role.team("[TEAM_ID]")),     // all team members can read
        Permission.read(Role.any()),                 // anyone (including guests) can read
    ]
)
```

### Upload de archivos con permisos

```swift
let file = try await storage.createFile(
    bucketId: "[BUCKET_ID]",
    fileId: ID.unique(),
    file: InputFile.fromPath("/path/to/file.png"),
    permissions: [
        Permission.read(Role.any()),
        Permission.update(Role.user("[USER_ID]")),
        Permission.delete(Role.user("[USER_ID]")),
    ]
)
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.read(Role.any())` sobre datos sensibles** — hace el recurso legible públicamente

