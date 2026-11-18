---
name: appwrite-rust
description: Skill del SDK de Appwrite para Rust. Úsala al construir aplicaciones Rust server-side con Appwrite. Cubre la configuración async del cliente con API keys, gestión de usuarios, operaciones de base de datos/tabla/fila de TablesDB, storage de archivos, ejecuciones de functions, permisos, consultas y manejo de errores. Usa el paquete `appwrite` de crates.io y Tokio.
---


# Appwrite Rust SDK

## Instalación

```bash
cargo add appwrite
cargo add tokio --features full
cargo add serde_json
```

## Configuración del cliente

El SDK de Rust es async. Úsalo desde un runtime de Tokio y autentica server-side con una API key.

```rust
use appwrite::Client;
use std::env;

let client = Client::new()
    .set_endpoint("https://<REGION>.cloud.appwrite.io/v1")
    .set_project(env::var("APPWRITE_PROJECT_ID")?)
    .set_key(env::var("APPWRITE_API_KEY")?);
```

### Esqueleto completo de servidor

```rust
use appwrite::query::Query;
use appwrite::services::Users;
use appwrite::Client;
use std::env;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new()
        .set_endpoint("https://<REGION>.cloud.appwrite.io/v1")
        .set_project(env::var("APPWRITE_PROJECT_ID")?)
        .set_key(env::var("APPWRITE_API_KEY")?);

    let users = Users::new(&client);
    let _list = users
        .list(Some(vec![Query::limit(25).to_string()]), None, Some(true))
        .await?;

    Ok(())
}
```

## Ejemplos de código

### Gestión de usuarios

```rust
use appwrite::id::ID;
use appwrite::query::Query;
use appwrite::services::Users;

let users = Users::new(&client);

// Create user
let _user = users
    .create(
        ID::unique(),
        Some("user@example.com"),
        None,
        Some("password123"),
        Some("User Name"),
    )
    .await?;

// List users
let _list = users
    .list(Some(vec![Query::limit(25).to_string()]), None, Some(true))
    .await?;

// Get user
let _fetched = users.get("[USER_ID]").await?;

// Delete user
users.delete("[USER_ID]").await?;
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` en código nuevo. Usa `Databases` solo si el proyecto existente depende explícitamente de la API de Databases heredada.
>
> **Convención de llamada del SDK de Rust:** Los métodos son async, usan parámetros posicionales y representan los parámetros opcionales de la API como `Option<T>`. Pasa `None` para los valores opcionales que no quieras establecer.

```rust
use appwrite::id::ID;
use appwrite::permission::Permission;
use appwrite::query::Query;
use appwrite::role::Role;
use appwrite::services::TablesDB;
use serde_json::json;

let tables_db = TablesDB::new(&client);

// Create database
let _database = tables_db
    .create(ID::unique(), "My Database", Some(true))
    .await?;

// Create table
let _table = tables_db
    .create_table(
        "[DATABASE_ID]",
        ID::unique(),
        "articles",
        Some(vec![
            Permission::read(Role::any()).to_string(),
            Permission::create(Role::users(None)).to_string(),
        ]),
        Some(true),
        Some(true),
        None,
        None,
    )
    .await?;

// Create row
let _row = tables_db
    .create_row(
        "[DATABASE_ID]",
        "[TABLE_ID]",
        ID::unique(),
        json!({
            "title": "Hello World",
            "done": false
        }),
        None,
        None,
    )
    .await?;

// Query rows
let _rows = tables_db
    .list_rows(
        "[DATABASE_ID]",
        "[TABLE_ID]",
        Some(vec![
            Query::equal("done", false).to_string(),
            Query::limit(10).to_string(),
        ]),
        None,
        Some(true),
        None,
    )
    .await?;

// Get row
let _row = tables_db
    .get_row("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]", None, None)
    .await?;

// Update row
let _updated = tables_db
    .update_row(
        "[DATABASE_ID]",
        "[TABLE_ID]",
        "[ROW_ID]",
        Some(json!({ "done": true })),
        None,
        None,
    )
    .await?;

// Delete row
tables_db
    .delete_row("[DATABASE_ID]", "[TABLE_ID]", "[ROW_ID]", None)
    .await?;
```

#### Tipos de columnas string

> **Nota:** El tipo de columna `string` heredado está deprecado. Usa tipos de columna string explícitos en las tablas nuevas.

| Type | Max characters | Indexing | Storage |
|------|---------------|----------|---------|
| `varchar` | 16,383 | Índice completo (si size <= 768) | Inline en la fila |
| `text` | 16,383 | Solo prefijo | Fuera de página |
| `mediumtext` | 4,194,303 | Solo prefijo | Fuera de página |
| `longtext` | 1,073,741,823 | Solo prefijo | Fuera de página |

- `varchar` se almacena inline y cuenta dentro del límite de 64 KB por fila. Prefiérelo para campos cortos e indexados como nombres, slugs e identificadores.
- `text`, `mediumtext` y `longtext` se almacenan fuera de página, así que no consumen el presupuesto de tamaño de fila. `size` no es necesario para estos tipos.

```rust
use appwrite::enums::{OrderBy, TablesDBIndexType};

// Short, indexed string
let _title = tables_db
    .create_varchar_column(
        "[DATABASE_ID]",
        "[TABLE_ID]",
        "title",
        255,
        true,
        None,
        None,
        None,
    )
    .await?;

// Off-page longer text
let _summary = tables_db
    .create_text_column("[DATABASE_ID]", "[TABLE_ID]", "summary", false, None, None, None)
    .await?;

let _body = tables_db
    .create_mediumtext_column("[DATABASE_ID]", "[TABLE_ID]", "body", false, None, None, None)
    .await?;

let _raw_data = tables_db
    .create_longtext_column("[DATABASE_ID]", "[TABLE_ID]", "raw_data", false, None, None, None)
    .await?;

// Index only the varchar column fully.
let _index = tables_db
    .create_index(
        "[DATABASE_ID]",
        "[TABLE_ID]",
        "title_idx",
        TablesDBIndexType::Key,
        vec!["title"],
        Some(vec![OrderBy::Asc]),
        Some(vec![255]),
    )
    .await?;
```

### Métodos de consulta

Los métodos de `TablesDB` aceptan `Option<Vec<String>>` para las consultas. Convierte cada `Query` a string.

```rust
use appwrite::query::Query;
use serde_json::Value;

let queries = vec![
    Query::equal("status", "published").to_string(),
    Query::not_equal("archived", true).to_string(),
    Query::greater_than("views", 100).to_string(),
    Query::less_than_equal("priority", 5).to_string(),
    Query::between("score", 1, 100).to_string(),
    Query::is_not_null("publishedAt").to_string(),
    Query::search("title", "rust appwrite").to_string(),
    Query::contains("tags", "rust").to_string(),
    Query::order_desc("$createdAt").to_string(),
    Query::limit(25).to_string(),
    Query::offset(50).to_string(),
];

let multi_value = Query::equal(
    "status",
    Value::Array(vec![
        Value::String("draft".to_string()),
        Value::String("published".to_string()),
    ]),
)
.to_string();

let related = Query::relationship(
    "comments",
    vec![
        Query::equal("approved", true).to_string(),
        Query::limit(25).to_string(),
    ],
)
.to_string();

// Aggregations (pass "" for no alias)
let aggregations = vec![
    Query::count("*", "total").to_string(),
    Query::sum("amount", "revenue").to_string(),
    Query::avg("score", "avgScore").to_string(),
    Query::min("price", "minPrice").to_string(),
    Query::max("price", "maxPrice").to_string(),
];

// Grouping
let grouping = vec![
    Query::group_by(["status", "year"]).to_string(),
    Query::having([Query::greater_than("total", 1).to_string()]).to_string(),
    Query::distinct().to_string(),
];

// Joins (table id, left key, right key, operator, alias; "" for no alias)
let joins = vec![
    Query::join("orders", "$id", "customerId", "=", "").to_string(),
    Query::left_join("orders", "$id", "customerId", "=", "ord").to_string(),
    Query::left_join_on(
        "orders",
        "ord",
        [
            Query::on("$id", "customerId", "=").to_string(),
            Query::equal("ord.status", "paid").to_string(),
        ],
    )
    .to_string(),
];

// Fluent builder. Pass build() into list methods.
let queries = Query::builder()
    .equal("status", "published")
    .left_join("orders", "$id", "customerId", "=", "ord")
    .count("*", "total")
    .group_by(["status"])
    .limit(25)
    .build();

let _rows = tables_db
    .list_rows("[DATABASE_ID]", "[TABLE_ID]", Some(queries), None, None, None)
    .await?;
```

### Storage de archivos

```rust
use appwrite::id::ID;
use appwrite::input_file::InputFile;
use appwrite::permission::Permission;
use appwrite::query::Query;
use appwrite::role::Role;
use appwrite::services::Storage;

let storage = Storage::new(&client);

// Create bucket
let _bucket = storage
    .create_bucket(
        ID::unique(),
        "Uploads",
        Some(vec![
            Permission::read(Role::any()).to_string(),
            Permission::create(Role::users(None)).to_string(),
        ]),
        Some(true),
        Some(true),
        Some(30_000_000),
        Some(vec!["jpg".to_string(), "png".to_string(), "pdf".to_string()]),
        None,
        Some(true),
        Some(true),
        Some(true),
    )
    .await?;

// Upload from disk
let input = InputFile::from_path("avatar.png", Some("image/png")).await?;
let _file = storage
    .create_file(
        "[BUCKET_ID]",
        ID::unique(),
        input,
        Some(vec![Permission::read(Role::any()).to_string()]),
    )
    .await?;

// Upload from bytes
let input = InputFile::from_bytes(b"hello".to_vec(), "hello.txt", Some("text/plain"));
let _file = storage.create_file("[BUCKET_ID]", ID::unique(), input, None).await?;

// List files
let _files = storage
    .list_files("[BUCKET_ID]", Some(vec![Query::limit(10).to_string()]), None, Some(true))
    .await?;

// Download file bytes
let _content = storage
    .get_file_download("[BUCKET_ID]", "[FILE_ID]", None)
    .await?;

// Delete file
storage.delete_file("[BUCKET_ID]", "[FILE_ID]").await?;
```

### Functions

```rust
use appwrite::enums::ExecutionMethod;
use appwrite::query::Query;
use appwrite::services::Functions;
use serde_json::json;

let functions = Functions::new(&client);

// List functions
let _functions = functions
    .list(Some(vec![Query::limit(25).to_string()]), None, Some(true))
    .await?;

// Execute function
let _execution = functions
    .create_execution(
        "[FUNCTION_ID]",
        Some(r#"{"hello":"world"}"#),
        Some(false),
        Some("/jobs/sync"),
        Some(ExecutionMethod::POST),
        Some(json!({ "content-type": "application/json" })),
        None,
    )
    .await?;

// Get execution
let _execution = functions
    .get_execution("[FUNCTION_ID]", "[EXECUTION_ID]")
    .await?;
```

### Permisos

```rust
use appwrite::permission::Permission;
use appwrite::role::Role;

let permissions = vec![
    Permission::read(Role::any()).to_string(),
    Permission::create(Role::users(None)).to_string(),
    Permission::update(Role::user("[USER_ID]", None)).to_string(),
    Permission::delete(Role::team("[TEAM_ID]", Some("owner"))).to_string(),
];
```

### Manejo de errores

```rust
match users.get("[USER_ID]").await {
    Ok(user) => {
        let _user = user;
    }
    Err(error) if error.status_code() == 404 => {
        eprintln!("User not found: {}", error.get_message());
    }
    Err(error) => {
        eprintln!("Appwrite error {}: {}", error.status_code(), error.get_message());
        return Err(Box::new(error) as Box<dyn std::error::Error>);
    }
}
```

## Errores frecuentes

- El SDK de Rust es actualmente un SDK server-side. Prefiere los SDKs de TypeScript/Web, Flutter, Apple, Android o React Native para los flujos de auth de cliente en navegador/móvil.
- Haz siempre `await` en las llamadas a servicios.
- Pasa `None` en los parámetros opcionales que no uses.
- Usa `TablesDB`, no la `Databases` heredada, en código nuevo de base de datos.
- Convierte las consultas con `.to_string()` antes de pasarlas a APIs que esperan `Option<Vec<String>>`.
- Usa `serde_json::json!({...})` para los datos de fila y los cuerpos JSON.
- Usa `InputFile::from_path(...).await?` o `InputFile::from_bytes(...)` para los uploads.
