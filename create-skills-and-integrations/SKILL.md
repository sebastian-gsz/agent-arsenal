---
name: create-skills-and-integrations
description: Crea skills nuevas siguiendo las mejores prácticas de autoría y conecta agentes, subagentes, servidores MCP o plugins al workflow. Úsala siempre que el usuario quiera crear una skill, convertir un workflow repetido en una skill reutilizable, mejorar el disparo de una skill o conectar un MCP server / plugin / subagente al servicio de un workflow — incluso si solo dice "hazme una skill", "automatiza esto", "conecta este MCP", "añade este plugin" o "que el agente use X".
---

# Crear skills con integraciones (MCP, plugins, agentes, commands)

Convierte un workflow repetido en una skill bien escrita y, cuando aporte valor, conecta a su servicio servidores MCP, plugins, commands o subagentes. Basada en las reglas de "Agent Skills vs. Rules vs. Commands" 

## Alcance

- Crear skills nuevas y mejorar existentes: description de routing, cuerpo procedimental, progressive disclosure.
- Decidir el vehículo correcto ANTES de crear: rule, command, skill o agente separado.
- Integrar herramientas al servicio de la skill: MCP, plugins, subagentes, commands.

NO aplica a: invariantes del repo (van a `AGENTS.md`/rules, no a una skill) ni preferencias puntuales (van en el prompt de la sesión).

## Restricciones

1. **Litmus test antes de crear**: "¿quiero que esta instrucción aplique incluso cuando no estoy pensando en ella?" Sí → es una **rule**, no una skill. Si aplica a toda tarea, es una regla del repo (modo de fallo *Everything Bagel*).
2. **La description es para routing, no para leer**: corta, con las keywords reales con las que se pediría la tarea. Los modelos sub-disparan skills, así que sé un poco "pushy": incluye qué hace Y cuándo dispararse, incluso si el usuario no nombra la skill.
3. **El cuerpo es un procedimiento, no una wiki**: pasos numerados, checklists y criterios de éxito verificables. El material pesado (reglas completas, catálogos, plantillas) va en `references/` y se enlaza con la indicación de cuándo leerlo.
4. `SKILL.md` < 500 líneas. Si crece, trocea en `references/`.
5. **Seguridad**: nunca instalar ni habilitar plugins, MCP servers, ni dependencias sin confirmación explícita del usuario. Secretos solo como `${VAR}` — jamás literales en archivos de la skill.

## Flujo

### Fase A — Triaje: ¿qué vehículo?

| Necesidad | Vehículo | Test |
|---|---|---|
| Aplica siempre, sin excepciones | **Rule** (AGENTS.md / rules) | ¿Debe aplicar aunque nadie piense en ella? |
| Tú la disparas a mano, es determinista | **Command** (`/cmd`) | ¿Es un atajo ergonómico tuyo? |
| Expertise opcional que el agente carga si la tarea lo pide | **Skill** | ¿Solo aplica a un tipo de tarea? |
| Otro perfil de trabajador: otro prompt, otro acceso, otro modelo | **Agente/subagente** | ¿Necesita aislamiento o permisos propios? |

Regla de escalada: **skill primero**; subagente solo si aparece (a) necesidad de otro LLM,
(b) scoping de permisos, (c) contaminación de contexto del agente principal.

### Fase B — Capturar intención

Extrae del historial de conversación si el workflow ya ocurrió (herramientas usadas, orden,
correcciones, formatos). Responde antes de redactar:

1. ¿Qué debe permitirle hacer al modelo?
2. ¿Cuándo debe dispararse? ¿Qué frases o contextos del usuario?
3. ¿Qué formato de salida se espera?
4. ¿Hay ejemplos entrada/salida que fijen el comportamiento?

Confirma los huecos con el usuario antes de redactar, no después.

### Fase C — Redactar la skill

1. Ubicación: estándar multi-herramienta `<repo>/.agents/skills/<nombre>/SKILL.md`; `~/`
   para skills personales; `.zcode/skills/` solo para hacer override (gana en descubrimiento).
   El `name` del frontmatter debe coincidir con la carpeta (kebab-case, 1–64 caracteres).
2. Copia [templates/plantilla-skill.md](templates/plantilla-skill.md) y sustituye.
3. Lee [references/reglas-autoria-skills.md](references/reglas-autoria-skills.md) antes de
   redactar: contiene la tabla completa reglas/commands/skills, el checklist de las 6 preguntas
   y los 4 modos de fallo con sus correcciones.
4. Escribe el cuerpo con las 6 preguntas: **Trigger** (description), **Inputs**, **Steps**,
   **Checks**, **Stop conditions**, **Recovery**.

### Fase D — Integraciones (opcional)

Lee [references/integraciones.md](references/integraciones.md) antes de conectar cualquiera.
Resumen de la escalera:

- **Rule de routing**: "Al hacer X, carga el skill `y`" — mantiene el prompt siempre-activo
  pequeño y hace la skill descubrible en el flujo real de trabajo.
- **Command**: atajo corto y estable que dispara la skill, con parámetros posicionales
  (`/pr 123 bot-integration` → skill `pr` + `$1` `$2`).
- **MCP server**: si la skill necesita datos o acciones externas (repo, navegador, BD, docs
  actualizadas). Configurar con secretos como `${VAR}`; empezar read-only; pocos servers.
- **Plugin**: si ya existe una skill/capacidad empaquetada oficial — usarla antes de
  reinventarla. Instalar/habilitar SOLO con confirmación explícita del usuario.
- **Subagente**: último escalón (ver Fase A). En paralelo, límites de archivos no solapados y
  worktree por agente.

### Fase E — Verificar

1. Prepara 2–3 prompts de prueba realistas: rutas concretas, fraseo casual, incluso typos —
   lo que un usuario escribiría de verdad.
2. Carga la skill y ejecuta cada prompt en un turno fresco. Revisa el resultado **y la traza**:
   si el modelo hace busywork (releeer archivos, ir en círculos), la skill está
   sobre-prescrita — recorta, no añadas más reglas.
3. Verifica la checklist de 6 preguntas completa y revisa los 4 modos de fallo
   (*Encyclopedia, Everything Bagel, Secret Handshake, Fragile Skill*).

## Criterios de éxito

- La skill dispara sola en al menos 2 de 3 prompts de prueba, sin forzar `/skill <nombre>`.
- El cuerpo responde las 6 preguntas y cada check es verificable por quien ejecuta.
- Ningún secreto literal; `SKILL.md` < 500 líneas; las referencias cargan solo bajo demanda.
- El usuario confirma que el output coincide con lo que quería (hecho ≠ aprobado).

## Condiciones de parada

- Instalación o habilitación de plugin/MCP pendiente de confirmación → pausar y preguntar.
- La instrucción que motivó la skill resulta ser invariante → proponer rule en `AGENTS.md`
  en lugar de seguir creando la skill.
- Description abstracta tras 2 revisiones (*Secret Handshake*) → pausar y pedir al usuario
  3 frases reales con las que pediría esta tarea.

## Recuperación

- **No dispara** → reescribir la description con las keywords reales del usuario
  (*Secret Handshake*).
- **Se rompe con cada cambio del repo** → mover los detalles hardcodeados a `references/` y
  dejar el cuerpo procedimental (*Fragile Skill*).
- **Genera ruido/busywork** → recortar lo que no tira de su peso; explicar el porqué de las
  reglas que quedan en lugar de ponerlas en MAYÚSCULAS.
- Tras cada corrección, volver a Fase E con los mismos prompts de prueba.
- Si un problema persiste tras ediciones dirigidas, prueba un reencuadre distinto antes de
  añadir más restricciones.

## Referencias (cargar solo cuando se necesiten)

- [references/reglas-autoria-skills.md](references/reglas-autoria-skills.md) — reglas completas
  de autoría: litmus test, tabla de decisión, progressive disclosure, checklist de 6 preguntas,
  modos de fallo.
- [references/integraciones.md](references/integraciones.md) — patrones de wiring: rules de
  routing, commands con params, MCP (`.mcp.json`, secretos, read-only primero), plugins,
  subagentes y worktrees.
- [templates/plantilla-skill.md](templates/plantilla-skill.md) — esqueleto mínimo para copiar.
