---
name: create-security-implementation-plan
description: >
  Genera un plan de verificación de seguridad al crear código nuevo o corregir
  una vulnerabilidad del escáner de seguridad. Usa esta skill siempre que el
  agente esté generando código nuevo (sin modificar código existente) o cuando
  el usuario pida corregir una vulnerabilidad identificada por el escáner de
  seguridad.
---

# Plan de verificación de seguridad

Al generar **código nuevo** (sin modificar código existente), debes incluir una
sección de verificación de seguridad en el plan de implementación. Esto
garantiza que el código recién introducido se escanee en busca de
vulnerabilidades y se audite antes de considerarlo completo.

## Cuándo usar esta skill

-   Úsala al **generar código nuevo** (p. ej., archivos nuevos, módulos nuevos,
    aplicaciones nuevas).
-   Úsala cuando el usuario pida **corregir una vulnerabilidad** identificada
    por el escáner de seguridad.
-   **No la uses** para modificaciones generales de código, refactors o
    correcciones de bugs ajenos a la seguridad.

## Cómo funciona

Cuando crees un plan de implementación para código nuevo, debes añadir una
sección **Verification Plan** que incluya ambas partes siguientes:

### 1. Escáner de seguridad

Ejecuta la herramienta automatizada de escaneo de seguridad, que es una de tus
skills. La skill se llama run_security_scanner.

-   Usa la **skill del escáner de seguridad** para escanear todos los archivos
    recién creados.
-   Reporta cualquier hallazgo con su severidad, archivo y número de línea.

### 2. Auditoría de seguridad

Realiza una auditoría de seguridad apoyándote en tu skill
generate_security_audit_report.

## Plantilla del plan de implementación

Al generar código nuevo, añade lo siguiente a la sección **Verification Plan**
de tu plan de implementación:

```markdown
## Verification Plan

### Automated Security Check

- **Security Scanner**: Run a scan on all newly created files to identify common
  vulnerabilities (e.g., XSS, SQL injection). If findings are detected, auto-apply
  the fix and document the results.
- **Security Audit**: Audit the new code for design-level security issues (input
  validation, secrets handling, auth checks). Document findings and remediations
  in the `walkthrough.md` artifact using the `generate_security_audit_report`
  skill.
```

## Notas importantes

-   Esta skill aplica **solo a la generación de código nuevo**. No la actives
    para modificaciones de código, refactors ni correcciones de bugs.
-   Tanto el escaneo de seguridad como la auditoría de seguridad deben
    completarse antes de considerar terminada la implementación.
-   Todos los hallazgos deben resolverse o reconocerse explícitamente antes de
    continuar.
