# COMPONENT_STRUCTURE.md

> Estructura basada exclusivamente en `docs/API.md`

---

## Estructura de Carpetas

```
/src
/app
/providers
    QueryProvider.tsx
    AuthProvider.tsx

/routes
    AppRouter.tsx
    ProtectedRoute.tsx
    RoleGuard.tsx

/pages
    /auth
        LoginPage.tsx

    /dashboard
        DashboardPage.tsx

    /customers
        CustomersPage.tsx
        CustomerForm.tsx

    /products
        ProductsPage.tsx
        ProductForm.tsx

    /inventario
        InventarioPage.tsx
        InventarioAddStockPage.tsx
        InventarioAdjustPage.tsx

    /shipments
        ShipmentsPage.tsx
        ShipmentCreatePage.tsx
        ShipmentValidatePage.tsx

/components
/ui
    Button.tsx
    Input.tsx
    Select.tsx
    Modal.tsx
    Table.tsx
    Badge.tsx
    Loader.tsx
    BarcodeViewer.tsx

    /layout
        Sidebar.tsx
        Header.tsx
        PageContainer.tsx

    /shared
        DataTable.tsx
        ConfirmDialog.tsx
        EmptyState.tsx
        ErrorState.tsx

    /forms
        CustomerFormFields.tsx
        ProductFormFields.tsx
        StockFormFields.tsx
        ShipmentFormFields.tsx
        ScannerInput.tsx

    /customers
        CustomerList.tsx

    /products
        ProductList.tsx
        ProductCard.tsx

    /inventario
        InventoryTable.tsx
        StockSummary.tsx

    /shipments
        ScannerInput.tsx
        ProductList.tsx
        ValidationPanel.tsx
        ShipmentSummary.tsx

/hooks
    useAuth.ts
    useUserRole.ts

/services
/api
    client.ts
    endpoints.ts

    /auth
        auth.service.ts

    /customers
        customers.service.ts

    /products
        products.service.ts

    /inventario
        inventario.service.ts

    /shipments
        shipments.service.ts

/store
    auth.store.ts
    ui.store.ts

/types
    auth.types.ts
    customer.types.ts
    product.types.ts
    inventario.types.ts
    shipment.types.ts
    api.types.ts

/utils
    formatters.ts
    validators.ts
    constants.ts
    roles.ts

/config
    env.ts
    roles.ts
    routes.ts

/styles
    globals.css

main.tsx
```

---

## Convenciones Clave

| Tipo | Descripción |
| -- | -- |
| `pages` | Vistas completas (rutas) |
| `components` | Reutilizables |
| `services` | Comunicación con API (singular) |
| `stores` | Estado global (Zustand) |
| `hooks` | Lógica reutilizable |
| `types` | Tipado estric

---

## mapping: Página → Endpoint

| Página | Endpoint | Método |
| -- | -- | -- |
| LoginPage | /public/login | POST |
| LoginPage | /private/auth/refresh-token | POST |
| CustomersPage | /private/customers | GET |
| CustomerForm | /private/customers | POST |
| CustomerForm | /private/customers/:id | PUT |
| ProductsPage | /private/products | GET |
| ProductForm | /private/products | POST |
| ProductForm | /private/products/:id | PUT |
| ProductsPage | /private/products/:id | DELETE |
| InventarioPage | /private/inventario | GET |
| InventarioAddStockPage | /private/inventario | POST |
| InventarioAdjustPage | /private/inventario/:id | PUT |
| ShipmentsPage | /private/shipments | GET |
| ShipmentCreatePage | /private/shipments | POST |
| ShipmentValidatePage | /private/validate_shipments/:id | POST |
| ShipmentsPage | /private/shipments/:id | DELETE |

---

## Control de Acceso por Roles

| Rol | Acceso |
| -- | -- |
| `admin` | Todo |
| `despacho` | Shipments (CRUD), Products (lectura) |
| `produccion` | Inventario (agregar stock), Login |

---

## Estado: Local vs Global

| Tipo | Dónde | Ejemplo |
| -- | -- | -- |
| **Global** (Zustand) | auth.store.ts | tokens, user |
| **Global** (Zustand) | ui.store.ts | sidebar, toasts |
| **Query Cache** (React Query) | useCustomersQuery | customers, products, inventario, shipments |
| **Local** | useState en página | Forms de creación/validación |

---

## Pendientes

Ninguno. Todos los endpoints definidos en API.md están cubiertos.

---

## Changelog

- 2024-01-15 - Actualizado con Products e Inventario basados en API.md