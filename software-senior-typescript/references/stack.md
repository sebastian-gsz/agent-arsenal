# Stack tecnológico obligatorio

## Frontend
- **Framework:** React 19+ con Vite
- **UI system:** shadcn/ui + Radix UI + Tailwind CSS v4
- **Estado de servidor:** TanStack Query
- **Estado de cliente/formularios:** Zustand / React Hook Form
- **Routing:** Wouter o React Router v7

## Backend
- **Runtime:** Node.js (ES Modules)
- **Framework:** Express v5 o Hono
- **ORM:** Drizzle ORM (SQL-like, type-safe)
- **Validación:** Zod (schemas compartidos con el frontend)

## Infraestructura
- **Gestor de paquetes:** pnpm
- **Tipado:** TypeScript en strict mode
- **Documentación de API:** OpenAPI 3.0 como Single Source of Truth
- **Generación de código:** Orval (clientes API desde OpenAPI), Drizzle Kit (migraciones y tipos desde el schema)

## Estructura de monorepo de referencia

```
<proyecto>/
├── package.json          (workspaces, scripts compartidos)
├── pnpm-workspace.yaml
├── packages/
│   ├── shared/           (tipos, schemas Zod, constantes — importado por todos)
│   ├── api/              (backend: Hono/Express + Drizzle)
│   └── web/              (frontend: React + Vite)
├── openapi.yaml          (contrato de API, fuente única)
└── tooling/              (tsconfig, eslint, prettier compartidos)
```

El flujo de tipos: schema Drizzle → tipos inferidos en `packages/shared` → cliente API generado por Orval desde `openapi.yaml` → componentes UI con tipos completos desde BD hasta el render.
