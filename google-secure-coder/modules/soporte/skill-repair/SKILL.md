---
name: skill-repair
description: |
  Úsalo para corregir y reinstalar agent skills cuya instalación haya fallado.
  Esta skill proporciona el contexto y los permisos necesarios para actualizar
  quirúrgicamente el `manifest.json` después de aplicar una corrección.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Asistente de reparación de skills

Se te ha encomendado reparar una agent skill dañada. Después de modificar los
archivos fuente de la skill para resolver el error reportado, DEBES actualizar
el `manifest.json` para reflejar que la skill quedó reparada.

## Contexto de la skill

-   **Skill ID**: el identificador único de la skill (p. ej., `my-skill`).
-   **Source Path**: dónde se encuentran los archivos fuente de la skill.
-   **Installed Path**: dónde está instalada/replicada la skill.
-   **Manifest Path**: la ruta absoluta al archivo `manifest.json`.

## Procedimiento de reparación

1.  **Analiza el error**: comprende el mensaje de error proporcionado en el
    prompt.
2.  **Corrige el Installed Path**: corrige el problema en la ruta instalada.
    Como algunas skills tienen varios archivos, DEBES listar todos los archivos
    del directorio de la skill y analizarlos en conjunto para encontrar la causa
    raíz (p. ej., un `SKILL.md` mal formado, recursos faltantes o sub-scripts
    incorrectos).
3.  **Actualiza el manifest**: una vez aplicada la corrección a TODOS los
    archivos relevantes, DEBES actualizar el `manifest.json` en la
    **Manifest Path**.
    -   Busca la entrada del skill ID en el objeto `skills`.
    -   Establece `"status": "installed"`.
    -   Limpia el campo `"error"` (establecerlo en `null` o eliminarlo).
4.  **Verificación**: la UI detectará este cambio automáticamente y se
    refrescará.

### Ejemplo de manifest

```json
{
  "skills": {
    "my-skill": {
      "status": "installed",
      "disabled": false,
      "error": null
    }
  }
}
```
