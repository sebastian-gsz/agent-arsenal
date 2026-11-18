---
name: appwrite-go
description: Skill del SDK de Appwrite para Go. Úsala al construir aplicaciones Go server-side con Appwrite. Cubre gestión de usuarios, CRUD de bases de datos/tablas, storage de archivos y functions vía API keys. Usa paquetes por servicio y el patrón de opciones funcionales.
---


# Appwrite Go SDK

## Instalación

```bash
go get github.com/appwrite/sdk-for-go
```

## Configuración del cliente

```go
import (
    "os"

    "github.com/appwrite/sdk-for-go/client"
    "github.com/appwrite/sdk-for-go/id"
    "github.com/appwrite/sdk-for-go/users"
    "github.com/appwrite/sdk-for-go/tablesdb"
    "github.com/appwrite/sdk-for-go/storage"
)

clt := client.New(
    client.WithEndpoint("https://<REGION>.cloud.appwrite.io/v1"),
    client.WithProject(os.Getenv("APPWRITE_PROJECT_ID")),
    client.WithKey(os.Getenv("APPWRITE_API_KEY")),
)
```

## Ejemplos de código

### Gestión de usuarios

```go
service := users.New(clt)

// Create user
user, err := service.Create(
    id.Unique(),
    "user@example.com",
    "password123",
    users.WithCreateName("User Name"),
)

// List users
list, err := service.List()

// Get user
fetched, err := service.Get("[USER_ID]")

// Delete user
_, err = service.Delete("[USER_ID]")
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere parámetros de opción funcional explícitos (p. ej., `tablesdb.WithUpdateRowData(...)`) sobre argumentos posicionales simples donde estén disponibles. Usa el estilo solo-posicional si el codebase existente ya lo usa o el usuario lo pide explícitamente.

```go
service := tablesdb.New(clt)

// Create database
db, err := service.Create(id.Unique(), "My Database")

// Create row
doc, err := service.CreateRow(
    "[DATABASE_ID]",
    "[TABLE_ID]",
    id.Unique(),
    map[string]interface{}{"title": "Hello World"},
)

// List rows
results, err := service.ListRows("[DATABASE_ID]", "[TABLE_ID]")

// Get row
row, err := service.GetRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]")

// Update row
_, err = service.UpdateRow(
    "[DATABASE_ID]",
    "[TABLE_ID]",
    "[ROW_ID]",
    tablesdb.WithUpdateRowData(map[string]interface{}{"title": "Updated"}),
)

// Delete row
_, err = service.DeleteRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]")
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

```go
// Create table with explicit string column types
_, err = service.CreateTable(
    "[DATABASE_ID]",
    id.Unique(),
    "articles",
    tablesdb.WithCreateTableColumns([]map[string]interface{}{
        {"key": "title",    "type": "varchar",    "size": 255, "required": true},
        {"key": "summary",  "type": "text",                    "required": false},
        {"key": "body",     "type": "mediumtext",              "required": false},
        {"key": "raw_data", "type": "longtext",                "required": false},
    }),
)
```

### Métodos de consulta

```go
import "github.com/appwrite/sdk-for-go/query"

// Filtering
query.Equal("field", "value")             // == (or pass slice for IN)
query.NotEqual("field", "value")          // !=
query.LessThan("field", 100)             // <
query.LessThanEqual("field", 100)        // <=
query.GreaterThan("field", 100)          // >
query.GreaterThanEqual("field", 100)     // >=
query.Between("field", 1, 100)           // 1 <= field <= 100
query.IsNull("field")                    // is null
query.IsNotNull("field")                 // is not null
query.StartsWith("field", "prefix")      // starts with
query.EndsWith("field", "suffix")        // ends with
query.Contains("field", "sub")           // contains
query.Search("field", "keywords")        // full-text search (requires index)

// Sorting
query.OrderAsc("field")
query.OrderDesc("field")

// Pagination
query.Limit(25)                          // max rows (default 25, max 100)
query.Offset(0)                          // skip N rows
query.CursorAfter("[ROW_ID]")            // cursor pagination (preferred)
query.CursorBefore("[ROW_ID]")

// Selection & Logic
query.Select([]string{"field1", "field2"})
query.Or([]string{query.Equal("a", 1), query.Equal("b", 2)})    // OR
query.And([]string{query.GreaterThan("age", 18), query.LessThan("age", 65)})  // AND (default)
query.Relationship("comments", []string{query.Equal("approved", true), query.Limit(25)})  // related documents

// Aggregations (optional alias)
query.Count("*", "total")
query.Sum("amount", "revenue")
query.Avg("score", "avgScore")
query.Min("price", "minPrice")
query.Max("price", "maxPrice")

// Grouping
query.GroupBy([]interface{}{"status", "year"})
query.Having([]string{query.GreaterThan("total", 1)})
query.Distinct()

// Joins (table id, left key, right key, operator, alias)
query.Join("orders", "$id", "customerId", "=", "")
query.LeftJoin("orders", "$id", "customerId", "=", "ord")
query.LeftJoinOn("orders", "ord", []string{
    query.On("$id", "customerId", ""),
    query.Equal("ord.status", "paid"),
})

// Fluent builder. Pass Build() into list methods.
queries := query.NewBuilder().
    Equal("status", "published").
    LeftJoin("orders", "$id", "customerId", "=", "ord").
    Count("*", "total").
    GroupBy([]interface{}{"status"}).
    Limit(25).
    Build()

_, _ = service.ListRows("[DATABASE_ID]", "[TABLE_ID]", service.WithListRowsQueries(queries))
```

### Storage de archivos

```go
import "github.com/appwrite/sdk-for-go/file"

service := storage.New(clt)

// Upload file
f, err := service.CreateFile(
    "[BUCKET_ID]",
    "[FILE_ID]",
    file.NewInputFile("/path/to/file.png", "file.png"),
)

// List files
files, err := service.ListFiles("[BUCKET_ID]")

// Delete file
_, err = service.DeleteFile("[BUCKET_ID]", "[FILE_ID]")
```

#### Métodos de fábrica de InputFile

```go
import "github.com/appwrite/sdk-for-go/file"

file.NewInputFile("/path/to/file.png", "file.png")          // from filesystem path
file.NewInputFileFromReader(reader, "file.png", size)        // from io.Reader (size required)
file.NewInputFileFromBytes(data, "file.png")                 // from []byte
```

### Teams

```go
import "github.com/appwrite/sdk-for-go/teams"

svc := teams.New(clt)

// Create team
team, err := svc.Create(id.Unique(), "Engineering")

// List teams
list, err := svc.List()

// Create membership (invite user by email)
membership, err := svc.CreateMembership(
    "[TEAM_ID]",
    []string{"editor"},
    teams.WithCreateMembershipEmail("user@example.com"),
)

// List memberships
members, err := svc.ListMemberships("[TEAM_ID]")

// Update membership roles
_, err = svc.UpdateMembership("[TEAM_ID]", "[MEMBERSHIP_ID]", []string{"admin"})

// Delete team
_, err = svc.Delete("[TEAM_ID]")
```

> **Acceso por roles:** Usa `role.Team("[TEAM_ID]")` para todos los miembros del team o `role.Team("[TEAM_ID]", "editor")` para un rol específico del team al asignar permisos.

### Functions serverless

```go
import "github.com/appwrite/sdk-for-go/functions"

svc := functions.New(clt)

// Execute function
execution, err := svc.CreateExecution(
    "[FUNCTION_ID]",
    functions.WithCreateExecutionBody(`{"key": "value"}`),
)

// List executions
executions, err := svc.ListExecutions("[FUNCTION_ID]")
```

#### Escribir un handler de function (runtime Go)

```go
// src/main.go — Appwrite Function entry point
package handler

import (
    "github.com/open-runtimes/types-for-go/v4/openruntimes"
)

func Main(context openruntimes.Context) openruntimes.Response {
    // context.Req.Body        — raw body (string)
    // context.Req.BodyJson    — parsed JSON (map[string]interface{})
    // context.Req.Headers     — headers (map[string]string)
    // context.Req.Method      — HTTP method
    // context.Req.Path        — URL path
    // context.Req.Query       — query params (map[string]string)

    context.Log("Processing: " + context.Req.Method + " " + context.Req.Path)

    if context.Req.Method == "GET" {
        return context.Res.Json(map[string]interface{}{"message": "Hello!"})
    }

    return context.Res.Json(map[string]interface{}{"success": true})
    // context.Res.Text("Hello")                  // plain text
    // context.Res.Empty()                         // 204
    // context.Res.Redirect("https://...")          // 302
}
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR que usan frameworks de Go (net/http, Gin, Echo, Chi, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```go
import (
    "github.com/appwrite/sdk-for-go/client"
    "github.com/appwrite/sdk-for-go/account"
)

// Admin client (reusable)
adminClient := client.New(
    client.WithEndpoint("https://<REGION>.cloud.appwrite.io/v1"),
    client.WithProject(os.Getenv("APPWRITE_PROJECT_ID")),
    client.WithKey(os.Getenv("APPWRITE_API_KEY")),
)

// Session client (create per-request)
sessionClient := client.New(
    client.WithEndpoint("https://<REGION>.cloud.appwrite.io/v1"),
    client.WithProject(os.Getenv("APPWRITE_PROJECT_ID")),
)

cookie, err := r.Cookie("a_session_[PROJECT_ID]")
if err == nil {
    sessionClient.SetSession(cookie.Value)
}
```

#### Inicio de sesión con email/contraseña

```go
http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
    svc := account.New(adminClient)
    session, err := svc.CreateEmailPasswordSession(r.FormValue("email"), r.FormValue("password"))
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // Cookie name must be a_session_<PROJECT_ID>
    http.SetCookie(w, &http.Cookie{
        Name:     "a_session_[PROJECT_ID]",
        Value:    session.Secret,
        HttpOnly: true,
        Secure:   true,
        SameSite: http.SameSiteStrictMode,
        Path:     "/",
    })

    w.Header().Set("Content-Type", "application/json")
    w.Write([]byte(`{"success": true}`))
})
```

#### Peticiones autenticadas

```go
http.HandleFunc("/user", func(w http.ResponseWriter, r *http.Request) {
    cookie, err := r.Cookie("a_session_[PROJECT_ID]")
    if err != nil {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }

    sessionClient := client.New(
        client.WithEndpoint("https://<REGION>.cloud.appwrite.io/v1"),
        client.WithProject(os.Getenv("APPWRITE_PROJECT_ID")),
        client.WithSession(cookie.Value),
    )

    svc := account.New(sessionClient)
    user, err := svc.Get()
    // Marshal user to JSON and write response
})
```

#### Flujo OAuth2 para SSR

```go
// Step 1: Redirect to OAuth provider
http.HandleFunc("/oauth", func(w http.ResponseWriter, r *http.Request) {
    svc := account.New(adminClient)
    redirectURL, err := svc.CreateOAuth2Token(
        "github",
        account.WithCreateOAuth2TokenSuccess("https://example.com/oauth/success"),
        account.WithCreateOAuth2TokenFailure("https://example.com/oauth/failure"),
    )
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, redirectURL, http.StatusFound)
})

// Step 2: Handle callback — exchange token for session
http.HandleFunc("/oauth/success", func(w http.ResponseWriter, r *http.Request) {
    svc := account.New(adminClient)
    session, err := svc.CreateSession(r.URL.Query().Get("userId"), r.URL.Query().Get("secret"))
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    http.SetCookie(w, &http.Cookie{
        Name: "a_session_[PROJECT_ID]", Value: session.Secret,
        HttpOnly: true, Secure: true, SameSite: http.SameSiteStrictMode, Path: "/",
    })
    w.Write([]byte(`{"success": true}`))
})
```

> **Seguridad de cookies:** Usa siempre `HttpOnly`, `Secure` y `SameSiteStrictMode` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `sessionClient.SetForwardedUserAgent(r.Header.Get("User-Agent"))` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```go
import "github.com/appwrite/sdk-for-go/apperr"

doc, err := service.GetRow("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]")
if err != nil {
    var appErr *apperr.AppwriteException
    if errors.As(err, &appErr) {
        fmt.Println(appErr.Message)    // human-readable message
        fmt.Println(appErr.Code)       // HTTP status code (int)
        fmt.Println(appErr.Type)       // error type (e.g. "document_not_found")
    }
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

Appwrite usa strings de permisos para controlar el acceso a los recursos. Cada permiso empareja una acción (`read`, `update`, `delete`, `create` o `write`, que otorga create + update + delete) con un rol destino. Por defecto, **ningún usuario tiene acceso** salvo que los permisos se establezcan explícitamente a nivel de fila/archivo o se hereden de la configuración de la tabla/bucket. Los permisos son arrays de strings construidos con los helpers `permission` y `role`.

```go
import (
    "github.com/appwrite/sdk-for-go/permission"
    "github.com/appwrite/sdk-for-go/role"
)
```

### Fila de base de datos con permisos

```go
doc, err := service.CreateRow(
    "[DATABASE_ID]",
    "[TABLE_ID]",
    "[ROW_ID]",
    map[string]interface{}{"title": "Hello World"},
    tablesdb.WithCreateRowPermissions([]string{
        permission.Read(role.User("[USER_ID]")),     // specific user can read
        permission.Update(role.User("[USER_ID]")),   // specific user can update
        permission.Read(role.Team("[TEAM_ID]")),     // all team members can read
        permission.Read(role.Any()),                 // anyone (including guests) can read
    }),
)
```

### Upload de archivos con permisos

```go
f, err := service.CreateFile(
    "[BUCKET_ID]",
    "[FILE_ID]",
    file.NewInputFile("/path/to/file.png", "file.png"),
    storage.WithCreateFilePermissions([]string{
        permission.Read(role.Any()),
        permission.Update(role.User("[USER_ID]")),
        permission.Delete(role.User("[USER_ID]")),
    }),
)
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`role.Any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`permission.Read(role.Any())` sobre datos sensibles** — hace el recurso legible públicamente

