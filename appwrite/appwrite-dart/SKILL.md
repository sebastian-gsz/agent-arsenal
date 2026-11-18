---
name: appwrite-dart
description: Skill del SDK de Appwrite para Dart. Úsala al construir apps Flutter (móvil, web, escritorio) o aplicaciones Dart server-side con Appwrite. Cubre auth de cliente (email, OAuth), consultas de base de datos, uploads con manejo nativo de archivos, suscripciones real-time, y administración server-side vía API keys para gestión de usuarios, administración de bases de datos, storage y functions.
---


# Appwrite Dart SDK

## Instalación

```bash
# Flutter (client-side)
flutter pub add appwrite

# Dart (server-side)
dart pub add dart_appwrite
```

## Configuración del cliente

### Client-side (Flutter)

```dart
import 'package:appwrite/appwrite.dart';

final client = Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]');
```

### Server-side (Dart)

```dart
import 'package:dart_appwrite/dart_appwrite.dart';

final client = Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject(Platform.environment['APPWRITE_PROJECT_ID']!)
    .setKey(Platform.environment['APPWRITE_API_KEY']!);
```

## Ejemplos de código

### Autenticación (client-side)

```dart
final account = Account(client);

// Signup
await account.create(userId: ID.unique(), email: 'user@example.com', password: 'password123', name: 'User Name');

// Login
final session = await account.createEmailPasswordSession(email: 'user@example.com', password: 'password123');

// OAuth login
await account.createOAuth2Session(provider: OAuthProvider.google);

// Get current user
final user = await account.get();

// Logout
await account.deleteSession(sessionId: 'current');
```

### Gestión de usuarios (server-side)

```dart
final users = Users(client);

// Create user
final user = await users.create(userId: ID.unique(), email: 'user@example.com', password: 'password123', name: 'User Name');

// List users
final list = await users.list(queries: [Query.limit(25)]);

// Get user
final fetched = await users.get(userId: '[USER_ID]');

// Delete user
await users.delete(userId: '[USER_ID]');
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere parámetros con nombre (p. ej., `databaseId: '...'`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```dart
final tablesDB = TablesDB(client);

// Create database (server-side only)
final db = await tablesDB.create(databaseId: ID.unique(), name: 'My Database');

// Create table (server-side only)
final col = await tablesDB.createTable(databaseId: '[DATABASE_ID]', tableId: ID.unique(), name: 'My Table');

// Create row
final doc = await tablesDB.createRow(
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: ID.unique(),
    data: {'title': 'Hello', 'done': false},
);

// Query rows
final results = await tablesDB.listRows(
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    queries: [Query.equal('done', false), Query.limit(10)],
);

// Get row
final row = await tablesDB.getRow(databaseId: '[DATABASE_ID]', tableId: '[TABLE_ID]', rowId: '[ROW_ID]');

// Update row
await tablesDB.updateRow(
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: '[ROW_ID]',
    data: {'done': true},
);

// Delete row
await tablesDB.deleteRow(
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: '[ROW_ID]',
);
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

```dart
// Create table with explicit string column types
await tablesDB.createTable(
    databaseId: '[DATABASE_ID]',
    tableId: ID.unique(),
    name: 'articles',
    columns: [
        {'key': 'title',    'type': 'varchar',    'size': 255, 'required': true},   // inline, fully indexable
        {'key': 'summary',  'type': 'text',                    'required': false},  // off-page, prefix index only
        {'key': 'body',     'type': 'mediumtext',              'required': false},  // up to ~4 M chars
        {'key': 'raw_data', 'type': 'longtext',                'required': false},  // up to ~1 B chars
    ],
);
```

### Métodos de consulta

```dart
// Filtering
Query.equal('field', 'value')             // == (or pass list for IN)
Query.notEqual('field', 'value')          // !=
Query.lessThan('field', 100)              // <
Query.lessThanEqual('field', 100)         // <=
Query.greaterThan('field', 100)           // >
Query.greaterThanEqual('field', 100)      // >=
Query.between('field', 1, 100)            // 1 <= field <= 100
Query.isNull('field')                     // is null
Query.isNotNull('field')                  // is not null
Query.startsWith('field', 'prefix')       // starts with
Query.endsWith('field', 'suffix')         // ends with
Query.contains('field', 'sub')            // contains
Query.search('field', 'keywords')         // full-text search (requires index)

// Sorting
Query.orderAsc('field')
Query.orderDesc('field')

// Pagination
Query.limit(25)                           // max rows (default 25, max 100)
Query.offset(0)                           // skip N rows
Query.cursorAfter('[ROW_ID]')             // cursor pagination (preferred)
Query.cursorBefore('[ROW_ID]')

// Selection & Logic
Query.select(['field1', 'field2'])        // return only specified fields
Query.or([Query.equal('a', 1), Query.equal('b', 2)])   // OR
Query.and([Query.greaterThan('age', 18), Query.lessThan('age', 65)])  // AND (default)
Query.relationship('comments', [Query.equal('approved', true), Query.limit(25)])  // related documents

// Aggregations (optional alias)
Query.count('*', 'total')
Query.sum('amount', 'revenue')
Query.avg('score', 'avgScore')
Query.min('price', 'minPrice')
Query.max('price', 'maxPrice')

// Grouping
Query.groupBy(['status', 'year'])
Query.having([Query.greaterThan('total', 1)])
Query.distinct()

// Joins (table id, left key, right key, operator, alias)
Query.join('orders', r'$id', 'customerId')
Query.leftJoin('orders', r'$id', 'customerId', '=', 'ord')
Query.leftJoin('orders', 'ord', [
  Query.on(r'$id', 'customerId'),
  Query.equal('ord.status', 'paid'),
])

// Fluent builder. Pass build() into list methods.
final queries = Query.builder()
    .equal('status', 'published')
    .leftJoin('orders', r'$id', 'customerId', '=', 'ord')
    .count('*', 'total')
    .groupBy(['status'])
    .limit(25)
    .build();

await tablesDB.listRows(
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    queries: queries,
);
```

### Storage de archivos

```dart
final storage = Storage(client);

// Upload file
final file = await storage.createFile(
    bucketId: '[BUCKET_ID]',
    fileId: ID.unique(),
    file: InputFile.fromPath(path: '/path/to/file.png', filename: 'file.png'),
);

// Get file preview
final preview = storage.getFilePreview(bucketId: '[BUCKET_ID]', fileId: '[FILE_ID]', width: 300, height: 300);

// List files
final files = await storage.listFiles(bucketId: '[BUCKET_ID]');

// Delete file
await storage.deleteFile(bucketId: '[BUCKET_ID]', fileId: '[FILE_ID]');
```

#### Métodos de fábrica de InputFile

```dart
// Client-side (Flutter)
InputFile.fromPath(path: '/path/to/file.png', filename: 'file.png')    // from path
InputFile.fromBytes(bytes: uint8List, filename: 'file.png')            // from Uint8List

// Server-side (Dart)
InputFile.fromPath(path: '/path/to/file.png', filename: 'file.png')
InputFile.fromBytes(bytes: uint8List, filename: 'file.png')
```

### Teams

```dart
final teams = Teams(client);

// Create team
final team = await teams.create(teamId: ID.unique(), name: 'Engineering');

// List teams
final list = await teams.list();

// Create membership (invite user by email)
final membership = await teams.createMembership(
    teamId: '[TEAM_ID]',
    roles: ['editor'],
    email: 'user@example.com',
);

// List memberships
final members = await teams.listMemberships(teamId: '[TEAM_ID]');

// Update membership roles
await teams.updateMembership(teamId: '[TEAM_ID]', membershipId: '[MEMBERSHIP_ID]', roles: ['admin']);

// Delete team
await teams.delete(teamId: '[TEAM_ID]');
```

> **Acceso por roles:** Usa `Role.team('[TEAM_ID]')` para todos los miembros del team o `Role.team('[TEAM_ID]', 'editor')` para un rol específico del team al asignar permisos.

### Suscripciones real-time (client-side)

```dart
final realtime = Realtime(client);

// Subscribe to row changes
final subscription = realtime.subscribe([
    Channel.tablesdb('[DATABASE_ID]').table('[TABLE_ID]').row(),
]);
subscription.stream.listen((response) {
    print(response.events);   // e.g. ['tablesdb.*.tables.*.rows.*.create']
    print(response.payload);  // the affected resource
});

// Subscribe to multiple channels
final multi = realtime.subscribe([
    Channel.tablesdb('[DATABASE_ID]').table('[TABLE_ID]').row(),
    Channel.bucket('[BUCKET_ID]').file(),
]);

// Cleanup
subscription.close();
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
| `memberships.[MEMBERSHIP_ID]` | Una membresía específica |
| `functions.[FUNCTION_ID].executions` | Actualizaciones de ejecución de functions |

Campos de la respuesta: `events` (array), `payload` (recurso), `channels` (coinciden), `timestamp` (ISO 8601).

### Functions serverless (server-side)

```dart
final functions = Functions(client);

// Execute function
final execution = await functions.createExecution(functionId: '[FUNCTION_ID]', body: '{"key": "value"}');

// List executions
final executions = await functions.listExecutions(functionId: '[FUNCTION_ID]');
```

#### Escribir un handler de function (runtime Dart)

```dart
// lib/main.dart — Appwrite Function entry point
Future<dynamic> main(final context) async {
    // context.req.body        — raw body (String)
    // context.req.bodyJson    — parsed JSON (Map or null)
    // context.req.headers     — headers (Map)
    // context.req.method      — HTTP method
    // context.req.path        — URL path
    // context.req.query       — query params (Map)

    context.log('Processing: ${context.req.method} ${context.req.path}');

    if (context.req.method == 'GET') {
        return context.res.json({'message': 'Hello from Appwrite Function!'});
    }

    return context.res.json({'success': true});      // JSON
    // return context.res.text('Hello');              // plain text
    // return context.res.empty();                    // 204
    // return context.res.redirect('https://...');    // 302
}
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR que usan Dart server-side (Dart Frog, Shelf, etc.) emplean el **SDK de servidor** (`dart_appwrite`) para manejar la auth. Necesitas dos clientes:

- **Admin client** — uses an API key, creates sessions, bypasses rate limits (reusable singleton)
- **Session client** — uses a session cookie, acts on behalf of a user (create per-request, never share)

```dart
import 'package:dart_appwrite/dart_appwrite.dart';

// Admin client (reusable)
final adminClient = Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]')
    .setKey(Platform.environment['APPWRITE_API_KEY']!);

// Session client (create per-request)
final sessionClient = Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]');

final session = request.cookies['a_session_[PROJECT_ID]'];
if (session != null) {
    sessionClient.setSession(session);
}
```

#### Inicio de sesión con email/contraseña

```dart
final account = Account(adminClient);
final session = await account.createEmailPasswordSession(
    email: body['email'],
    password: body['password'],
);

// Cookie name must be a_session_<PROJECT_ID>
response.headers.add('Set-Cookie',
    'a_session_[PROJECT_ID]=${session.secret}; '
    'HttpOnly; Secure; SameSite=Strict; '
    'Expires=${HttpDate.format(DateTime.parse(session.expire))}; Path=/');
```

#### Peticiones autenticadas

```dart
final session = request.cookies['a_session_[PROJECT_ID]'];
if (session == null) {
    return Response(statusCode: 401, body: 'Unauthorized');
}

final sessionClient = Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]')
    .setSession(session);

final account = Account(sessionClient);
final user = await account.get();
```

#### Flujo OAuth2 para SSR

```dart
// Step 1: Redirect to OAuth provider
final account = Account(adminClient);
final redirectUrl = await account.createOAuth2Token(
    provider: OAuthProvider.github,
    success: 'https://example.com/oauth/success',
    failure: 'https://example.com/oauth/failure',
);
return Response(statusCode: 302, headers: {'Location': redirectUrl});

// Step 2: Handle callback — exchange token for session
final account = Account(adminClient);
final session = await account.createSession(
    userId: request.uri.queryParameters['userId']!,
    secret: request.uri.queryParameters['secret']!,
);
// Set session cookie as above
```

> **Seguridad de cookies:** Usa siempre `HttpOnly`, `Secure` y `SameSite=Strict` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `sessionClient.setForwardedUserAgent(request.headers['user-agent'])` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```dart
import 'package:appwrite/appwrite.dart';
// AppwriteException is included in the main import

try {
    final row = await tablesDB.getRow(databaseId: '[DATABASE_ID]', tableId: '[TABLE_ID]', rowId: '[ROW_ID]');
} on AppwriteException catch (e) {
    print(e.message);    // human-readable message
    print(e.code);       // HTTP status code (int)
    print(e.type);       // error type (e.g. 'document_not_found')
    print(e.response);   // full response body (Map)
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

```dart
import 'package:appwrite/appwrite.dart';
// Permission and Role are included in the main package import
```

### Fila de base de datos con permisos

```dart
final doc = await tablesDB.createRow(
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: ID.unique(),
    data: {'title': 'Hello World'},
    permissions: [
        Permission.read(Role.user('[USER_ID]')),     // specific user can read
        Permission.update(Role.user('[USER_ID]')),   // specific user can update
        Permission.read(Role.team('[TEAM_ID]')),     // all team members can read
        Permission.read(Role.any()),                 // anyone (including guests) can read
    ],
);
```

### Upload de archivos con permisos

```dart
final file = await storage.createFile(
    bucketId: '[BUCKET_ID]',
    fileId: ID.unique(),
    file: InputFile.fromPath(path: '/path/to/file.png', filename: 'file.png'),
    permissions: [
        Permission.read(Role.any()),
        Permission.update(Role.user('[USER_ID]')),
        Permission.delete(Role.user('[USER_ID]')),
    ],
);
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.read(Role.any())` sobre datos sensibles** — hace el recurso legible públicamente

