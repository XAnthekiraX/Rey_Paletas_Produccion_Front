# PROJECT.md

> Proyecto basado exclusivamente en `docs/API.md`

---

## Nombre del Proyecto

Sistema de Gestión de Pedidos y Despachos - Rey Paletas

---

## Objetivo

Desarrollar un sistema que permita controlar de forma precisa:

- Gestión de productos (catálogo)
- Control de inventario (stock)
- Creación y gestión de pedidos (envíos)
- Validación de productos antes de despacho
- Gestión de clientes

Reduciendo errores operativos como:

- Envío de productos incorrectos
- Cantidades erróneas
- Falta de trazabilidad

---

## Problema que Resuelve

Actualmente el proceso es manual y propenso a errores:

- No hay verificación real entre pedido y despacho
- No existe validación antes del envío
- No hay control claro de inventario

El sistema introduce:

- Validación obligatoria de envíos
- Control de inventario en tiempo real
- Trazabilidad completa de pedidos

---

## Tipos de Usuarios

### Admin

- Control total del sistema
- Gestión de productos (CRUD)
- Gestión de clientes (CRUD)
- Gestión de inventario (ver, ajustar)
- Acceso a pedidos y validación

### Despacho

- Crear pedidos
- Listar pedidos
- Validar pedidos (escaneo)
- Ver productos (lectura)
- Ver clientes (lectura)

### Producción

- Agregar stock al inventario
- Ver inventario
- Ver productos (lectura)

---

## Flujo General del Sistema

1. Admin crea productos (catálogo)
2. Producción registra entrada de inventario
3. Admin/Producción ajusta stock si es necesario
4. Despacho crea pedido (selecciona cliente + productos)
5. Sistema crea pedido en estado "por confirmar"
6. Despacho escanea productos reales
7. Sistema valida: pedido vs productos escaneados
8. Si hay errores: se muestran diferencias
9. Si todo correcto: se confirma el envío y descuenta inventario

---

## Funcionalidades Principales (según API)

| Funcionalidad | Endpoint | Rol |
| -- | -- | -- |
| Autenticación | POST /public/login | all |
| Renovación token | POST /private/auth/refresh-token | all |
| Listar clientes | GET /private/customers | admin, despacho |
| Crear cliente | POST /private/customers | admin |
| Editar cliente | PUT /private/customers/:id | admin |
| Eliminar cliente | DELETE /private/customers/:id | admin |
| Listar productos | GET /private/products | admin, despacho, produccion |
| Crear producto | POST /private/products | admin |
| Editar producto | PUT /private/products/:id | admin |
| Eliminar producto | DELETE /private/products/:id | admin |
| Listar inventario | GET /private/inventario | admin, produccion |
| Agregar stock | POST /private/inventario | admin, produccion |
| Ajustar stock | PUT /private/inventario/:id | admin |
| Listar pedidos | GET /private/shipments | admin, despacho |
| Crear pedido | POST /private/shipments | admin, despacho |
| Validar pedido | POST /private/validate_shipments/:id | admin, despacho |
| Cancelar pedido | DELETE /private/shipments/:id | admin |
| Ver métricas | GET /private/metricas | admin, despacho |

---

## Recursos

### Customers (Clientes)

- CRUD completo
- Datos: name, city, document_number
- Admin: CRUD completo
- Despacho: solo lectura

### Products (Productos)

- CRUD completo
- Datos: name, barcode, barcode_url
- Auto-generación de barcode
- Admin: CRUD completo
- Despacho: solo lectura
- Producción: solo lectura

### Inventario

- Lista con stock
- Agregar stock (con origen y notas)
- Ajustar stock (+/-)
- Admin y producción

### Shipments (Pedidos)

- Crear, listar, filtrar
- Validar con escaneo
- Cancelar (solo admin)
- Estados: por confirmar, confirmado, cancelado

---

## Reglas Clave

- Ningún envío puede confirmarse sin validación
- El inventario se afecta automáticamente al validar/cancelar
- Solo admin puede gestionar productos y clientes
- Producción solo puede agregar stock y ver productos
- Despacho opera: crear, listar, validar pedidos (no puede cancelar)
- Despacho puede ver clientes y productos (solo lectura)

---

## Estados de Pedido

| Estado | Descripción | Acciones disponibles |
| -- | -- | -- |
| `por confirmar` | Pedido creado, pendiente validar | validar, cancelar |
| `confirmado` | Validado y stock descontado | cancelar |
| `cancelado` | Cancelado | ninguna |

---

## Prioridades del Sistema

1. Precisión en las validaciones
2. Velocidad en flujo de despacho
3. Control de inventario
4. Simplicidad de uso
5. Trazabilidad de pedidos

---

## Changelog

- 2024-01-15 - Actualizado con endpoints de Products e Inventario