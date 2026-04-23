# Documentación de APIs

> Endpoints del sistema Rey Paletas - Backend

---

## Autenticación

Endpoints públicos para gestión de sesiones de usuarios.

### Login

#### POST /public/login

Autentica al usuario y retorna tokens de sesión.

**Request:**

```json
{
  "email": "admin@ejemplo.com",
  "password": "password_seguro_123"
}
```

**Respuesta (200):**

```json
{
  "estado": "exito",
  "access_token": "eyJhbGciOiJFUzI1NiIsImtpZCI6IjljNmU",
  "refresh_token": "xksagklhkgb7e",
  "user": {
    "id": "45d21cdba8sd-55asd5-asd5as2das8s-0-c5688834c5ae",
    "email": "usuario@gmail.com",
    "role": "role"
  }
}
```

**Detalle de campos:**

| Campo           | Descripción                                   |
| --------------- | --------------------------------------------- |
| `access_token`  | Token de acceso (vigencia: 10 horas)          |
| `refresh_token` | Token para renovar acceso                     |
| `expires_in`    | Segundos hasta expiración (36000)             |
| `user.email`    | Email del usuario autenticado                 |
| `user.role`     | Rol del usuario (admin, produccion, despacho) |

**Códigos de error:**

| Código | Descripción                        |
| ------ | ---------------------------------- |
| 400    | Email o password no proporcionados |
| 401    | Credenciales inválidas             |

---

### Renovar Token

#### POST /private/auth/refresh-token

Renueva el access_token usando el refresh_token.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Request:**

```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Respuesta (200):**

```json
{
  "access_token": "nuevo_access_token...",
  "refresh_token": "nuevo_refresh_token...",
  "expires_in": 36000
}
```

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 401    | Token no proporcionado o inválido |

---

### Crear customer

#### POST /private/customers

Crea un nuevo customer.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Request:**

```json
{
  "name": "Juan Pérez",
  "city": "Bogotá",
  "document_number": "1234567890"
}
```

**Detalle de campos:**

| Campo             | Descripción                           |
| ----------------- | ------------------------------------- |
| `name`            | Nombre del customer (requerido)       |
| `city`            | Ciudad (requerido)                    |
| `document_number` | Número de documento único (requerido) |

**Respuesta (201):**

```json
{
  "id": "uuid-nuevo",
  "name": "Juan Pérez",
  "city": "Bogotá",
  "document_number": "1234567890",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 400    | Datos inválidos                   |
| 400    | Stock insuficiente                |
| 401    | Token no proporcionado o inválido |

**Error (400) - Stock insuficiente:**

```json
{
  "estado": "error",
  "mensaje": "Stock insuficiente",
  "stock_errors": [
    {
      "product_id": "uuid-producto-1",
      "name": "Paleta Mango",
      "requested": 150,
      "available": 30
    },
    {
      "product_id": "uuid-producto-2",
      "name": "Paleta Fresa",
      "requested": 150,
      "available": 50
    }
  ]
}
```

---

### Actualizar customer

#### PUT /private/customers/:id

Actualiza un customer existente.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Request:**

```json
{
  "name": "Juan Pérez Actualizado",
  "city": "Cali",
  "document_number": "1234567890"
}
```

**Respuesta (200):**

```json
{
  "id": "uuid-customer",
  "name": "Juan Pérez Actualizado",
  "city": "Cali",
  "document_number": "1234567890",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 400    | Datos inválidos                   |
| 401    | Token no proporcionado o inválido |
| 404    | Customer no encontrado            |

---

### Eliminar customer

#### DELETE /private/customers/:id

Elimina un customer del sistema.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Respuesta (204):**

```
(empty body)
```

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 401    | Token no proporcionado o inválido |
| 404    | Customer no encontrado            |

---

## Despachos

Gestión de pedidos/despachos. Accessible para admin y despacho.

**Prefijo:** `/private`

### Lista de estados

| Estado          | Descripción                             |
| --------------- | --------------------------------------- |
| `por confirmar` | Pedido creado, pendiente de validar     |
| `confirmado`    | Validado y stock descontado             |
| `cancelado`     | Cancelado (stockdevuelto si confirmado) |

### Crear pedido

#### POST /private/shipments

Crea un nuevo pedido en estado "por confirmar". Crea el shipment y los items relacionados.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Request:**

```json
{
  "customer_id": "uuid-del-customer",
  "items": [
    {
      "product_id": "uuid-producto-1",
      "quantity": 10
    },
    {
      "product_id": "uuid-producto-2",
      "quantity": 5
    }
  ],
  "notes": "Pedido para tienda central"
}
```

**Detalle de campos:**

| Campo                | Descripción                    |
| -------------------- | ------------------------------ |
| `customer_id`        | ID del customer (requerido)    |
| `items`              | Array de productos (requerido) |
| `items[].product_id` | ID del producto                |
| `items[].quantity`   | Cantidad solicitada            |
| `notes`              | Notas opcionales               |

**Respuesta (201):**

```json
{
  "estado": "exito",
  "id": "uuid-shipment",
  "destination": "Juan Pérez",
  "origin": "despacho",
  "status": "por confirmar",
  "notes": "Pedido para tienda central",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "items": [
    { "product_id": "uuid-producto-1", "name": "Paleta Mango", "quantity": 10 },
    { "product_id": "uuid-producto-2", "name": "Paleta Fresa", "quantity": 5 }
  ]
}
```

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 400    | Datos inválidos                   |
| 401    | Token no proporcionado o inválido |

---

### Listar pedidos

#### GET /private/shipments

Retorna todos los pedidos. Opcionalmente filtrar por estado o buscar por documento/nombre del customer.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Query params (opcionales):**

| Param    | Descripción                   | Ejemplo           |
| -------- | ----------------------------- | ----------------- |
| `status` | Filtro por estado             | confirmado        |
| `search` | Buscar por documento o nombre | Juan o 1234567890 |

**Ejemplos:**

```
GET /private/shipments
GET /private/shipments?status=por confirmar
GET /private/shipments?search=Juan
GET /private/shipments?search=1234567890
GET /private/shipments?status=confirmado&search=Juan
```

**Respuesta (200):**

```json
{
  "estado": "exito",
  "data": [
    {
      "id": "uuid-shipment",
      "destination": "Juan Pérez",
      "origin": "despacho",
      "status": "por confirmar",
      "notes": "Pedido #123",
      "created_at": "2024-01-15T10:30:00Z",
      "items": [
        { "product_id": "uuid-1", "name": "Paleta Mango", "quantity": 10 }
      ]
    },
    {
      "id": "uuid-shipment-2",
      "destination": "María García",
      "origin": "despacho",
      "status": "confirmado",
      "notes": "Pedido #124",
      "created_at": "2024-01-14T14:00:00Z",
      "items": [
        { "product_id": "uuid-2", "name": "Paleta Fresa", "quantity": 5 }
      ]
    }
  ]
}
```

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 401    | Token no proporcionado o inválido |

---

### Validar pedido

#### POST /private/validate_shipments/:id

**Path params:**

| Param | Descripción                 |
| ----- | --------------------------- |
| `:id` | UUID del shipment a validar |

Valida productos escaneados contra los items del pedido. Si todo correcto, cambia estado a "confirmado" y descuenta del inventario.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Request:**

```json
{
  "items": [
    { "product_id": "uuid-producto-1", "quantity": 10 },
    { "product_id": "uuid-producto-2", "quantity": 5 }
  ]
}
```

| Campo                | Descripción                              |
| -------------------- | ---------------------------------------- |
| `items`              | Array de productos validados (requerido) |
| `items[].product_id` | UUID del producto                        |
| `items[].quantity`   | Cantidad escaneada                       |

**Respuesta (200) - Validación exitosa:**

```json
{
  "estado": "exito",
  "message": "Pedido confirmado",
  "id": "uuid-shipment",
  "status": "confirmado"
}
```

**Respuesta (200) - Con diferencias:**

```json
{
  "estado": "error",
  "differences": [
    {
      "product_id": "uuid-producto-1",
      "name": "Paleta Mango",
      "error": "cantidad_incorrecta",
      "esperado": 30,
      "recibido": 50
    },
    {
      "product_id": "uuid-producto-2",
      "name": "Paleta Fresa",
      "error": "producto_faltante"
    },
    {
      "product_id": "uuid-producto-3",
      "name": "Paleta Coco",
      "error": "producto_no_esperado"
    }
  ]
}
```

| Campo        | Descripción                              |
| ------------ | ---------------------------------------- |
| `product_id` | UUID del producto                        |
| `name`       | Nombre del producto                      |
| `error`      | Tipo de error                            |
| `esperado`   | Cantidad esperada (cantidad_incorrecta)  |
| `recibido`   | Cantidad escaneada (cantidad_incorrecta) |

| Error                  | Descripción                                   |
| ---------------------- | --------------------------------------------- |
| `cantidad_incorrecta`  | Cantidad escaneada no coincide con esperada   |
| `producto_faltante`    | Producto esperado pero no fue escaneado       |
| `producto_no_esperado` | Producto escaneado que no estaba en el pedido |

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 400    | Datos inválidos                   |
| 401    | Token no proporcionado o inválido |
| 404    | Pedido no encontrado              |

---

### Cancelar pedido

#### DELETE /private/shipments/:id

Cancela un pedido. Si estaba confirmado, devuelve el stock al inventario.

**Headers:**

```
Authorization: Bearer <access_token>
```

**Respuesta (200) - Cancelado exitosamente:**

```json
{
  "estado": "exito",
  "message": "Pedido cancelado",
  "stock_returned": 15
}
```

**Detalle:**

| Estado inicial  | Acción                              |
| --------------- | ----------------------------------- |
| `por confirmar` | Solo cambia a cancelado             |
| `confirmado`    | Cambia a cancelado + devuelve stock |

**Códigos de error:**

| Código | Descripción                       |
| ------ | --------------------------------- |
| 401    | Token no proporcionado o inválido |
| 404    | Pedido no encontrado              |

---
