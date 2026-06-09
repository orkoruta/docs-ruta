# RUTA — Plan de Desarrollo Fase 2: Cliente API (Logística-as-a-Service)

Alineado con `mvp_alcance.md` (sección Fase 2) y `all_ruta.md` (sección 20).
**Inicio estimado:** tras deploy a producción de Fase 1 (6.INFRA-4).
**Duración estimada:** 4-5 días de trabajo paralelo (4-6 semanas calendario).

---

## Hipótesis a validar

Una empresa con su propia plataforma de venta puede integrar RUTA en un día
y usarlo como su backend logístico: crea pedidos via `POST /api/orders` con
una API key, recibe webhooks cuando el estado cambia, y el resto del flujo
logístico (mapa de asignación, courier, PICKUP) funciona igual que en Fase 1.

## Criterio de salida de Fase 2

- 1 Cliente API piloto integrado y operando en producción.
- 10+ pedidos reales procesados via API key.
- Tiempo de integración de nuevo Cliente API: < 1 día.
- Webhook reliability > 99%.
- Cero violaciones de aislamiento multi-tenant.

---

## Convenciones

### Notación de tareas

`F2.[WAVE].[TRACK]-[N] — Título`

- **WAVE:** 1 a 4.
- **TRACK:** `SHARED`, `BACK`, `ADMIN`, `QA`.

### Estimaciones

- **S** = Small (1-4h) · **M** = Medium (4h-1d) · **L** = Large (1-3d)

### Estados

`[ ]` pendiente · `[/]` en progreso · `[x]` completado · `[-]` cancelado

### Definition of Done

Igual que Fase 1: ningún avance se da por terminado sin verificación.

- Tipos/validators: typecheck + unit tests.
- Backend/API: typecheck + unit + integration tests (DB, auth, tenant, estados).
- Frontend: typecheck/build + prueba visual o E2E del flujo.
- Si una prueba no puede correr, registrar explícitamente por qué.

### Regla de no-avance

> No avanzar a la siguiente wave si existen: errores, warnings críticos,
> inconsistencias, conflictos de archivos, pruebas fallidas o comportamiento
> incompleto en la wave anterior.

---

## Qué reutiliza Fase 2 de Fase 1 (sin cambios)

- Tabla `client_api_keys` — ya existe en BD y schema Prisma.
- Campo `client_type = 'API'` en `clients` — ya existe.
- `actor_api_key_id` en `audit_events` — ya existe.
- `TransitionContext.clientType` en el state machine — ya existe.
- Código `LOGISTICS_ONLY_FEATURE_UNAVAILABLE` — ya existe (parcialmente).
- Flujo SHIP completo (Flujo 2) — backend + frontend completos.
- Flujo PICKUP completo (Flujo 3) — backend + frontend completos.
- Mapa de asignación de couriers — completo.
- Vista del repartidor — completa.
- Webhooks salientes pg-boss — infraestructura completa.
- Auditoría, multi-tenant, RLS — completos.

---

## Mapa anti-conflicto de archivos

| Archivo | SHARED-1 | BACK-1 | BACK-2 | BACK-3 | BACK-4 | BACK-5 | BACK-6 | ADMIN-1 | ADMIN-2 | ADMIN-3 | QA-1 | QA-2 |
|---------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| packages-ruta/shared/* | ✏️ | | | | | | | | | | | |
| api_keys.service.ts (NUEVO) | | ✏️ | | | | | | | | | | |
| admin_api_keys.ts (NUEVO) | | ✏️ | | | | | | | | | | |
| api_key_auth.ts (NUEVO) | | | ✏️ | | | | | | | | | |
| middleware/auth.ts | | | ✏️ | | | | | | | | | |
| state_machine.ts | | | | ✏️ | | | | | | | | |
| orders.service.ts | | | | ✏️ | | ✏️† | | | | | | |
| api_client_orders.ts (NUEVO) | | | | | ✏️ | | | | | | | |
| app.ts | | ✏️ | | | ✏️ | | | | | | | |
| admin_orders.ts | | | | | | ✏️ | | | | | | |
| buyer_orders.ts / buyer_payment.ts | | | | | | ✏️ | | | | | | |
| webhooks_outgoing.service.ts | | | | | | | ✏️ | | | | | |
| courier_ops.service.ts | | | | | | | ✏️ | | | | | |
| admin/api-keys/* (NUEVO) | | | | | | | | ✏️ | | | | |
| admin/orders/page.tsx | | | | | | | | | ✏️ | | | |
| admin/orders/[id]/page.tsx | | | | | | | | | ✏️ | | | |
| ruta-admin/clients/* | | | | | | | | | | ✏️ | | |

† F2.3.BACK-6 (webhooks) debe iniciar DESPUÉS del merge de F2.2.BACK-3 para evitar conflicto en `orders.service.ts`.

---

## Wave 1 — Prerequisito (1 agente, ~4h)

Debe completarse y publicarse antes de iniciar Wave 2.

---

### F2.1.SHARED-1 — Bump `@orkoruta/shared` con schemas de Fase 2 [M]

**Estado:** `[ ]` pendiente

**Objetivo.** Publicar `@orkoruta/shared@1.4.0` con los tipos y validators
necesarios para que back y front implementen Fase 2 tipadamente.

**Archivos a crear/modificar:**

- `packages-ruta/shared/src/validators/api_key.schema.ts` (NUEVO)
  - `createApiKeySchema`: `{ name, scopes, expires_at? }`
  - `apiKeyListQuerySchema`: paginación básica
  - `revokeApiKeySchema`: `{ reason? }`
- `packages-ruta/shared/src/validators/api_order.schema.ts` (NUEVO)
  - `createApiOrderSchema`: `{ external_reference?, delivery_type, initial_state, items[], delivery_address? | pickup_point_id?, payment_method, customer_info? }`
  - `apiOrderListQuerySchema`: filtros por status, from, to, page
- `packages-ruta/shared/src/types/api_key.types.ts` (NUEVO)
  - Tipos inferidos de los schemas anteriores
- `packages-ruta/shared/src/constants/error_codes.ts` (MODIFICAR)
  - Verificar/agregar: `API_KEY_INVALID`, `API_KEY_REVOKED`, `API_KEY_EXPIRED`, `SCOPE_NOT_ALLOWED`
  - `LOGISTICS_ONLY_FEATURE_UNAVAILABLE` ya debe existir — confirmar
- `packages-ruta/shared/src/index.ts` (MODIFICAR) — re-exportar los nuevos schemas
- `packages-ruta/shared/package.json` (MODIFICAR) — bump `1.3.0 → 1.4.0`

**Archivos que NO debe tocar:** `backend-ruta/`, `frontend-ruta/`, `packages-ruta/db/`.

**Dependencias previas:** ninguna.

**Scopes iniciales definidos:** `orders:write`, `orders:read`.
(Expandir en Fase 3 si se necesita más granularidad.)

**Criterios de aceptación:**
- `pnpm build` EXIT 0 en `packages-ruta`
- `pnpm test` EXIT 0 en `packages-ruta`
- `@orkoruta/shared@1.4.0` publicado en GitHub Packages (publicación manual con PAT)
- `pnpm add @orkoruta/shared` desde `backend-ruta` resuelve `1.4.0`

**Pruebas obligatorias:**
- Tests unitarios Zod: input válido → no lanza. Input inválido (falta campo, tipo incorrecto) → lanza con mensaje.
- Mínimo 1 test por schema nuevo.

**Riesgos de conflicto:** ninguno — único agente tocando `packages-ruta/shared`.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

## Wave 2 — Backend independiente (3 agentes en paralelo, ~4-8h c/u)

Inician después de confirmar que `@orkoruta/shared@1.4.0` está disponible.
ADMIN-2 puede iniciar en paralelo con esta wave (no depende de los nuevos paquetes).

---

### F2.2.BACK-1 — Gestión de API Keys: servicio + endpoints admin [L]

**Estado:** `[ ]` pendiente

→ depende de: F2.1.SHARED-1

**Objetivo.** ADMIN_CLIENT puede crear, listar y revocar API keys desde el
panel admin. El secreto solo se muestra una vez al crearla.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/api_keys.service.ts` (NUEVO)
  - `generateApiKey(clientId, input, actorUserId)` — genera `key_id` (`rk_` + cuid random), `secret` (hex 32 bytes), hashea `secret` con HMAC-SHA256 (no argon2 — es secreto máquina-a-máquina, no password humano), inserta en `client_api_keys`
  - `listApiKeys(clientId)` — retorna keys sin revelar `secret_hash`
  - `revokeApiKey(clientId, keyId, reason, actorUserId)` — pone `revoked_at`, audita
  - Audita en `audit_events` con `actor_user_id` (las acciones sobre keys son acciones de usuario humano)
- `backend-ruta/api/src/routes/admin_api_keys.ts` (NUEVO)
  - `GET /admin/api-keys` — lista (solo `ADMIN_CLIENT`, `ADMIN_RUTA`)
  - `POST /admin/api-keys` — crear; response incluye `{ key_id, secret, scopes, expires_at }` — secreto solo aquí
  - `DELETE /admin/api-keys/:key_id` — revocar
- `backend-ruta/api/src/app.ts` (MODIFICAR) — montar el nuevo router en `/admin/api-keys`

**Archivos que NO debe tocar:** `state_machine.ts`, `orders.service.ts`, `middleware/auth.ts`, cualquier route existente que no sea `app.ts`.

**Decisión de diseño — hashing:**
Usar HMAC-SHA256 (no argon2) para validar la API key en cada request. Razón: las API keys son secretos generados por el sistema (alta entropía), no passwords humanos. HMAC-SHA256 es suficientemente seguro y no introduce latencia en cada request.

**Criterios de aceptación:**
- `POST /admin/api-keys` → 201 con `{ key_id, secret, scopes }`. El campo `secret` nunca aparece en ningún otro response.
- `GET /admin/api-keys` → lista con `key_id`, `name`, `scopes`, `status` (ACTIVE/REVOKED/EXPIRED), `last_used_at`, sin `secret_hash`.
- `DELETE /admin/api-keys/:key_id` con key ya revocada → 409 `IDEMPOTENCY_CONFLICT`.
- ADMIN_CLIENT de otro tenant → 403 `FORBIDDEN`.
- `pnpm typecheck` EXIT 0, `pnpm test` EXIT 0.

**Pruebas obligatorias (mínimo 8 tests):**
- Crear key → aparece en lista → revocar → no aparece como activa.
- Revocar key ya revocada → 409.
- ADMIN_CLIENT de tenant B no ve keys de tenant A (aislamiento).
- Crear con `expires_at` en el pasado → validación rechaza.
- OPERATOR_CLIENT → 403 (no puede gestionar keys).
- Idempotencia en `POST` con mismo `X-Idempotency-Key`.

**Riesgos de conflicto:** `app.ts` también lo modifica F2.3.BACK-4. Merge de 2 líneas separadas — trivial.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

### F2.2.BACK-2 — Middleware de autenticación por API Key [M]

**Estado:** `[ ]` pendiente

→ depende de: F2.1.SHARED-1

**Objetivo.** Nuevo middleware que autentica requests de máquina-a-máquina
usando `Authorization: Bearer <key_id>.<secret>`, con validación HMAC-SHA256.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/middleware/api_key_auth.ts` (NUEVO)
  - Parsea `Authorization: Bearer <key_id>.<secret>` (separador: `.`, formato fijo)
  - Busca `client_api_keys` por `key_id`
  - Valida: `HMAC-SHA256(secret) === secret_hash`
  - Valida: `revoked_at IS NULL` → si no, 401 `API_KEY_REVOKED`
  - Valida: `expires_at > NOW()` si no es null → si no, 401 `API_KEY_EXPIRED`
  - Valida scope requerido (recibe scope como parámetro): si no está en `scopes[]` → 403 `SCOPE_NOT_ALLOWED`
  - Actualiza `last_used_at`, `last_used_ip` (fire-and-forget con setImmediate — no bloquea response)
  - Popula `req.user = { clientId, userType: 'API_CLIENT', apiKeyId, scopes }`
- `backend-ruta/api/src/middleware/auth.ts` (MODIFICAR mínimo)
  - Extender el type `AuthenticatedUser` para incluir `userType: 'API_CLIENT'` y `apiKeyId?: bigint`

**Archivos que NO debe tocar:** ningún route existente, ningún service existente. Solo crea el middleware y extiende el type.

**Criterios de aceptación:**
- Header válido, scope correcto → `req.user` populado correctamente, `last_used_at` actualizado en BD.
- Key inexistente → 401 `API_KEY_INVALID`.
- Secret incorrecto → 401 `API_KEY_INVALID` (mismo mensaje para no revelar si el key_id existe).
- Key revocada → 401 `API_KEY_REVOKED`.
- Key expirada → 401 `API_KEY_EXPIRED`.
- Scope faltante → 403 `SCOPE_NOT_ALLOWED`.
- Sin header `Authorization` → 401 `AUTHENTICATION_REQUIRED`.
- `pnpm typecheck` EXIT 0.

**Pruebas obligatorias (mínimo 8 unit tests):**
- Happy path con mock de BD y argon2/HMAC.
- Cada caso de error listado en criterios.
- Verificar que `last_used_at` se actualiza (spy en Prisma client).

**Riesgos de conflicto:** modifica solo `auth.ts` (3-5 líneas al type). Sin conflicto con otros agentes.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

### F2.2.BACK-3 — State Machine + createApiClientOrder() [L]

**Estado:** `[ ]` pendiente

→ depende de: F2.1.SHARED-1

**Objetivo.** Extender el state machine para soportar la creación directa
de pedidos desde un Cliente API en estado avanzado (saltando Flujo 1 completo),
e implementar la función de creación.

**Diseño de `initial_state`:**
El Cliente API elige en qué estado entra el pedido según su caso de uso:
- `PREPARING` → ya preparó el pedido, solo necesita asignación de courier.
- `AWAITING_COURIER_ASSIGNMENT` → ya está listo para despacho.
- `READY_TO_SHIP` → tiene courier externo y solo necesita seguimiento.
- `READY_FOR_PICKUP` → el cliente va a un punto físico a recoger.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/orders/state_machine.ts` (MODIFICAR)
  - Agregar actor `'API_CLIENT'` al type `TransitionActor`.
  - Las transiciones logísticas existentes (PREPARING → AWAITING_COURIER_ASSIGNMENT, etc.) deben aceptar `'API_CLIENT'` como actor donde corresponda.
  - No agregar un estado ficticio `INITIAL_API`; la creación directa bypasea el state machine para el INSERT inicial, pero las transiciones posteriores usan el machine normal.
- `backend-ruta/api/src/services/orders/orders.service.ts` (MODIFICAR — agregar función)
  - `createApiClientOrder(clientId, input, apiKeyId)`:
    - Valida `client_type === 'API'` del cliente (si es FULL → error).
    - Valida input con `createApiOrderSchema` de `@orkoruta/shared`.
    - Resuelve `initial_status` del campo `input.initial_state`.
    - Inserta en `orders` directamente con el estado correcto (no pasa por DRAFT).
    - Inserta en `order_state_history` con el estado inicial y `actor = 'API_CLIENT'`.
    - Inserta `audit_event` con `actor_api_key_id`.
    - Llama a `emitWebhookIfSubscribed('ORDER_CREATED', ...)` — si hay suscripción.
    - Retorna el pedido creado serializado.
  - Los productos del pedido API son **flat data** en el payload (no referencias al catálogo de RUTA). Se guardan como `order_items` con los campos provistos.
  - `payment_method` válidos: `PREPAID_EXTERNAL` (ya pagado en plataforma del cliente) o `COD` (contra entrega).
  - Campo adicional en `orders`: `order_source = 'API'` (o usar campo existente si hay).

**Archivos que NO debe tocar:** ningún route, `admin_orders.ts`, `buyer_orders.ts`, `api_keys.service.ts`.

**Criterios de aceptación:**
- `createApiClientOrder()` con `delivery_type=SHIP, initial_state=PREPARING` → pedido en `PREPARING`.
- `createApiClientOrder()` con `delivery_type=PICKUP` → pedido en `READY_FOR_PICKUP`.
- Cliente FULL intentando usar esta función → error `VALIDATION_ERROR`.
- `order_state_history` tiene la transición registrada con actor `'API_CLIENT'`.
- `audit_events.actor_api_key_id` correcto.
- `pnpm typecheck` EXIT 0, tests relacionados pasan.

**Pruebas obligatorias (mínimo 8 tests):**
- Cada `initial_state` válido crea pedido en el estado correcto.
- `initial_state` inválido → error de validación.
- `payment_method = PREPAID_EXTERNAL` → no requiere Wompi.
- `payment_method = COD` → registra método correctamente.
- Cliente `FULL` → error rechazado.
- `order_state_history` con estado inicial presente.

**Riesgos de conflicto:** `orders.service.ts` también lo toca F2.3.BACK-6 (Wave 3). Resolución: F2.3.BACK-6 debe iniciar DESPUÉS de que este PR esté mergeado. Ver nota en mapa anti-conflicto.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

### F2.2.ADMIN-2 — Vista diferenciada de pedidos de Cliente API [M]

**Estado:** `[ ]` pendiente

→ depende de: ninguna (puede iniciar en paralelo con Wave 2)

**Objetivo.** En el panel admin, los pedidos creados via API tienen un badge
visible y no muestran acciones que no les aplican (Flujo 1, reembolsos, disputas).

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/app/admin/orders/page.tsx` (MODIFICAR)
  - Badge "API" junto al número de pedido cuando `order_source === 'API'` (o `client_type === 'API'`).
  - Filtro opcional por origen (UI / API).
- `frontend-ruta/admin/src/app/admin/orders/[id]/page.tsx` (MODIFICAR)
  - Badge "Pedido vía API" en el encabezado del detalle.
  - Ocultar: botón "Aceptar pedido", "Rechazar pedido", sección de validación, botón "Iniciar reembolso".
  - Mostrar: todos los botones logísticos (asignar courier, preparar, listo para despacho, etc.).
- `frontend-ruta/admin/src/lib/orders.api.ts` (REVISAR/MODIFICAR)
  - Verificar que el campo `order_source` o `client_type` viene en el response del backend; si no, agregar al fetch del detalle de pedido y a la lista.

**Archivos que NO debe tocar:** ningún archivo de backend, nada fuera de `admin/orders/`.

**Criterios de aceptación:**
- Pedido de origen API → badge visible en lista y en detalle.
- Botones de Flujo 1 no aparecen para pedidos API.
- Botones logísticos siguen funcionando.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Pruebas obligatorias:**
- typecheck + build. Verificación visual con datos mock si es posible.

**Riesgos de conflicto:** ninguno — solo modifica `admin/orders/`.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

## Wave 3 — Backend dependiente + Frontend (4-5 agentes en paralelo)

Inician cuando toda Wave 2 esté mergeada y CI verde.
**Excepción:** F2.3.BACK-6 espera el merge de F2.2.BACK-3 específicamente antes de tocar `orders.service.ts`.

---

### F2.3.BACK-4 — Endpoints `/api/orders` (Cliente API) [L]

**Estado:** `[ ]` pendiente

→ depende de: F2.2.BACK-2 (middleware api_key_auth), F2.2.BACK-3 (createApiClientOrder)

**Objetivo.** Exponer los 4 endpoints que consume la plataforma del Cliente API.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/routes/api_client_orders.ts` (NUEVO)
  - `POST /api/orders` — scope `orders:write`
    - Auth: `apiKeyAuth('orders:write')`
    - Llama a `createApiClientOrder(clientId, body, apiKeyId)`
    - Response 201: `{ id, order_status, delivery_type, items, created_at, state_history_url }`
  - `GET /api/orders` — scope `orders:read`
    - Filtros: `?status=&from=&to=&page=&page_size=`
    - Response 200: `{ data: [], pagination: {} }`
  - `GET /api/orders/:id` — scope `orders:read`
    - Response 200: pedido con `state_history[]`
  - `POST /api/orders/:id/cancel` — scope `orders:write`
    - Cancela si el estado lo permite (usa state machine)
    - Response 200 o 422 `INVALID_STATE_TRANSITION`
- `backend-ruta/api/src/app.ts` (MODIFICAR) — montar router `/api` con `apiKeyAuth` como base

**Archivos que NO debe tocar:** ningún route existente de `/admin/`, `/buyer/`, `/courier/`, `/ruta-admin/`. No toca auth JWT.

**Criterios de aceptación:**
- Sin API key → 401.
- API key de cliente B usada en endpoint de cliente A → 403 (tenant isolation).
- Key válida, scope `orders:write`, body correcto → 201 con pedido creado.
- `GET /api/orders/:id` con pedido de otro cliente → 404.
- Cancelar pedido ya en `DELIVERED` → 422 `INVALID_STATE_TRANSITION`.
- `GET /api/orders` paginación funciona.
- `pnpm typecheck` EXIT 0, tests pasan.

**Pruebas obligatorias (mínimo 10 tests de integración):**
- Happy path SHIP: crear → estado correcto.
- Happy path PICKUP: crear → estado correcto.
- Tenant isolation en cada endpoint.
- Scope incorrecto → 403.
- Cancelar en estado no cancelable → 422.
- Paginación: página 2 retorna datos distintos.
- `external_reference` duplicado dentro del mismo cliente → 409 (si se implementa unicidad).

**Riesgos de conflicto:** `app.ts` también lo modifica F2.2.BACK-1. Merge de 2 líneas — trivial.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

### F2.3.BACK-5 — Auditoría LOGISTICS_ONLY_FEATURE_UNAVAILABLE [M]

**Estado:** `[ ]` pendiente

→ depende de: F2.2.BACK-2 (para `req.user.userType === 'API_CLIENT'`)

**Objetivo.** Garantizar que TODOS los endpoints bloqueados para Cliente API
retornan 422 `LOGISTICS_ONLY_FEATURE_UNAVAILABLE`, de forma sistemática y
sin excepciones.

**Inventario de endpoints a bloquear:**

| Endpoint | Flujo bloqueado |
|----------|-----------------|
| `POST /buyer/orders` | Flujo 1 — no hay BUYERs en Cliente API |
| `POST /buyer/orders/:id/initiate-payment` | Flujo 1 — no hay Wompi |
| `POST /admin/orders/:id/accept` | Flujo 1 — aceptación del vendedor |
| `POST /admin/orders/:id/reject` | Flujo 1 — rechazo del vendedor |
| Cualquier endpoint de reembolsos (cuando exista en Fase 3) | Flujo 4 |
| Cualquier endpoint de devoluciones post-cierre | Flujo 7 |
| Cualquier endpoint de disputas | Bloque 15 |
| Cualquier endpoint de recurrencia | Flujo 5 |
| Cualquier endpoint de pedidos corporativos | Flujo 6 |

**Archivos a modificar:**

- `backend-ruta/api/src/routes/admin_orders.ts` — revisar todos los endpoints y agregar guard donde falte
- `backend-ruta/api/src/routes/buyer_orders.ts` — guard en creación de DRAFT
- `backend-ruta/api/src/routes/buyer_payment.ts` — guard en initiate-payment
- Crear helper `assertClientIsFull(clientId, db)` o `assertNotApiClient(clientId, db)` en `lib/` si no existe, para reutilizar en todos los guards

**Archivos que NO debe tocar:** `state_machine.ts`, `orders.service.ts`, los routes nuevos de Fase 2 (`api_client_orders.ts`, `admin_api_keys.ts`).

**Criterios de aceptación:**
- Inventario completo documentado (tabla con cada endpoint revisado y su resultado).
- Cada endpoint bloqueado retorna 422 `LOGISTICS_ONLY_FEATURE_UNAVAILABLE` cuando `client_type === 'API'`.
- Endpoints logísticos (asignar courier, preparar, mark-ready, courier ops) NO están bloqueados.
- `pnpm typecheck` EXIT 0.

**Pruebas obligatorias:**
- 1 test por endpoint bloqueado: llamar con contexto de API client → 422.
- 1 test de regresión por endpoint bloqueado: llamar con cliente FULL → no 422.

**Riesgos de conflicto:** modifica múltiples routes existentes. Sin solape con `api_client_orders.ts` (es nuevo). Sin solape con `orders.service.ts` (no lo toca).

**Registro de ejecución:**

```
Fecha:
PR:
Endpoints auditados:
Resultado:
Notas:
```

---

### F2.3.BACK-6 — Webhooks: completitud para Cliente API [M]

**Estado:** `[ ]` pendiente

→ depende de: F2.1.SHARED-1 (event types), F2.2.BACK-3 (orders.service.ts mergeado — esperar este merge antes de tocar ese archivo)

**Objetivo.** Verificar y completar los eventos de webhook disparados durante
el flujo logístico, para que el Cliente API reciba notificaciones en todos
los puntos relevantes de su pedido.

**Eventos de webhook que deben existir:**

| Evento | Cuándo se dispara | Servicio que lo emite |
|--------|-------------------|-----------------------|
| `ORDER_CREATED` | Al crear pedido via API | `createApiClientOrder()` |
| `COURIER_ASSIGNED` | Al asignar courier | `courier_assignment.service.ts` |
| `ORDER_SHIPPED` | Al iniciar envío | `courier_ops.service.ts` |
| `ORDER_OUT_FOR_DELIVERY` | Al salir a entregar | `courier_ops.service.ts` |
| `ORDER_DELIVERED` | Al marcar entregado | `courier_ops.service.ts` |
| `ORDER_DELIVERY_FAILED` | Al fallo de intento | `courier_ops.service.ts` |
| `ORDER_CANCELLED` | Al cancelar | `orders.service.ts` |
| `ORDER_RETURN_TO_ORIGIN` | Al iniciar retorno | `courier_ops.service.ts` |
| `PAYMENT_COLLECTED` | Al registrar cobro COD | `collection.service.ts` |
| `ORDER_READY_FOR_PICKUP` | Al marcar listo para recoger | `pickup_ops.service.ts` |
| `ORDER_PICKED_UP` | Al confirmar recogida | `pickup_ops.service.ts` |

**Archivos a revisar/modificar:**

- `backend-ruta/api/src/services/webhooks_outgoing.service.ts` (REVISAR) — función `emitWebhookIfSubscribed` o similar; verificar que acepta todos los event types
- `backend-ruta/api/src/services/orders/courier_ops.service.ts` (REVISAR/MODIFICAR) — agregar llamadas a `emitWebhook` donde falten
- `backend-ruta/api/src/services/orders/pickup_ops.service.ts` (REVISAR/MODIFICAR)
- `backend-ruta/api/src/services/orders/collection.service.ts` (REVISAR/MODIFICAR)
- `backend-ruta/api/src/services/orders/orders.service.ts` (REVISAR/MODIFICAR) — solo agregar llamadas; el `createApiClientOrder` ya lo hace

**Payload mínimo de cada webhook:**
```json
{
  "event_type": "ORDER_DELIVERED",
  "client_id": 7,
  "order_id": 5001,
  "timestamp": "2026-05-31T14:30:00Z",
  "data": {
    "order_status": "DELIVERED",
    "delivery_type": "SHIP"
  }
}
```

**Archivos que NO debe tocar:** `state_machine.ts`, `api_keys.service.ts`, los routes nuevos.

**Criterios de aceptación:**
- Tabla de eventos completa: todos los eventos listados tienen su llamada a `emitWebhook`.
- No hay eventos duplicados (si el evento ya existía desde Sprint 6, no se duplica).
- El payload tiene la estructura mínima definida.
- La firma HMAC es correcta (ya implementada en Sprint 6).
- `pnpm typecheck` EXIT 0.

**Pruebas obligatorias:**
- 1 test por evento nuevo: simular transición → verificar que se encola un job en pg-boss con el event_type correcto.
- Test de no-duplicación: si el evento ya existía, verificar que no se encola dos veces.

**Riesgos de conflicto:** toca `orders.service.ts` que F2.2.BACK-3 también modifica. **Iniciar solo después de que F2.2.BACK-3 esté mergeado.**

**Registro de ejecución:**

```
Fecha:
PR:
Eventos auditados y agregados:
Resultado:
Notas:
```

---

### F2.3.ADMIN-1 — UI de gestión de API Keys [L]

**Estado:** `[ ]` pendiente

→ depende de: F2.2.BACK-1 (endpoints admin api-keys disponibles)

**Objetivo.** ADMIN_CLIENT puede crear, ver y revocar API keys sin tocar código,
directamente desde el panel admin.

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/lib/api_keys.api.ts` (NUEVO)
  - `listApiKeys()`, `createApiKey(input)`, `revokeApiKey(keyId, reason?)`
- `frontend-ruta/admin/src/app/admin/api-keys/page.tsx` (NUEVO)
  - Lista de keys con columnas: nombre, `key_id` visible, scopes, estado, último uso
- `frontend-ruta/admin/src/app/admin/api-keys/_components/ApiKeyTable.tsx` (NUEVO)
- `frontend-ruta/admin/src/app/admin/api-keys/_components/CreateApiKeyDialog.tsx` (NUEVO)
  - Muestra el secreto UNA sola vez en un campo copiable (botón "Copiar")
  - Advertencia clara: "Este secreto no podrá recuperarse. Guárdalo ahora."
  - Al cerrar el diálogo, el secreto desaparece de la UI
- `frontend-ruta/admin/src/app/admin/api-keys/_components/RevokeApiKeyDialog.tsx` (NUEVO)
  - Confirmación antes de revocar
- `frontend-ruta/admin/src/components/RutaSidebar.tsx` (MODIFICAR)
  - Agregar enlace "API Keys" en la sección de configuración, visible **solo** cuando `client_type === 'API'`

**Archivos que NO debe tocar:** nada fuera de `admin/api-keys/` y el sidebar. No toca `orders/`, `settings/`, `ruta-admin/`.

**Criterios de aceptación:**
- Crear key: modal muestra secreto con botón copiar. Al cerrar y volver, no se puede ver el secreto.
- Lista: muestra `key_id`, nombre, scopes, estado (Activa / Revocada / Expirada), fecha último uso.
- Revocar: confirmación previa. Key revocada aparece como "Revocada" en la lista.
- El enlace "API Keys" en el sidebar solo aparece para clientes de tipo API.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Pruebas obligatorias:**
- typecheck + build obligatorios.
- Verificación visual del flujo completo (crear → copiar secreto → cerrar → revocar).

**Riesgos de conflicto:** `RutaSidebar.tsx` también podría tocarse en F2.3.ADMIN-3. Coordinar: este agente agrega el link a API Keys; F2.3.ADMIN-3 agrega la pestaña de API Keys en settings del cliente. Son secciones distintas del sidebar — merge trivial si hay conflicto.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

### F2.3.ADMIN-3 — Configuración de cliente: tipo API en creación y settings [S]

**Estado:** `[ ]` pendiente

→ depende de: F2.2.BACK-1 (para saber que la pestaña API Keys tiene endpoints)

**Objetivo.** ADMIN_RUTA puede crear un cliente de tipo API desde el panel.
El detalle del cliente muestra una pestaña "API Keys" cuando es tipo API.

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/app/ruta-admin/clients/new/page.tsx` (MODIFICAR)
  - Al elegir `client_type = 'API'`: ocultar el campo `frontend_mode`.
  - Al elegir `client_type = 'FULL'`: mostrar `frontend_mode` como requerido.
- `frontend-ruta/admin/src/app/ruta-admin/clients/[id]/page.tsx` (MODIFICAR)
  - Si el cliente es tipo API: mostrar pestaña "API Keys" (o enlace a `admin/api-keys` de ese cliente en Vista de Control).
  - Mostrar claramente el tipo del cliente en el encabezado.
- `frontend-ruta/admin/src/app/admin/settings/page.tsx` (REVISAR)
  - Si existe una pestaña de configuración general, agregar sección "API Keys" visible solo para clientes API.

**Archivos que NO debe tocar:** nada de backend, nada de `orders/`, nada de `api-keys/` (esa carpeta la construye F2.3.ADMIN-1).

**Criterios de aceptación:**
- Formulario de creación: seleccionar "API" oculta `frontend_mode`. Seleccionar "Full" lo requiere.
- Detalle de cliente API: muestra enlace o tab a gestión de API Keys.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Pruebas obligatorias:**
- typecheck + build.

**Riesgos de conflicto:** ninguno — modifica solo `ruta-admin/clients/`.

**Registro de ejecución:**

```
Fecha:
PR:
Tests ejecutados:
Resultado:
Notas:
```

---

## Wave 4 — QA (2 agentes en paralelo, ~5-6h c/u)

Inician cuando toda Wave 3 esté mergeada y CI verde.

---

### F2.4.QA-1 — Tests: API key management + middleware de autenticación [M]

**Estado:** `[ ]` pendiente

→ depende de: Wave 3 completa

**Objetivo.** Cobertura de tests completa para el flujo de generación,
validación y revocación de API keys.

**Archivos a crear:**

- `backend-ruta/api/src/tests/api_keys.test.ts` (NUEVO) — integración de `api_keys.service.ts`
- `backend-ruta/api/src/tests/api_key_auth.test.ts` (NUEVO) — unit del middleware

**Criterios de aceptación:**
- `pnpm test` EXIT 0 en `backend-ruta`.
- Coverage de `api_keys.service.ts` ≥ 90%.
- Coverage de `api_key_auth.ts` ≥ 95%.

**Pruebas obligatorias:**
- Ciclo completo: generar key → usar en request → verificar `last_used_at` actualizado.
- Cada caso de error del middleware: key inválida, revocada, expirada, scope incorrecto.
- Aislamiento: key de cliente A rechazada para endpoints de cliente B.
- Idempotencia en `POST /admin/api-keys` (mismo `X-Idempotency-Key`).
- Revocar key → usarla → 401 `API_KEY_REVOKED`.

**Riesgos de conflicto:** ninguno — solo crea archivos de test nuevos.

**Registro de ejecución:**

```
Fecha:
PR:
Cobertura api_keys.service.ts:
Cobertura api_key_auth.ts:
Tests totales backend:
Resultado:
```

---

### F2.4.QA-2 — Tests: flujo completo de Cliente API [L]

**Estado:** `[ ]` pendiente

→ depende de: Wave 3 completa

**Objetivo.** Tests de integración end-to-end del flujo de pedido de Cliente
API, cobertura de LOGISTICS_ONLY_FEATURE_UNAVAILABLE y suite E2E Playwright
mínima.

**Archivos a crear:**

- `backend-ruta/api/src/tests/api_client_orders.test.ts` (NUEVO) — integración completa
- `frontend-ruta/tests/e2e/api_client_flow.spec.ts` (NUEVO, opcional si hay tiempo)

**Flujos a cubrir:**

1. Crear pedido SHIP via API → asignar courier via admin → courier entrega → webhook `ORDER_DELIVERED`.
2. Crear pedido PICKUP via API → marcar listo para recoger → cliente recoge → webhook `ORDER_PICKED_UP`.
3. Crear pedido COD → courier cobra → webhook `PAYMENT_COLLECTED`.
4. Cancelar pedido antes de despacho via API → 200.
5. Intentar cancelar pedido ya despachado via API → 422 `INVALID_STATE_TRANSITION`.
6. Verificar que todos los endpoints de Flujo 1, 4-7 y disputas retornan 422 para cliente API.

**Criterios de aceptación:**
- `pnpm test` EXIT 0 en `backend-ruta`.
- `pnpm test:e2e` EXIT 0 en `frontend-ruta` (si se implementa E2E).
- Los 6 flujos pasan.
- Ningún endpoint bloqueado escapa al guard.

**Pruebas obligatorias:**
- Los 6 flujos listados.
- Test de regresión: flujos de Cliente Full no se ven afectados por los cambios de Fase 2.

**Riesgos de conflicto:** ninguno — solo crea archivos de test nuevos.

**Registro de ejecución:**

```
Fecha:
PR:
Flujos cubiertos:
Tests totales backend:
Tests E2E (si aplica):
Resultado:
```

---

## Resumen del plan

### Orden recomendado

```
Día 1 AM  → Agente 1: F2.1.SHARED-1 (publicar @orkoruta/shared@1.4.0)
Día 1 PM  → Agentes 2+3+4: F2.2.BACK-1 + F2.2.BACK-2 + F2.2.BACK-3 (paralelo)
             Agente 5: F2.2.ADMIN-2 (puede iniciar sin esperar Wave 2)
Día 2 AM  → Verificar Wave 2: todos los PRs mergeados, CI verde
Día 2 PM  → Agentes 2+3+4+5+nuevo: F2.3.BACK-4 + F2.3.BACK-5 + F2.3.ADMIN-1 + F2.3.ADMIN-3 (paralelo)
             F2.3.BACK-6 inicia solo tras confirmar merge de F2.2.BACK-3
Día 3 AM  → Verificar Wave 3: todos los PRs mergeados, CI verde en backend y frontend
Día 3 PM  → Agentes QA-A + QA-B: F2.4.QA-1 + F2.4.QA-2 (paralelo)
Día 4 AM  → Verificar Wave 4: tests completos, CI verde
Día 4 PM  → Deploy a staging, onboarding del Cliente API piloto
```

### Checklist pre-inicio

- [ ] `@orkoruta/shared@1.3.0` instalado sin errores en backend y frontend
- [ ] BD dev accesible: `149.130.168.24:26432/rutadb`
- [ ] `client_api_keys` existe en BD dev: `\dt ruta.client_api_keys`
- [ ] Al menos un cliente con `client_type = 'API'` en datos de seed
- [ ] GitHub PAT disponible para publicar `@orkoruta/shared@1.4.0`
- [ ] `pnpm test` en `backend-ruta/main` verde (línea base)
- [ ] `pnpm build` en `frontend-ruta/main` verde (línea base)
- [ ] F2.3.BACK-6 coordinado para iniciar después del merge de F2.2.BACK-3
- [ ] Ningún agente tiene instrucción de tocar `ruta_postgres.sql` (tabla `client_api_keys` ya existe — sin migración de BD nueva)

### Tabla de avance

| Tarea | Estado | PR | Fecha | Agente |
|-------|:------:|----|-------|--------|
| F2.1.SHARED-1 | `[ ]` | — | — | — |
| F2.2.BACK-1 | `[ ]` | — | — | — |
| F2.2.BACK-2 | `[ ]` | — | — | — |
| F2.2.BACK-3 | `[ ]` | — | — | — |
| F2.2.ADMIN-2 | `[ ]` | — | — | — |
| F2.3.BACK-4 | `[ ]` | — | — | — |
| F2.3.BACK-5 | `[ ]` | — | — | — |
| F2.3.BACK-6 | `[ ]` | — | — | — |
| F2.3.ADMIN-1 | `[ ]` | — | — | — |
| F2.3.ADMIN-3 | `[ ]` | — | — | — |
| F2.4.QA-1 | `[ ]` | — | — | — |
| F2.4.QA-2 | `[ ]` | — | — | — |

### Criterio de salida de Fase 2

- [ ] 1 Cliente API piloto integrado y operando en staging/producción
- [ ] 10+ pedidos reales procesados via API key
- [ ] Tiempo de integración < 1 día documentado
- [ ] Webhook reliability > 99% (verificar en historial de entregas)
- [ ] `pnpm test` EXIT 0 en `backend-ruta` (≥ 4100 tests)
- [ ] `pnpm build` EXIT 0 en `frontend-ruta`
- [ ] CI verde en ambos repos en `main`
- [ ] Cero violaciones de aislamiento multi-tenant en suite QA
