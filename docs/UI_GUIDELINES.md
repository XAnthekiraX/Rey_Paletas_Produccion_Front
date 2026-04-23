# UI_GUIDELINES.md

## Enfoque general

- UI operativa, rápida y sin fricción
- Flujo continuo para escaneo
- Soporte dual:
  - ingreso manual
  - escaneo (input automático)

---

## Stack UI

- TailwindCSS v4.2
- Iconos: @iconify/react

---

## Sistema de colores

Definidos como variables (customizables):

- primary
- secondary
- tertiary
- quaternary

Uso:

- primary → acciones principales
- secondary → acciones secundarias
- tertiary → fondos / superficies
- quaternary → estados / acentos

---

## Layout

### AppLayout

- Sidebar (por rol)
- Header
- Contenido principal

---

## Componentes base

### Button

- primary
- secondary
- ghost
- destructive

---

### Input

#### Tipos

- text
- number
- scanner

#### Scanner behavior

- autofocus persistente
- captura inmediata
- limpia input después de cada lectura

---

## Patrón clave: "Carrito de productos"

Aplicado en:

- Crear Envío
- Validar Envío
- Registro de Inventario

---

### Estructura visual

Zona superior:

- Scanner activo
- Botón: "Ingreso manual" (toggle)

Zona principal:

- Lista tipo carrito

---

### Item del carrito

- nombre producto
- cantidad
- controles:
  - incrementar
  - decrementar
  - eliminar

---

### Comportamiento

- Escaneo:
  - si producto existe → incrementa cantidad
  - si no existe → agrega nuevo item

- Manual:
  - búsqueda de producto
  - selección + cantidad

---

## Toggle: Escaneo vs Manual

### Requisito

- Botón visible:
  - "Modo manual"
  - "Modo escaneo"

### Comportamiento

- Escaneo activo:
  - input visible y enfocado
  - formulario oculto

- Manual activo:
  - formulario visible
  - scanner desactivado

---

## Formularios con ocultamiento

Aplicar en:

- Crear Envío
- Validar Envío
- Inventario

---

### Estados UI

- showScanner = true
- showForm = false

Toggle:

- alterna entre ambos modos

---

## Validación de Envíos (UI crítica)

### Vista tipo comparación

Producto | Esperado | Escaneado | Estado

Estados:

- correcto
- faltante
- sobrante
- incorrecto

---

### Feedback visual

- correcto → indicador claro
- error → resaltado fuerte
- diferencias visibles sin scroll excesivo

---

## Tablas

- Filtros
- Búsqueda
- Acciones por fila

---

## Feedback

- toast success / error
- indicadores en tiempo real al escanear

---

## Accesibilidad operativa

- navegación por teclado
- foco automático en scanner
- botones grandes

---

## Consideraciones críticas

- Nunca perder foco del scanner
- No bloquear UI durante escaneo
- Minimizar clicks
- Carrito siempre visible
- Feedback inmediato por cada acción
