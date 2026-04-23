# AGENTS.md

## Project State

This repo contains **only specification documents** - no source code. Build full frontend from `docs/` specs.

## Required Setup (when implementing)

1. Copy `.env.example` → `.env` and fill values
2. Backend expected (conecta directamente a API)

## Documentation Order (read first)

1. `docs/PROJECT.md` - Business overview, users, flows
2. `docs/API.md` - Backend endpoints (authoritative)
3. `docs/FRONTEND_SPEC.md` - Architecture decisions
4. `docs/STATE_MANAGEMENT.md` - State design

## Stack

| Technology | Version |
|------------|---------|
| Build | Vite |
| SPA | React + TypeScript |
| Routing | react-router-dom ^7 |
| HTTP | axios |
| UI | Tailwind CSS v4 |
| Server state | TanStack Query ^5 |
| Client state | Zustand ^4 |
| Forms | React Hook Form ^7 |
| Validation | Zod ^3 |
| Testing | Vitest |

## Roles & Access

| Rol | Access |
| -- | -- |
| admin | All |
| produccion | /inventario, /inventario/create, GET /products |
| despacho | /shipments/*, /shipments/create, /shipments/:id/validate, GET /customers, GET /products |

## Endpoints por Rol

| Endpoint | admin | despacho | produccion |
|----------|:-----:|:--------:|:----------:|
| POST /public/login | ✅ | ✅ | ✅ |
| GET /private/customers | ✅ | ✅ | ❌ |
| POST /private/customers | ✅ | ❌ | ❌ |
| PUT /private/customers/:id | ✅ | ❌ | ❌ |
| DELETE /private/customers/:id | ✅ | ❌ | ❌ |
| GET /private/products | ✅ | ✅ | ✅ |
| POST /private/products | ✅ | ❌ | ❌ |
| PUT /private/products/:id | ✅ | ❌ | ❌ |
| DELETE /private/products/:id | ✅ | ❌ | ❌ |
| GET /private/inventario | ✅ | ❌ | ✅ |
| POST /private/inventario | ✅ | ❌ | ✅ |
| PUT /private/inventario/:id | ✅ | ❌ | ❌ |
| GET /private/shipments | ✅ | ✅ | ❌ |
| POST /private/shipments | ✅ | ✅ | ❌ |
| POST /private/validate_shipments/:id | ✅ | ✅ | ❌ |
| DELETE /private/shipments/:id | ✅ | ❌ | ❌ |
| GET /private/metricas | ✅ | ✅ | ❌ |

## Roles Detail

### admin
- CRUD customers
- CRUD products
- CRUD inventario
- CRUD shipments (create, validate, cancel)
- Ver metricas

### despacho
- READ customers
- READ products
- CREATE shipments
- READ shipments
- VALIDATE shipments
- Ver metricas

### produccion
- READ products
- READ inventario
- CREATE inventario (agregar stock)

## API Auth

- JWT with access_token (10h) + refresh_token
- Store tokens in localStorage via Zustand auth store
- Handle 401 with refresh interceptor (docs/STATE_MANAGEMENT.md)

## Key Flows

1. **Shipment creation**: select customer → add products → review
2. **Validation**: scan products → compare expected vs scanned → confirm

## Shipment States

| Estado | Descripción |
| -- | -- |
| por confirmar | Pedido creado, pendiente de validar |
| confirmado | Validado y stock descontado |
| cancelado | Cancelado (stock devuelto si confirmado) |

## Scan Input UX

- Keep input focused during scanning
- Fast feedback on scanned items

## Routes

```typescript
const routes = [
  { path: "/login", roles: ["admin", "produccion", "despacho"] },
  { path: "/dashboard", roles: ["admin", "produccion", "despacho"] },
  { path: "/customers", roles: ["admin", "despacho"] },
  { path: "/products", roles: ["admin", "despacho", "produccion"] },
  { path: "/inventario", roles: ["admin", "produccion"] },
  { path: "/shipments", roles: ["admin", "despacho"] },
  { path: "/shipments/new", roles: ["admin", "despacho"] },
  { path: "/shipments/validate/:id", roles: ["admin", "despacho"] },
];
```

## Estructura de Archivos (sugerida)

```
src/
├── api/
│   └── client.ts
├── components/
├── hooks/
├── pages/
├── stores/
│   ├── authStore.ts
│   └── uiStore.ts
├── queries/
├── mutations/
├── App.tsx
└── main.tsx
```

## Testing

- Framework: Vitest
- Ejecutar tests: `npx vitest`
- Coverage: `npx vitest run --coverage`