# FRONTEND_SPEC.md

## Arquitectura general

- Tipo: SPA (Single Page Application)
- Framework sugerido: Vite + React + TypeScript
- Routing: basado en roles
- Protección de rutas según permisos

---

## Layouts

### AuthLayout

- Usado en: Login
- Sin navegación lateral

### AppLayout

- Sidebar + Header
- Navegación por módulos según rol

---

## Rutas

### Públicas

- /login

### Privadas (según rol)

#### Admin

- /dashboard
- /products
- /customers
- /inventario
- /movimientos
- /shipments

#### Despacho

- /dashboard
- /shipments/create
- /shipments
- /shipments/:id/validate

#### Producción

- /inventario
- /inventario/create

---

## Pantallas

### Login

**Objetivo**
Autenticar usuario

**Componentes**

- Formulario (email/password)
- Botón login
- Manejo de error

**Estados**

- loading
- error

---

### Dashboard

**Objetivo**
Visualizar métricas

**API**

- GET /private/metricas

**Componentes**

- Cards de métricas
- Resumen general

**Estados**

- loading
- error
- success

---

### Productos (Admin)

#### Lista

**API**

- GET /private/products

**Componentes**

- Tabla
- Buscador
- Botón crear
- Acciones (editar/eliminar)

---

#### Crear / Editar

**API**

- POST /private/products
- PUT /private/products/:id

**Componentes**

- Formulario
- Inputs dinámicos

---

### Clientes (Admin)

#### Lista

**API**

- GET /private/customers

**Componentes**

- Tabla
- Buscador
- CRUD actions

---

#### Crear / Editar

**API**

- POST /private/customers
- PUT /private/customers/:id

---

### Inventario

#### Vista general

**API**

- GET /private/inventario

**Componentes**

- Tabla de stock
- Filtros

---

#### Registro (Producción)

**API**

- POST /private/inventario

**Componentes**

- Scanner input
- Input manual
- Lista de productos agregados
- Botón confirmar

---

### Movimientos

**API**

- GET /private/movimientos

**Componentes**

- Tabla histórica
- Filtros por fecha/producto

---

### Envíos

#### Lista

**API**

- GET /private/shipments

**Componentes**

- Tabla
- Estado del envío
- Acciones (ver, validar, eliminar si admin)

---

### Crear Envío (Despacho)

**API**

- POST /private/shipments

**Flujo UI**

1. Selección de cliente
2. Definición del pedido (productos esperados)
3. Zona de escaneo

**Componentes**

- Selector de cliente
- Lista de productos esperados
- Scanner input
- Lista de productos escaneados
- Indicador de diferencias

**Estados**

- editing
- validating
- error

---

### Validar Envío

**API**

- POST /private/validate_shipments/:id

**Componentes**

- Comparación visual:
  - Esperado vs Escaneado
- Indicadores:
  - faltantes
  - sobrantes
  - incorrectos
- Botón validar

---

### Detalle de Envío

**API**

- GET /private/shipments

**Componentes**

- Información del cliente
- Lista de productos
- Estado del envío

---

## Componentes reutilizables

- Table
- Form
- Input (normal + scanner)
- Modal
- Alert
- Badge (estado)
- Loader

---

## Estados globales

- Usuario autenticado
- Rol
- Token
- Configuración base (API URL)

---

## Manejo de errores

- Mostrar errores de API en UI
- Fallback visual en fallos críticos
- Reintentos en requests clave

---

## Navegación

- Sidebar dinámico según rol
- Rutas protegidas
- Redirección automática si no autorizado

---

## Consideraciones clave

- Flujo de escaneo debe ser rápido (input enfocado siempre)
- Validación visual clara (colores / estados)
- Evitar recargas innecesarias
- UX optimizada para uso operativo (no decorativa)
