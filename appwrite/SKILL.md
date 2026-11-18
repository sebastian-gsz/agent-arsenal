---
name: appwrite
description: Hub de las skills de Appwrite — enruta a la skill del SDK según lenguaje o plataforma (TypeScript/JS, Flutter/Dart, Kotlin, Swift, Python, PHP, Go, Ruby, Rust, .NET) y a la CLI para despliegue y operación. Úsala como punto de entrada cuando el trabajo toque Appwrite (auth, bases de datos/tables, storage, functions, real-time, teams, sites) o se mencionen Appwrite, Appwrite Cloud, appwrite.config.json, desplegar funciones, generar SDKs/tipos — también cuando no sepas qué skill de Appwrite aplicar.
---

# Appwrite — Hub

Suite de skills de Appwrite bajo `appwrite-*/`. **No resuelvas la tarea aquí:** identifica lenguaje/plataforma y tarea, carga la skill del SDK correspondiente y, si hay despliegue u operación de proyectos, añade la CLI.

## Mapa de skills

| Skill | Plataforma / ámbito | Cuándo usarla |
|---|---|---|
| [`appwrite-typescript`](appwrite-typescript/SKILL.md) | Web (JS/TS), React Native, Node.js, Deno, SSR | Apps de navegador o React Native, backends Node/Deno, autenticación SSR |
| [`appwrite-dart`](appwrite-dart/SKILL.md) | Flutter (móvil/web/escritorio) y Dart server | Apps Flutter multiplataforma o servicios Dart |
| [`appwrite-kotlin`](appwrite-kotlin/SKILL.md) | Android nativo y Kotlin/JVM | Apps Android (OAuth con Activity, corutinas) o backends JVM |
| [`appwrite-swift`](appwrite-swift/SKILL.md) | iOS, macOS, watchOS, tvOS y Swift server | Apps nativas Apple con real-time async/await |
| [`appwrite-python`](appwrite-python/SKILL.md) | Python (Django, Flask, FastAPI) | Backends y admin server-side en Python |
| [`appwrite-php`](appwrite-php/SKILL.md) | PHP (Laravel, Symfony) | Backends y admin server-side en PHP |
| [`appwrite-go`](appwrite-go/SKILL.md) | Go (paquetes por servicio, functional options) | Backends y admin server-side en Go |
| [`appwrite-ruby`](appwrite-ruby/SKILL.md) | Ruby (Rails, Sinatra) | Backends y admin server-side en Ruby |
| [`appwrite-rust`](appwrite-rust/SKILL.md) | Rust (crates.io `appwrite`, Tokio) | Backends async en Rust: TablesDB, storage, functions |
| [`appwrite-dotnet`](appwrite-dotnet/SKILL.md) | .NET / C# (ASP.NET, Blazor) | Backends y admin server-side en .NET |
| [`appwrite-cli`](appwrite-cli/SKILL.md) | CLI, CI y scripting | Desplegar/pull/push functions, sites y esquema, `appwrite.config.json` vs `appwrite client`, auth en CI, generar SDK/tipos (`appwrite generate`, `appwrite types`), variables de functions/sites |

## Cómo enrutar

1. **App cliente** (web, móvil, escritorio): carga la skill del SDK del lenguaje del cliente. La autenticación de cliente usa sesión/OAuth; cubre también consultas, uploads, real-time y teams.
2. **Backend o admin server-side**: carga la skill del lenguaje del servidor. El admin usa API keys — nunca mezcles credenciales de cliente y servidor en el mismo flujo.
3. **Proyecto mixto (cliente + servidor)**: carga ambas skills pertinentes; cada una cubre su lado del límite.
4. **Desplegar u operar** (deploy de functions/sites, pull/push de esquema de tables, variables de entorno remotas, auth en CI, endpoints regionales de Cloud): añade [`appwrite-cli`](appwrite-cli/SKILL.md) **además** del SDK — es transversal a todos los lenguajes.
5. **No sabes qué lenguaje o el proyecto combina varios**: empieza por la fila de la tabla que coincida con el archivo que vas a tocar; si la tarea es de infraestructura (CI, deploy, esquema), empieza por la CLI.

## Notas

- Las skills hijas están en inglés y son independientes entre sí: ninguna referencia a las demás; este hub es el único punto de enrutado.
- La CLI no cataloga comandos: ante la duda de flags, `appwrite <comando> --help`.
- Cloud usa endpoints regionales: el proyecto manda (`appwrite.config.json`), no el host de login — la CLI detalla las trampas.
