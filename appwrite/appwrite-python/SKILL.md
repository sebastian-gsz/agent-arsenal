---
name: appwrite-python
description: Skill del SDK de Appwrite para Python. Úsala al construir aplicaciones Python server-side con Appwrite, incluidas las integraciones con Django, Flask y FastAPI. Cubre gestión de usuarios, CRUD de bases de datos/tablas, storage de archivos y functions vía API keys.
---


# Appwrite Python SDK

## Instalación

```bash
pip install appwrite
```

## Configuración del cliente

```python
from appwrite.client import Client
from appwrite.id import ID
from appwrite.query import Query
from appwrite.services.users import Users
from appwrite.services.tables_db import TablesDB
from appwrite.services.storage import Storage
from appwrite.services.functions import Functions
from appwrite.enums.o_auth_provider import OAuthProvider

import os

client = (Client()
    .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
    .set_project(os.environ['APPWRITE_PROJECT_ID'])
    .set_key(os.environ['APPWRITE_API_KEY']))
```

## Ejemplos de código

### Gestión de usuarios

```python
users = Users(client)

# Create user
user = users.create(ID.unique(), 'user@example.com', None, 'password123', 'User Name')

# List users
result = users.list([Query.limit(25)])

# Get user
fetched = users.get('[USER_ID]')

# Delete user
users.delete('[USER_ID]')
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere argumentos por palabra clave (p. ej., `database_id='...'`) sobre argumentos posicionales en todas las llamadas a métodos del SDK. Usa el estilo posicional solo si el codebase existente ya lo usa o el usuario lo pide explícitamente.

```python
tables_db = TablesDB(client)

# Create database
db = tables_db.create(ID.unique(), 'My Database')

# Create row
doc = tables_db.create_row('[DATABASE_ID]', '[TABLE_ID]', ID.unique(), {
    'title': 'Hello World'
})

# Query rows
results = tables_db.list_rows('[DATABASE_ID]', '[TABLE_ID]', [
    Query.equal('title', 'Hello World'),
    Query.limit(10)
])

# Get row
row = tables_db.get_row('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]')

# Update row
tables_db.update_row('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]', {
    'title': 'Updated'
})

# Delete row
tables_db.delete_row('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]')
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

```python
# Create table with explicit string column types
tables_db.create_table(
    database_id='[DATABASE_ID]',
    table_id=ID.unique(),
    name='articles',
    columns=[
        {'key': 'title',    'type': 'varchar',    'size': 255, 'required': True},   # inline, fully indexable
        {'key': 'summary',  'type': 'text',                    'required': False},  # off-page, prefix index only
        {'key': 'body',     'type': 'mediumtext',              'required': False},  # up to ~4 M chars
        {'key': 'raw_data', 'type': 'longtext',                'required': False},  # up to ~1 B chars
    ]
)
```

### Métodos de consulta

```python
# Filtering
Query.equal('field', 'value')             # == (or pass list for IN)
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
Query.contains('field', 'sub')            # contains (string or array)
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
Query.or_queries([Query.equal('a', 1), Query.equal('b', 2)])   # OR
Query.and_queries([Query.greater_than('age', 18), Query.less_than('age', 65)])  # AND (default)
Query.relationship('comments', [Query.equal('approved', True), Query.limit(25)])  # related documents

# Aggregations (optional alias)
Query.count('*', 'total')
Query.sum('amount', 'revenue')
Query.avg('score', 'avg_score')
Query.min('price', 'min_price')
Query.max('price', 'max_price')

# Grouping
Query.group_by(['status', 'year'])
Query.having([Query.greater_than('total', 1)])
Query.distinct()

# Joins (table id, left key, right key, operator, alias)
Query.join('orders', '$id', 'customerId')
Query.left_join('orders', '$id', 'customerId', '=', 'ord')
Query.left_join('orders', 'ord', [
    Query.on('$id', 'customerId'),
    Query.equal('ord.status', 'paid'),
])

# Fluent builder. Pass build() into list methods.
queries = (
    Query.builder()
    .equal('status', 'published')
    .left_join('orders', '$id', 'customerId', '=', 'ord')
    .count('*', 'total')
    .group_by(['status'])
    .limit(25)
    .build()
)

tables_db.list_rows('[DATABASE_ID]', '[TABLE_ID]', queries)
```

### Storage de archivos

```python
from appwrite.input_file import InputFile

storage = Storage(client)

# Upload file
file = storage.create_file('[BUCKET_ID]', ID.unique(), InputFile.from_path('/path/to/file.png'))

# List files
files = storage.list_files('[BUCKET_ID]')

# Delete file
storage.delete_file('[BUCKET_ID]', '[FILE_ID]')
```

#### Métodos de fábrica de InputFile

```python
from appwrite.input_file import InputFile

InputFile.from_path('/path/to/file.png')             # from filesystem path
InputFile.from_bytes(byte_data, 'file.png')          # from bytes
InputFile.from_string('Hello world', 'hello.txt')    # from string content
```

### Teams

```python
from appwrite.services.teams import Teams

teams = Teams(client)

# Create team
team = teams.create(ID.unique(), 'Engineering')

# List teams
team_list = teams.list()

# Create membership (invite user by email)
membership = teams.create_membership('[TEAM_ID]', roles=['editor'], email='user@example.com')

# List memberships
members = teams.list_memberships('[TEAM_ID]')

# Update membership roles
teams.update_membership('[TEAM_ID]', '[MEMBERSHIP_ID]', roles=['admin'])

# Delete team
teams.delete('[TEAM_ID]')
```

> **Acceso por roles:** Usa `Role.team('[TEAM_ID]')` para todos los miembros del team o `Role.team('[TEAM_ID]', 'editor')` para un rol específico del team al asignar permisos.

### Functions serverless

```python
functions = Functions(client)

# Execute function
execution = functions.create_execution('[FUNCTION_ID]', body='{"key": "value"}')

# List executions
executions = functions.list_executions('[FUNCTION_ID]')
```

#### Escribir un handler de function (runtime Python)

```python
# src/main.py — Appwrite Function entry point
def main(context):
    # context.req — request object
    #   .body        — raw request body (string)
    #   .body_json   — parsed JSON body (dict, or None if not JSON)
    #   .headers     — request headers (dict)
    #   .method      — HTTP method (GET, POST, etc.)
    #   .path        — URL path
    #   .query       — parsed query parameters (dict)
    #   .query_string — raw query string

    context.log('Processing: ' + context.req.method + ' ' + context.req.path)

    if context.req.method == 'GET':
        return context.res.json({'message': 'Hello from Appwrite Function!'})

    data = context.req.body_json or {}
    if 'name' not in data:
        context.error('Missing name field')
        return context.res.json({'error': 'Name is required'}, 400)

    # Response methods
    return context.res.json({'success': True})                    # JSON response
    # return context.res.text('Hello')                           # plain text
    # return context.res.empty()                                 # 204 No Content
    # return context.res.redirect('https://example.com')         # 302 Redirect
    # return context.res.send('data', 200, {'X-Custom': '1'})   # custom response
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR (Flask, Django, FastAPI, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```python
from appwrite.client import Client
from appwrite.services.account import Account
from flask import request, jsonify, make_response, redirect

# Admin client (reusable)
admin_client = (Client()
    .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
    .set_project('[PROJECT_ID]')
    .set_key(os.environ['APPWRITE_API_KEY']))

# Session client (create per-request)
session_client = (Client()
    .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
    .set_project('[PROJECT_ID]'))

session = request.cookies.get('a_session_[PROJECT_ID]')
if session:
    session_client.set_session(session)
```

#### Inicio de sesión con email/contraseña

```python
@app.post('/login')
def login():
    account = Account(admin_client)
    session = account.create_email_password_session(
        request.json['email'], request.json['password']
    )

    # Cookie name must be a_session_<PROJECT_ID>
    resp = make_response(jsonify({'success': True}))
    resp.set_cookie('a_session_[PROJECT_ID]', session['secret'],
                    httponly=True, secure=True, samesite='Strict',
                    expires=session['expire'], path='/')
    return resp
```

#### Peticiones autenticadas

```python
@app.get('/user')
def get_user():
    session = request.cookies.get('a_session_[PROJECT_ID]')
    if not session:
        return jsonify({'error': 'Unauthorized'}), 401

    session_client = (Client()
        .set_endpoint('https://<REGION>.cloud.appwrite.io/v1')
        .set_project('[PROJECT_ID]')
        .set_session(session))

    account = Account(session_client)
    return jsonify(account.get())
```

#### Flujo OAuth2 para SSR

```python
# Step 1: Redirect to OAuth provider
@app.get('/oauth')
def oauth():
    account = Account(admin_client)
    redirect_url = account.create_o_auth2_token(
        OAuthProvider.Github,
        'https://example.com/oauth/success',
        'https://example.com/oauth/failure',
    )
    return redirect(redirect_url)

# Step 2: Handle callback — exchange token for session
@app.get('/oauth/success')
def oauth_success():
    account = Account(admin_client)
    session = account.create_session(request.args['userId'], request.args['secret'])

    resp = make_response(jsonify({'success': True}))
    resp.set_cookie('a_session_[PROJECT_ID]', session['secret'],
                    httponly=True, secure=True, samesite='Strict',
                    expires=session['expire'], path='/')
    return resp
```

> **Seguridad de cookies:** Usa siempre `httponly`, `secure` y `samesite='Strict'` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `session_client.set_forwarded_user_agent(request.headers.get('user-agent'))` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```python
from appwrite.exception import AppwriteException

try:
    row = tables_db.get_row('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]')
except AppwriteException as e:
    print(e.message)    # human-readable error message
    print(e.code)       # HTTP status code (int)
    print(e.type)       # Appwrite error type string (e.g. 'document_not_found')
    print(e.response)   # full response body (dict)
```

**Códigos de error comunes:**

| Código | Significado |
|------|---------|
| `401` | No autorizado — falta la sesión/API key o no es válida |
| `403` | Prohibido — permisos insuficientes para esta acción |
| `404` | No encontrado — el recurso no existe |
| `409` | Conflicto — ID duplicado o violación de restricción unique |
| `429` | Limitado por tasa — demasiadas solicitudes, reintenta tras un backoff |

## Permisos y roles (crítico)

Appwrite usa strings de permisos para controlar el acceso a los recursos. Cada permiso empareja una acción (`read`, `update`, `delete`, `create` o `write`, que otorga create + update + delete) con un rol destino. Por defecto, **ningún usuario tiene acceso** salvo que los permisos se establezcan explícitamente a nivel de fila/archivo o se hereden de la configuración de la tabla/bucket. Los permisos son arrays de strings construidos con los helpers `Permission` y `Role`.

```python
from appwrite.permission import Permission
from appwrite.role import Role
```

### Fila de base de datos con permisos

```python
doc = tables_db.create_row('[DATABASE_ID]', '[TABLE_ID]', ID.unique(), {
    'title': 'Hello World'
}, [
    Permission.read(Role.user('[USER_ID]')),     # specific user can read
    Permission.update(Role.user('[USER_ID]')),   # specific user can update
    Permission.read(Role.team('[TEAM_ID]')),     # all team members can read
    Permission.read(Role.any()),                 # anyone (including guests) can read
])
```

### Upload de archivos con permisos

```python
file = storage.create_file('[BUCKET_ID]', ID.unique(), InputFile.from_path('/path/to/file.png'), [
    Permission.read(Role.any()),
    Permission.update(Role.user('[USER_ID]')),
    Permission.delete(Role.user('[USER_ID]')),
])
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.read(Role.any())` sobre datos sensibles** — hace el recurso legible públicamente

