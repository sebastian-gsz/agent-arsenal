---
name: standard-operating-procedure
description: Guía paso a paso en 3 fases (Sistema → Proceso → SOP) para crear o actualizar Procedimientos Operativos Estándar (SOP/POE) con dueño, trigger, pasos y escalación, y guardarlos en sops/ del proyecto. Úsala siempre que el usuario quiera documentar un proceso, escribir un SOP, POE, procedimiento, manual de procedimientos, protocolo o instrucción de trabajo, o modificar uno ya existente — aunque no diga "SOP".
---

# Documentar un SOP por Niveles (Sistema → Proceso → SOP)

Convierte un proceso de negocio en un SOP profesional siguiendo la jerarquía de documentación: el SOP vive dentro de un Proceso, y el Proceso dentro de un Sistema.

## Alcance

- Cubre: crear SOPs nuevos y actualizar SOPs generados con esta skill.
- NO cubre: políticas e invariantes del repo (van a `AGENTS.md`/rules, no a una skill), documentación no procedimental (tutoriales, guías de arquitectura) ni auditorías de certificación formal (ISO, etc.).

## Guardrail: fases en orden

No redactes un SOP (Fase 3) sin haber completado antes la Fase 1 (Sistema) y la Fase 2 (Proceso) en esta conversación. Un SOP sin sistema ni proceso queda huérfano: sin dueño que lo mantenga, sin límites de alcance y sin contexto para quien lo ejecute.

Si el usuario pide directamente "escríbeme el SOP de X", explica en una frase por qué necesitas las dos fases anteriores y arranca por la Fase 1. Si el usuario insiste en saltarse las fases, no discutas más: documenta con lo que dé, marcando los campos que falten como `<PENDIENTE>` y avisando de qué quedó pendiente. Las actualizaciones de un SOP ya existente están exentas de este guardrail: el Sistema y el Proceso ya constan en el archivo.

## Inputs

Antes de preguntar nada:

1. Busca en `sops/` de la raíz del proyecto (y en la documentación del repo: README, `docs/`) si ya existe un SOP o material para este proceso.
2. Extrae de la conversación lo que ya ocurrió: si el workflow ya se describió, no lo re-preguntes; confirma solo los huecos.

## Flujo de Trabajo (3 Fases)

### Fase 1 — Sistema
Pregunta al usuario:
> *"¿A qué categoría completa de trabajo o sistema pertenece este proceso?"*
*(Ejemplos: "Gestión de proveedores e inventario", "Atención al cliente", "Onboarding de personal")*

⚠️ **No continúes a la Fase 2 sin una respuesta concreta.**

### Fase 2 — Proceso
Pide la secuencia completa de pasos del proceso a **nivel alto** (visión general sin entrar aún en el detalle del paso a paso).
*(Ejemplo: "Detectar riesgo → Confirmar atraso → Decidir respaldo → Comprar → Notificar")*

De esa secuencia, propone un nombre corto para el Proceso y confírmalo con el usuario (la plantilla necesita ese campo).

⚠️ **Confirma con el usuario que la secuencia esté completa antes de pasar a la Fase 3.**

### Fase 3 — SOP (Detalle)
Selecciona cuál paso específico del Proceso se documentará como SOP (pregúntalo solo si el contexto no lo deja claro) y recolecta la siguiente información **haciendo una sola pregunta a la vez**:

1. **Dueño:** Rol o puesto responsable (ej. *Gerente de Compras*), nunca el nombre de una persona concreta.
2. **Se activa cuando (Trigger):** El desencadenante o umbral exacto. Si la respuesta es vaga (ej. *"cuando sea necesario"*), solicita una condición o cifra concreta.
3. **Pasos:** La secuencia exacta y detallada de acciones a ejecutar, numerada.
4. **Excepción / Escalación:** Cuándo este procedimiento NO aplica y ante quién o cómo hay que escalar.

Los campos **Código**, **Versión** y **Aprobado por** no se preguntan: los propone el agente (ver "Generación del Archivo Final").

## Generación del Archivo Final

Con la información recolectada, genera y guarda un archivo Markdown en `sops/<nombre-del-sop-slug>.md` **de la raíz del proyecto donde trabaja el agente** (crea la carpeta `sops/` si no existe). No lo guardes dentro de la carpeta de la skill: la skill puede estar instalada en modo solo lectura.

Valores derivados:

- **Código:** propón uno corto y estable a partir del sistema/proceso (ej. `SOP-COM-001`); si ya hay SOPs con código, usa el siguiente número libre.
- **Versión:** `1.0` en creación.
- **Aprobado por:** el rol dueño, salvo que el usuario indique otro rol.

### Actualización de un SOP existente

Si el SOP ya existe en `sops/` (o el usuario pide actualizar uno), actualízalo en vez de recrearlo: re-pregunta solo los campos que cambian, conserva el contenido que no cambia y actualiza **Última revisión** con la fecha actual. La **Versión** se incrementa en el número menor (X.0 → X.1) ante cualquier cambio de contenido operativo (trigger, pasos, excepción, dueño, aprobador); para erratas de redacción, mantén la versión.

### Plantilla del SOP:

```markdown
SOP: <Nombre del SOP>
Código: <Código propuesto>
Sistema: <Nombre del Sistema>
Proceso: <Nombre del Proceso>
Dueño: <Rol del Dueño>
Versión: <X.Y>
Aprobado por: <Rol aprobador; por defecto el dueño>
Última revisión: <Fecha actual YYYY-MM-DD>
Se activa cuando: <Trigger exacto>

Pasos:
1. <Paso 1>
2. <Paso 2>
3. <Paso 3>

Excepción / cuándo escalar: <Regla de excepción o escalación>
```

## Checks (antes de guardar)

- Todos los campos de la plantilla están llenos, sin placeholders ni `<PENDIENTE>` (si los hay, avisa al usuario explícitamente).
- El dueño es un rol, no una persona.
- El trigger es una condición o cifra concreta, no una vaguedad ("cuando sea necesario").
- Los pasos los puede ejecutar alguien que no participó en esta conversación.
- La excepción/escalación está presente y nombra ante quién escalar.

## Criterios de éxito

- Archivo creado o actualizado en `sops/` de la raíz del proyecto, con confirmación al usuario y la ruta exacta.
- La checklist de Checks pasa.

## Stop conditions

- Datos imposibles de concretar tras 2 intentos (no hay dueño claro, el trigger sigue vago) → pausa y pide la decisión al usuario.
- No se puede escribir en `sops/` → informa al usuario y pide una ruta alternativa; no la inventes.

## Recovery

- **Secuencia incompleta en Fase 2** → pregunta explícitamente por el paso anterior, el siguiente y los casos borde (¿y si falla?, ¿y si nadie responde?).
- **Trigger vago tras re-preguntar** → propone 2–3 umbrales de ejemplo para que elija (ej. "atraso > 15 días", "deuda > 5.000 €").
- **SOP existente con formato distinto a la plantilla** → migra los campos al formato sin borrar contenido; los campos que falten se tratan como actualización normal.
- Tras cada corrección, vuelve a pasar la checklist de Checks.
