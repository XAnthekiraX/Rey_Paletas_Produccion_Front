# Mapeo de Datos Backend → Frontend

> Documento que mapea los endpoints del backend hacia su uso en el frontend.
> Toda la información aquí presentada se deriva exclusivamente de `docs/API.md`.

---

## Tabla de Contenidos

1. [Módulos del Sistema](#módulos-del-sistema)
2. [Estructuras de Datos Principales](#estructuras-de-datos-principales)
3. [Endpoint → Pantalla](#endpoint--pantalla)
4. [Autenticación](#autenticación)
5. [Customers](#customers)
6. [Products](#products)
7. [Inventario](#inventario)
8. [Shipments (Despachos)](#shipments-despachos)
9. [Transformaciones Necesarias](#transformaciones-necesarias)
10. [Reglas de Frontend](#reglas-de-frontend)
11. [Dependencias entre Datos](#dependencias-entre-datos)

---

## Módulos del Sistema

| Módulo | Prefijo | Roles permitidos |
| ------ | -- | -- |
| Autenticación | `/public` | Público |
| Customers | `/private` | Admin |
| Products | `/private` | Admin |
| Inventario | `/private` | Admin, producción |
| Shipments | `/private` | Admin, despacho |

---

## Estructuras de Datos Principales

### User

```typescript
interface User {
  id: string;
  email: string;
  role: "admin" | "produccion" | "despacho";
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `id` | string (UUID) | Sí | Identificador único del usuario |
| `email` | string | Sí | Email del usuario |
| `role` | string | Sí | Rol del usuario |

### Customer

```typescript
interface Customer {
  id: string;
  name: string;
  city: string;
  document_number: string;
  created_at: string;
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `id` | string (UUID) | Sí (respuesta) | Identificador único |
| `name` | string | Sí | Nombre del customer |
| `city` | string | Sí | Ciudad |
| `document_number` | string | Sí | Número de documento único |
| `created_at` | string (ISO 8601) | Sí (respuesta) | Fecha de creación |

### Product

```typescript
interface Product {
  id: string;
  name: string;
  barcode: string;
  barcode_url: string;
  created_at: string;
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `id` | string (UUID) | Sí | Identificador único |
| `name` | string | Sí | Nombre del producto |
| `barcode` | string | Sí | Código de barras |
| `barcode_url` | string | Sí | URL de imagen del barcode |
| `created_at` | string (ISO 8601) | Sí | Fecha de creación |

### Inventario

```typescript
interface Inventario {
  id: string;
  product_id: string;
  quantity: number;
  products: {
    name: string;
    barcode: string;
  };
  updated_at: string;
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `id` | string (UUID) | Sí | ID del registro de inventario |
| `product_id` | string (UUID) | Sí | ID del producto |
| `quantity` | number | Sí | Cantidad en stock |
| `products.name` | string | Sí | Nombre del producto |
| `products.barcode` | string | Sí | Barcode del producto |
| `updated_at` | string (ISO 8601) | Sí | Última actualización |

### StockItem (para agregar/ajustar)

```typescript
interface StockItem {
  product_id: string;
  quantity: number;
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `product_id` | string (UUID) | Sí | ID del producto |
| `quantity` | number | Sí | Cantidad a agregar/ajustar |

### ShipmentItem

```typescript
interface ShipmentItem {
  product_id: string;
  name: string;
  quantity: number;
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `product_id` | string (UUID) | Sí | ID del producto |
| `name` | string | No (respuesta) | Nombre del producto |
| `quantity` | number | Sí | Cantidad |

### Shipment

```typescript
interface Shipment {
  id: string;
  destination: string;
  origin: string;
  status: "por confirmar" | "confirmado" | "cancelado";
  notes?: string;
  created_at: string;
  updated_at: string;
  items: ShipmentItem[];
}
```

| Campo | Tipo | Requerido | Descripción |
| -- | -- | -- | -- |
| `id` | string (UUID) | Sí | ID del shipment |
| `destination` | string | Sí | Nombre del customer destino |
| `origin` | string | Sí | Origen ("despacho") |
| `status` | string | Sí | Estado del pedido |
| `notes` | string | No | Notas adicionales |
| `created_at` | string (ISO 8601) | Sí | Fecha de creación |
| `updated_at` | string (ISO 8601) | Sí | Fecha de actualización |
| `items` | ShipmentItem[] | Sí | Lista de productos |

### ValidatedItem (para validación)

```typescript
interface ValidatedItem {
  product_id: string;
  quantity: number;
}
```

### ShipmentDifference

```typescript
interface ShipmentDifference {
  product_id: string;
  name: string;
  error: "cantidad_incorrecta" | "producto_faltante" | "producto_no_esperado";
  esperado?: number;
  recibido?: number;
}
```

### StockError

```typescript
interface StockError {
  product_id: string;
  name: string;
  requested: number;
  available: number;
}
```

---

## Endpoint → Pantalla

### Auth

| Método | Endpoint | Pantalla | Descripción |
| -- | -- | -- | -- |
| POST | `/public/login` | Login | Autentica usuario y retorna tokens |
| POST | `/private/auth/refresh-token` | (interno) | Renueva access_token |

### Customers

| Método | Endpoint | Pantalla | Descripción |
| -- | -- | -- | -- |
| GET | `/private/customers` | Customers | Lista customers |
| POST | `/private/customers` | Customers (crear) | Crea nuevo customer |
| PUT | `/private/customers/:id` | Customers (editar) | Actualiza customer |
| DELETE | `/private/customers/:id` | Customers (eliminar) | Elimina customer |

### Products

| Método | Endpoint | Pantalla | Descripción |
| -- | -- | -- | -- |
| GET | `/private/products` | Products | Lista productos |
| POST | `/private/products` | Products (crear) | Crea nuevo producto |
| PUT | `/private/products/:id` | Products (editar) | Actualiza producto |
| DELETE | `/private/products/:id` | Products (eliminar) | Elimina producto |

### Inventario

| Método | Endpoint | Pantalla | Descripción |
| -- | -- | -- | -- |
| GET | `/private/inventario` | Inventario | Lista inventario |
| POST | `/private/inventario` | Inventario (agregar) | Agrega stock |
| PUT | `/private/inventario/:id` | Inventario (ajustar) | Ajusta stock |

### Shipments

| Método | Endpoint | Pantalla | Descripción |
| -- | -- | -- | -- |
| POST | `/private/shipments` | Nuevo Pedido | Crea nuevo pedido |
| GET | `/private/shipments` | Lista Pedidos | Lista pedidos con filtros |
| POST | `/private/validate_shipments/:id` | Validar Pedido | Valida productos escaneados |
| DELETE | `/private/shipments/:id` | Lista Pedidos | Cancela pedido |

---

## Autenticación

### POST /public/login

**Pantalla:** Login

**Request:**

```typescript
interface LoginRequest {
  email: string;
  password: string;
}
```

**Response (200):**

```typescript
interface LoginResponse {
  estado: "exito";
  access_token: string;
  refresh_token: string;
  user: {
    id: string;
    email: string;
    role: string;
  };
}
```

**Uso en UI:**

- Guardar `access_token` y `refresh_token`
- Guardar `user` (id, email, role)
- Redireccionar según rol

---

### POST /private/auth/refresh-token

**Pantalla:** (interno)

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```typescript
interface RefreshTokenRequest {
  refresh_token: string;
}
```

**Response (200):**

```typescript
interface RefreshTokenResponse {
  access_token: string;
  refresh_token: string;
  expires_in: number;
}
```

---

## Customers

### GET /private/customers

**Pantalla:** Customers

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```typescript
interface CustomersResponse {
  estado: "exito";
  data: Customer[];
}
```

---

### POST /private/customers

**Pantalla:** Customers - Crear

**Request:**

```typescript
interface CreateCustomerRequest {
  name: string;
  city: string;
  document_number: string;
}
```

**Response (201):**

```typescript
interface CustomerResponse {
  id: string;
  name: string;
  city: string;
  document_number: string;
  created_at: string;
}
```

---

### PUT /private/customers/:id

**Pantalla:** Customers - Editar

**Request:**

```typescript
interface UpdateCustomerRequest {
  name: string;
  city: string;
  document_number: string;
}
```

---

### DELETE /private/customers/:id

**Pantalla:** Customers - Eliminar

**Response (204):** `(empty body)`

---

## Products

### GET /private/products

**Pantalla:** Products

**Response (200):**

```typescript
interface ProductsResponse {
  estado: "exito";
  data: Product[];
}
```

---

### POST /private/products

**Pantalla:** Products - Crear

**Headers:** `Authorization: Bearer <access_token>` (admin)

**Request:**

```typescript
interface CreateProductRequest {
  name: string;
}
```

**Response (201):**

```typescript
interface ProductResponse {
  estado: "exito";
  message: "Producto creado";
  data: Product;
}
```

---

### PUT /private/products/:id

**Pantalla:** Products - Editar

**Request:**

```typescript
interface UpdateProductRequest {
  name: string;
}
```

---

### DELETE /private/products/:id

**Pantalla:** Products - Eliminar

**Response (200):**

```typescript
{
  estado: "exito";
  message: "Producto eliminado";
}
```

---

## Inventario

### GET /private/inventario

**Pantalla:** Inventario

**Query params (opcionales):**

| Param | Descripción | Ejemplo |
| -- | -- | -- |
| `sort` | Ordenar por campo | `quantity desc` |

**Response (200):**

```typescript
interface InventarioResponse {
  estado: "exito";
  data: Inventario[];
}
```

---

### POST /private/inventario

**Pantalla:** Inventario - Agregar Stock

**Headers:** `Authorization: Bearer <access_token>` (admin, produccion)

**Request:**

```typescript
interface AddStockRequest {
  items: StockItem[];
  origin: "production" | "other";
  notes?: string;
}
```

**Response (201):**

```typescript
{
  estado: "exito";
  message: "Stock registrado";
  data: Array<{
    product_id: string;
    quantity_added: number;
  }>;
}
```

---

### PUT /private/inventario/:id

**Pantalla:** Inventario - Ajustar Stock

**Request:**

```typescript
interface AdjustStockRequest {
  quantity: number;
}
```

> Nota: Positivo para sumar, negativo para restar.

**Response (200):**

```typescript
{
  estado: "exito";
  message: "Stock actualizado";
  data: Inventario;
}
```

---

## Shipments

### Estados de Pedido

| Estado | Descripción | Acciones disponibles |
| -- | -- | -- |
| `por confirmar` | Pedido creado, pendiente de validar | validar, cancelar |
| `confirmado` | Validado y stock descontado | cancelar |
| `cancelado` | Cancelado (stockdevuelto si confirmado) | ninguna |

---

### POST /private/shipments

**Pantalla:** Nuevo Pedido

**Request:**

```typescript
interface CreateShipmentRequest {
  customer_id: string;
  items: Array<{ product_id: string; quantity: number }>;
  notes?: string;
}
```

**Response (201):**

```typescript
interface ShipmentResponse {
  estado: "exito";
  id: string;
  destination: string;
  origin: string;
  status: "por confirmar";
  notes: string;
  created_at: string;
  updated_at: string;
  items: ShipmentItem[];
}
```

---

### GET /private/shipments

**Pantalla:** Lista Pedidos

**Query params (opcionales):**

| Param | Descripción | Ejemplo |
| -- | -- | -- |
| `status` | Filtro por estado | `confirmado` |
| `search` | Buscar por documento o nombre | `Juan` |

**Response (200):**

```typescript
interface ShipmentsListResponse {
  estado: "exito";
  data: Shipment[];
}
```

---

### POST /private/validate_shipments/:id

**Pantalla:** Validar Pedido (Escaneo)

**Request:**

```typescript
interface ValidateShipmentRequest {
  items: ValidatedItem[];
}
```

**Response (200) - Éxito:**

```typescript
{
  estado: "exito";
  message: "Pedido confirmado";
  id: string;
  status: "confirmado";
}
```

**Response (200) - Con diferencias:**

```typescript
{
  estado: "error";
  differences: ShipmentDifference[];
}
```

| Error | Descripción |
| -- | -- |
| `cantidad_incorrecta` | Cantidad escaneada no coincides |
| `producto_faltante` | Producto esperado no escaneado |
| `producto_no_esperado` | Producto escaneado no en pedido |

---

### DELETE /private/shipments/:id

**Pantalla:** Lista Pedidos (Cancelar)

**Response (200):**

```typescript
{
  estado: "exito";
  message: "Pedido cancelado";
  stock_returned: number;
}
```

**Comportamiento:**

| Estado inicial | Acción |
| -- | -- |
| `por confirmar` | Solo cambia a cancelado |
| `confirmado` | Cambia a cancelado + devuelve stock |

---

## Transformaciones Necesarias

### Fechas

```typescript
function formatDate(dateString: string): string {
  return new Date(dateString).toLocaleDateString("es-CO");
}

function formatDateTime(dateString: string): string {
  return new Date(dateString).toLocaleString("es-CO", {
    dateStyle: "short",
    timeStyle: "short"
  });
}
```

### Estados de Pedido

```typescript
const statusConfig = {
  "por confirmar": { label: "Por Confirmar", color: "warning" },
  "confirmado": { label: "Confirmado", color: "success" },
  "cancelado": { label: "Cancelado", color: "error" }
};
```

### Estados de Stock

```typescript
const originConfig = {
  "production": { label: "Producción", color: "info" },
  "other": { label: "Otro", color: "neutral" }
};
```

---

## Reglas de Frontend

### Validaciones Antes de Enviar

| Recurso | Campos requeridos |
| -- | -- |
| Login | email, password |
| Customer | name, city, document_number |
| Product | name |
| Stock | items (mínimo 1), origin |
| Shipment | customer_id, items (mínimo 1) |
| Validate | items |

### Datos que NO deben confiarse al Frontend

| Dato | Razón |
| -- | -- |
| access_token completo | JWT puede ser decodificado |
| refresh_token | Manejar en httpOnly cookies |
| Contraseñas | Nunca almacenar |

---

## Dependencias entre Datos

### Entidades y Relaciones

```
User (1)
  └── Customer (N)
        └── Shipment (N)

Product (1)
  ├── Inventario (1)
  └── ShipmentItem (N)
```

### Dependencias

| Entidad | Depende de | Cardinalidad |
| -- | -- | -- |
| Shipment | Customer | N:1 |
| ShipmentItem | Product | N:1 |
| Inventario | Product | N:1 |

### Orden de Carga

1. User (login)
2. Products (para inventario y shipments)
3. Customers (para shipments)
4. Inventario (para stock)
5. Shipments (pedidos)

---

## Changelog

- 2024-01-15 - Documento basado en API.md actualizado
- Agregados: Products e Inventario