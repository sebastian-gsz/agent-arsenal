---
name: run-poc
description: >
  Genera y razona una Prueba de Concepto (PoC) para verificar que una
  corrección de seguridad es efectiva. Usa esta skill después de que se haya
  identificado una vulnerabilidad y se haya aplicado un parche de corrección.
  El artifact PoC resultante demuestra si la vulnerabilidad sigue siendo
  explotable después de la remediación.
---

# Ejecutar PoC

Esta skill produce un **artifact de verificación PoC** — un script que
explotaría una vulnerabilidad conocida, acompañado de un análisis paso a paso
de por qué la corrección evita la explotación. Se usa como paso final de
verificación en el pipeline de remediación de seguridad, después de aplicar
`determine_threat_model`, `create_security_implementation_plan` y el parche de
corrección.

## Cuándo usar esta skill

-   Úsala **después de aplicar un parche de corrección** para verificar que la
    remediación es efectiva.
-   Úsala cuando necesites **validar que un hallazgo del escáner ya no es
    explotable** después de cambios en el código.
-   **No la uses** antes de aplicar una corrección — usa `determine_threat_model`
    para evaluar los hallazgos antes de la remediación.
-   **No la uses** para modificaciones generales de código o tareas ajenas a
    la seguridad.

## Requisitos previos

Antes de ejecutar esta skill, asegúrate de que:

-   El artifact `implementation_plan.md` contiene una sección
    `## Security Threat Model` (producida por `determine_threat_model`).
-   Se ha aplicado un parche de corrección para atender la vulnerabilidad.
-   Conoces el tipo específico de vulnerabilidad, los archivos afectados y el
    hallazgo del escáner o del modelo de amenazas.

## Paso 1: Revisar el modelo de amenazas y la corrección

Lee la sección `## Security Threat Model` de `implementation_plan.md` y el
parche de corrección aplicado. Céntrate solo en los hallazgos clasificados
como **verdaderos positivos** en el modelo de amenazas:

-   Qué vulnerabilidad de verdadero positivo se identificó (tipo, severidad,
    ruta de código afectada).
-   Qué puntos de entrada y límites de confianza son relevantes para este
    hallazgo.
-   Qué cambió la corrección y por qué debería prevenir la explotación.

## Paso 2: Describir el PoC

Describe un escenario de explotación concreto que demuestre la vulnerabilidad.
Debe incluir:

-   Qué entrada o petición construiría un atacante.
-   Qué ruta de código sigue el exploit a través del código del usuario (ahora
    parcheado).
-   Cuál sería el resultado malicioso esperado si el código siguiera siendo
    vulnerable (p. ej., lectura de archivos no autorizada, ejecución de
    scripts inyectados, escalada de privilegios).

### Ejemplos de PoC por tipo de vulnerabilidad

Úsalos como guía para describir el PoC según el tipo de vulnerabilidad:

#### Path Traversal

-   **Escenario de explotación:** Un atacante envía una petición como `GET
    /files?name=../../etc/passwd` para leer archivos fuera del directorio
    previsto.
-   **Qué rastrear:** ¿El código parcheado normaliza o rechaza las secuencias
    `../` antes de construir la ruta del archivo? ¿Valida que la ruta resuelta
    permanezca dentro del directorio permitido?

#### Cross-Site Scripting (XSS)

-   **Escenario de explotación:** Un atacante envía un campo de formulario o
    parámetro de URL que contiene `<script>document.cookie</script>`, el cual
    se renderiza en la respuesta HTML sin escapar.
-   **Qué rastrear:** ¿El código parcheado sanitiza o escapa la entrada del
    usuario antes de insertarla en el DOM o en la plantilla HTML? ¿La
    corrección aplica a todos los contextos de salida (cuerpo HTML,
    atributos, JavaScript)?

#### SQL Injection

-   **Escenario de explotación:** Un atacante envía un formulario de login con
    el usuario `admin'
    OR '1'='1` para eludir la autenticación o extraer datos.
-   **Qué rastrear:** ¿El código parcheado usa consultas parametrizadas o
    prepared statements en lugar de concatenación de cadenas? ¿La corrección
    cubre todos los puntos de construcción de consultas?

#### Server-Side Request Forgery (SSRF)

-   **Escenario de explotación:** Un atacante proporciona una URL como
    `http://169.254.169.254/latest/meta-data/` en un campo de webhook o de
    descarga de imágenes para acceder a metadatos internos de la nube.
-   **Qué rastrear:** ¿El código parcheado valida y restringe la URL/hostname
    de destino antes de hacer la petición? ¿Bloquea los rangos de IP internas
    y los endpoints de metadatos de la nube?

#### Insecure Deserialization

-   **Escenario de explotación:** Un atacante envía un objeto serializado
    manipulado (p. ej., un payload pickle malicioso o JSON con contaminación
    de `__proto__`) que ejecuta código arbitrario al deserializarse.
-   **Qué rastrear:** ¿El código parcheado usa métodos de deserialización
    seguros? ¿Valida o restringe los tipos que pueden deserializarse?

## Paso 3: Razonar el PoC

**No** ejecutes el script del PoC. En su lugar, razónalo paso a paso:

1.  Rastrea el flujo de datos desde la entrada de explotación del PoC a través
    del código parcheado.
2.  Identifica dónde la corrección intercepta o neutraliza el exploit.
3.  Determina el resultado esperado — el exploit debería fallar gracias a la
    corrección.
4.  Si el exploit todavía tendría éxito, marca la corrección como incompleta
    y explica por qué.

## Paso 4: Escribir la verificación PoC en el walkthrough

Crea o actualiza el **artifact** `walkthrough.md` con una sección
`## PoC Verification`. Esto muestra el análisis del PoC en la UI de walkthrough
de Antigravity junto a la Auditoría de seguridad, reuniendo toda la evidencia
posterior a las correcciones en un solo lugar.

Usa la siguiente estructura para la sección:

```markdown
## PoC Verification

### <Vulnerability ID or Description>

#### Vulnerability Summary
| Field               | Value                              |
|---------------------|------------------------------------|
| Type                | <e.g., Path Traversal, XSS>        |
| Severity            | <e.g., High, Medium>               |
| Affected File       | <file:line>                        |
| Vulnerability Class | <e.g., Path Traversal, XSS>        |

#### Fix Summary
<Brief description of what the fix changed and why.>

#### Reasoning Analysis
| Step | Description              | Result                             |
|------|--------------------------|------------------------------------|
| 1    | <Trace exploit input>    | <What happens>                     |
| 2    | <Where fix intercepts>   | <How it blocks>                    |
| 3    | <Expected outcome>       | Exploit blocked / Exploit succeeds |

#### Conclusion
<**Fix verified** or **Fix incomplete** with explanation.>
```

Si se verificaron múltiples vulnerabilidades, añade una subsección (`###`) por
cada una bajo el único encabezado `## PoC Verification`.

## Notas importantes

-   La sección `## PoC Verification` de `walkthrough.md` la lee la skill
    `generate_security_audit_report` para referenciar cruzadamente los
    resultados de las correcciones.
-   Si el análisis de razonamiento muestra que la corrección está incompleta,
    recomienda los cambios adicionales específicos necesarios.
-   Esta skill complementa pero no reemplaza volver a ejecutar el escáner de
    seguridad después de una corrección — el escáner busca patrones conocidos,
    mientras que el PoC prueba la explotabilidad real.
