---
name: appwrite-php
description: Skill del SDK de Appwrite para PHP. Úsala al construir aplicaciones PHP server-side con Appwrite, incluidas las integraciones con Laravel y Symfony. Cubre gestión de usuarios, CRUD de bases de datos/tablas, storage de archivos y functions vía API keys.
---


# Appwrite PHP SDK

## Instalación

```bash
composer require appwrite/appwrite
```

## Configuración del cliente

```php
use Appwrite\Client;
use Appwrite\ID;
use Appwrite\Query;
use Appwrite\Services\Users;
use Appwrite\Services\TablesDB;
use Appwrite\Services\Storage;
use Appwrite\Services\Functions;
use Appwrite\InputFile;

$client = (new Client())
    ->setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    ->setProject(getenv('APPWRITE_PROJECT_ID'))
    ->setKey(getenv('APPWRITE_API_KEY'));
```

## Ejemplos de código

### Gestión de usuarios

```php
$users = new Users($client);

// Create user
$user = $users->create(ID::unique(), 'user@example.com', null, 'password123', 'User Name');

// List users
$list = $users->list([Query::limit(25)]);

// Get user
$fetched = $users->get('[USER_ID]');

// Delete user
$users->delete('[USER_ID]');
```

### Operaciones de base de datos

> **Nota:** Usa `TablesDB` (no la clase `Databases` deprecada) en todo código nuevo. Usa `Databases` solo si el codebase existente ya depende de ella o el usuario lo pide explícitamente.
>
> **Consejo:** Prefiere argumentos con nombre (PHP 8+, p. ej., `databaseId: '...'`) en todas las llamadas a métodos del SDK. Usa argumentos posicionales solo si el codebase existente ya los usa o el usuario lo pide explícitamente.

```php
$tablesDB = new TablesDB($client);

// Create database
$db = $tablesDB->create(ID::unique(), 'My Database');

// Create row
$doc = $tablesDB->createRow('[DATABASE_ID]', '[TABLE_ID]', ID::unique(), [
    'title' => 'Hello World'
]);

// Query rows
$results = $tablesDB->listRows('[DATABASE_ID]', '[TABLE_ID]', [
    Query::equal('title', ['Hello World']),
    Query::limit(10)
]);

// Get row
$row = $tablesDB->getRow('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]');

// Update row
$tablesDB->updateRow('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]', [
    'title' => 'Updated'
]);

// Delete row
$tablesDB->deleteRow('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]');
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

```php
// Create table with explicit string column types
$tablesDB->createTable('[DATABASE_ID]', ID::unique(), 'articles', [
    ['key' => 'title',    'type' => 'varchar',    'size' => 255, 'required' => true],
    ['key' => 'summary',  'type' => 'text',                      'required' => false],
    ['key' => 'body',     'type' => 'mediumtext',                'required' => false],
    ['key' => 'raw_data', 'type' => 'longtext',                  'required' => false],
]);
```

### Métodos de consulta

```php
// Filtering
Query::equal('field', ['value'])            // == (always pass array)
Query::notEqual('field', ['value'])         // !=
Query::lessThan('field', 100)              // <
Query::lessThanEqual('field', 100)         // <=
Query::greaterThan('field', 100)           // >
Query::greaterThanEqual('field', 100)      // >=
Query::between('field', 1, 100)            // 1 <= field <= 100
Query::isNull('field')                     // is null
Query::isNotNull('field')                  // is not null
Query::startsWith('field', 'prefix')       // starts with
Query::endsWith('field', 'suffix')         // ends with
Query::contains('field', ['sub'])          // contains (string or array)
Query::search('field', 'keywords')         // full-text search (requires index)

// Sorting
Query::orderAsc('field')
Query::orderDesc('field')

// Pagination
Query::limit(25)                           // max rows (default 25, max 100)
Query::offset(0)                           // skip N rows
Query::cursorAfter('[ROW_ID]')             // cursor pagination (preferred)
Query::cursorBefore('[ROW_ID]')

// Selection & Logic
Query::select(['field1', 'field2'])        // return only specified fields
Query::or([Query::equal('a', [1]), Query::equal('b', [2])])   // OR
Query::and([Query::greaterThan('age', 18), Query::lessThan('age', 65)])  // AND (default)
Query::relationship('comments', [Query::equal('approved', [true]), Query::limit(25)])  // related documents

// Aggregations (optional alias)
Query::count('*', 'total')
Query::sum('amount', 'revenue')
Query::avg('score', 'avgScore')
Query::min('price', 'minPrice')
Query::max('price', 'maxPrice')

// Grouping
Query::groupBy(['status', 'year'])
Query::having([Query::greaterThan('total', 1)])
Query::distinct()

// Joins (table id, left key, right key, operator, alias)
Query::join('orders', '$id', 'customerId')
Query::leftJoin('orders', '$id', 'customerId', '=', 'ord')

// Nested join ON: extra predicates stay on the JOIN, not WHERE
Query::leftJoin('orders', 'ord', [
    Query::on('$id', 'customerId'),
    Query::equal('ord.status', 'paid'),
])

// Fluent builder. Pass build() into list methods.
$queries = Query::builder()
    ->equal('status', 'published')
    ->leftJoin('orders', '$id', 'customerId', '=', 'ord')
    ->count('*', 'total')
    ->groupBy(['status'])
    ->limit(25)
    ->build();

$tablesDB->listRows('[DATABASE_ID]', '[TABLE_ID]', $queries);
```

### Storage de archivos

```php
$storage = new Storage($client);

// Upload file
$file = $storage->createFile('[BUCKET_ID]', ID::unique(), InputFile::withPath('/path/to/file.png'));

// List files
$files = $storage->listFiles('[BUCKET_ID]');

// Delete file
$storage->deleteFile('[BUCKET_ID]', '[FILE_ID]');
```

#### Métodos de fábrica de InputFile

```php
use Appwrite\InputFile;

InputFile::withPath('/path/to/file.png')                    // from filesystem path
InputFile::withData('Hello world', 'hello.txt')             // from string content
```

### Teams

```php
$teams = new Teams($client);

// Create team
$team = $teams->create(ID::unique(), 'Engineering');

// List teams
$list = $teams->list();

// Create membership (invite user by email)
$membership = $teams->createMembership('[TEAM_ID]', ['editor'], email: 'user@example.com');

// List memberships
$members = $teams->listMemberships('[TEAM_ID]');

// Update membership roles
$teams->updateMembership('[TEAM_ID]', '[MEMBERSHIP_ID]', ['admin']);

// Delete team
$teams->delete('[TEAM_ID]');
```

> **Acceso por roles:** Usa `Role::team('[TEAM_ID]')` para todos los miembros del team o `Role::team('[TEAM_ID]', 'editor')` para un rol específico del team al asignar permisos.

### Functions serverless

```php
$functions = new Functions($client);

// Execute function
$execution = $functions->createExecution('[FUNCTION_ID]', '{"key": "value"}');

// List executions
$executions = $functions->listExecutions('[FUNCTION_ID]');
```

#### Escribir un handler de function (runtime PHP)

```php
// src/main.php — Appwrite Function entry point
return function ($context) {
    // $context->req->body        — raw body (string)
    // $context->req->bodyJson    — parsed JSON (array or null)
    // $context->req->headers     — headers (array)
    // $context->req->method      — HTTP method
    // $context->req->path        — URL path
    // $context->req->query       — query params (array)

    $context->log('Processing: ' . $context->req->method . ' ' . $context->req->path);

    if ($context->req->method === 'GET') {
        return $context->res->json(['message' => 'Hello from Appwrite Function!']);
    }

    $data = $context->req->bodyJson ?? [];
    if (!isset($data['name'])) {
        $context->error('Missing name field');
        return $context->res->json(['error' => 'Name is required'], 400);
    }

    return $context->res->json(['success' => true]);      // JSON
    // return $context->res->text('Hello');                // plain text
    // return $context->res->empty();                      // 204
    // return $context->res->redirect('https://...');      // 302
};
```

### Autenticación SSR (Server-Side Rendering)

Las apps SSR (Laravel, Symfony, etc.) emplean el **SDK de servidor** para manejar la auth. Necesitas dos clientes:

- **Cliente admin** — usa una API key, crea sesiones y evita los rate limits (singleton reutilizable)
- **Cliente de sesión** — usa una cookie de sesión, actúa en nombre de un usuario (crea uno por petición, nunca lo compartas)

```php
use Appwrite\Client;
use Appwrite\Services\Account;

// Admin client (reusable)
$adminClient = (new Client())
    ->setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    ->setProject('[PROJECT_ID]')
    ->setKey(getenv('APPWRITE_API_KEY'));

// Session client (create per-request)
$sessionClient = (new Client())
    ->setEndpoint('https://<REGION>.cloud.appwrite.io/v1')
    ->setProject('[PROJECT_ID]');

$session = $_COOKIE['a_session_[PROJECT_ID]'] ?? null;
if ($session) {
    $sessionClient->setSession($session);
}
```

#### Inicio de sesión con email/contraseña

```php
$account = new Account($adminClient);
$session = $account->createEmailPasswordSession($email, $password);

// Cookie name must be a_session_<PROJECT_ID>
setcookie('a_session_[PROJECT_ID]', $session['secret'], [
    'httpOnly' => true,
    'secure' => true,
    'sameSite' => 'strict',
    'expires' => strtotime($session['expire']),
    'path' => '/',
]);
```

#### Peticiones autenticadas

```php
$session = $_COOKIE['a_session_[PROJECT_ID]'] ?? null;
if (!$session) {
    http_response_code(401);
    exit;
}

$sessionClient->setSession($session);
$account = new Account($sessionClient);
$user = $account->get();
```

#### Flujo OAuth2 para SSR

```php
// Step 1: Redirect to OAuth provider
$account = new Account($adminClient);
$redirectUrl = $account->createOAuth2Token(
    OAuthProvider::GITHUB(),
    'https://example.com/oauth/success',
    'https://example.com/oauth/failure',
);
header('Location: ' . $redirectUrl);

// Step 2: Handle callback — exchange token for session
$account = new Account($adminClient);
$session = $account->createSession($_GET['userId'], $_GET['secret']);

setcookie('a_session_[PROJECT_ID]', $session['secret'], [
    'httpOnly' => true, 'secure' => true, 'sameSite' => 'strict',
    'expires' => strtotime($session['expire']), 'path' => '/',
]);
```

> **Seguridad de cookies:** Usa siempre `httpOnly`, `secure` y `sameSite: 'strict'` para prevenir XSS. El nombre de la cookie debe ser `a_session_<PROJECT_ID>`.

> **Reenvío del user agent:** Llama a `$sessionClient->setForwardedUserAgent($_SERVER['HTTP_USER_AGENT'])` para registrar la información del navegador del usuario final a efectos de depuración y seguridad.

## Error Handling

```php
use Appwrite\AppwriteException;

try {
    $row = $tablesDB->getRow('[DATABASE_ID]', '[TABLE_ID]', '[ROW_ID]');
} catch (AppwriteException $e) {
    echo $e->getMessage();   // human-readable error message
    echo $e->getCode();      // HTTP status code (int)
    echo $e->getType();      // Appwrite error type string (e.g. 'document_not_found')
    echo $e->getResponse();  // full response body (array)
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

```php
use Appwrite\Permission;
use Appwrite\Role;
```

### Fila de base de datos con permisos

```php
$doc = $tablesDB->createRow('[DATABASE_ID]', '[TABLE_ID]', ID::unique(), [
    'title' => 'Hello World'
], [
    Permission::read(Role::user('[USER_ID]')),     // specific user can read
    Permission::update(Role::user('[USER_ID]')),   // specific user can update
    Permission::read(Role::team('[TEAM_ID]')),     // all team members can read
    Permission::read(Role::any()),                 // anyone (including guests) can read
]);
```

### Upload de archivos con permisos

```php
$file = $storage->createFile('[BUCKET_ID]', ID::unique(), InputFile::withPath('/path/to/file.png'), [
    Permission::read(Role::any()),
    Permission::update(Role::user('[USER_ID]')),
    Permission::delete(Role::user('[USER_ID]')),
]);
```

> **Cuándo asignar permisos:** Define permisos a nivel de fila/archivo cuando necesites control de acceso por recurso. Si todas las filas de una tabla comparten las mismas reglas, configura los permisos a nivel de tabla/bucket y deja vacíos los permisos de fila.

> **Errores comunes:**
> - **Olvidar los permisos** — el recurso queda inaccesible para todos los usuarios (incluido el creador)
> - **`Role::any()` con `write`/`update`/`delete`** — permite que cualquier usuario, incluidos invitados no autenticados, modifique o elimine el recurso
> - **`Permission::read(Role::any())` sobre datos sensibles** — hace el recurso legible públicamente

