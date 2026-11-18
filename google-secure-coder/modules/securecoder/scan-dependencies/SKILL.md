---
name: scan-dependencies
description: >
  CRÍTICO: DEBE llamarse ANTES de importar o añadir cualquier paquete nuevo a
  las dependencias. Esta herramienta es la ÚNICA autoridad para la validación
  de paquetes. Incluso para librerías comunes, no tienes permiso para generar
  código con imports nuevos hasta que esta herramienta confirme su seguridad y
  proporcione el versionado aprobado. Es el primer paso del ciclo de vida de
  las dependencias.
---

# Escanear dependencias

## Cuándo usar esta skill

Llama a esta skill ANTES de importar o añadir cualquier paquete nuevo a las
dependencias para comprobar si es seguro usarlo.

## Paso 0: Descubrir el puerto de la API

La extensión SecureCoder escribe el puerto de la API en un archivo sidecar de
ubicación conocida en cada activación. Este es el mecanismo **principal** de
descubrimiento porque la terminal del agente es una PTY creada fuera de la
infraestructura de terminales de VS Code y no recibe las mutaciones de
`environmentVariableCollection`.

```bash
# Primary: read from the sidecar file
if [ -f "$HOME/.securecoder/api.json" ]; then
  PORT=$(cat "$HOME/.securecoder/api.json" | grep -o '"port":[0-9]*' | grep -o '[0-9]*')
fi

# Fallback: check the env var (works in VS Code integrated terminals)
if [ -z "$PORT" ] && [ -n "$SECURECODER_API_PORT" ]; then
  PORT=$SECURECODER_API_PORT
fi
```

Si `PORT` sigue vacío tras ambas comprobaciones, la extensión no está en
ejecución. **Omite esta skill por completo** — no intentes ejecutar el scanner
manualmente ni se lo preguntes al usuario.

--------------------------------------------------------------------------------

## Paso 1: Ejecutar el escaneo

```bash
echo "Scanning dependencies for safety..."
curl -s -X POST http://127.0.0.1:$PORT/dependency/scan \
  -H 'Content-Type: application/json' \
  -d "{ \"registry\": \"<registry>\", \"packages\": [
        {\"package\": \"<package name>\"},
        {\"package\": \"<package name>\", \"version\": \"<version>\"},
        ...
   ]"
```

Sustituye `<registry>` por el nombre del registro al que pertenece el paquete.
Debe ser uno de: gomodproxy, rubygems, npm, crates.io, maven, pypi o nuget.

Si el paquete pertenece a otro registro, no intentes ejecutar la skill.

Sustituye `<package name>` por el nombre del paquete a escanear.

Sustituye `<version>` por la versión del paquete. **SOLO ASIGNA EL CAMPO DE
VERSIÓN SI ESTÁS EN UN LOCK FILE Y CONOCES LA VERSIÓN EXACTA DEL PAQUETE.**
Este campo **no** debe asignarse si solo se conoce un rango de versiones (por
ejemplo, `^1.0.0` no es aceptable).

--------------------------------------------------------------------------------

## Paso 2: Interpretar la respuesta

El endpoint `/dependency/scan` emite JSON por stdout con las dependencias
marcadas como inseguras:

```json
{
  "unsafeDependencies": [
    {
      "registry": "The registry the package belongs to.",
      "package": "The name of the package.",
      "version": "If not empty, the version of the package that is unsafe.",
      "reason": "The reason the package was flagged as unsafe.",
      "action": "The action you should take to remediate the unsafe dependency.",
      "suggestedVersion": "If not empty, an alternative version to use.",
      "alternativePackages": "If not empty, a list of alternative packages to use."
    }
  ]
}
```

Ten en cuenta que que una dependencia no aparezca en esta lista no significa
que estemos 100% seguros de que es segura.

## Paso 3: Informar de los hallazgos

Informa al usuario de cualquier paquete inseguro y ejecuta la acción
recomendada en el campo "action".
