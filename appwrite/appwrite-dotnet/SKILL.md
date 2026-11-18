---
name: appwrite-dotnet
description: Skill del SDK de Appwrite para .NET. Úsala al construir aplicaciones C# o .NET server-side con Appwrite, incluidas las integraciones con ASP.NET y Blazor. Cubre gestión de usuarios, CRUD de bases de datos/tablas, storage de archivos y functions vía API keys.
---


# Appwrite .NET SDK

## Instalación

```bash
dotnet add package Appwrite
```

## Configuración del cliente

```csharp
using Appwrite;
using Appwrite.Services;
using Appwrite.Models;

var client = new Client()
    .SetEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .SetProject(Environment.GetEnvironmentVariable("APPWRITE_PROJECT_ID"))
    .SetKey(Environment.GetEnvironmentVariable("APPWRITE_API_KEY"));
```

## Ejemplos de código

### Gestión de usuarios

```csharp
var users = new Users(client);

// Create user
var user = await users.Create(ID.Unique(), "user@example.com", null, "password123", "User Name");

// List users
var list = await users.List(new List<string> { Query.Limit(25) });

// Get user
var fetched = await users.Get("[USER_ID]");

// Delete user
await users.Delete("[USER_ID]");
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere argumentos con nombre (p. ej., `databaseId: "..."`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```csharp
var tablesDB = new TablesDB(client);

// Create database
var db = await tablesDB.Create(ID.Unique(), "My Database");

// Create row
var doc = await tablesDB.CreateRow("[DATABASE_ID]", "[TABLE_ID]", ID.Unique(),
    new Dictionary<string, object> { { "title", "Hello World" } });

// Query rows
var results = await tablesDB.ListRows("[DATABASE_ID]", "[TABLE_ID]",
    new List<string> { Query.Equal("title", "Hello World"), Query.Limit(10) });

// Get row
var row = await tablesDB.GetRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]");

// Update row
await tablesDB.UpdateRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]",
    new Dictionary<string, object> { { "title", "Updated" } });

// Delete row
await tablesDB.DeleteRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]");
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

```csharp
// Create table with explicit string column types
await tablesDB.CreateTable("[DATABASE_ID]", ID.Unique(), "articles",
    new List<object> {
        new { key = "title",    type = "varchar",    size = 255, required = true  },  // inline, fully indexable
        new { key = "summary",  type = "text",                   required = false },  // off-page, prefix index only
        new { key = "body",     type = "mediumtext",             required = false },  // up to ~4 M chars
        new { key = "raw_data", type = "longtext",               required = false },  // up to ~1 B chars
    });
```

### Métodos de consulta

```csharp
// Filtering
Query.Equal("field", "value")             // == (or pass array for IN)
Query.NotEqual("field", "value")          // !=
Query.LessThan("field", 100)             // <
Query.LessThanEqual("field", 100)        // <=
Query.GreaterThan("field", 100)          // >
Query.GreaterThanEqual("field", 100)     // >=
Query.Between("field", 1, 100)           // 1 <= field <= 100
Query.IsNull("field")                    // is null
Query.IsNotNull("field")                 // is not null
Query.StartsWith("field", "prefix")      // starts with
Query.EndsWith("field", "suffix")        // ends with
Query.Contains("field", "sub")           // contains
Query.Search("field", "keywords")        // full-text search (requires index)

// Sorting
Query.OrderAsc("field")
Query.OrderDesc("field")

// Pagination
Query.Limit(25)                          // max rows (default 25, max 100)
Query.Offset(0)                          // skip N rows
Query.CursorAfter("[ROW_ID]")            // cursor pagination (preferred)
Query.CursorBefore("[ROW_ID]")

// Selection & Logic
Query.Select(new List<string> { "field1", "field2" })
Query.Or(new List<string> { Query.Equal("a", 1), Query.Equal("b", 2) })   // OR
Query.And(new List<string> { Query.GreaterThan("age", 18), Query.LessThan("age", 65) })  // AND (default)
Query.Relationship("comments", new List<string> { Query.Equal("approved", true), Query.Limit(25) })  // related documents

// Aggregations (optional alias)
Query.Count("*", "total")
Query.Sum("amount", "revenue")
Query.Avg("score", "avgScore")
Query.Min("price", "minPrice")
Query.Max("price", "maxPrice")

// Grouping
Query.GroupBy(new List<string> { "status", "year" })
Query.Having(new List<string> { Query.GreaterThan("total", 1) })
Query.Distinct()

// Joins (table id, left key, right key, operator, alias)
Query.Join("orders", "$id", "customerId")
Query.LeftJoin("orders", "$id", "customerId", "=", "ord")
Query.LeftJoin("orders", "ord", new List<string> {
    Query.On("$id", "customerId"),
    Query.Equal("ord.status", "paid"),
})

// Fluent builder. Pass Build() into list methods.
var queries = new Query.Builder()
    .Equal("status", "published")
    .LeftJoin("orders", "$id", "customerId", "=", "ord")
    .Count("*", "total")
    .GroupBy(new List<string> { "status" })
    .Limit(25)
    .Build();

await tablesDB.ListRows("[DATABASE_ID]", "[TABLE_ID]", queries);
```

### Storage de archivos

```csharp
var storage = new Storage(client);

// Upload file
var file = await storage.CreateFile("[BUCKET_ID]", ID.Unique(), InputFile.FromPath("/path/to/file.png"));

// List files
var files = await storage.ListFiles("[BUCKET_ID]");

// Delete file
await storage.DeleteFile("[BUCKET_ID]", "[FILE_ID]");
```

#### Métodos de fábrica de InputFile

```csharp
using Appwrite.Models;

InputFile.FromPath("/path/to/file.png")                          // from filesystem path
InputFile.FromBytes(byteArray, "file.png", "image/png")          // from byte[]
InputFile.FromStream(stream, "file.png", "image/png", size)      // from Stream (size required)
```

### Teams

```csharp
var teams = new Teams(client);

// Create team
var team = await teams.Create(ID.Unique(), "Engineering");

// List teams
var list = await teams.List();

// Create membership (invite user by email)
var membership = await teams.CreateMembership(
    teamId: "[TEAM_ID]",
    roles: new List<string> { "editor" },
    email: "user@example.com"
);

// List memberships
var members = await teams.ListMemberships("[TEAM_ID]");

// Update membership roles
await teams.UpdateMembership("[TEAM_ID]", "[MEMBERSHIP_ID]", new List<string> { "admin" });

// Delete team
await teams.Delete("[TEAM_ID]");
```

> **Acceso por roles:** Usa `Role.Team("[TEAM_ID]")` para todos los miembros del team o `Role.Team("[TEAM_ID]", "editor")` para un rol específico del team al asignar permisos.

### Functions serverless

```csharp
var functions = new Functions(client);

// Execute function
var execution = await functions.CreateExecution("[FUNCTION_ID]", body: "{\"key\": \"value\"}");

// List executions
var executions = await functions.ListExecutions("[FUNCTION_ID]");
```

#### Escribir un handler de function (runtime .NET)

```csharp
// src/Main.cs — Appwrite Function entry point
using System.Text.Json;

public async Task<RuntimeOutput> Main(RuntimeContext context)
{
    // context.Req.Body        — raw body (string)
    // context.Req.BodyJson    — parsed JSON (JsonElement)
    // context.Req.Headers     — headers (Dictionary)
    // context.Req.Method      — HTTP method
    // context.Req.Path        — URL path
    // context.Req.Query       — query params (Dictionary)

    context.Log($"Processing: {context.Req.Method} {context.Req.Path}");

    if (context.Req.Method == "GET")
        return context.Res.Json(new { message = "Hello from Appwrite Function!" });

    return context.Res.Json(new { success = true });      // JSON
    // context.Res.Text("Hello");                         // plain text
    // context.Res.Empty();                               // 204
    // context.Res.Redirect("https://...");               // 302
}
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR que usan frameworks .NET (ASP.NET, Blazor Server, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```csharp
using Appwrite;
using Appwrite.Services;

// Admin client (reusable)
var adminClient = new Client()
    .SetEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .SetProject("[PROJECT_ID]")
    .SetKey(Environment.GetEnvironmentVariable("APPWRITE_API_KEY"));

// Session client (create per-request)
var sessionClient = new Client()
    .SetEndpoint("https://<REGION>.cloud.appwrite.io/v1")
    .SetProject("[PROJECT_ID]");

var session = Request.Cookies["a_session_[PROJECT_ID]"];
if (session != null)
{
    sessionClient.SetSession(session);
}
```

#### Inicio de sesión con email/contraseña (ASP.NET Minimal API)

```csharp
app.MapPost("/login", async (HttpContext ctx, LoginRequest body) =>
{
    var account = new Account(adminClient);
    var session = await account.CreateEmailPasswordSession(body.Email, body.Password);

    // Cookie name must be a_session_<PROJECT_ID>
    ctx.Response.Cookies.Append("a_session_[PROJECT_ID]", session.Secret, new CookieOptions
    {
        HttpOnly = true,
        Secure = true,
        SameSite = SameSiteMode.Strict,
        Path = "/",
    });

    return Results.Ok(new { success = true });
});
```

#### Peticiones autenticadas

```csharp
app.MapGet("/user", async (HttpContext ctx) =>
{
    var session = ctx.Request.Cookies["a_session_[PROJECT_ID]"];
    if (session == null) return Results.Unauthorized();

    var sessionClient = new Client()
        .SetEndpoint("https://<REGION>.cloud.appwrite.io/v1")
        .SetProject("[PROJECT_ID]")
        .SetSession(session);

    var account = new Account(sessionClient);
    var user = await account.Get();
    return Results.Ok(user);
});
```

#### Flujo OAuth2 para SSR

```csharp
// Step 1: Redirect to OAuth provider
app.MapGet("/oauth", async () =>
{
    var account = new Account(adminClient);
    var redirectUrl = await account.CreateOAuth2Token(
        provider: OAuthProvider.Github,
        success: "https://example.com/oauth/success",
        failure: "https://example.com/oauth/failure"
    );
    return Results.Redirect(redirectUrl);
});

// Step 2: Handle callback — exchange token for session
app.MapGet("/oauth/success", async (HttpContext ctx, string userId, string secret) =>
{
    var account = new Account(adminClient);
    var session = await account.CreateSession(userId, secret);

    ctx.Response.Cookies.Append("a_session_[PROJECT_ID]", session.Secret, new CookieOptions
    {
        HttpOnly = true, Secure = true, SameSite = SameSiteMode.Strict, Path = "/",
    });

    return Results.Ok(new { success = true });
});
```

> **Seguridad de cookies:** Usa siempre `HttpOnly`, `Secure` y `SameSite = SameSiteMode.Strict` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `sessionClient.SetForwardedUserAgent(ctx.Request.Headers["User-Agent"])` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```csharp
using Appwrite;

try
{
    var row = await tablesDB.GetRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]");
}
catch (AppwriteException e)
{
    Console.WriteLine(e.Message);    // human-readable message
    Console.WriteLine(e.Code);       // HTTP status code (int)
    Console.WriteLine(e.Type);       // error type (e.g. "document_not_found")
    Console.WriteLine(e.Response);   // full response body
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

```csharp
using Appwrite;
// Permission and Role are included in the main namespace
```

### Fila de base de datos con permisos

```csharp
var doc = await tablesDB.CreateRow("[DATABASE_ID]", "[TABLE_ID]", ID.Unique(),
    new Dictionary<string, object> { { "title", "Hello World" } },
    new List<string>
    {
        Permission.Read(Role.User("[USER_ID]")),     // specific user can read
        Permission.Update(Role.User("[USER_ID]")),   // specific user can update
        Permission.Read(Role.Team("[TEAM_ID]")),     // all team members can read
        Permission.Read(Role.Any()),                 // anyone (including guests) can read
    });
```

### Upload de archivos con permisos

```csharp
var file = await storage.CreateFile("[BUCKET_ID]", ID.Unique(),
    InputFile.FromPath("/path/to/file.png"),
    new List<string>
    {
        Permission.Read(Role.Any()),
        Permission.Update(Role.User("[USER_ID]")),
        Permission.Delete(Role.User("[USER_ID]")),
    });
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.Any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.Read(Role.Any())` sobre datos sensibles** — hace el recurso legible públicamente

