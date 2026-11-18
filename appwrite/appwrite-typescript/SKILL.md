---
name: appwrite-typescript
description: Skill del SDK de Appwrite para TypeScript. Úsala al construir apps JavaScript/TypeScript de navegador, apps móviles React Native o backends Node.js/Deno server-side con Appwrite. Cubre auth de cliente (email, OAuth, anónima), consultas de base de datos, uploads de archivos, suscripciones real-time, y administración server-side vía API keys para gestión de usuarios, administración de bases de datos, storage y functions.
---


# Appwrite TypeScript SDK

## Instalación

```bash
# Web
npm install appwrite

# React Native
npm install react-native-appwrite

# Node.js / Deno
npm install node-appwrite
```

## Configuración del cliente

### Client-side (Web / React Native)

```typescript
// Web
import { Client, Account, TablesDB, Storage, ID, Query } from 'appwrite';

// React Native
import { Client, Account, TablesDB, Storage, ID, Query } from 'react-native-appwrite';

const client = new Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]');
```

### Server-side (Node.js / Deno)

```typescript
import { Client, Users, TablesDB, Storage, Functions, ID, Query } from 'node-appwrite';

const client = new Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject(process.env.APPWRITE_PROJECT_ID)
    .setKey(process.env.APPWRITE_API_KEY);
```

## Ejemplos de código

### Autenticación (client-side)

```typescript
const account = new Account(client);

// Email signup
await account.create({
    userId: ID.unique(),
    email: 'user@example.com',
    password: 'password123',
    name: 'User Name'
});

// Email login
const session = await account.createEmailPasswordSession({
    email: 'user@example.com',
    password: 'password123'
});

// OAuth login (Web)
account.createOAuth2Session({
    provider: OAuthProvider.Github,
    success: 'https://example.com/success',
    failure: 'https://example.com/fail',
    scopes: ['repo', 'user'] // optional — provider-specific scopes
});

// Get current user
const user = await account.get();

// Logout
await account.deleteSession({ sessionId: 'current' });
```

### Inicio de sesión OAuth 2 (React Native)

> **Importante:** `createOAuth2Session()` **no** funciona en React Native. En su lugar debes usar `createOAuth2Token()` con deep linking.

#### Preparación

Instala las dependencias necesarias:

```bash
npx expo install react-native-appwrite react-native-url-polyfill
npm install expo-auth-session expo-web-browser expo-linking
```

Define el esquema de URL en tu `app.json`:

```json
{
  "expo": {
    "scheme": "appwrite-callback-[PROJECT_ID]"
  }
}
```

#### Flujo OAuth

```typescript
import { Client, Account, OAuthProvider } from 'react-native-appwrite';
import { makeRedirectUri } from 'expo-auth-session';
import * as WebBrowser from 'expo-web-browser';

const client = new Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]');

const account = new Account(client);

async function oauthLogin(provider: OAuthProvider) {
    // Create deep link that works across Expo environments
    const deepLink = new URL(makeRedirectUri({ preferLocalhost: true }));
    const scheme = `${deepLink.protocol}//`; // e.g. 'exp://' or 'appwrite-callback-[PROJECT_ID]://'

    // Get the OAuth login URL
    const loginUrl = await account.createOAuth2Token({
        provider,
        success: `${deepLink}`,
        failure: `${deepLink}`,
    });

    // Open browser and listen for the scheme redirect
    const result = await WebBrowser.openAuthSessionAsync(`${loginUrl}`, scheme);

    if (result.type !== 'success') return;

    // Extract credentials from the redirect URL
    const url = new URL(result.url);
    const secret = url.searchParams.get('secret');
    const userId = url.searchParams.get('userId');

    // Create session with the OAuth credentials
    await account.createSession({ userId, secret });
}

// Usage
await oauthLogin(OAuthProvider.Github);
await oauthLogin(OAuthProvider.Google);
```

### Gestión de usuarios (server-side)

```typescript
const users = new Users(client);

// Create user
const user = await users.create({
    userId: ID.unique(),
    email: 'user@example.com',
    password: 'password123',
    name: 'User Name'
});

// List users
const list = await users.list({ queries: [Query.limit(25)] });

// Get user
const fetched = await users.get({ userId: '[USER_ID]' });

// Delete user
await users.delete({ userId: '[USER_ID]' });
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere el estilo de llamada con parámetros en objeto (p. ej., `{ databaseId: '...' }`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```typescript
const tablesDB = new TablesDB(client);

// Create database (server-side only)
const db = await tablesDB.create({ databaseId: ID.unique(), name: 'My Database' });

// Create table (server-side only)
const col = await tablesDB.createTable({
    databaseId: '[DATABASE_ID]',
    tableId: ID.unique(),
    name: 'My Table'
});

// Create row
const doc = await tablesDB.createRow({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: ID.unique(),
    data: { title: 'Hello World', content: 'Example content' }
});

// List rows with query
const results = await tablesDB.listRows({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    queries: [Query.equal('status', 'active'), Query.limit(10)]
});

// Get row
const row = await tablesDB.getRow({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: '[ROW_ID]'
});

// Update row
await tablesDB.updateRow({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: '[ROW_ID]',
    data: { title: 'Updated Title' }
});

// Delete row
await tablesDB.deleteRow({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: '[ROW_ID]'
});
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

```typescript
// Create table with explicit string column types
await tablesDB.createTable({
    databaseId: '[DATABASE_ID]',
    tableId: ID.unique(),
    name: 'articles',
    columns: [
        { key: 'title',    type: 'varchar',    size: 255, required: true  },  // inline, fully indexable
        { key: 'summary',  type: 'text',                  required: false },  // off-page, prefix index only
        { key: 'body',     type: 'mediumtext',            required: false },  // up to ~4 M chars
        { key: 'raw_data', type: 'longtext',              required: false },  // up to ~1 B chars
    ]
});
```

#### Genéricos de TypeScript

```typescript
import { Models } from 'appwrite';
// Server-side: import from 'node-appwrite'

// Define a typed interface for your row data
interface Todo {
    title: string;
    done: boolean;
    priority: number;
}

// listRows returns Models.DocumentList<Models.Document> by default
// Cast or use generics for typed results
const results = await tablesDB.listRows({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    queries: [Query.equal('done', false)]
});

// Each document includes built-in fields alongside your data
const doc = results.documents[0];
doc.$id;            // string — unique row ID
doc.$createdAt;     // string — ISO 8601 creation timestamp
doc.$updatedAt;     // string — ISO 8601 update timestamp
doc.$permissions;   // string[] — permission strings
doc.$databaseId;    // string
doc.$collectionId;  // string

// Common model types
// Models.User<Preferences>  — user account
// Models.Session             — auth session
// Models.File                — storage file metadata
// Models.Team                — team object
// Models.Execution           — function execution result
// Models.DocumentList<T>     — paginated list with total count
```

### Métodos de consulta

```typescript
// Filtering
Query.equal('field', 'value')           // field == value (or pass array for IN)
Query.notEqual('field', 'value')        // field != value
Query.lessThan('field', 100)            // field < value
Query.lessThanEqual('field', 100)       // field <= value
Query.greaterThan('field', 100)         // field > value
Query.greaterThanEqual('field', 100)    // field >= value
Query.between('field', 1, 100)          // 1 <= field <= 100
Query.isNull('field')                   // field is null
Query.isNotNull('field')                // field is not null
Query.startsWith('field', 'prefix')     // string starts with prefix
Query.endsWith('field', 'suffix')       // string ends with suffix
Query.contains('field', 'substring')    // string/array contains value
Query.search('field', 'keywords')       // full-text search (requires full-text index)

// Sorting
Query.orderAsc('field')                 // sort ascending
Query.orderDesc('field')                // sort descending

// Pagination
Query.limit(25)                         // max rows returned (default 25, max 100)
Query.offset(0)                         // skip N rows
Query.cursorAfter('[ROW_ID]')           // paginate after this row ID (preferred for large datasets)
Query.cursorBefore('[ROW_ID]')          // paginate before this row ID

// Selection
Query.select(['field1', 'field2'])      // return only specified fields

// Logical
Query.or([Query.equal('a', 1), Query.equal('b', 2)])   // OR condition
Query.and([Query.greaterThan('age', 18), Query.lessThan('age', 65)])  // explicit AND (queries are AND by default)
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
Query.join('orders', '$id', 'customerId')
Query.leftJoin('orders', '$id', 'customerId', '=', 'ord')
Query.leftJoin('orders', 'ord', [
    Query.on('$id', 'customerId'),
    Query.equal('ord.status', 'paid'),
])

// Fluent builder. Pass build() into list methods.
const queries = Query.builder()
    .equal('status', 'published')
    .leftJoin('orders', '$id', 'customerId', '=', 'ord')
    .count('*', 'total')
    .groupBy(['status'])
    .limit(25)
    .build();

await tablesDB.listRows({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    queries,
});
```

### Storage de archivos

```typescript
const storage = new Storage(client);

// Upload file (client-side — from file input)
const file = await storage.createFile({
    bucketId: '[BUCKET_ID]',
    fileId: ID.unique(),
    file: document.getElementById('file-input').files[0]
});

// Upload file (server-side — from path)
import { InputFile } from 'node-appwrite/file';

const file2 = await storage.createFile({
    bucketId: '[BUCKET_ID]',
    fileId: ID.unique(),
    file: InputFile.fromPath('/path/to/file.png', 'file.png')
});

// List files
const files = await storage.listFiles({ bucketId: '[BUCKET_ID]' });

// Get file preview (image)
const preview = storage.getFilePreview({
    bucketId: '[BUCKET_ID]',
    fileId: '[FILE_ID]',
    width: 300,
    height: 300
});

// Download file
const download = await storage.getFileDownload({
    bucketId: '[BUCKET_ID]',
    fileId: '[FILE_ID]'
});

// Delete file
await storage.deleteFile({ bucketId: '[BUCKET_ID]', fileId: '[FILE_ID]' });
```

#### Métodos de fábrica de InputFile (server-side)

```typescript
import { InputFile } from 'node-appwrite/file';

InputFile.fromPath('/path/to/file.png', 'file.png')          // from filesystem path
InputFile.fromBuffer(buffer, 'file.png')                       // from Buffer
InputFile.fromStream(readableStream, 'file.png', size)         // from ReadableStream (size in bytes required)
InputFile.fromPlainText('Hello world', 'hello.txt')            // from string content
```

### Teams

```typescript
const teams = new Teams(client);

// Create team
const team = await teams.create({ teamId: ID.unique(), name: 'Engineering' });

// List teams
const list = await teams.list();

// Create membership (invite a user by email)
const membership = await teams.createMembership({
    teamId: '[TEAM_ID]',
    roles: ['editor'],
    email: 'user@example.com',
});

// List memberships
const members = await teams.listMemberships({ teamId: '[TEAM_ID]' });

// Update membership roles
await teams.updateMembership({
    teamId: '[TEAM_ID]',
    membershipId: '[MEMBERSHIP_ID]',
    roles: ['admin'],
});

// Delete team
await teams.delete({ teamId: '[TEAM_ID]' });
```

> **Acceso por roles:** Usa `Role.team('[TEAM_ID]')` para todos los miembros del team o `Role.team('[TEAM_ID]', 'editor')` para un rol específico del team al asignar permisos.

### Suscripciones real-time (client-side)

```typescript
import { Realtime, Channel } from 'appwrite';

const realtime = new Realtime(client);

// Subscribe to row changes
const subscription = await realtime.subscribe(
    Channel.tablesdb('[DATABASE_ID]').table('[TABLE_ID]').row(),
    (response) => {
        console.log(response.events);   // e.g. ['tablesdb.*.tables.*.rows.*.create']
        console.log(response.payload);  // the affected resource
    }
);

// Subscribe to a specific row
await realtime.subscribe(
    Channel.tablesdb('[DATABASE_ID]').table('[TABLE_ID]').row('[ROW_ID]'),
    (response) => { /* ... */ }
);

// Subscribe to multiple channels
await realtime.subscribe([
    Channel.tablesdb('[DATABASE_ID]').table('[TABLE_ID]').row(),
    Channel.bucket('[BUCKET_ID]').file(),
], (response) => { /* ... */ });

// Unsubscribe
await subscription.close();
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
| `teams.[TEAM_ID]` | Cambios en un team específico |
| `memberships` | Cambios en las membresías de team del usuario |
| `memberships.[MEMBERSHIP_ID]` | Una membresía específica |
| `functions.[FUNCTION_ID].executions` | Actualizaciones de ejecución de una function |

El objeto `response` incluye: `events` (array de strings de eventos), `payload` (el recurso afectado), `channels` (canales que coinciden) y `timestamp` (ISO 8601).

### Functions serverless (server-side)

```typescript
const functions = new Functions(client);

// Execute function
const execution = await functions.createExecution({
    functionId: '[FUNCTION_ID]',
    body: JSON.stringify({ key: 'value' })
});

// List executions
const executions = await functions.listExecutions({ functionId: '[FUNCTION_ID]' });
```

#### Escribir un handler de function (runtime Node.js)

Al desplegar tu propia Appwrite Function, el archivo de punto de entrada debe exportar una función async por defecto:

```typescript
// src/main.js (or src/main.ts)
export default async ({ req, res, log, error }) => {
    // Request properties
    // req.body        — raw request body (string)
    // req.bodyJson    — parsed JSON body (object, or undefined if not JSON)
    // req.headers     — request headers (object)
    // req.method      — HTTP method (GET, POST, PUT, DELETE, PATCH)
    // req.path        — URL path (e.g. '/hello')
    // req.query       — parsed query parameters (object)
    // req.queryString — raw query string

    log('Processing request: ' + req.method + ' ' + req.path);

    if (req.method === 'GET') {
        return res.json({ message: 'Hello from Appwrite Function!' });
    }

    const data = req.bodyJson;
    if (!data?.name) {
        error('Missing name field');
        return res.json({ error: 'Name is required' }, 400);
    }

    // Response methods
    return res.json({ success: true });                    // JSON (sets Content-Type automatically)
    // return res.text('Hello');                           // plain text
    // return res.empty();                                 // 204 No Content
    // return res.redirect('https://example.com');         // 302 Redirect
    // return res.send('data', 200, { 'X-Custom': '1' }); // custom body, status, headers
};
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR (Next.js, SvelteKit, Nuxt, Remix, Astro) emplean el **SDK de servidor** (`node-appwrite`) para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```typescript
import { Client, Account, OAuthProvider } from 'node-appwrite';

// Admin client (reusable)
const adminClient = new Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]')
    .setKey(process.env.APPWRITE_API_KEY);

// Session client (create per-request)
const sessionClient = new Client()
    .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    .setProject('[PROJECT_ID]');

const session = req.cookies['a_session_[PROJECT_ID]'];
if (session) {
    sessionClient.setSession(session);
}
```

#### Inicio de sesión con email/contraseña

```typescript
app.post('/login', async (req, res) => {
    const account = new Account(adminClient);
    const session = await account.createEmailPasswordSession({
        email: req.body.email,
        password: req.body.password,
    });

    // Cookie name must be a_session_<PROJECT_ID>
    res.cookie('a_session_[PROJECT_ID]', session.secret, {
        httpOnly: true,
        secure: true,
        sameSite: 'strict',
        expires: new Date(session.expire),
        path: '/',
    });

    res.json({ success: true });
});
```

#### Peticiones autenticadas

```typescript
app.get('/user', async (req, res) => {
    const session = req.cookies['a_session_[PROJECT_ID]'];
    if (!session) return res.status(401).json({ error: 'Unauthorized' });

    // Create a fresh session client per request
    const sessionClient = new Client()
        .setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
        .setProject('[PROJECT_ID]')
        .setSession(session);

    const account = new Account(sessionClient);
    const user = await account.get();
    res.json(user);
});
```

#### Flujo OAuth2 para SSR

```typescript
// Step 1: Redirect to OAuth provider
app.get('/oauth', async (req, res) => {
    const account = new Account(adminClient);
    const redirectUrl = await account.createOAuth2Token({
        provider: OAuthProvider.Github,
        success: 'https://example.com/oauth/success',
        failure: 'https://example.com/oauth/failure',
    });
    res.redirect(redirectUrl);
});

// Step 2: Handle callback — exchange token for session
app.get('/oauth/success', async (req, res) => {
    const account = new Account(adminClient);
    const session = await account.createSession({
        userId: req.query.userId,
        secret: req.query.secret,
    });

    res.cookie('a_session_[PROJECT_ID]', session.secret, {
        httpOnly: true, secure: true, sameSite: 'strict',
        expires: new Date(session.expire), path: '/',
    });
    res.json({ success: true });
});
```

> **Seguridad de cookies:** Usa siempre `httpOnly`, `secure` y `sameSite: 'strict'` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `sessionClient.setForwardedUserAgent(req.headers['user-agent'])` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```typescript
import { AppwriteException } from 'appwrite';
// Server-side: import from 'node-appwrite'

try {
    const doc = await tablesDB.getRow({
        databaseId: '[DATABASE_ID]',
        tableId: '[TABLE_ID]',
        rowId: '[ROW_ID]',
    });
} catch (err) {
    if (err instanceof AppwriteException) {
        console.log(err.message);   // human-readable error message
        console.log(err.code);      // HTTP status code (number)
        console.log(err.type);      // Appwrite error type string (e.g. 'document_not_found')
        console.log(err.response);  // full response body (object)
    }
}
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

```typescript
import { Permission, Role } from 'appwrite';
// Server-side: import from 'node-appwrite'
```

### Fila de base de datos con permisos

```typescript
const doc = await tablesDB.createRow({
    databaseId: '[DATABASE_ID]',
    tableId: '[TABLE_ID]',
    rowId: ID.unique(),
    data: { title: 'Hello World' },
    permissions: [
        Permission.read(Role.user('[USER_ID]')),     // specific user can read
        Permission.update(Role.user('[USER_ID]')),   // specific user can update
        Permission.read(Role.team('[TEAM_ID]')),     // all team members can read
        Permission.read(Role.any()),                 // anyone (including guests) can read
    ]
});
```

### Upload de archivos con permisos

```typescript
const file = await storage.createFile({
    bucketId: '[BUCKET_ID]',
    fileId: ID.unique(),
    file: document.getElementById('file-input').files[0],
    permissions: [
        Permission.read(Role.any()),
        Permission.update(Role.user('[USER_ID]')),
        Permission.delete(Role.user('[USER_ID]')),
    ]
});
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role.any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission.read(Role.any())` sobre datos sensibles** — hace el recurso legible públicamente

