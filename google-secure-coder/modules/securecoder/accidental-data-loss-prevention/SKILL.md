---
name: accidental-data-loss-prevention
description: |
  **DETENTE Y VERIFICA**: Antes de ejecutar cualquier comando o herramienta que
  produzca una pérdida de datos irreversible, DEBES obtener el consentimiento
  explícito del usuario.
  Ante la duda, pregunta. Es mejor esperar la confirmación que eliminar por
  accidente datos de producción o recursos críticos del proyecto.
  Úsala para:
  - SQL: DROP TABLE/VIEW/SCHEMA/DATABASE, TRUNCATE, o DELETE masivo (sin WHERE
    o usando 1=1).
  - Cloud Storage: gsutil rm o gcloud storage rm dirigidos a datos de
    producción o buckets críticos.
  - Infraestructura: gcloud projects delete, eliminación de recursos de
    Spanner/BigQuery/Dataproc, eliminación de secretos o destrucción de claves
    KMS.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Prevención de pérdida accidental de datos

> [!CAUTION]
>
> **DETENTE Y VERIFICA**: Antes de ejecutar cualquier comando o herramienta que
> produzca una pérdida de datos irreversible, **DEBES** obtener el
> consentimiento explícito del usuario.

## Procedimiento obligatorio

1.  **Detén la ejecución**: **No** ejecutes el comando.
2.  **Solicita el consentimiento**: Explica claramente al usuario:
    -   El **impacto** de esta eliminación.
    -   **Por qué** crees que es necesaria.
    -   Una solicitud de su **aprobación explícita** para continuar.
3.  **Espera**: Continúa solo si el usuario otorga un consentimiento claro y
    afirmativo en la conversación.
