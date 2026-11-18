# Reglas de autoría de skills (fuente: builder.io/blog/agent-skills-rules-commands + skill-creator)

Lee este archivo en Fase C (redacción) o Fase E (verificación). No necesitas memorizarlo:
vuelve a las tablas cuando tengas una decisión que tomar.

## 1. El modelo mental: rules vs commands vs skills

| Concepto | Quién lo dispara | Para qué | Coste de contexto | Anti-patrón |
|---|---|---|---|---|
| **Rules** | La herramienta (siempre) | Requisitos del repo, invariantes, seguridad, convenciones | Se paga siempre | Meter tutoriales en las rules |
| **Commands** | Tú (escribes `/cmd`) | Workflows repetibles, intención explícita | Se paga al usarlo | Convertir cada preferencia en command |
| **Skills** | El agente (bajo demanda) | Playbooks de tarea específica | Se paga solo si se necesita | Hacer skills que en realidad son rules |

### Litmus test

> "¿Quiero que esta instrucción aplique incluso cuando no estoy pensando en ella?"
> Sí → **rule**. No → **skill**.

Ejemplos de la fuente:

- "Nunca commitear archivos `.env`" → **rule**.
- "El design system usa estos nombres de tokens" → **rule**.
- "Al tocar código de billing, ejecutar estos 3 tests de integración" → **skill**.
- "Al escribir release notes, seguir este formato y checklist" → **skill**.

### Árbol de decisión

```text
¿Aplica a TODA tarea, siempre?
├─ Sí → RULE (AGENTS.md / rules del repo)
└─ No → ¿Lo disparas tú explícitamente para tomar el control?
         ├─ Sí → COMMAND (atajo ergonómico, determinista)
         └─ No → ¿El agente decide si la necesita según la tarea?
                  ├─ Sí → SKILL (playbook bajo demanda)
                  └─ ¿Y necesita además otro prompt/acceso/modelo?
                         └─ Sí → AGENTE o SUBAGENTE (último escalón)
```

## 2. Description = routing

La description es la única parte de la skill que el agente ve en cada sesión. Las reglas:

1. **Corta, específica y densa en keywords**: las palabras con las que el usuario realmente
   pediría la tarea (incluye sinónimos y frases casuales).
2. **Incluye qué hace Y cuándo dispararse.** Los modelos tienden a *sub*-disparar: sé un poco
   "pushy". Mal: "Cómo construir un dashboard de datos internos". Bien: "…Úsala siempre que
   el usuario mencione dashboards, métricas internas o quiera mostrar datos de la empresa,
   aunque no diga 'dashboard'".
3. **Test del Secret Handshake**: si el agente nunca carga la skill, la description es
   abstracta o poética. Reescríbela con el vocabulario real del usuario.
4. Nota de nomenclatura: Cursor llamó "rules" a lo que otras herramientas llaman "skills"
   (rules aplicadas por descripción). Lo que importa es el triple
   invariante-siempre / atajo-determinista / expertise-bajo-demanda, no la etiqueta.

## 3. Progressive disclosure ("lazy loading para contexto")

Tres capas de carga:

1. **Metadata** (`name` + `description`): siempre en contexto. Mantenla corta.
2. **Cuerpo del SKILL.md**: se carga solo cuando la skill dispara. Objetivo < 500 líneas.
3. **Archivos adjuntos** (`references/`, `scripts/`, `assets/`): se leen bajo demanda,
   tamaño en principio ilimitado.

Si el cuerpo crece, separa el detalle por dominio en referencias y dile al modelo cuándo
leer cada una ("si el objetivo es AWS, lee references/aws.md antes de seguir").

El contexto es un presupuesto estricto: cada palabra genérica es espacio que no queda para
el código, los logs de error y el plan. Las skills no hacen al agente más listo — hacen la
información más enfocada y fácil de recuperar.

## 4. Checklist de las 6 preguntas (toda skill debe responderlas)

| Pregunta | Dónde vive |
|---|---|
| **Trigger**: ¿cuándo exactamente debe cargarla el agente? | `description` |
| **Inputs**: ¿qué info necesita del usuario o del repo antes de empezar? | Sección Inputs |
| **Steps**: ¿cuál es el procedimiento? | Sección Steps numerada |
| **Checks**: ¿cómo se prueba que funcionó? | Criterios de éxito / checklist |
| **Stop conditions**: ¿cuándo pausar y pedir un humano? | Sección explícita |
| **Recovery**: ¿qué pasa si un check falla? | Sección explícita |

## 5. Los 4 modos de fallo

| Modo | Síntoma | Corrección |
|---|---|---|
| **The Encyclopedia** | La skill lee como página de wiki | Trocear en archivos pequeños en `references/`; la skill tira solo de lo que necesita en el momento |
| **The Everything Bagel** | Aplica a toda tarea | No es una skill: es una rule o convención del repo. Migrarla |
| **The Secret Handshake** | El agente nunca la carga | Description demasiado abstracta: reescribirla con las frases reales del usuario |
| **The Fragile Skill** | Se rompe con cada cambio del repo | Demasiados hardcodes: mover los detalles a archivos referenciados y dejar el cuerpo procedimental |

## 6. Estilo de redacción

- Forma imperativa ("Lee el archivo antes de editar").
- **Explica el porqué** cuando la regla no es obvia: los modelos siguen mejor la guía cuando
  entienden la razón; los MUST y NEVER en mayúsculas suelen indicar que falta explicación,
  no volumen.
- **Ejemplos > reglas**: si la skill produce salida estructurada, incluye un ejemplo literal
  del formato. Si debe usarse una herramienta concreta, muestra la llamada.
- "Done" claro: reduce ambigüedad, no añade ruido.

## 7. Ubicación y descubrimiento (ZCode)

Prioridad de descubrimiento (gana la primera que coincida):

1. `<proyecto>/.zcode/skills/<nombre>/SKILL.md` — para *override* de una skill existente.
2. `<proyecto>/.agents/skills/<nombre>/SKILL.md` — ubicación estándar multi-herramienta
   (default para skills nuevas).
3. `~/.zcode/skills/<nombre>/SKILL.md` y `~/.agents/skills/<nombre>/SKILL.md` — personales.

`name` = kebab-case, 1–64 caracteres, debe coincidir con el nombre de la carpeta.
Para actualizar una skill instalada de solo-lectura (caché de plugin): copiarla a
`~/.agents/skills/<nombre>/` y editar ahí; el mismo nombre + ruta de mayor prioridad la
reemplaza en descubrimiento.

## 8. Iteración

- Probar con 2–3 prompts realistas (rutas concretas, typos, fraseo casual), revisar resultado
  **y traza** con el usuario.
- Generalizar: la skill debe funcionar con inputs que ni tú ni el usuario han visto aún.
- "Añade una regla la segunda vez que veas el mismo error" — y tras cada corrección manual,
  actualizar la skill para que el fallo no recurra.
- Si un problema resiste ediciones dirigidas, prueba otro reencuadre en vez de apilar
  restricciones; las reglas sobre-ajustadas envejecen mal.
