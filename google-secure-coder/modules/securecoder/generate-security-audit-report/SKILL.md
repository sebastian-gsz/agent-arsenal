---
name: generate-security-audit-report
description: >
  Genera un reporte de auditoría de seguridad como artifact de walkthrough
  tras completar la generación de código. Esta skill documenta todas las
  vulnerabilidades que se detectaron y corrigieron durante la sesión,
  proporcionando a los usuarios una visión general completa de la seguridad.
---

# Generar reporte de auditoría de seguridad

Tras completar la generación de código, produce un artifact de walkthrough de
**Auditoría de seguridad** que documente cada vulnerabilidad detectada y
remediada durante la sesión. Esto da a los usuarios visibilidad total de la
postura de seguridad del código generado.

## Cuándo usar esta skill

-   Usa esta skill al **final de una tarea de generación de código**, cuando
    todo el escaneo y la corrección de seguridad se hayan completado.
-   Usa esta skill cuando el plan de verificación de la skill
    `create_security_implementation_plan` se haya ejecutado y se hayan
    encontrado vulnerabilidades.
-   Usa esta skill siempre que `run_security_scanner` se haya invocado durante
    la sesión y haya producido hallazgos.
-   **No la uses** si no se realizó ningún escaneo de seguridad o no se generó
    código.

## Requisitos previos

Antes de generar este reporte debes haber:

1.  Ejecutado la skill `run_security_scanner` sobre todos los archivos
    generados.
2.  Aplicado correcciones para las vulnerabilidades detectadas usando
    herramientas de edición de archivos o remediación manual.
3.  Re-escaneado los archivos corregidos para confirmar que las
    vulnerabilidades están resueltas.

## Paso 1: Recopilar los datos del escaneo

Recoge lo siguiente de la sesión:

-   **Resultados iniciales del escaneo**: La salida JSON de
    `run_security_scanner` antes de cualquier corrección.
-   **Acciones de remediación**: Qué se cambió y en qué archivos.
-   **Resultados del escaneo posterior a las correcciones**: La salida JSON de
    `run_security_scanner` después de aplicar las correcciones.
-   **Hallazgos ignorados**: Obtén la lista de supresiones actual desde la API
    local:

    ```bash
    curl -s http://127.0.0.1:$PORT/ignored
    ```

    Cada `IgnoredEntry` tiene `ruleId`, `filePath`, `reason` y `timestamp`.
    Mapea el `reason` a un estado del reporte usando la tabla siguiente.

## Paso 2: Generar el artifact de walkthrough

Crea o actualiza el artifact `walkthrough.md` con una sección **SecureCoder
Security Audit**. El reporte debe incluir las secciones siguientes.

> [!NOTE] Si la skill `run_poc` también se usó durante esta sesión, ya
> existirá una sección `## PoC Verification` en `walkthrough.md`. Escribe la
> sección **SecureCoder Security Audit** **antes** de esa sección, para que la
> evidencia posterior a las correcciones fluya de forma natural: Auditoría →
> Verificación PoC. De manera similar, si se usó `determine_threat_model`, la
> sección `## Security Threat Model` vive en `implementation_plan.md` — no
> necesitas copiarla aquí, solo referenciarla como contexto previo.

### Encabezado

```markdown
# SecureCoder Security Audit

**Status**: Completed
**Scanned Files**: <count>
**Vulnerabilities Found**: <count>
**Vulnerabilities Fixed**: <count>
```

> [!TIP] SecureCoder escanea automáticamente tu directorio de trabajo en busca
> de fallas de seguridad casi en tiempo real, ejecutando
> `run_security_scanner` en segundo plano y generando dinámicamente
> correcciones dirigidas.

### Reporte de vulnerabilidades

Presenta todos los hallazgos en una tabla con las siguientes columnas:

| Columna          | Descripción                                              |
| ---------------- | -------------------------------------------------------- |
| ID de vulnerabilidad | Un ID único para el hallazgo (p. ej., `CS-XSS-001`)  |
| Archivo          | El archivo que contiene la vulnerabilidad                |
| Línea            | El número de línea donde ocurre la vulnerabilidad        |
| Descripción      | Una descripción clara y legible de la vulnerabilidad     |
:                  : que incluya el vector de ataque, el patrón de código     :
:                  : afectado y por qué es peligrosa                          :
| Severidad        | La severidad de la salida del escaneo: `High`, `Medium`  |
:                  : o `Low`                                                  :
| Estado           | `Fixed` u `Open`                                         |
| Remediación      | Una explicación concisa de la corrección aplicada        |

Mapea los valores de severidad del escáner al reporte así:

Severidad del escáner | Severidad en el reporte
--------------------- | -----------------------
`ERROR`               | `High`
`WARNING`             | `Medium`
`INFO`                | `Low`

Valores de la columna **Estado** y su origen:

Estado           | Cuándo usarlo
---------------- | -----------------------------------------------------
`Fixed`          | La vulnerabilidad se corrigió durante la sesión
`Open`           | Vulnerabilidad detectada pero aún no atendida
`False Positive` | Entrada ignorada con reason que contiene "False Positive"
`Accepted Risk`  | Entrada ignorada con reason que contiene "Accepted Risk"
`Won't Fix`      | Entrada ignorada con reason que contiene "Won't Fix"
`Suppressed`     | Entrada ignorada con cualquier otro reason

#### Formato del ID de vulnerabilidad

Usa el formato `CS-<CLASS>-<NNN>` donde:

-   `CS` = prefijo de SecureCoder
-   `<CLASS>` = Código corto de la clase de vulnerabilidad (p. ej., `XSS`,
    `SQLI`, `EXEC`, `SECRETS`, `PATH`)
-   `<NNN>` = Número secuencial, rellenado con ceros hasta 3 dígitos

#### Tabla de ejemplo

```markdown
| Vulnerability ID | File | Line | Description | Severity | Status | Remediation |
|---|---|---|---|---|---|---|
| CS-XSS-001 | app.js | 10 | Cross-Site Scripting (XSS) via `innerHTML`. The unsanitized `city` input string was being rendered directly into the DOM inside the `.error` element. This could permit arbitrary JavaScript execution if malicious payload strings were used as the search term. | High | Fixed | Replaced `innerHTML` assignment with `textContent`, ensuring that all characters are safely encoded as text literals by the browser before rendering. |
| CS-PATH-001 | routes.js | 42 | Path Traversal via unsanitised `req.params.file`. | Medium | False Positive | The path is validated upstream by a middleware layer not visible in the scanned file. Suppressed via API. |
```

### Resumen de falsos positivos

Después de la tabla principal de vulnerabilidades, añade una sección de
**falsos positivos** si alguna entrada de `GET /ignored` tiene un reason que
mapee a `False Positive`, `Accepted Risk` o `Won't Fix`:

```markdown
## Suppressed Findings

| Finding | File | Reason | Suppressed At |
|---|---|---|---|
| SQL Injection | db.js | Accepted Risk - parameterised query used at call site | 2024-03-20 |
| Path Traversal | routes.js | False Positive - path validated by upstream middleware | 2024-03-20 |
```

Incluye esta sección aunque haya un solo hallazgo suprimido. Si no hay
hallazgos suprimidos, omite esta sección por completo.

## Paso 3: Presentar el reporte al usuario

Al presentar el trabajo completado al usuario:

1.  **En tu mensaje de respuesta**, destaca el impacto de seguridad de
    SecureCoder:

    -   Menciona que SecureCoder estuvo activo durante la generación de
        código.
    -   Indica el número y la severidad de las vulnerabilidades detectadas.
    -   Nombra los archivos específicos y los tipos de vulnerabilidad
        encontrados.
    -   Explica que las correcciones se aplicaron automáticamente antes de que
        el usuario necesitara ejecutar el código.

2.  **Adjunta el artifact de walkthrough** y enlázalo para que el usuario
    pueda abrir y revisar el reporte completo de auditoría de seguridad.
    Asegúrate de que esté formateado correctamente.

### Ejemplo de respuesta

```
I've completed building the initial version of the Weather App!

During the generation process, **SecureCoder** was running actively in the background. It detected two critical Cross-Site Scripting (XSS) vulnerabilities in `app.js`.

I was able to automatically intervene and patch these vulnerabilities before you even needed to run the code.

I've attached the complete **SecureCoder Security Audit** for your review which details exactly what was found and how it was remediated.
```

## Paso 4: Manejar casos límite

| Escenario                           | Acción                                 |
| ----------------------------------- | -------------------------------------- |
| Sin vulnerabilidades encontradas    | Genera igualmente el reporte con un    |
:                                     : estado "No vulnerabilities detected" y :
:                                     : la lista de archivos escaneados. Esto  :
:                                     : confirma que el escaneo se realizó.    :
| Vulnerabilidades encontradas y no corregidas | Márcalas como `Open` en la    |
:                                     : columna Estado y explica por qué no se :
:                                     : corrigieron automáticamente (p. ej.,   :
:                                     : requiere revisión manual, lógica de    :
:                                     : negocio compleja).                     :
| Errores del escáner                 | Incluye los errores del array `errors` |
:                                     : de la salida JSON en una sección       :
:                                     : "Scanner Errors" aparte.               :
| Múltiples pasadas de escaneo        | Documenta cada pasada. Muestra los     |
:                                     : hallazgos iniciales, la corrección     :
:                                     : aplicada y los resultados del          :
:                                     : re-escaneo.                            :
