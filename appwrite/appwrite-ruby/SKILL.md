---
name: appwrite-ruby
description: Skill del SDK de Appwrite para Ruby. Úsala al construir aplicaciones Ruby server-side con Appwrite, incluidas las integraciones con Rails y Sinatra. Cubre gestión de usuarios, CRUD de bases de datos/tablas, storage de archivos y functions vía API keys.
---


# Appwrite Ruby SDK

## Instalación

```bash
gem install appwrite
```

## Configuración del cliente

```ruby
require 'appwrite'

include Appwrite

client = Client.new
    .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
    .set_project(ENV['APPWRITE_PROJECT_ID'])
    .set_key(ENV['APPWRITE_API_KEY'])
```

## Ejemplos de código

### Gestión de usuarios

```ruby
users = Users.new(client)

# Create user
user = users.create(user_id: ID.unique, email: 'user@example.com', password: 'password123', name: 'User Name')

# List users
list = users.list(queries: [Query.limit(25)])

# Get user
fetched = users.get(user_id: '[USER_ID]')

# Delete user
users.delete(user_id: '[USER_ID]')
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere argumentos por palabra clave (p. ej., `database_id: '...'`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```ruby
tables_db = TablesDB.new(client)

# Create database
db = tables_db.create(database_id: ID.unique, name: 'My Database')

# Create row
doc = tables_db.create_row(
    database_id: '[DATABASE_ID]',
    table_id: '[TABLE_ID]',
    row_id: ID.unique,
    data: { title: 'Hello World' }
)

# Query rows
results = tables_db.list_rows(
    database_id: '[DATABASE_ID]',
    table_id: '[TABLE_ID]',
    queries: [Query.equal('title', 'Hello World'), Query.limit(10)]
)

# Get row
row = tables_db.get_row(database_id: '[DATABASE_ID]', table_id: '[TABLE_ID]', row_id: '[ROW_ID]')

# Update row
tables_db.update_row(
    database_id: '[DATABASE_ID]',
    table_id: '[TABLE_ID]',
    row_id: '[ROW_ID]',
    data: { title: 'Updated' }
)

# Delete row
tables_db.delete_row(database_id: '[DATABASE_ID]', table_id: '[TABLE_ID]', row_id: '[ROW_ID]')
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

```ruby
# Create table with explicit string column types
tables_db.create_table(
    database_id: '[DATABASE_ID]',
    table_id: ID.unique,
    name: 'articles',
    columns: [
        { key: 'title',    type: 'varchar',    size: 255, required: true  },  # inline, fully indexable
        { key: 'summary',  type: 'text',                  required: false },  # off-page, prefix index only
        { key: 'body',     type: 'mediumtext',            required: false },  # up to ~4 M chars
        { key: 'raw_data', type: 'longtext',              required: false },  # up to ~1 B chars
    ]
)
```

### Métodos de consulta

```ruby
# Filtering
Query.equal('field', 'value')             # == (or pass array for IN)
Query.not_equal('field', 'value')         # !=
Query.less_than('field', 100)             # <
Query.less_than_equal('field', 100)       # <=
Query.greater_than('field', 100)          # >
Query.greater_than_equal('field', 100)    # >=
Query.between('field', 1, 100)            # 1 <= field <= 100
Query.is_null('field')                    # is null
Query.is_not_null('field')                # is not null
Query.starts_with('field', 'prefix')      # starts with
Query.ends_with('field', 'suffix')        # ends with
Query.contains('field', 'sub')            # contains
Query.search('field', 'keywords')         # full-text search (requires index)

# Sorting
Query.order_asc('field')
Query.order_desc('field')

# Pagination
Query.limit(25)                           # max rows (default 25, max 100)
Query.offset(0)                           # skip N rows
Query.cursor_after('[ROW_ID]')            # cursor pagination (preferred)
Query.cursor_before('[ROW_ID]')

# Selection & Logic
Query.select(['field1', 'field2'])        # return only specified fields
Query.or([Query.equal('a', 1), Query.equal('b', 2)])   # OR
Query.and([Query.greater_than('age', 18), Query.less_than('age', 65)])  # AND (default)
Query.relationship('comments', [Query.equal('approved', true), Query.limit(25)])  # related documents

# Aggregations (optional alias)
Query.count('*', 'total')
Query.sum('amount', 'revenue')
Query.avg('score', 'avg_score')
Query.min('price', 'min_price')
Query.max('price', 'max_price')

# Grouping
Query.group_by(['status', 'year'])
Query.having([Query.greater_than('total', 1)])
Query.distinct

# Joins (table id, left key, right key, operator, alias)
Query.join('orders', '$id', 'customerId')
Query.left_join('orders', '$id', 'customerId', '=', 'ord')
Query.left_join('orders', 'ord', [
    Query.on('$id', 'customerId'),
    Query.equal('ord.status', 'paid'),
])

# Fluent builder. Pass build() into list methods.
queries = Query.builder
    .equal('status', 'published')
    .left_join('orders', '$id', 'customerId', '=', 'ord')
    .count('*', 'total')
    .group_by(['status'])
    .limit(25)
    .build

tables_db.list_rows(database_id: '[DATABASE_ID]', table_id: '[TABLE_ID]', queries: queries)
```

### Storage de archivos

```ruby
storage = Storage.new(client)

# Upload file
file = storage.create_file(bucket_id: '[BUCKET_ID]', file_id: ID.unique, file: InputFile.from_path('/path/to/file.png'))

# List files
files = storage.list_files(bucket_id: '[BUCKET_ID]')

# Delete file
storage.delete_file(bucket_id: '[BUCKET_ID]', file_id: '[FILE_ID]')
```

#### Métodos de fábrica de InputFile

```ruby
InputFile.from_path('/path/to/file.png')             # from filesystem path
InputFile.from_string('Hello world', 'hello.txt')    # from string content
```

### Teams

```ruby
teams = Teams.new(client)

# Create team
team = teams.create(team_id: ID.unique, name: 'Engineering')

# List teams
list = teams.list

# Create membership (invite user by email)
membership = teams.create_membership(
    team_id: '[TEAM_ID]',
    roles: ['editor'],
    email: 'user@example.com'
)

# List memberships
members = teams.list_memberships(team_id: '[TEAM_ID]')

# Update membership roles
teams.update_membership(team_id: '[TEAM_ID]', membership_id: '[MEMBERSHIP_ID]', roles: ['admin'])

# Delete team
teams.delete(team_id: '[TEAM_ID]')
```

> **Acceso por roles:** Usa `Role.team('[TEAM_ID]')` para todos los miembros del team o `Role.team('[TEAM_ID]', 'editor')` para un rol específico del team al asignar permisos.

### Functions serverless

```ruby
functions = Functions.new(client)

# Execute function
execution = functions.create_execution(function_id: '[FUNCTION_ID]', body: '{"key": "value"}')

# List executions
executions = functions.list_executions(function_id: '[FUNCTION_ID]')
```

#### Escribir un handler de function (runtime Ruby)

```ruby
# src/main.rb — Appwrite Function entry point
def main(context)
    # context.req.body         — raw body (String)
    # context.req.body_json    — parsed JSON (Hash or nil)
    # context.req.headers      — headers (Hash)
    # context.req.method       — HTTP method
    # context.req.path         — URL path
    # context.req.query        — query params (Hash)

    context.log("Processing: #{context.req.method} #{context.req.path}")

    if context.req.method == 'GET'
        return context.res.json({ message: 'Hello from Appwrite Function!' })
    end

    context.res.json({ success: true })          # JSON
    # context.res.text('Hello')                  # plain text
    # context.res.empty                          # 204
    # context.res.redirect('https://...')         # 302
end
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR que usan frameworks de Ruby (Rails, Sinatra, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```ruby
require 'appwrite'
include Appwrite

# Admin client (reusable)
admin_client = Client.new
    .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
    .set_project('[PROJECT_ID]')
    .set_key(ENV['APPWRITE_API_KEY'])

# Session client (create per-request)
session_client = Client.new
    .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
    .set_project('[PROJECT_ID]')

session = cookies['a_session_[PROJECT_ID]']
session_client.set_session(session) if session
```

#### Inicio de sesión con email/contraseña (Sinatra)

```ruby
post '/login' do
    account = Account.new(admin_client)
    session = account.create_email_password_session(
        email: params[:email],
        password: params[:password]
    )

    # Cookie name must be a_session_<PROJECT_ID>
    response.set_cookie('a_session_[PROJECT_ID]', {
        value: session.secret,
        httponly: true,
        secure: true,
        same_site: :strict,
        path: '/',
    })

    content_type :json
    { success: true }.to_json
end
```

#### Peticiones autenticadas

```ruby
get '/user' do
    session = request.cookies['a_session_[PROJECT_ID]']
    halt 401, { error: 'Unauthorized' }.to_json unless session

    session_client = Client.new
        .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
        .set_project('[PROJECT_ID]')
        .set_session(session)

    account = Account.new(session_client)
    user = account.get

    content_type :json
    user.to_json
end
```

#### Flujo OAuth2 para SSR

```ruby
# Step 1: Redirect to OAuth provider
get '/oauth' do
    account = Account.new(admin_client)
    redirect_url = account.create_o_auth2_token(
        provider: OAuthProvider::GITHUB,
        success: 'https://example.com/oauth/success',
        failure: 'https://example.com/oauth/failure'
    )
    redirect redirect_url
end

# Step 2: Handle callback — exchange token for session
get '/oauth/success' do
    account = Account.new(admin_client)
    session = account.create_session(
        user_id: params[:userId],
        secret: params[:secret]
    )

    response.set_cookie('a_session_[PROJECT_ID]', {
        value: session.secret,
        httponly: true, secure: true, same_site: :strict, path: '/',
    })

    content_type :json
    { success: true }.to_json
end
```

> **Seguridad de cookies:** Usa siempre `httponly`, `secure` y `same_site: :strict` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `session_client.set_forwarded_user_agent(request.user_agent)` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```ruby
require 'appwrite'
include Appwrite

begin
    row = tables_db.get_row(database_id: '[DATABASE_ID]', table_id: '[TABLE_ID]', row_id: '[ROW_ID]')
rescue Appwrite::Exception => e
    puts e.message    # human-readable message
    puts e.code       # HTTP status code (Integer)
    puts e.type       # error type (e.g. 'document_not_found')
    puts e.response   # full response body (Hash)
end
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

```ruby
# Permission and Role are included in the main require
require 'appwrite'
include Appwrite
```

### Fila de base de datos con permisos

```ruby
doc = tables_db.create_row(
    database_id: '[DATABASE_ID]',
    table_id: '[TABLE_ID]',
    row_id: ID.unique,
    data: { title: 'Hello World' },
    permissions: [
        Permission.read(Role.user('[USER_ID]')),     # specific user can read
        Permission.update(Role.user('[USER_ID]')),   # specific user can update
        Permission.read(Role.team('[TEAM_ID]')),     # all team members can read
        Permission.read(Role.any),                   # anyone (including guests) can read
    ]
)
```

### Upload de archivos con permisos

```ruby
file = storage.create_file(
    bucket_id: '[BUCKET_ID]',
    file_id: ID.unique,
    file: InputFile.from_path('/path/to/file.png'),
    permissions: [
        Permission.read(Role.any),
        Permission.update(Role.user('[USER_ID]')),
        Permission.delete(Role.user('[USER_ID]')),
    ]
)
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.any` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.read(Role.any)` sobre datos sensibles** — hace el recurso legible públicamente

