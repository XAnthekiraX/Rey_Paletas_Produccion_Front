# STATE_MANAGEMENT.md

> Estado de la aplicación para el frontend de Rey Paletas.
> Derivado exclusivamente de `docs/API.md`.

---

## Enfoque General

- Estado global mínimo
- Estado server manejado con cache
- Separación clara:
  - **server state**: datos de API (react-query)
  - **client state**: UI, formularios, interacciones (zustand)

---

## Librerías Recomendadas

| Propósito | Librería | Versión |
| -- | -- | -- |
| Server state | React Query (TanStack Query) | ^5 |
| Global state | Zustand | ^4 |
| Formularios | React Hook Form | ^7 |
| Validación | Zod | ^3 |

---

## Endpoints API

### Clasificación por Tipo

| Método | Endpoint | Tipo | Recurso | Roles |
| -- | -- | -- | -- | -- |
| POST | `/public/login` | Mutation | auth | all |
| POST | `/private/auth/refresh-token` | Mutation | auth | all |
| GET | `/private/customers` | Query | customers | admin, despacho |
| POST | `/private/customers` | Mutation | customers | admin |
| PUT | `/private/customers/:id` | Mutation | customers | admin |
| DELETE | `/private/customers/:id` | Mutation | customers | admin |
| GET | `/private/products` | Query | products | admin, despacho, produccion |
| POST | `/private/products` | Mutation | products | admin |
| PUT | `/private/products/:id` | Mutation | products | admin |
| DELETE | `/private/products/:id` | Mutation | products | admin |
| GET | `/private/inventario` | Query | inventario | admin, produccion |
| POST | `/private/inventario` | Mutation | inventario | admin, produccion |
| PUT | `/private/inventario/:id` | Mutation | inventario | admin |
| GET | `/private/shipments` | Query | shipments | admin, despacho |
| POST | `/private/shipments` | Mutation | shipments | admin, despacho |
| POST | `/private/validate_shipments/:id` | Mutation | shipments | admin, despacho |
| DELETE | `/private/shipments/:id` | Mutation | shipments | admin |
| GET | `/private/metricas` | Query | metricas | admin, despacho |

---

## Query Keys

```typescript
const queryKeys = {
  // Auth
  auth: ["auth"],
  authUser: () => ["auth", "user"],

  // Customers
  customers: () => ["customers"],
  customer: (id: string) => ["customers", id],

  // Products
  products: () => ["products"],
  product: (id: string) => ["products", id],

  // Inventario
  inventario: () => ["inventario"],
  inventarioItem: (id: string) => ["inventario", id],

  // Shipments
  shipments: () => ["shipments"],
  shipmentsByStatus: (status: string) => ["shipments", { status }],
  shipment: (id: string) => ["shipments", id],
};
```

---

## Queries (GET)

### GET /private/customers

```typescript
// Key: ["customers"]
// Stale time: 60 segundos

function useCustomersQuery() {
  return useQuery({
    queryKey: queryKeys.customers,
    queryFn: () => api.get("/customers"),
  });
}
```

### GET /private/products

```typescript
// Key: ["products"]
// Stale time: 60 segundos

function useProductsQuery() {
  return useQuery({
    queryKey: queryKeys.products,
    queryFn: () => api.get("/products"),
  });
}
```

### GET /private/inventario

```typescript
interface UseInventarioQueryProps {
  sort?: "quantity asc" | "quantity desc" | "updated_at asc" | "updated_at desc";
}

// Key: ["inventario"] o ["inventario", { sort }]

function useInventarioQuery(props?: UseInventarioQueryProps) {
  return useQuery({
    queryKey: props?.sort 
      ? ["inventario", { sort: props.sort }]
      : queryKeys.inventario(),
    queryFn: () => api.get("/inventario", { params: props }),
  });
}
```

### GET /private/shipments

```typescript
interface UseShipmentsQueryProps {
  status?: "por confirmar" | "confirmado" | "cancelado";
  search?: string;
}

// Key: ["shipments"] o ["shipments", { status, search }]

function useShipmentsQuery(props?: UseShipmentsQueryProps) {
  return useQuery({
    queryKey: props?.status || props?.search
      ? ["shipments", { status: props?.status, search: props?.search }]
      : queryKeys.shipments(),
    queryFn: () => api.get("/shipments", { params: props }),
  });
}
```

---

## Mutations

### Login

```typescript
// POST /public/login

interface LoginVariables {
  email: string;
  password: string;
}

function useLoginMutation() {
  return useMutation({
    mutationKey: ["auth", "login"],
    mutationFn: (variables: LoginVariables) => 
      api.post("/public/login", variables),
    onSuccess: (data) => {
      authStore.getState().setTokens(data.access_token, data.refresh_token);
      authStore.getState().setUser(data.user);
    },
  });
}
```

### Refresh Token

```typescript
// POST /private/auth/refresh-token

function useRefreshTokenMutation() {
  return useMutation({
    mutationKey: ["auth", "refreshToken"],
    mutationFn: () => {
      const refreshToken = authStore.getState().refresh_token;
      return api.post("/private/auth/refresh-token", { refresh_token: refreshToken });
    },
    onSuccess: (data) => {
      authStore.getState().setTokens(data.access_token, data.refresh_token);
    },
  });
}
```

### Create Customer

```typescript
// POST /private/customers

interface CreateCustomerVariables {
  name: string;
  city: string;
  document_number: string;
}

function useCreateCustomerMutation() {
  return useMutation({
    mutationKey: ["customers", "create"],
    mutationFn: (variables: CreateCustomerVariables) =>
      api.post("/customers", variables),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.customers() });
    },
  });
}
```

### Update Customer

```typescript
// PUT /private/customers/:id

interface UpdateCustomerVariables {
  id: string;
  name: string;
  city: string;
  document_number: string;
}

function useUpdateCustomerMutation() {
  return useMutation({
    mutationKey: ["customers", "update"],
    mutationFn: ({ id, ...data }: UpdateCustomerVariables) =>
      api.put(`/customers/${id}`, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: queryKeys.customer(id) });
      queryClient.invalidateQueries({ queryKey: queryKeys.customers() });
    },
  });
}
```

### Delete Customer

```typescript
// DELETE /private/customers/:id

function useDeleteCustomerMutation() {
  return useMutation({
    mutationKey: ["customers", "delete"],
    mutationFn: (id: string) => api.delete(`/customers/${id}`),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.customers() });
    },
  });
}
```

### Create Product

```typescript
// POST /private/products

interface CreateProductVariables {
  name: string;
}

function useCreateProductMutation() {
  return useMutation({
    mutationKey: ["products", "create"],
    mutationFn: (variables: CreateProductVariables) =>
      api.post("/products", variables),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.products() });
    },
  });
}
```

### Update Product

```typescript
// PUT /private/products/:id

interface UpdateProductVariables {
  id: string;
  name: string;
}

function useUpdateProductMutation() {
  return useMutation({
    mutationKey: ["products", "update"],
    mutationFn: ({ id, name }: UpdateProductVariables) =>
      api.put(`/products/${id}`, { name }),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: queryKeys.product(id) });
      queryClient.invalidateQueries({ queryKey: queryKeys.products() });
    },
  });
}
```

### Delete Product

```typescript
// DELETE /private/products/:id

function useDeleteProductMutation() {
  return useMutation({
    mutationKey: ["products", "delete"],
    mutationFn: (id: string) => api.delete(`/products/${id}`),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.products() });
    },
  });
}
```

### Add Stock

```typescript
// POST /private/inventario

interface AddStockVariables {
  items: Array<{ product_id: string; quantity: number }>;
  origin: "production" | "other";
  notes?: string;
}

function useAddStockMutation() {
  return useMutation({
    mutationKey: ["inventario", "add"],
    mutationFn: (variables: AddStockVariables) =>
      api.post("/inventario", variables),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.inventario() });
    },
  });
}
```

### Adjust Stock

```typescript
// PUT /private/inventario/:id

interface AdjustStockVariables {
  id: string;
  quantity: number;
}

function useAdjustStockMutation() {
  return useMutation({
    mutationKey: ["inventario", "adjust"],
    mutationFn: ({ id, quantity }: AdjustStockVariables) =>
      api.put(`/inventario/${id}`, { quantity }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.inventario() });
    },
  });
}
```

### Create Shipment

```typescript
// POST /private/shipments

interface CreateShipmentVariables {
  customer_id: string;
  items: Array<{ product_id: string; quantity: number }>;
  notes?: string;
}

function useCreateShipmentMutation() {
  return useMutation({
    mutationKey: ["shipments", "create"],
    mutationFn: (variables: CreateShipmentVariables) =>
      api.post("/shipments", variables),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.shipments() });
    },
  });
}
```

### Validate Shipment

```typescript
// POST /private/validate_shipments/:id

interface ValidateShipmentVariables {
  id: string;
  items: Array<{ product_id: string; quantity: number }>;
}

function useValidateShipmentMutation() {
  return useMutation({
    mutationKey: ["shipments", "validate"],
    mutationFn: ({ id, items }: ValidateShipmentVariables) =>
      api.post(`/validate_shipments/${id}`, { items }),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: queryKeys.shipment(id) });
      queryClient.invalidateQueries({ queryKey: queryKeys.shipments() });
    },
  });
}
```

### Cancel Shipment

```typescript
// DELETE /private/shipments/:id

function useCancelShipmentMutation() {
  return useMutation({
    mutationKey: ["shipments", "cancel"],
    mutationFn: (id: string) => api.delete(`/shipments/${id}`),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.shipments() });
    },
  });
}
```

---

## Invalidación de Caché

| Mutation | Queries a invalidar |
| -- | -- |
| createCustomer | `["customers"]` |
| updateCustomer | `["customers"]`, `["customers", id]` |
| deleteCustomer | `["customers"]` |
| createProduct | `["products"]` |
| updateProduct | `["products"]`, `["products", id]` |
| deleteProduct | `["products"]` |
| addStock | `["inventario"]` |
| adjustStock | `["inventario"]`, `["inventario", id]` |
| createShipment | `["shipments"]` |
| validateShipment | `["shipments"]`, `["shipments", id]` |
| cancelShipment | `["shipments"]` |

---

## Estado Global (Zustand)

### Auth Store

```typescript
interface AuthState {
  access_token: string | null;
  refresh_token: string | null;
  user: {
    id: string;
    email: string;
    role: "admin" | "produccion" | "despacho";
  } | null;
  isAuthenticated: boolean;
  
  setTokens: (access: string, refresh: string) => void;
  setUser: (user: AuthState["user"]) => void;
  logout: () => void;
  initialize: () => void;
}

const useAuthStore = create<AuthState>((set) => ({
  access_token: null,
  refresh_token: null,
  user: null,
  isAuthenticated: false,
  
  setTokens: (access_token, refresh_token) => {
    localStorage.setItem("access_token", access_token);
    localStorage.setItem("refresh_token", refresh_token);
    set({ access_token, refresh_token, isAuthenticated: true });
  },
  
  setUser: (user) => set({ user }),
  
  logout: () => {
    localStorage.removeItem("access_token");
    localStorage.removeItem("refresh_token");
    set({ access_token: null, refresh_token: null, user: null, isAuthenticated: false });
  },
  
  initialize: () => {
    const access_token = localStorage.getItem("access_token");
    const refresh_token = localStorage.getItem("refresh_token");
    if (access_token && refresh_token) {
      set({ access_token, refresh_token, isAuthenticated: true });
    }
  },
}));
```

### UI Store

```typescript
interface UIState {
  sidebarOpen: boolean;
  globalLoading: boolean;
  toasts: Array<{ id: string; message: string; type: "success" | "error" | "info" }>;
  
  toggleSidebar: () => void;
  setGlobalLoading: (loading: boolean) => void;
  addToast: (message: string, type: "success" | "error" | "info") => void;
  removeToast: (id: string) => void;
}

const useUIStore = create<UIState>((set, get) => ({
  sidebarOpen: true,
  globalLoading: false,
  toasts: [],
  
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setGlobalLoading: (globalLoading) => set({ globalLoading }),
  
  addToast: (message, type) => {
    const id = crypto.randomUUID();
    set((state) => ({ toasts: [...state.toasts, { id, message, type }] }));
    setTimeout(() => get().removeToast(id), 5000);
  },
  
  removeToast: (id) => set((state) => ({ 
    toasts: state.toasts.filter((t) => t.id !== id) 
  })),
}));
```

---

## Estado Local por Pantallas

### CreateShipmentForm

```typescript
interface CreateShipmentFormState {
  customer_id: string | null;
  items: Array<{ product_id: string; quantity: number }>;
  notes: string;
  step: "customer" | "items" | "review";
  
  setCustomer: (id: string) => void;
  addItem: (product_id: string, quantity: number) => void;
  removeItem: (product_id: string) => void;
  updateItemQuantity: (product_id: string, quantity: number) => void;
  setNotes: (notes: string) => void;
  setStep: (step: "customer" | "items" | "review") => void;
  reset: () => void;
}
```

### ValidateShipmentForm

```typescript
interface ValidateShipmentFormState {
  shipmentId: string;
  scannedItems: Array<{ product_id: string; quantity: number }>;
  isValidating: boolean;
  validationResult: {
    estado: "exito" | "error";
    message?: string;
    differences?: Array<{
      product_id: string;
      name: string;
      error: string;
      esperado?: number;
      recibido?: number;
    }>;
  } | null;
  
  addScannedItem: (product_id: string, quantity: number) => void;
  removeScannedItem: (product_id: string) => void;
  clearScannedItems: () => void;
  setValidating: (validating: boolean) => void;
  setResult: (result: ValidateShipmentFormState["validationResult"]) => void;
  reset: () => void;
}
```

### AddStockForm

```typescript
interface AddStockFormState {
  items: Array<{ product_id: string; quantity: number }>;
  origin: "production" | "other";
  notes: string;
  
  addItem: (product_id: string, quantity: number) => void;
  removeItem: (product_id: string) => void;
  updateItemQuantity: (product_id: string, quantity: number) => void;
  setOrigin: (origin: "production" | "other") => void;
  setNotes: (notes: string) => void;
  reset: () => void;
}
```

---

## Manejo de Autenticación

### Headers

```typescript
api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().access_token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

### Renovación Automática (401 → refresh)

```typescript
let isRefreshing = false;
let failedQueue: Array<{
  resolve: (value: unknown) => void;
  reject: (reason?: unknown) => void;
}> = [];

api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token) => {
          originalRequest.headers.Authorization = `Bearer ${token}`;
          return api(originalRequest);
        });
      }
      
      originalRequest._retry = true;
      isRefreshing = true;
      
      try {
        const refreshToken = useAuthStore.getState().refresh_token;
        const response = await api.post("/private/auth/refresh-token", { refresh_token });
        
        const { access_token, refresh_token } = response.data;
        useAuthStore.getState().setTokens(access_token, refresh_token);
        
        failedQueue.forEach((prom) => prom.resolve(access_token));
        failedQueue = [];
        
        originalRequest.headers.Authorization = `Bearer ${access_token}`;
        return api(originalRequest);
      } catch (err) {
        failedQueue.forEach((prom) => prom.reject(err));
        failedQueue = [];
        
        useAuthStore.getState().logout();
        window.location.href = "/login";
        
        return Promise.reject(err);
      } finally {
        isRefreshing = false;
      }
    }
    
    return Promise.reject(error);
  }
);
```

### Tiempos

| Token | Vigencia |
| -- | -- |
| access_token | 10 horas (36000 segundos) |
| refresh_token | No especificado |

---

## Protección de Rutas por Roles

### Roles

| Rol | Acceso |
| -- | -- |
| admin | Todo |
| produccion | Inventario, Productos (lectura) |
| despacho | Shipments, Customers (lectura), Productos (lectura) |

### Rutas

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

### Componente

```typescript
function ProtectedRoute({ children, allowedRoles }: { 
  children: React.ReactNode;
  allowedRoles: Array<"admin" | "produccion" | "despacho">;
}) {
  const user = useAuthStore((state) => state.user);
  
  if (!user || !allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return <>{children}</>;
}
```

---

## Configuración de React Query

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30 * 1000,
      gcTime: 5 * 60 * 1000,
      refetchOnWindowFocus: true,
      retry: 1,
    },
    mutations: {
      retry: 0,
    },
  },
});
```

### staleTime por Recurso

| Recurso | staleTime |
| -- | -- |
| Customers | 60 segundos |
| Products | 60 segundos |
| Inventario | 30 segundos |
| Shipments | 30 segundos |

---

## Estructura de Archivos

```
src/
├── api/
│   ├── client.ts
│   └── endpoints.ts
├── stores/
│   ├── useAuthStore.ts
│   └── useUIStore.ts
├── queries/
│   ├── keys.ts
│   ├── useCustomers.ts
│   ├── useProducts.ts
│   ├── useInventario.ts
│   └── useShipments.ts
├── mutations/
│   ├── useAuthMutations.ts
│   ├── useCustomerMutations.ts
│   ├── useProductMutations.ts
│   ├── useInventarioMutations.ts
│   └── useShipmentMutations.ts
├── hooks/
│   ├── useAuth.ts
│   └── usePermissions.ts
├── components/
│   └── ProtectedRoute.tsx
├── App.tsx
└── main.tsx
```

---

## Buenos Prácticas

1. **Separación Server vs Client State**: Datos de API en react-query, UI en zustand
2. **Minimizar Estado Global**: Solo auth y UI principal
3. **No Duplicar Lógica**: Backend valida permisos
4. **Invalidar Tras Mutations**: Siempre invalidar queries relacionadas
5. **Manejar 401 Globalmente**: Interceptor de axios

---

## Changelog

- 2024-01-15 - Actualizado con endpoints de Products e Inventario