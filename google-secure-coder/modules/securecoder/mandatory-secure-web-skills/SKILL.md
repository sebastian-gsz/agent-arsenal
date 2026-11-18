<!-- disableFinding(SKILLS_METADATA_MISSING) -->
<!-- disableFinding(SKILLS_BODY_LENGTH) -->
---
name: mandatory-secure-web-skills
description: >
  CRÍTICO: DEBES usar esta skill para TODAS las tareas de generación de código
  y de diseño. Define reglas obligatorias de codificación segura para
  aplicaciones web. DEBES aplicar estrictamente estas reglas en el acceso a
  base de datos, el manejo de archivos, la gestión de sesiones y el renderizado
  del frontend para prevenir vulnerabilidades críticas. ¡NO ignores ninguna
  sección!
---

## Directrices de codificación segura

*   Validación y sanitización de entradas: Trata todos los datos externos como
    no confiables. Valida las entradas contra una allow-list de tipos,
    longitudes y formatos esperados para prevenir la inyección SQL y el
    cross-site scripting (XSS).
*   Codificación de salida: Convierte los datos a un formato seguro antes de
    enviarlos al cliente. Esto garantiza que los navegadores traten los
    scripts potencialmente maliciosos como texto plano y no como código
    ejecutable.
*   Autenticación y autorización: Exige autenticación fuerte para las páginas
    no públicas. Usa protocolos modernos como OAuth 2.0 u OpenID Connect para
    verificar usuarios, y JSON Web Tokens (JWT) para una gestión de sesiones
    segura y sin estado. Nunca hardcodees secretos JWT, API keys, ni uses
    cadenas de fallback en el código fuente.
*   Gestión de contraseñas: Usa librerías consolidadas para el hasheo de
    contraseñas (p. ej., Argon2 o bcrypt) con salts únicos, y nunca almacenes
    contraseñas en texto plano.
*   Principio de mínimo privilegio: Concede a usuarios y procesos solo los
    permisos mínimos necesarios para desempeñar sus funciones. Usa el control
    de acceso basado en roles (RBAC) para gestionar estos permisos de forma
    centralizada.
*   Gestión de sesiones segura: Genera en el servidor identificadores de
    sesión aleatorios y criptográficamente fuertes. Protege estos tokens
    usando las flags HttpOnly y Secure en las cookies y aplicando tiempos de
    inactividad cortos.
*   Arquitectura segura (patrón BFF): Usa un Backend-for-Frontend (BFF) para
    que el frontend se comunique con una capa BFF segura del lado del servidor
    en lugar de hacerlo directamente con APIs públicas, manteniendo las claves
    sensibles fuera del cliente.
*   Seguridad de rutas y del sistema de archivos: Nunca confíes en la entrada
    del usuario (incluidos los nombres de archivos subidos y los metadatos de
    archivos extraídos como etiquetas/formatos ID3) en las rutas de archivos.
    Sanitiza las entradas (p. ej., usando `path.basename()`) para eliminar las
    secuencias de traversal (`../`, ``..\`) antes de pasarlas a sinks
    como``fs.writeFile`,`fs.readFile`, o`path.join`. Evita la sanitización
    personalizada como`split('/')`, que falla contra las barras invertidas de
    Windows. Cuando restrinjas el acceso a directorios específicos, resuelve
    las rutas por completo y verifica estrictamente el límite del directorio
    (p. ej., exige una barra final al usar comprobaciones de prefijo
    como`resolved.startsWith(sandboxDir + path.sep)`) para evitar bypasses por
    coincidencia parcial (p. ej.,`/sandbox-malicious`eludiendo una`/sandbox`
    comprobación).
*   Ejecución de comandos del sistema: No pases entrada de usuario sin validar
    directamente a sinks de ejecución (`exec`, `spawn`, etc.). Valida siempre
    las rutas de los binarios y los argumentos contra una allow-list estricta
    y predefinida en el código. Asegúrate de que los directorios de ejecución
    también se verifiquen estrictamente contra bypasses por coincidencia
    parcial (p. ej., exigiendo una barra final durante las comprobaciones de
    prefijo).
*   Manejo de errores y logging: Muestra mensajes de error genéricos a los
    usuarios mientras registras información de diagnóstico detallada de forma
    segura para los desarrolladores. Asegúrate de que los logs no contengan
    datos sensibles como contraseñas o tokens de sesión.
*   Cifrado de datos: Cifra los datos sensibles tanto en reposo como en
    tránsito (usando TLS 1.2 o superior)
*   Fail Safe: Cuando algo falla, falla cerrado (deniega el acceso)
*   Criptografía: Usa librerías consolidadas y primitivas seguras. Usa cifrado
    autenticado y hashes criptográficos seguros. Usa el PRNG seguro del
    sistema operativo.
*   Deserialización: Nunca uses formatos de deserialización inseguros.
*   Carga de código: Nunca cargues código remotamente a menos que verifiques
    su origen.

## Pruebas

*   Los servidores **DEBEN** escuchar en localhost o 127.0.0.1 durante las
    pruebas. Los servidores **NO DEBEN** escuchar en 0.0.0.0.

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Skill de frontend web seguro

Para diseñar y escribir un frontend web seguro, sigue las secciones de este
documento

## Inyección XSS

## Skill de prevención de XSS para frontend web seguro

Sigue estos principios guía y añádelos a tu plan:

## Escape nativo del framework

*   **NO DEBES** confiar en datos provenientes de la base de datos, archivos
    de configuración, archivos subidos, peticiones del usuario, etc., que el
    usuario que interactúa con la app pueda controlar directa o indirectamente.
*   **DEBES** escapar o validar los datos no confiables en todas las respuestas
    salientes: HTML, Javascript, CSS, cabeceras HTTP, etc.
*   **DEBES** apoyarte en el auto-escape nativo del framework (p. ej., JSX de
    React, interpolación de Angular).
*   **DEBES** siempre entrecomillar los atributos HTML al usar variables en
    las plantillas para prevenir el attribute breakout y XSS.
    *   **Vulnerable (negativo):** `html <div class={{ var }}>`
    *   **Seguro (positivo):** `html <div class="{{ var }}">`
*   **NO DEBES** usar métodos inseguros como `dangerouslySetInnerHTML` de React
    o `bypassSecurityTrustHtml` de Angular sin una aprobación de seguridad
    explícita y una justificación rigurosa.
*   **DEBES** usar `DOMPurify` al renderizar entradas de cadenas HTML crudas
    inevitables (p. ej., de editores de texto enriquecido).

### Ejemplos

*   **Código vulnerable (React):** `tsx <div dangerouslySetInnerHTML={{ __html:
    userInput }} />`
*   **Código seguro (React con DOMPurify):** `tsx <div dangerouslySetInnerHTML={{
    __html: DOMPurify.sanitize(userInput) }} />`

## Manipulación del DOM con JavaScript vanilla

*   **NO DEBES** usar propiedades/métodos DOM inseguros como `innerHTML`,
    `outerHTML`, `document.write` o `insertAdjacentHTML`.
*   **DEBES** usar `textContent` o `innerText` para insertar texto de forma
    segura.
*   **DEBES** usar `document.createElement()`, `setAttribute()` y
    `appendChild()` para construir elementos estructurales del DOM en lugar de
    cadenas HTML crudas.
*   **DEBES** usar `element.replaceChildren()` o `element.textContent = ''`
    para limpiar el contenido de un elemento en lugar de
    `element.innerHTML = ''`.
*   **DEBES** usar `DOMParser` o `document.createElementNS()` para insertar
    estructuras estáticas complejas (como SVG) y evitar por completo las
    asignaciones a `innerHTML`, incluso con cadenas estrictamente predefinidas
    en el código.

### Ejemplos

*   **Código vulnerable (Vanilla JS):** `javascript element.innerHTML = "<span>" +
    userInput + "</span>";`
*   **Código seguro (Vanilla JS):** `javascript const span =
    document.createElement('span'); span.textContent = userInput;
    element.appendChild(span);`
*   **Código seguro (limpiar elementos):** `javascript element.replaceChildren();`
*   **Código seguro (SVG estático):** `javascript const doc = new
    DOMParser().parseFromString('<svg>...</svg>', 'image/svg+xml');
    element.appendChild(doc.documentElement);`

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Almacenamiento y sesión

## Skill de almacenamiento y sesión para frontend web seguro

Sigue estos principios guía y añádelos a tu plan:

## Almacenamiento de datos sensibles

*   **NO DEBES** almacenar tokens de autenticación sensibles (Session IDs,
    Bearer tokens) en `localStorage` o `sessionStorage` debido a la exposición
    a vulnerabilidades XSS.
*   **DEBES** apoyarte en cookies seguras con `HttpOnly`, `Secure`,
    `SameSite=Lax` (o `Strict`) para la gestión de sesiones, que el navegador
    envía automáticamente.

### Ejemplos

*   **Código vulnerable (TypeScript):** `typescript
    localStorage.setItem('auth_token', response.token); // Vulnerable to XSS
    theft`
*   **Código seguro (estrategia backend/cookie):**
    *   *Los tokens los establece en cookies directamente el servicio de
        backend/auth; no son accesibles vía JS.*

## Ciclo de vida de la sesión

*   **DEBES** limpiar el estado del lado cliente (p. ej., Redux, Context,
    variables locales) al cerrar sesión.
*   **DEBES** provocar una recarga completa de la página o una redirección
    para limpiar la caché al cerrar sesión.

### Ejemplos

*   **Código seguro (manejador de logout en TypeScript):** `typescript function
    handleLogout() { authService.clearSession(); // Clears memory state
    window.location.href = '/login'; // Redirect triggers clean state }`

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Configuración

## Skill de configuración para frontend web seguro

Sigue estos principios guía y añádelos a tu plan:

## Política de Seguridad de Contenido (CSP)

*   **DEBES** implementar una Política de Seguridad de Contenido (CSP)
    estricta para mitigar los riesgos de XSS.
*   **DEBERÍAS** usar nonces para los scripts inline o limitar estrictamente
    `script-src` a orígenes confiables.
*   **NO DEBES** usar `unsafe-inline` o `unsafe-eval` en la CSP sin una
    revisión de seguridad explícita.

### Ejemplos

*   **Cabecera Content-Security-Policy segura:** `http Content-Security-Policy:
    default-src 'self'; script-src 'self' 'nonce-random123'; object-src 'none';`

## Integridad de subrecursos (SRI)

*   **DEBES** usar hashes de Subresource Integrity (SRI) al cargar recursos
    desde CDNs que no son de primera parte. Usa un gestor de paquetes en lugar
    de un CDN para las dependencias cuando sea posible, y fija siempre una
    versión de parche exacta de una dependencia al usar un CDN ajeno. Deja una
    nota para que el usuario actualice estas versiones.

## Protección contra clickjacking

*   **DEBES** configurar protecciones anti-clickjacking usando el scoping
    correcto de `X-Frame-Options: DENY` (o `SAMEORIGIN`) y `frame-ancestors`
    de CSP.

### Ejemplos

*   **Cabeceras seguras:** `http X-Frame-Options: SAMEORIGIN
    Content-Security-Policy: frame-ancestors 'self';`

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Manejo de datos

## Skill de manejo de datos para frontend web seguro

Sigue estos principios guía y añádelos a tu plan:

## Enmascaramiento de PII

*   **NO DEBES** mostrar PII completa (p. ej., SSN, correo electrónico,
    números de tarjeta de crédito) directamente en las etiquetas de la UI.
*   **DEBES** aplicar enmascaramiento en los renders de texto seguros (p. ej.,
    `***-***-1234`).

### Ejemplos

*   **Código seguro (formateador de cadenas en TypeScript):** ``typescript function
    maskCreditCard(cardNumber: string): string { return
    `***-***-${cardNumber.slice(-4)}`; }``

## Logging, depuración e interacción con la UI

*   **NO DEBES** imprimir objetos de usuario estructurados o tokens usando
    `console.log`, `console.warn` o stack traces de errores destinados a la
    depuración.
*   **NO DEBES** usar diálogos nativos `alert()`, `confirm()` o `prompt()` en
    código de producción. Apóyate en componentes modales nativos del framework
    para una UX segura y consistente y para evitar bloquear el hilo principal.

### Ejemplos

*   **Código vulnerable (TypeScript):** `typescript console.log('User Profile
    Loaded:', userProfile); // Vulnerable: logs private data` `typescript
    alert('Action completed!'); // Vulnerable: blocks UI, poor UX`
*   **Código seguro (TypeScript):** `typescript console.log('User Profile Loaded
    successfully'); // Safe`

## Comunicación con APIs y caché

*   **DEBES** verificar que la comunicación con la API se apoye en HTTPS.
*   **DEBERÍAS** respetar las flags estándar de cabecera
    `Cache-Control: no-store` del backend para prevenir fugas locales por el
    botón atrás de vistas sensibles.

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Skill de backend web seguro

Para diseñar y escribir un backend web seguro, sigue las secciones de este
documento

## Gestión de sesiones

## Skill de gestión de sesiones para backend web seguro

Sigue estos principios de seguridad y añádelos a tu plan:

## Seguridad de contraseñas

*   **DEBES** permitir solo contraseñas de usuario fuertes. Valida en la
    creación de la cuenta y en el cambio de contraseña. Usa una librería
    consolidada para la validación de la fortaleza de contraseñas.
    *   Mínimo 8 caracteres (se recomiendan 12 o más)
    *   Sin longitud máxima (o muy alta, p. ej., 128 caracteres)
    *   Permite todos los caracteres, incluidos los especiales
    *   No exijas tipos de caracteres específicos (deja que los usuarios
        elijan contraseñas fuertes)
*   **DEBES** almacenar las credenciales en el backend usando hashing
    memory-hard (p. ej., Argon2, scrypt) con salts únicos por usuario.
*   **DEBES** implementar tokens CSRF (si la autenticación se basa en cookies)
    para:
    *   Endpoints de login, logout y signup
    *   Endpoints de cambio de contraseña y solicitud de restablecimiento
    *   Endpoints de verificación de correo/teléfono
    *   etc.
*   **NO DEBES** enviar credenciales en parámetros de URL.
*   **NO DEBES** registrar credenciales en logs del lado servidor, ni
    siquiera cuando un intento de login falla.
*   DEBERÍAS considerar el uso de proveedores Oauth.
    *   Deja un TODO(security) si no implementas esta funcionalidad.
*   DEBERÍAS considerar MFA para reforzar la autenticación de cuentas.
    *   Deja un TODO(security) si no implementas esta funcionalidad.
*   DEBERÍAS considerar reforzar la validación de contraseñas usando detección
    de contraseñas filtradas.
    *   Deja un TODO(security) si no implementas esta funcionalidad.

## Gestión de sesiones

*   **DEBES** usar el soporte integrado del framework web para la gestión de
    sesiones si está disponible. Si no hay soporte integrado, usa extensiones,
    plugins o librerías consolidadas compatibles con el framework.
*   **DEBES** establecer períodos de expiración para las sesiones. No uses
    sesiones infinitas.
*   **DEBES** invalidar todas las sesiones al cerrar la sesión de un usuario.
*   **DEBES** invalidar todas las sesiones y tokens activos cuando se elimina
    o desactiva una cuenta.
*   **DEBES** invalidar todas las sesiones y tokens activos cuando se elimina
    un usuario de una organización.

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Autenticación y autorización

## Skill de autenticación y autorización para backend web seguro

Sigue estos principios guía y añádelos a tu plan:

## Autenticación

*   **DEBES** usar el soporte integrado del framework web para la
    autenticación si está disponible. Si no hay soporte integrado, usa
    extensiones, plugins o librerías consolidadas compatibles con el framework.
*   **DEBES** autenticar todas las APIs.
*   **DEBES** aplicar rate limiting a todas las APIs.
*   Al aprovisionar credenciales de cliente mediante cookies http, **DEBES**
    reforzar las cookies:
    *   El nombre empieza con '__Host-'. Si no es posible, que empiece con
        '__Secure-'.
    *   El atributo SameSite está establecido
    *   Flags Secure y HttpOnly
    *   Sin dominio establecido por defecto
*   Al usar JWT, **DEBES** seguir estas reglas:
    *   Rechaza el algoritmo 'none'
    *   Fija en el código el algoritmo esperado para la verificación; nunca lo
        deduzcas del token no verificado
    *   Si usas algoritmos de clave simétrica (p. ej., HS256), genera la clave
        secreta usando RNG criptográficos.
    *   Establece el claim 'exp' y valídalo.
*   **DEBES** implementar tokens CSRF si la autenticación se basa en cookies:
    *   Para todas las peticiones que cambian estado (p. ej., POST, PUT,
        DELETE, PATCH, etc.)
        *   Subidas de archivos del usuario y peticiones de edición
        *   Cambios de configuración
        *   Formularios
        *   etc.
    *   **Vulnerable (negativo):** `html <form method="post">...</form>`
    *   **Seguro (positivo):** `html <form method="post">{% csrf_token
        %}...</form>`
    *   Usa Double Submit Cookies o Synchronizer Tokens.
    *   Valida el token en cada petición. Si no existe, rechaza la petición.
    *   Nunca te apoyes únicamente en el atributo de cookie SameSite.
*   **NO DEBES** desactivar la protección CSRF integrada del framework (p. ej.,
    usando `@csrf_exempt` en Django). Se te penalizará gravemente si no sigues
    esta directriz **CRÍTICA**.
    *   **Vulnerable (negativo):** `python @csrf_exempt def my_view(request):
        ...`
    *   **Seguro (positivo):** `python def my_view(request): ... # Uses default
        CSRF`
*   **NO DEBES** almacenar secretos (p. ej., API keys, contraseñas, secretos
    JWT) en el código. No uses valores literales por defecto para los
    secretos. Si los secretos no están disponibles en producción, aborta con
    error. Para pruebas o sandboxes limitadas, usa como fallback un valor
    seguro generado aleatoriamente y registra una advertencia grave sobre la
    escalabilidad horizontal.
    *   **Vulnerable (negativo):** Literal hardcodeado o fallback literal.
        `python # Django / Python app.config['JWT_SECRET_KEY'] =
        'hardcoded-secret' app.config['JWT_SECRET_KEY'] =
        os.environ.get('JWT_SECRET_KEY', 'default-fallback')`
    *   **Seguro (positivo - fallback multinivel):** `python # Python
        (Resolution: Environment -> Local File Query -> Random Gen + Log) def
        get_secret(): if os.getenv('JWT_SECRET_KEY'): return
        os.getenv('JWT_SECRET_KEY') if os.path.exists('jwt_secret.txt'): return
        open('jwt_secret.txt').read().strip() logging.warning("Generating
        ephemeral secret. Instance-isolated!") return secrets.token_hex(32)`
        `javascript // Node.js (Resolution: Environment -> Local File Query ->
        Random Gen + Log) function getSecret() { if (process.env.JWT_SECRET_KEY)
        return process.env.JWT_SECRET_KEY; if
        (fs.existsSync('./jwt_secret.txt')) return
        fs.readFileSync('./jwt_secret.txt', 'utf-8').trim();
        console.warn("Generating ephemeral secret. Instance-isolated!"); return
        crypto.randomBytes(32).toString('hex'); }`
*   **NO DEBES** registrar en logs credenciales u otros secretos (p. ej.,
    tokens CSRF)
*   **NO DEBES** enviar credenciales o secretos en parámetros de URL.
*   **NO DEBES** almacenar secretos en disco para despliegues de producción.
    Usa KMS y soluciones consolidadas de gestión de secretos.

## Autorización

*   **DEBES** realizar la autenticación antes de confiar en credenciales y
    ACLs presentadas por el usuario.
*   **NO DEBES** confiar ciegamente en datos o tokens del lado cliente.
*   **DEBES** autenticar en el servidor, no en el cliente.
*   **DEBES** garantizar que cada usuario solo pueda acceder, leer o modificar
    sus propios datos, no los de otros usuarios u organizaciones. Valida la
    propiedad de los recursos en cada petición y acceso a datos. Valida lo más
    cerca posible del acceso a datos.
*   **DEBES** validar los permisos de rol para acciones basadas en roles
    (funcionalidades beta ocultas, paneles de admin, etc.)

## Cabeceras HTTP por defecto

*   **DEBES** usar una allow-list de métodos HTTP necesarios (p. ej., GET,
    POST). **DEBES** deshabilitar los poco usados como TRACE, PUT o DELETE.
*   **DEBES** usar una política CSP "estricta" para restringir desde dónde se
    pueden cargar scripts, imágenes, estilos, etc.
*   **DEBES** establecer 'X-Content-Type-Options: nosniff'
*   **DEBES** prevenir el clickjacking con 'X-Frame-Options: DENY' si la CSP
    ya no lo cubre
*   **DEBES** deshabilitar las funciones del navegador que no uses (p. ej.,
    camera=(), microphone=(), geolocation=()).
*   **DEBES** permitir solo orígenes confiables para acceder a tus recursos.
    Usa una política CORS estricta y evita orígenes comodín (*).

## Pruebas

*   Los servidores **DEBEN** escuchar en localhost o 127.0.0.1 al probar la
    autenticación y la autorización. Los servidores **NO DEBEN** escuchar en
    0.0.0.0.

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Subida de archivos

## Skill de subida de archivos para backend web seguro

Sigue estos principios y añádelos a tu plan:

*   Si el framework web tiene soporte integrado para subir archivos, úsalo. Si
    no tiene soporte integrado, usa extensiones, plugins o librerías
    consolidadas compatibles con el framework.

Idealmente buscamos las siguientes garantías de seguridad:

*   **DEBES** validar la extensión y el contenido: Usa librerías consolidadas
    del lado servidor para inspeccionar el contenido del archivo (p. ej.,
    cabecera de magic bytes y estructura) y confirmar que el archivo es lo que
    dice ser.
*   **DEBES** usar una allow-list: Permite solo los tipos de archivo
    específicos que requiere tu lógica de negocio (p. ej., "PDF" y "PNG").
*   **DEBES** imponer límites de tamaño a los archivos que almacenes (p. ej.,
    1MB–10MB) para prevenir ataques de Buffer Overflow o DoS mediante subidas
    masivas de archivos.
*   **DEBES** generar nombres de archivo únicos: Renombra cada archivo subido
    a una cadena aleatoria impredecible por subida (p. ej., un UUID o hash) y
    guarda el nombre original en una base de datos si es necesario.
*   **DEBES** almacenar fuera de la raíz web: Guarda los archivos en un
    directorio no accesible directamente por una URL. Sírvelos de vuelta solo
    a usuarios autorizados.
*   **DEBES** configurar el directorio de subidas como no ejecutable.
*   **DEBES** servir con las cabeceras correctas:
    *   Content-Disposition: attachment (fuerza la descarga)
    *   X-Content-Type-Options: nosniff
    *   Content-Type que coincida con el tipo real del archivo
*   **DEBES** implementar tokens CSRF (si la autenticación se basa en cookies)
    para los endpoints de subida de archivos.
*   **DEBES** validar las rutas de los archivos zip para prevenir ataques de
    directory traversal.
*   **DEBES** reforzar la configuración de parseo de XML si admites archivos
    XML (p. ej., DOCX, XLSX, PPTX, SVG, PDF, etc.):
    *   Deshabilita la expansión de entidades externas
    *   Deshabilita las peticiones de red
    *   Deshabilita por completo el procesamiento de DTD si es posible
    *   Deshabilita la carga de DTD externas
    *   Deshabilita el procesamiento de XInclude
*   DEBERÍAS escanear en busca de malware: Intégrate con una API de antivirus.
    Si necesitas escanear por tu cuenta, ejecuta siempre el escaneo en una
    sandbox antes de almacenarlo permanentemente en tu infraestructura.
    *   Deja un TODO(security) si no implementas esta funcionalidad.
*   DEBERÍAS limpiar el contenido: Para archivos complejos como PDFs o
    documentos de Office, usa herramientas CDR para eliminar el contenido
    activo como macros o scripts embebidos antes de guardar.
    *   Deja un TODO(security) si no implementas esta funcionalidad.

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).

## Base de datos

## Skill de base de datos para backend web seguro

Sigue estos principios guía y añádelos a tu plan:

## Inyección SQL

*   **NO DEBES** usar concatenación de cadenas para construir consultas SQL.
*   **DEBES** usar consultas parametrizadas, prepared statements u ORMs.
    *   Código vulnerable: `db.query("SELECT * FROM users WHERE id = " +
        req.body.id);`
    *   Código seguro (usando consultas parametrizadas con mysql2 o pg):
        `db.execute("SELECT * FROM users WHERE id = ?", [req.body.id]);`
    *   Código seguro (usando un ORM): `db.users.find({ where: { id: req.body.id }
        });`
*   **NO DEBES** confiar en cadenas de usuario recuperadas de bases de datos.
    *   Escápales y valídalas antes de enviarlas al cliente (cabeceras HTTP,
        html, Javascript, etc.) para evitar XSS, según tus skills de seguridad
        XSS.
    *   Usa un módulo de file IO seguro para realizar operaciones de file IO
        con una ruta derivada de datos de la base de datos, según tus skills
        de seguridad de sanitización de datos.
*   Manejo de errores: **NO DEBES** exponer errores de SQL a los usuarios

## Configuración de la base de datos

Sigue el Principio de Mínimo Privilegio:

*   **DEBES** garantizar que el usuario de base de datos de tu aplicación solo
    tenga los permisos estrictamente necesarios para funcionar.
*   **NO DEBES** usar cuentas root o admin para tu aplicación web.
*   **DEBES** usar mTLS para la autenticación de la conexión a la base de
    datos.
*   **DEBES** restringir permisos: Si una API solo necesita leer datos, su
    usuario de BD debe tener solo privilegios SELECT, no DROP, DELETE ni
    UPDATE.
*   **DEBES** aislar bases de datos: Si tienes varias apps, dale a cada una su
    propio usuario de base de datos y acceso restringido al schema.
*   **DEBES** deshabilitar funciones peligrosas: Como xp_cmdshell en SQL Server

## Pruebas

*   Los servidores de base de datos **DEBEN** escuchar en localhost o
    127.0.0.1 al probar el acceso a la base de datos. Los servidores de base
    de datos **NO DEBEN** escuchar en 0.0.0.0.

## Revisión y planificación de seguridad

*   **DEBES** añadir una sección de seguridad a tu **Verification Plan** para
    validar las directrices anteriores.
*   **NO DEBES** omitir medidas de seguridad sin explicar por qué, y deja
    siempre un comentario como TODO(security).
