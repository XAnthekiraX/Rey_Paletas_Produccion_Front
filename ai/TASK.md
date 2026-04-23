# Tareas del Proyecto - Rey Paletas Frontend

## Proyecto: Sistema de Gestión de Pedidos y Despachos

## Configuración Inicial

- [ ] 1. Inicializar proyecto con Vite + React + TypeScript
- [ ] 2. Instalar dependencias (react-router-dom, axios, TanStack Query, Zustand, React Hook Form, Zod, Tailwind CSS v4)
- [ ] 3. Configurar Tailwind con colores personalizados
- [ ] 4. Configurar ESLint y TypeScript
- [ ] 5. Crear estructura de carpetas (`src/`)
- [ ] 6. Configurar `.env` (API_URL)

## API Client

- [ ] 7. Crear cliente axios con interceptores (auth, 401 refresh)
- [ ] 8. Crear stores (authStore, uiStore)

## Autenticación

- [ ] 9. Crear página de Login
- [ ] 10. Implementar ProtectedRoute component
- [ ] 11. Configurar routing con autenticación

## Layouts

- [ ] 12. Crear AuthLayout (login)
- [ ] 13. Crear AppLayout (sidebar + header dinámico por rol)
- [ ] 14. Crear Navigation/Sidebar con permisos por rol

## Dashboard

- [ ] 15. Crear página Dashboard con métricas
- [ ] 16. Implementar query de métricas (GET /private/metricas)

## Módulo Customers

- [ ] 17. Crear página Customers (lista con búsqueda)
- [ ] 18. Crear modal/-formulario para crear/editar customer
- [ ] 19. Implementar queries y mutations de customers
- [ ] 20. Agregar protección de rutas (solo admin)

## Módulo Products

- [ ] 21. Crear página Products (lista con búsqueda)
- [ ] 22. Crear modal/-formulario para crear/editar producto
- [ ] 23. Implementar queries y mutations de products
- [ ] 24. Mostrar barcode/url del producto

## Módulo Inventario

- [ ] 25. Crear página Inventario (lista con stock)
- [ ] 26. Implementar query de inventario
- [ ] 27. Crear página Agregar Stock (solo produccion/admin)
- [ ] 28. Implementar mutation de agregar stock

## Módulo Shipments

- [ ] 29. Crear página Shipments (lista con filtros)
- [ ] 30. Implementar query de shipments con filtros
- [ ] 31. Crear página Nuevo Shipment (wizard: cliente → productos → review)
- [ ] 32. Implementar mutation de crear shipment
- [ ] 33. Crear página Validar Shipment (escaneo + validación)
- [ ] 34. Implementar mutation de validar shipment
- [ ] 35. Mostrar diferencias en validación (faltantes, sobrantes, incorrectos)
- [ ] 36. Implementar mutation de cancelar shipment (solo admin)

## UI Components

- [ ] 37. Crear Table component reutilizable
- [ ] 38. Crear Form components (Input, Select, etc)
- [ ] 39. Crear ScannerInput component (mantiene foco)
- [ ] 40. Crear Modal component
- [ ] 41. Crear Alert/Toast component
- [ ] 42. Crear Badge component para estados
- [ ] 43. Crear Loading/Spinner component

## Estilos

- [ ] 44. Aplicar colores del tema en toda la app
- [ ] 45. Crear tema claro con colores specified
- [ ] 46. Responsive design

## Testing

- [ ] 47. Configurar Vitest
- [ ] 48. Crear tests básicos para components
- [ ] 49. Crear tests para stores y queries

## Build & Deploy

- [ ] 50. Configurar build para producción
- [ ] 51. Crear scripts en package.json

---

## Colores del Tema

```css
--color-primary: #492360;      /* Púrpura oscuro */
--color-secondary: #fef0e7;    /* Crema claro */
--color-tertiary: #f5bf3b;     /* Amarillo dorado */
--color-quaternary: #e3023a;  /* Rojo intenso */
```

---

## Notas

- Proyecto conecta directamente al backend API (no Supabase en frontend)
- Backend maneja permisos por rol
- Frontend solo protege rutas según rol del usuario