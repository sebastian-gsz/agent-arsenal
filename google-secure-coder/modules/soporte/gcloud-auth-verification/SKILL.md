---
name: gcloud-auth-verification
description: Guía para identificar y resolver la falta de autenticación de Google
  Cloud y de Application Default Credentials (ADC). Usa esta skill si `gcloud`, `bq`,
  `dataform` o las librerías de Python devuelven errores de autenticación.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Manejo de problemas de autenticación

## Mensajes de error comunes

1.  **CLI de gcloud/bq**:
    -   `ERROR: (bq) You do not currently have an active account selected.`
    -   `No credentialed accounts.`
    -   `Configuration error: No account is currently active.`
2.  **Fallos de ejecución (Python/notebooks)**:
    -   `google.auth.exceptions.DefaultCredentialsError: Could not automatically
        determine credentials.`
    -   `Forbidden: 403 Access Denied` (cuando es claramente un problema de
        autenticación).

## Paso de verificación

Antes de pedirle al usuario que inicie sesión, verifica de forma independiente
el estado de la autenticación ejecutando: `gcloud auth list` * Si la salida
contiene `No credentialed accounts.`, pasa a los pasos de **Acción correctiva**
de abajo. * Si hay una cuenta *en la lista* pero el usuario sigue recibiendo un
error `403 Access Denied`, el problema probablemente sean los **permisos IAM**
(p. ej., roles de BigQuery faltantes) de su cuenta activa, y no una falta de
autenticación. En ese caso, investiga los permisos en lugar de pedirle que
inicie sesión de nuevo.

## Acción correctiva

Cuando se confirme que faltan credenciales, **NO** intentes resolver las
credenciales vía código ni con herramientas alternativas. Las credenciales
deben establecerlas el usuario.

**Detente y pide al usuario que ejecute los siguientes comandos en su terminal:**

1.  **Para autenticar la CLI de gcloud**: `gcloud auth login`

2.  **Para configurar Application Default Credentials (ADC)** (necesario para la
    CLI de BQ Y la mayoría de las librerías/notebooks):
    `gcloud auth application-default login`

## Verificación posterior al login

Después de que el usuario confirme que inició sesión, verifica con:
`gcloud auth list`
Luego continúa con la tarea original.
