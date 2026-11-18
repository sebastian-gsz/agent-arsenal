---
name: run-security-scanner
description: >
  Ejecuta el escáner de seguridad sobre los archivos fuente para detectar
  vulnerabilidades. Usa esta skill para escanear archivos en busca de problemas
  de seguridad comunes como XSS, inyección SQL, secretos hardcodeados y otras
  vulnerabilidades clasificadas por CWE.
---

# Ejecutar el escáner de seguridad

Esta skill ejecuta el backend de escaneo configurado para escanear los
archivos fuente en busca de vulnerabilidades de seguridad. SecureCoder admite
dos backends: **Wiz CLI** (principal) y un **escáner integrado** (fallback).
El backend activo lo determina la configuración `securecoder.scannerBackend`,
que se establece durante el onboarding.

## Paso 0: Descubrir el puerto de la API

La extensión SecureCoder escribe el puerto de la API en un archivo sidecar de
ubicación conocida en cada activación. Este es el mecanismo de descubrimiento
**principal** porque la terminal del agente es una PTY creada fuera de la
infraestructura de terminales de VS Code y no recibe las mutaciones de
`environmentVariableCollection`.

```bash
# Primary: read from the sidecar file
if [ -f "$HOME/.securecoder/api.json" ]; then
  PORT=$(cat "$HOME/.securecoder/api.json" | grep -o '"port":[0-9]*' | grep -o '[0-9]*')
fi

# Fallback: check the env var (works in VS Code integrated terminals)
if [ -z "$PORT" ] && [ -n "$SECURECODER_API_PORT" ]; then
  PORT=$SECURECODER_API_PORT
fi
```

Si `PORT` sigue vacío después de ambas comprobaciones, la extensión no está en
ejecución. **Omite esta skill por completo** -- no intentes ejecutar el
escáner manualmente ni se lo preguntes al usuario.

Verifica que el servidor sea alcanzable:

```bash
curl -s http://127.0.0.1:$PORT/config
```

Si esto devuelve una respuesta JSON, continúa con el Paso 1.

--------------------------------------------------------------------------------

## Paso 1: Determinar el backend de escaneo activo

```bash
curl -s http://127.0.0.1:$PORT/config
```

Respuesta:

```json
{"scannerBackend": "wiz"}
```

Valor de `scannerBackend` | Significado
------------------------- | ---------------------------------
`"wiz"`                   | Wiz CLI es el backend activo
`"semgrep"`               | El escáner integrado está activo
`null`                    | Sin escáner configurado — omite el escaneo

Si `scannerBackend` es `null`, omite los pasos restantes.

--------------------------------------------------------------------------------

## Paso 2: Ejecutar el escaneo

```bash
echo "Requesting SecureCoder security scan for <absolute_path_to_file>..."
curl -s -X POST http://127.0.0.1:$PORT/scan \
  -H 'Content-Type: application/json' \
  -d "{\"filePath\": \"<absolute_path_to_file>\"}"
```

Reemplaza `<absolute_path_to_file>` por la ruta absoluta del archivo a
verificar (p. ej., `/Users/you/project/routes/media.js`).

La extensión ejecuta internamente el escáner configurado y devuelve una
respuesta normalizada — sin resolución de rutas de binarios, detección de
plataforma ni parseo de salida específico del backend.

--------------------------------------------------------------------------------

## Paso 3: Parsear la respuesta

El endpoint `/scan` siempre devuelve la misma forma de `ScanResponse`
independientemente de qué backend (Wiz o escáner integrado) esté activo:

```json
{
  "findings": [
    {
      "subcategory": "<rule_id>",
      "message": "<human-readable description>",
      "location": {
        "path": "<absolute_file_path>",
        "range": {
          "textRange": {
            "startLine": 42,
            "startColumn": 4,
            "endLine": 42,
            "endColumn": 30
          }
        }
      },
      "labels": {
        "severity": "HIGH",
        "cwe": "CWE-22",
        "category": "security",
        "vulnerability_class": "Path Traversal"
      }
    }
  ],
  "errors": []
}
```

Campos clave:

| Campo                                | Descripción                        |
| ------------------------------------ | ---------------------------------- |
| `subcategory`                        | Identificador estable de la regla  |
:                                      : del hallazgo (uso interno; no lo   :
:                                      : muestres al usuario)               :
| `message`                            | Descripción legible de la          |
:                                      : vulnerabilidad                     :
| `location.path`                      | Ruta absoluta del archivo afectado |
| `location.range.textRange.startLine` | Número de línea (índice desde 1)   |
| `labels.severity`                    | `HIGH`, `MEDIUM` o `LOW`           |
| `labels.cwe`                         | Identificadores CWE, separados por |
:                                      : comas (p. ej. `CWE-22,CWE-73`)     :

--------------------------------------------------------------------------------

## Paso 4: Reportar los hallazgos

Para cada elemento de `findings`, reporta:

```
**[SEVERITY] [vulnerability_class]**
File: <path>, Lines: <startLine>-<endLine>
CWE: <cwe>
Description: <message>
```

Usa `labels.vulnerability_class` (p. ej., `Path Traversal`, `XSS`,
`SQL Injection`) como etiqueta `[vulnerability_class]`. **No** incluyas el ID
crudo de la regla ni el nombre del backend de escaneo en el reporte — céntrate
en el hallazgo en sí.

Tras enumerar todos los hallazgos, proporciona un resumen: número total de
hallazgos, desglose por severidad y lista de categorías CWE distintas.

-   Si `findings` está vacío y `errors` está vacío → reporta que el escaneo se
    completó sin vulnerabilidades detectadas.
-   Si `errors` no está vacío → reporta los errores e indica que el escaneo
    puede estar incompleto.

--------------------------------------------------------------------------------

## Paso 5: Ignorar una vulnerabilidad (supresión programática)

Si el escáner señala algo que es un **falso positivo** o que el usuario aceptó
explícitamente, el agente puede suprimirlo **silenciosamente** vía la API
local — no se muestra ninguna UI de VS Code al usuario.

### Marcar un hallazgo como ignorado

Usa `codeSnippet` (el texto exacto de la línea señalada) en lugar de
`lineNumber`. El servidor encuentra la línea coincidente por contenido
recortado, por lo que las diferencias de espacios al inicio/final se ignoran.
Esto hace que la supresión sea inmune a los desplazamientos de números de
línea causados por ediciones en otras partes del archivo.

```bash
echo "Suppressing finding in <absolute_path_to_file>..."
curl -s -X POST http://127.0.0.1:$PORT/ignore \
  -H 'Content-Type: application/json' \
  -d '{
    "filePath": "<absolute_path_to_file>",
    "ruleId": "<subcategory_from_finding>",
    "codeSnippet": "<exact_text_of_flagged_line>",
    "vulnerabilityClass": "<vulnerability_class_from_labels>",
    "reason": "False Positive"
  }'
```

Usa `content_at_line` del prompt del hallazgo (o
`location.range.textRange.startLine` de la respuesta de `/scan`) para obtener
el texto de la línea señalada y usarlo como `codeSnippet`.

Si el mismo texto de línea aparece varias veces en el archivo, pasa también
`lineNumber` como pista — el servidor elegirá la ocurrencia más cercana a esa
línea:

```bash
-d '{
    "filePath": "...",
    "ruleId": "...",
    "codeSnippet": "<flagged_line_text>",
    "lineNumber": 42,
    "reason": "False Positive"
  }'
```

**Campos de la petición:**

| Campo                | Tipo   | Requerido | Descripción                  |
| -------------------- | ------ | --------- | ---------------------------- |
| `filePath`           | string | sí        | Ruta absoluta del            |
:                      :        :           : archivo afectado             :
| `ruleId`             | string | sí        | ID de la regla / clase de    |
:                      :        :           : vulnerabilidad del escaneo   :
| `codeSnippet`        | string | sí*       | Texto exacto de la línea     |
:                      :        :           : señalada (recortado para     :
:                      :        :           : coincidir)                   :
| `lineNumber`         | number | sí*       | Número de línea (índice      |
:                      :        :           : desde 1) — úsalo cuando      :
:                      :        :           : `codeSnippet` esté ausente   :
:                      :        :           : o como pista de              :
:                      :        :           : desambiguación               :
| `vulnerabilityClass` | string | no        | `labels.vulnerability_class` |
:                      :        :           : del hallazgo                 :
| `reason`             | string | no        | Razón legible para humanos   |
:                      :        :           : (por defecto\: `"Suppressed  :
:                      :        :           : via API"`)                   :

\* Proporciona al menos uno de `codeSnippet` o `lineNumber`. Se prefiere
`codeSnippet`.

**Valores de `reason` recomendados:**

Valor              | Significado
------------------ | ------------------
`"False Positive"` | Error del escáner
`"Accepted Risk"`  | Conocido y aceptado
`"Won't Fix"`      | No priorizado

**Respuesta en caso de éxito:**

```json
{
  "success": true,
  "vulnId": "/path/to/file.js:41:injection/sql",
  "contentHash": "3a7b9f..."
}
```

El registro de supresión se indexa por el **hash de contenido del texto de
línea recortado**, por lo que sobrevive a cualquier edición posterior que solo
desplace líneas cercanas. El dashboard y CodeLens se actualizan
automáticamente tras llamar a este endpoint — no se requiere ningún paso
adicional.

### Inspeccionar la lista de ignorados actual

```bash
curl -s http://127.0.0.1:$PORT/ignored
```

```json
{
  "entries": [
    {
      "vulnId":      "/path/to/file.js:41:injection/sql",
      "ruleId":      "injection/sql",
      "filePath":    "/path/to/file.js",
      "contentHash": "3a7b9f...",
      "reason":      "False Positive",
      "timestamp":   1711000000000
    }
  ]
}
```

Usa `GET /ignored` en el reporte de auditoría de seguridad para identificar
qué hallazgos fueron suprimidos como falsos positivos, e inclúyelos en la
sección de **falsos positivos** del reporte.

--------------------------------------------------------------------------------

## Solución de problemas

| Problema                            | Solución                               |
| ----------------------------------- | -------------------------------------- |
| `SECURECODER_API_PORT` sin definir  | La extensión no está activa, o esta    |
:                                     : terminal se abrió antes de la          :
:                                     : activación — abre una nueva terminal   :
| Conexión rechazada en el puerto     | La extensión puede haberse reiniciado  |
:                                     : con un nuevo puerto — abre una nueva   :
:                                     : terminal para obtener la variable de   :
:                                     : entorno actualizada                    :
| `scannerBackend: null`              | Sin escáner configurado — dirige al    |
:                                     : usuario a ejecutar `SecureCoder\:      :
:                                     : Select Security Scanner` desde la      :
:                                     : paleta de comandos (Command Palette)   :
| `errors` no vacío en la respuesta del escaneo | Consulta el canal de salida  |
:                                     : de SecureCoder en VS Code para más     :
:                                     : detalles                               :
