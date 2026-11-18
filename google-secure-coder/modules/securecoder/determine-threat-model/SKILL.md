---
name: determine-threat-model
description: >
  Construye un modelo de amenazas para el repositorio o componente actual. Usa
  esta skill para identificar puntos de entrada, límites de confianza, rutas de
  datos sensibles y áreas de revisión prioritarias. El artifact de modelo de
  amenazas resultante lo usan otras skills para contextualizar los hallazgos
  del escáner y distinguir verdaderos positivos de falsos positivos.
---

# Determinar el modelo de amenazas

Esta skill produce un **artifact de modelo de amenazas** — un resumen de
seguridad breve de cómo funciona un repositorio o componente. Otras skills
(`create_security_implementation_plan`, `generate_security_audit_report`) usan
este artifact como contexto para priorizar escaneos, evaluar hallazgos y
generar reportes de auditoría.

## Cuándo usar esta skill

-   Úsala cuando necesites **evaluar si un hallazgo del escáner es una
    vulnerabilidad real o un falso positivo**.
-   Úsala cuando el usuario pida un **modelo de amenazas** o una **visión
    general de seguridad** de un componente.
-   Úsala al **iniciar una revisión de seguridad** de un componente nuevo o
    desconocido.
-   **No la uses** para modificaciones generales de código o tareas ajenas a
    la seguridad.

## Paso 1: Identificar el propósito del componente

Examina el repositorio o el componente para responder:

-   ¿Qué hace este componente?
-   ¿Quiénes son sus consumidores (usuarios, otros servicios, herramientas
    internas)?
-   ¿Cuál es el contexto de despliegue (internet público, red interna, CLI)?

Revisa los README, los archivos fuente de nivel superior, los archivos de
configuración y los archivos de configuración de build para construir esta
comprensión.

## Paso 2: Mapear puntos de entrada y entradas no confiables

Identifica todos los puntos por donde los datos externos entran al sistema:

Tipo de punto de entrada | Ejemplos
------------------------ | -------------------------------------------------
Endpoints HTTP / gRPC    | Handlers de API, rutas de servlets, métodos de servicios RPC
Argumentos de CLI        | Flags, argumentos posicionales, stdin
Entradas de archivos     | Archivos de configuración, archivos subidos, archivos temporales
Variables de entorno     | Feature flags, secretos pasados por env
Lecturas de BD / colas   | Mensajes de Pub/Sub, filas de tablas no confiables
Comunicación entre procesos | Memoria compartida, pipes, sockets

Para cada punto de entrada, anota:

-   Si la entrada es **confiable** (p. ej., de un servicio interno autenticado)
    o **no confiable** (p. ej., de un usuario final).
-   Qué validación o sanitización se aplica.

## Paso 3: Identificar límites de confianza y supuestos de autenticación

Documenta:

-   **Autenticación**: ¿Cómo se identifican los llamantes? (OAuth, mTLS, API
    keys, ninguno)
-   **Autorización**: ¿Cómo se comprueban los permisos? (IAM, ACLs,
    comprobaciones de roles, ninguna)
-   **Confianza implícita**: ¿Hay supuestos como "solo los servicios backend
    confiables llaman a esto" o "solo los usuarios internos tienen acceso"?
-   **Cruces de límites**: ¿Dónde se mueven los datos entre zonas de confianza
    (p. ej., frontend → backend, servicio → base de datos, usuario → admin)?

## Paso 4: Mapear rutas de datos sensibles y acciones privilegiadas

Rastrea por dónde fluyen los datos sensibles dentro del componente:

-   **Secretos**: API keys, tokens, credenciales — ¿dónde se leen, almacenan y
    transmiten?
-   **PII**: Datos de usuario, direcciones de correo, direcciones IP — ¿cómo
    se gestionan y se registran en logs?
-   **Operaciones privilegiadas**: Escrituras de archivos, ejecución de shell,
    llamadas de red, mutaciones de base de datos, concesión de permisos.

## Paso 5: Identificar áreas de revisión prioritarias

Con base en los pasos 1–4, enumera las áreas que deben revisarse primero.
Prioriza por:

1.  Código que maneja **entradas no confiables** con **validación insuficiente**.
2.  Código que realiza **acciones privilegiadas** (exec, I/O de archivos,
    llamadas de red).
3.  Código que cruza **límites de confianza** sin comprobaciones de
    autenticación adecuadas.
4.  Código que maneja **datos sensibles** (secretos, PII, tokens).

## Paso 6: Evaluar los hallazgos del escáner contra el modelo de amenazas

Cuando haya resultados del escáner, evalúa cada hallazgo contra el contexto
del modelo de amenazas. Para cada hallazgo, determina:

-   **¿Es alcanzable el código señalado desde un punto de entrada no
    confiable?**
-   **¿El contexto de autenticación/confianza del modelo de amenazas mitiga el
    riesgo?**
-   **¿Es explotable la vulnerabilidad dado el contexto de despliegue?**

Clasifica cada hallazgo como:

Disposición                   | Significado
----------------------------- | -------------------------------------------------
**Verdadero positivo**        | El hallazgo es una vulnerabilidad real y explotable
                              | dado el modelo de amenazas.
**Falso positivo**            | El hallazgo no es explotable debido a los límites
                              | de confianza, la autenticación o la funcionalidad
                              | prevista.
**Requiere revisión manual**  | Contexto insuficiente para determinarlo; márcalo
                              | para revisión humana.

Proporciona una justificación de una línea para cada clasificación.

## Paso 7: Escribir el modelo de amenazas en el plan de implementación

Crea o actualiza el **artifact** `implementation_plan.md` con una sección
`## Security Threat Model`. Esto muestra el modelo de amenazas directamente en
la UI de planificación de Antigravity junto al plan de corrección. Otras
skills (`create_security_implementation_plan`, `generate_security_audit_report`
y `run_poc`) deben leer la sección `## Security Threat Model` de
`implementation_plan.md` cuando necesiten contexto del modelo de amenazas.

Usa la siguiente estructura para la sección:

```markdown
## Security Threat Model

### Component Overview
<Brief description of the component, its consumers, and deployment context.>

### Entry Points and Untrusted Inputs
| Entry Point | Type | Trusted? | Validation |
|---|---|---|---|
| <endpoint> | <type> | Yes/No | <description> |

### Trust Boundaries and Auth Assumptions
- **Authentication**: <method>
- **Authorization**: <method>
- **Implicit trust**: <assumptions>

### Sensitive Data Paths
| Data Type | Source | Destination | Protection |
|---|---|---|---|
| <type> | <source> | <dest> | <protection> |

### Privileged Actions
| Action | Location | Guard |
|---|---|---|
| <action> | <file:line> | <auth check or none> |

### Priority Review Areas
1. <area and rationale>
2. <area and rationale>

### Finding Dispositions
| Finding | Severity | Disposition | Rationale |
|---|---|---|---|
| <vulnerability_class> at <file:line> | <severity> | True Positive / False Positive / Needs Review | <one-line reason> |
```

## Notas importantes

-   La sección `## Security Threat Model` de `implementation_plan.md` es lo
    **primero que hay que revisar** si los hallazgos del escáner parecen
    extraños — proporciona el contexto que guía la priorización.
-   Debes tener en cuenta la intención y el modelo de amenazas del componente
    para determinar si un problema es un problema de seguridad válido o una
    funcionalidad prevista.
-   Puedes obtener los puntos de entrada, los puntos de entrada web y las
    acciones de usuario para ayudar a determinar el uso previsto del
    componente.
-   Tanto la skill `create_security_implementation_plan` como
    `generate_security_audit_report` deben hacer referencia a la sección
    `## Security Threat Model` de `implementation_plan.md` cuando exista.
