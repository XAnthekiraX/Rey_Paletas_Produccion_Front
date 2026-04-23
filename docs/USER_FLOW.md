# USER_FLOW.md

## Roles del sistema

### Admin

Acceso total a todos los endpoints.
Puede crear, editar, eliminar y consultar cualquier recurso.

### Despacho

Acceso a:

- Crear envíos
- Consultar datos necesarios (productos, clientes, inventario)
- Validar envíos

Restricciones:

- No puede modificar ni eliminar registros existentes

### Producción

Acceso a:

- Consultar inventario
- Registrar entradas de inventario

Restricciones:

- No puede modificar ni eliminar registros existentes

---

## Flujo: Admin

1. Login
2. Acceso al dashboard
3. Gestión completa:
   - Productos (CRUD)
   - Clientes (CRUD)
   - Inventario (ajustes)
   - Envíos (visualización, eliminación si aplica)
4. Monitoreo de métricas

---

## Flujo: Despacho (Core del sistema)

### 1. Login

1. Acceso al dashboard

### 2. Crear Envío

1. Seleccionar cliente
2. Crear pedido base (estructura del envío)
3. Ingresar productos esperados (pedido)

### 3. Proceso de despacho (escaneo)

1. Escanear productos (o ingreso manual)
2. Construir lista de productos reales escaneados

### 4. Validación

1. Comparar:
   - Productos esperados (pedido)
   - Productos escaneados

2. Resultado:
   - Si coincide:
     - Confirmar envío
     - Registrar envío final

   - Si no coincide:
     - Detectar:
       - Faltantes
       - Excedentes
       - Productos incorrectos
     - Corregir:
       - Agregar faltantes
       - Remover incorrectos
     - Revalidar

3. Confirmación final del envío

---

## Flujo: Producción

### 1. Login

### 2. Registro de inventario

1. Escanear productos o ingreso manual
2. Registrar cantidades
3. Enviar registro al sistema

Resultado:

- Actualización del inventario

---

## Reglas clave del sistema

- Solo Admin puede usar:
  - PUT
  - DELETE

- Despacho y Producción:
  - Solo GET y POST

- Validación de envíos es obligatoria antes de confirmar

- El inventario debe reflejar:
  - Entradas (producción)
  - Salidas (despacho)

---

## Consideraciones críticas

- El flujo de despacho debe ser rápido y tolerante a errores
- El escaneo debe permitir correcciones en tiempo real
- La validación debe ser clara y explícita
- El sistema debe prevenir confirmaciones incorrectas
