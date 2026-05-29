# Sprint 3 — Flujo SHIP completo

Última actualización: 2026-05-29.

---

## Estado general

| Sprint | Estado |
|---|---|
| 0, 1, 2 | ✅ Completos — todos los PRs mergeados a `main` |
| **3** | ✅ Completo — todos los PRs mergeados a `main` |
| 4–6 | 🔜 Pendientes — siguiente: Sprint 4 PICKUP |

**Meta Sprint 3:** Pedido con entrega a domicilio (SHIP) completado de extremo a extremo:
asignación de repartidor → entrega → cobro → confirmación. Flujo de cancelación post-despacho y retorno al origen cubiertos.

---

## Contexto técnico clave (leer antes de lanzar agentes)

- **State machine:** todas las transiciones del flujo SHIP ya están codificadas en
  `api/src/services/orders/state_machine.ts`. Los agentes BACK **no modifican** ese archivo.
- **`requestCancel` endpoint:** ya existe en `buyer_orders.ts` y `orders.service.ts`.
  El agente BACK-C parte de esa base.
- **`app.ts` es zona de conflicto:** ningún agente de Wave 1 toca ese archivo.
  Los mounts nuevos se integran en el paso post-wave.
- **Contrato API:** `docs-ruta/contrato_api.md` sección 5 "COURIER" y sección
  "admin/orders" tienen los endpoints exactos a implementar.

---

## Mapa de propiedad de archivos — Wave 1

> Cada agente trabaja únicamente en los archivos de su columna. Sin traslapes.

| Agente | Archivos nuevos (crea) | Archivos modificados | NO toca |
|---|---|---|---|
| BACK-A | `courier_assignment.service.ts`, `admin_order_assignment.ts`, `courier_assignment.test.ts` | — | `app.ts`, `orders.service.ts`, `admin_orders.ts` |
| BACK-B | `courier_orders.service.ts`, `collection.service.ts`, `courier_orders.ts`, `courier_orders.test.ts` | — | `app.ts`, `orders.service.ts`, `buyer_orders.ts` |
| BACK-C | `cancellation.test.ts` | `orders.service.ts`, `admin_orders.ts` | `app.ts`, `buyer_orders.ts`* |
| BACK-D | `auto_confirm_delivered.job.ts` | `maintenance_boss.ts`, `maintenance_jobs.test.ts` | Todo lo demás |
| FRONT-A | `courier/` (directorio completo nuevo), `courier_orders.api.ts` | — | Todo lo existente |

> \*`buyer_orders.ts` ya tiene `POST /:id/request-cancel` — BACK-C no lo toca.

---

## Wave 0 — Verificación pre-arranque ✅ COMPLETADO 2026-05-28

Resultado del grep ejecutado:

```
packages-ruta/shared/src/validators/order.schema.ts: assignCourierSchema ✅
recordCollectionSchema ❌ no existe
```

**Conclusión:**
- BACK-A, BACK-C, BACK-D, FRONT-A → pueden arrancar **ahora**.
- BACK-B → bloqueado hasta que `3.SHARED-1` publique `@orkoruta/shared@1.3.0`.

### 3.SHARED-1 — Bump @orkoruta/shared con schema de cobro

**Estado:** `[x]` publicado 2026-05-28 — `@orkoruta/shared@1.3.0`

**Repo:** `packages-ruta` | **Rama:** `feat/shared-3`

Agregar en `packages-ruta/shared/src/validators/order.schema.ts`:

```typescript
export const recordCollectionSchema = z.object({
  amount: z.number().positive(),
  payment_method: z.enum(['CASH', 'ELECTRONIC']),
  notes: z.string().max(500).optional(),
});

export type RecordCollectionInput = z.infer<typeof recordCollectionSchema>;
```

Exportar desde el barrel `index.ts`. Bump `version` a `1.3.0` en `package.json`.
Publicar con PAT local (CI workflow tiene limitaciones de permisos de org).
Actualizar `@orkoruta/shared` en `backend-ruta/api/package.json` a `^1.3.0` y hacer `pnpm install`.

**Verificación:** `pnpm --filter @orkoruta/shared typecheck && pnpm --filter @orkoruta/shared test`

---

## Wave 1 — 5 agentes en paralelo

Arrancar los 5 agentes **simultáneamente** en ventanas/terminales separadas.
Cada uno trabaja en su propia rama desde `main` actualizado.

---

### BACK-A: `3.BACK-1` — Asignación de Repartidor

**Estado:** `[x]` mergeado 2026-05-29 — PR #9

**Rama:** `feat/back-3-1` (en `backend-ruta`)

---

#### PROMPT BACK-A

```
Eres un agente de desarrollo trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.BACK-1 — Asignación de Repartidor

Implementa el servicio y los endpoints para asignar/desasignar couriers a pedidos.

## Contexto del proyecto

Stack: Express.js + TypeScript, Prisma, Zod, Vitest, pino.
Repo: backend-ruta. Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/backend-ruta
Branch de trabajo: crea y trabaja en `feat/back-3-1` desde main actualizado.

Documentos a leer primero:
- api/CLAUDE.md — reglas no negociables del proyecto
- docs-ruta/contrato_api.md — sección "admin/orders" (endpoints de asignación)
- docs-ruta/flujos/flujo_2_ship_completo.txt — flujo SHIP
- api/src/services/orders/state_machine.ts — transiciones ya codificadas

## Archivos existentes de referencia

Lee estos antes de implementar:
- api/src/services/orders/orders.service.ts — patrón de service existente
- api/src/routes/admin_orders.ts — patrón de router existente
- api/src/middleware/auth.js — requireAdminClient, requireOperatorClient
- api/src/lib/errors.ts — errores tipados

## Lo que debes crear (SOLO estos archivos)

1. `api/src/services/orders/courier_assignment.service.ts`
   - `assignCourier(clientId, orderId, courierUserId, actingUser)`:
     verifica que el pedido esté en `AWAITING_COURIER_ASSIGNMENT`,
     verifica que el courier exista, esté ACTIVE y pertenezca al mismo client_id,
     usa optimistic lock (campo `version` de la tabla orders),
     transiciona a `COURIER_ASSIGNED` vía `state_machine.assertTransition()`,
     registra en `order_state_history` (el trigger lo hace automáticamente).
   - `unassignCourier(clientId, orderId, actingUser)`:
     verifica que esté en `COURIER_ASSIGNED`,
     transiciona a `AWAITING_COURIER_ASSIGNMENT`.
   - `getAvailableCouriers(clientId)`:
     couriers ACTIVE del mismo client_id, sin pedido en curso
     (sin pedido en estado COURIER_ASSIGNED..ARRIVED_AT_CUSTOMER).

2. `api/src/routes/admin_order_assignment.ts`
   Router con 3 endpoints (ver contrato_api.md):
   - `GET  /admin/orders/map` — pedidos en AWAITING_COURIER_ASSIGNMENT con coords
   - `GET  /admin/orders/:id/available-couriers`
   - `POST /admin/orders/:id/assign-courier` — body: { courier_user_id }
   - `POST /admin/orders/:id/unassign-courier`
   Auth requerida: requireAdminClient o requireOperatorClient.
   Todos los POST requieren X-Idempotency-Key.

3. `api/src/__tests__/courier_assignment.test.ts`
   Tests unitarios sin BD (mock prisma como en otros tests existentes):
   - assignCourier: OK, pedido en estado incorrecto (422), courier inactivo (422),
     courier de otro tenant (403), optimistic lock fallido (409).
   - unassignCourier: OK, estado incorrecto.
   - getAvailableCouriers: retorna solo los disponibles.

## Lo que NO debes tocar

- `api/src/app.ts` — NO agregues el mount. Documenta en el PR:
  "Agregar a app.ts: app.use('/admin', adminOrderAssignmentRouter)"
- `api/src/services/orders/state_machine.ts` — ya tiene las transiciones
- `api/src/services/orders/orders.service.ts`
- Cualquier archivo existente de routes o services

## Definition of Done

- `pnpm typecheck` EXIT 0
- `pnpm test` pasa (todos los tests existentes + los nuevos)
- `pnpm lint` EXIT 0
- PR abierto contra main con título "[3.BACK-1] Asignación de Repartidor"
```

---

### BACK-B: `3.BACK-2` + `3.BACK-3` — Endpoints COURIER + Cobro contra entrega

**Estado:** `[x]` mergeado 2026-05-29 — PR #12

**Rama:** `feat/back-3-2-3` (en `backend-ruta`)

---

#### PROMPT BACK-B

```
Eres un agente de desarrollo trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.BACK-2 + 3.BACK-3 — Endpoints COURIER y cobro contra entrega

Implementa todos los endpoints del portal del courier (sus propios pedidos)
y el registro de cobro contra entrega con evidencia.

## Contexto del proyecto

Stack: Express.js + TypeScript, Prisma, Zod, Vitest, Multer, pino.
Repo: backend-ruta. Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/backend-ruta
Branch de trabajo: crea y trabaja en `feat/back-3-2-3` desde main actualizado.

Documentos a leer primero:
- api/CLAUDE.md — reglas no negociables
- docs-ruta/contrato_api.md — sección "5. COURIER (vista móvil)" — endpoints exactos
- docs-ruta/flujos/flujo_2_ship_completo.txt — flujo SHIP completo
- api/src/services/orders/state_machine.ts — transiciones ya codificadas (no modificar)

## Archivos existentes de referencia

Lee estos antes de implementar:
- api/src/services/orders/orders.service.ts — patrón de service
- api/src/routes/buyer_orders.ts — patrón de router con auth
- api/src/middleware/auth.ts — middleware requireCourier (verifícalo; si no existe, créalo)
- api/src/services/payments.service.ts — patrón de upload/evidencia si existe

## Lo que debes crear (SOLO estos archivos)

1. `api/src/services/orders/courier_orders.service.ts`
   Servicio para todas las acciones del courier. Regla: SIEMPRE verifica que
   `order.assigned_courier_id === req.user.id` antes de cualquier transición.
   Métodos a implementar (siguiendo contrato_api.md):
   - `getCourierOrders(clientId, courierId, filters)` — lista paginada
   - `getCourierOrderById(clientId, courierId, orderId)` — detalle
   - `startShipping(clientId, courierId, orderId)` — COURIER_ASSIGNED → SHIPPED
   - `markOutForDelivery(clientId, courierId, orderId)` — IN_TRANSIT → OUT_FOR_DELIVERY
   - `arrive(clientId, courierId, orderId)` — → ARRIVED_AT_CUSTOMER
   - `markDelivered(clientId, courierId, orderId)` — requiere cobro previo si COD
   - `recordFailedAttempt(clientId, courierId, orderId, reason)` — → DELIVERY_ATTEMPTED
   Todas usan `state_machine.assertTransition()`. Nunca modifican `order_status` directamente.

2. `api/src/services/orders/collection.service.ts`
   - `recordCollection(clientId, courierId, orderId, amount, evidenceBuffer, mimeType)`:
     verifica que el pedido esté en estado que permita cobro COD,
     verifica que amount > 0,
     sube evidencia a file storage (stub: guarda en campo `evidence_url` con
     placeholder "pending-upload/{orderId}/{timestamp}" hasta que file storage esté listo),
     transiciona payment_status a PAYMENT_COLLECTED_CASH o PAYMENT_COLLECTED_ELECTRONIC
     según payment_method del pedido,
     audita la operación.

3. `api/src/routes/courier_orders.ts`
   Router con todos los endpoints de contrato_api.md sección 5:
   - GET  /courier/me
   - GET  /courier/orders/assigned
   - GET  /courier/orders/:id
   - POST /courier/orders/:id/start-shipping
   - POST /courier/orders/:id/mark-out-for-delivery
   - POST /courier/orders/:id/arrive
   - POST /courier/orders/:id/record-collection  (multipart/form-data)
   - POST /courier/orders/:id/mark-delivered
   - POST /courier/orders/:id/attempt-failed
   - POST /courier/orders/:id/return-to-origin
   - POST /courier/orders/:id/upload-evidence    (multipart)
   Auth: requireCourier en todos. X-Idempotency-Key en todos los POST.

4. `api/src/__tests__/courier_orders.test.ts`
   Tests unitarios (mock prisma):
   - Acceso a pedido de otro courier → 403
   - Transición permitida → 200
   - Transición no permitida → 422 INVALID_STATE_TRANSITION
   - recordCollection sin COD → 422
   - recordCollection OK → payment_status actualizado

## Lo que NO debes tocar

- `api/src/app.ts` — NO agregues el mount. Documenta en PR:
  "Agregar a app.ts: app.use('/courier', courierOrdersRouter)"
- `api/src/services/orders/state_machine.ts` — ya tiene todas las transiciones
- `api/src/services/orders/orders.service.ts`
- `api/src/routes/buyer_orders.ts`
- `api/src/routes/admin_orders.ts`

## Definition of Done

- `pnpm typecheck` EXIT 0
- `pnpm test` pasa
- `pnpm lint` EXIT 0
- PR "[3.BACK-2+3] Endpoints COURIER y cobro contra entrega"
```

---

### BACK-C: `3.BACK-4` + `3.BACK-5` — Cancelación post-despacho + RETURN_TO_ORIGIN

**Estado:** `[x]` mergeado 2026-05-29 — PR #11

**Rama:** `feat/back-3-4-5` (en `backend-ruta`)

---

#### PROMPT BACK-C

```
Eres un agente de desarrollo trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.BACK-4 + 3.BACK-5 — Cancelación post-despacho + RETURN_TO_ORIGIN

Implementa el flujo completo de cancelación post-despacho (solicitud del buyer,
aprobación/rechazo del admin) y el flujo de retorno al origen.

## Contexto del proyecto

Stack: Express.js + TypeScript, Prisma, Zod, Vitest, pino.
Repo: backend-ruta. Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/backend-ruta
Branch de trabajo: crea y trabaja en `feat/back-3-4-5` desde main actualizado.

Documentos a leer primero:
- api/CLAUDE.md — reglas no negociables
- docs-ruta/contrato_api.md — sección "admin/orders" endpoints de cancel request
- docs-ruta/flujos/flujo_2_ship_completo.txt — sección de cancelación post-despacho
- api/src/services/orders/state_machine.ts — transiciones (CUSTOMER_CANCEL_REQUEST,
  CANCEL_REQUEST_APPROVED, CANCEL_REQUEST_REJECTED, RETURN_TO_ORIGIN, etc.)

## Diagnóstico previo (ya existe — verifica antes de implementar)

Lee estos archivos para entender qué hay:
- `api/src/services/orders/orders.service.ts` — ya tiene `requestCancel()`
- `api/src/routes/buyer_orders.ts` — ya tiene `POST /:id/request-cancel`
- `api/src/routes/admin_orders.ts` — verifica si tiene approve/reject cancel

## Lo que debes agregar

### En `api/src/services/orders/orders.service.ts` (extiende lo existente):

Si no existen, agregar:
- `approveCancelRequest(clientId, orderId, actingUser)`:
  verifica estado CUSTOMER_CANCEL_REQUEST,
  transiciona a CANCEL_REQUEST_APPROVED vía state_machine,
  audita con actor admin.
- `rejectCancelRequest(clientId, orderId, actingUser, reason)`:
  verifica estado CUSTOMER_CANCEL_REQUEST,
  transiciona a CANCEL_REQUEST_REJECTED,
  audita.
- `returnToOrigin(clientId, orderId, actingUser)`:
  válido desde múltiples estados (ver state_machine — CANCEL_REQUEST_APPROVED,
  DELIVERY_ATTEMPTED, PAYMENT_COLLECTION_PENDING, CASH_PAYMENT_REJECTED),
  transiciona a RETURN_TO_ORIGIN.
- `returnToOriginReceived(clientId, orderId, actingUser)`:
  RETURN_TO_ORIGIN → RETURN_TO_ORIGIN_RECEIVED.

### En `api/src/routes/admin_orders.ts` (extiende lo existente):

Agregar endpoints si no existen:
- `POST /admin/orders/:id/approve-cancel-request`
- `POST /admin/orders/:id/reject-cancel-request` — body: { reason }
- `POST /admin/orders/:id/return-to-origin`
- `POST /admin/orders/:id/return-to-origin-received`
Auth: requireAdminClient o requireOperatorClient. X-Idempotency-Key en todos.

### Archivo nuevo:

`api/src/__tests__/cancellation.test.ts`:
- approve cancel request: OK, estado incorrecto, rol incorrecto
- reject cancel request: OK, con reason
- returnToOrigin desde CANCEL_REQUEST_APPROVED: OK
- returnToOrigin desde DELIVERY_ATTEMPTED: OK
- returnToOriginReceived: OK

## Lo que NO debes tocar

- `api/src/app.ts` — los routers admin_orders y buyer_orders ya están montados
- `api/src/services/orders/state_machine.ts` — ya completo
- `api/src/routes/buyer_orders.ts` — `request-cancel` ya existe
- Cualquier archivo de courier (pertenece a BACK-B)

## Definition of Done

- Verifica primero qué métodos ya existen (no duplicar)
- `pnpm typecheck` EXIT 0
- `pnpm test` pasa (incluyendo tests pre-existentes)
- `pnpm lint` EXIT 0
- PR "[3.BACK-4+5] Cancelación post-despacho y RETURN_TO_ORIGIN"
```

---

### BACK-D: `3.BACK-6` — Auto-confirmación de entregados

**Estado:** `[x]` mergeado 2026-05-29 — PR #10

**Rama:** `feat/back-3-6` (en `backend-ruta`)

---

#### PROMPT BACK-D

```
Eres un agente de desarrollo trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.BACK-6 — Auto-confirmación de pedidos entregados

Implementa el job de mantenimiento que transiciona automáticamente pedidos
de DELIVERED a CONFIRMED_BY_SYSTEM cuando el buyer no confirma en el plazo configurado.

## Contexto del proyecto

Stack: Express.js + TypeScript, Prisma, pg-boss, Zod, Vitest, pino.
Repo: backend-ruta. Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/backend-ruta
Branch de trabajo: crea y trabaja en `feat/back-3-6` desde main actualizado.

Documentos a leer primero:
- api/CLAUDE.md — reglas no negociables
- docs-ruta/parametros_negocio.md — busca parámetros de auto-confirmación
- api/src/services/orders/state_machine.ts — transición DELIVERED → CONFIRMED_BY_SYSTEM
  ya existe con actor 'SYSTEM'

## Archivos existentes de referencia (LEE ANTES de codificar)

- `api/src/jobs/order_expiration.job.ts` — patrón exacto a seguir:
  cron, iteración por cliente activo, lectura de parámetros con getParameter(),
  withTenant(), query de pedidos vencidos, transición de estado.
- `api/src/jobs/maintenance_boss.ts` — singleton pg-boss, patrón de registro.
- `api/src/__tests__/maintenance_jobs.test.ts` — patrón de tests de jobs.
- `api/src/lib/parameters.ts` — función getParameter().

## Lo que debes crear

### Archivo nuevo: `api/src/jobs/auto_confirm_delivered.job.ts`

Cron: `0 * * * *` (cada hora, configurable).

Lógica:
1. Para cada cliente FULL activo (client_id > 0):
   - Leer parámetro `order.auto_confirm_after_hours` (o el nombre real que
     encuentres en parametros_negocio.md) con fallback al global (client_id=0)
     y fallback a código (48h).
   - Buscar pedidos en estado DELIVERED con
     `updated_at < NOW() - INTERVAL '<n> hours'`.
   - Para cada pedido: `state_machine.assertTransition(DELIVERED, CONFIRMED_BY_SYSTEM,
     'SYSTEM', {})` y actualizar en BD dentro de `withTenant()`.
2. Loggear con pino: cantidad de pedidos procesados por cliente.
3. Guard `NODE_ENV === 'test'` para no correr el scheduler en tests
   (mismo patrón que order_expiration.job.ts).

### Modificar: `api/src/jobs/maintenance_boss.ts`

Registrar el nuevo job siguiendo el patrón existente.

### Modificar: `api/src/__tests__/maintenance_jobs.test.ts`

Agregar casos para el nuevo job:
- Pedido en DELIVERED con tiempo vencido → transiciona a CONFIRMED_BY_SYSTEM
- Pedido en DELIVERED sin tiempo vencido → no se toca
- Pedido en otro estado → no se toca

## Lo que NO debes tocar

- `api/src/services/orders/state_machine.ts`
- `api/src/services/orders/orders.service.ts`
- Cualquier archivo de routes
- `api/src/app.ts`

## Definition of Done

- `pnpm typecheck` EXIT 0
- `pnpm test` pasa
- PR "[3.BACK-6] Job auto-confirmación de pedidos entregados"
```

---

### FRONT-A: `3.ADMIN-2` — Vista COURIER móvil-first

**Estado:** `[x]` mergeado 2026-05-29 — PR #14

**Rama:** `feat/admin-3-2` (en `frontend-ruta`)

---

#### PROMPT FRONT-A

```
Eres un agente de desarrollo trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.ADMIN-2 — Vista COURIER móvil-first

Implementa el portal del repartidor dentro del app admin de Next.js.
Es una sección completamente nueva, aislada, bajo la ruta /courier.

## Contexto del proyecto

Stack: Next.js 14 App Router + TypeScript + Tailwind. output: 'export' (SSG).
Repo: frontend-ruta, app: admin/
Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/frontend-ruta
Branch de trabajo: crea y trabaja en `feat/admin-3-2` desde main actualizado.

Documentos a leer primero:
- docs-ruta/contrato_api.md — sección "5. COURIER" — endpoints del courier
- docs-ruta/wireframes_mvp.md — pantallas del courier
- docs-ruta/diseno/galeria_estilos_ruta.md — design system Ruta*
- admin/src/app/(protected)/admin/orders/[id]/OrderDetailClient.tsx — patrón de UI existente

## Patrones a seguir (leer antes de crear)

- admin/src/lib/orders.api.ts — patrón de API client con fetch + credentials: 'include'
- admin/src/app/(protected)/admin/orders/page.tsx — patrón de lista con client component
- admin/src/lib/session-context.tsx — cómo leer el user/rol del contexto

## Lo que debes crear (TODO bajo directorios nuevos)

### `admin/src/lib/courier_orders.api.ts`
Tipos e funciones fetch para todos los endpoints de la sección 5 del contrato:
- `CourierOrder`, `CourierOrderDetail`, `CollectionPayload` — tipos
- `getAssignedOrders(filters?)`, `getCourierOrderById(id)`
- `startShipping(id, idempotencyKey)`, `markOutForDelivery(id, key)`,
  `arrive(id, key)`, `markDelivered(id, key)`, `attemptFailed(id, reason, key)`
- `recordCollection(id, payload: CollectionPayload, key)` — multipart/FormData
- Todas usan `credentials: 'include'` y base URL desde env `NEXT_PUBLIC_API_URL`

### `admin/src/app/(protected)/courier/layout.tsx`
Layout dedicado al courier:
- Fondo neutro, tipografía grande (móvil-first)
- Header simple: logo Ruta, nombre del courier, botón logout
- Guard de rol: si `session.user.user_type !== 'COURIER'` → redirect /login
- Sin `RutaSidebar` (el sidebar del admin no aplica al courier)

### `admin/src/app/(protected)/courier/page.tsx` + `_components/CourierDashboard.tsx`
Vista principal con 3 tabs: **Asignados** / **En camino** / **Completados**

Cada tab muestra lista de pedidos (tarjetas grandes, touch-friendly):
- Número de pedido, dirección de entrega, nombre del comprador, total
- Badge de estado con color
- CTA principal según estado (botón grande, 48px mínimo de altura)

### `admin/src/app/(protected)/courier/[id]/page.tsx` + `_components/CourierOrderDetail.tsx`
Detalle del pedido para el courier:
- Dirección completa con link a Google Maps (`maps.google.com?q=<dirección>`)
- Teléfono del comprador (clickeable, `tel:`)
- Resumen de items
- Historial de estados (timeline simplificado)
- Botones de acción condicionales según `order_status`:
  - COURIER_ASSIGNED → "Iniciar envío"
  - SHIPPED / IN_TRANSIT → "Marcar en camino" / "Llegar al destino"
  - ARRIVED_AT_CUSTOMER + COD → mostrar CollectionForm ANTES de "Entregado"
  - ARRIVED_AT_CUSTOMER + no COD → "Marcar como entregado"
  - Cualquier estado en campo → "Intento fallido"

### `admin/src/app/(protected)/courier/[id]/_components/CollectionForm.tsx`
Formulario de cobro contra entrega:
- Campo monto (número, COP)
- Input de foto: `<input type="file" accept="image/*" capture="environment">`
  (activa la cámara en móvil)
- Botón "Registrar cobro" → llama `recordCollection()`
- Solo se muestra si `payment_method` es COD y el estado lo requiere

## Notas importantes

- Los endpoints del courier aún NO existen en el backend (los está construyendo otro
  agente en paralelo). Escribe el API client con los tipos del contrato y asume que
  los endpoints estarán disponibles. Los errores de red mostrarán estado de carga.
- Compatibilidad con `output: export`: todas las páginas dinámicas necesitan
  `generateStaticParams`. Para `/courier/[id]` usa `generateStaticParams([{ id: '_' }])`.
- Usa componentes de `@orkoruta/ui` (RutaButton, RutaCard, etc.) para consistencia.
- NO uses RutaSidebar ni el layout del admin existente — el courier tiene su propio layout.

## Lo que NO debes tocar

- Ningún archivo existente de `admin/src/app/(protected)/admin/`
- `admin/src/app/(protected)/layout.tsx` (layout global del admin)
- `admin/src/lib/orders.api.ts` (del admin, no del courier)
- Nada de `storefront/`

## Definition of Done

- `pnpm --filter @ruta/admin typecheck` EXIT 0
- `pnpm --filter @ruta/admin lint` EXIT 0
- PR "[3.ADMIN-2] Vista COURIER móvil-first"
```

---

## Paso de Integración post-Wave 1 (~30 min)

**Responsable:** tú, después de mergear BACK-A y BACK-B.

Agregar 2 líneas a `api/src/app.ts`:

```typescript
import { adminOrderAssignmentRouter } from './routes/admin_order_assignment.js'; // 3.BACK-1
import { courierOrdersRouter } from './routes/courier_orders.js';                // 3.BACK-2+3

// En la sección de protected routes:
app.use('/admin', adminOrderAssignmentRouter);   // 3.BACK-1 — /admin/orders/map, assign, etc.
app.use('/courier', courierOrdersRouter);         // 3.BACK-2+3
```

Commit directo en main o como parte del último PR de la wave.

**Estado:** `[x]` completado 2026-05-29

---

## Orden de merge sugerido (Wave 1)

Para minimizar conflictos en `app.ts`:

1. **BACK-D** primero — solo toca jobs, cero conflicto
2. **BACK-C** — solo toca services/routes existentes, cero conflicto
3. **BACK-A** — crea archivos nuevos (app.ts mount en integración)
4. **BACK-B** — crea archivos nuevos (app.ts mount en integración)
5. **FRONT-A** — repo diferente, en cualquier momento

---

## Wave 2 — 1 agente (después de BACK-A mergeado)

### FRONT-B: `3.ADMIN-1` — Mapa de asignación

**Estado:** `[x]` mergeado 2026-05-29 — PR #15

**Rama:** `feat/admin-3-1` (en `frontend-ruta`)

**Condición de arranque:** BACK-A debe estar mergeado en `backend-ruta/main`.

---

#### PROMPT FRONT-B

```
Eres un agente de desarrollo trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.ADMIN-1 — Mapa de asignación de repartidores

Implementa la pantalla de asignación: mapa con pedidos pendientes y couriers
disponibles, con panel lateral para asignar. Es la herramienta operativa
principal del ADMIN_CLIENT para gestionar el flujo SHIP.

## Contexto del proyecto

Stack: Next.js 14 App Router + TypeScript + Tailwind + Leaflet. output: 'export'.
Repo: frontend-ruta, app: admin/
Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/frontend-ruta
Branch de trabajo: crea y trabaja en `feat/admin-3-1` desde main actualizado.

Documentos a leer primero:
- docs-ruta/contrato_api.md — endpoints GET /admin/orders/map y assign-courier
- docs-ruta/wireframes_mvp.md — pantalla de mapa de asignación
- docs-ruta/diseno/galeria_estilos_ruta.md — design system

## Patrones a seguir (leer antes de crear)

- admin/src/lib/orders.api.ts — patrón de API client
- admin/src/app/(protected)/admin/orders/[id]/OrderDetailClient.tsx — patrón UI
- admin/src/app/(protected)/admin/orders/page.tsx — patrón de lista

## Lo que debes crear

### `admin/src/lib/assignment.api.ts`
- `getPendingAssignmentOrders()` — GET /admin/orders/map
- `getAvailableCouriers(orderId)` — GET /admin/orders/:id/available-couriers
- `assignCourier(orderId, courierUserId, key)` — POST /admin/orders/:id/assign-courier
- `unassignCourier(orderId, key)` — POST /admin/orders/:id/unassign-courier
- Tipos: `PendingOrder` (con lat/lng), `AvailableCourier`

### `admin/src/app/(protected)/admin/orders/map/page.tsx`
Shell server con `generateStaticParams([])`.

### `admin/src/app/(protected)/admin/orders/map/_components/AssignmentMap.tsx`
('use client')
- Mapa Leaflet cargado via dynamic import (no SSR): `import dynamic from 'next/dynamic'`
- Markers para pedidos pendientes (ícono rojo) y couriers disponibles (ícono verde)
- Al clicar un pedido: seleccionarlo y abrir panel lateral
- Al clicar un courier: ver info básica
- Auto-refresh cada 30s con `setInterval`
- Leaflet via CDN en `<Script>` o via paquete npm — usa el patrón ya existente
  en el storefront checkout (AddressStep.tsx tiene mapa Leaflet — cópialo)

### `admin/src/app/(protected)/admin/orders/map/_components/OrdersPanel.tsx`
Panel lateral izquierdo:
- Lista de pedidos en AWAITING_COURIER_ASSIGNMENT
- Clic en pedido: highlight en mapa + mostrar couriers disponibles
- Botón "Asignar" → modal de confirmación → llamada a assignCourier()

### Modificar (mínimo): `admin/src/app/(protected)/admin/orders/[id]/OrderDetailClient.tsx`
Agregar una sección "Asignación de Repartidor" al detalle del pedido:
- Si `order_status === 'AWAITING_COURIER_ASSIGNMENT'`: botón "Ir al mapa de asignación"
  (link a `/admin/orders/map`)
- Si `order_status === 'COURIER_ASSIGNED'`: mostrar nombre del courier asignado
  + botón "Desasignar"

## Lo que NO debes tocar

- `admin/src/app/(protected)/courier/` (pertenece a FRONT-A)
- `admin/src/lib/orders.api.ts` — solo extiende con `assignment.api.ts`
- `admin/src/app/(protected)/admin/orders/page.tsx`
- Nada de `storefront/`

## Definition of Done

- `pnpm --filter @ruta/admin typecheck` EXIT 0
- `pnpm --filter @ruta/admin lint` EXIT 0
- PR "[3.ADMIN-1] Mapa de asignación de repartidores"
```

---

## Wave 3 — 1 agente (después de Wave 1 + Wave 2 completos)

### QA: `3.QA-1` — E2E flujo SHIP

**Estado:** `[x]` mergeado 2026-05-29 — frontend-ruta #16

**Rama:** `feat/qa-3-1` (en `frontend-ruta`)

**Condición de arranque:** todos los PRs de Wave 1 y Wave 2 mergeados.

---

#### PROMPT QA

```
Eres un agente de QA trabajando en el proyecto RUTA, una plataforma
SaaS multi-tenant para operaciones comerciales (Colombia, COP).

## Tu tarea: 3.QA-1 — E2E flujo SHIP con Playwright

Implementa la suite E2E que valida el flujo SHIP completo de extremo a extremo.

## Contexto del proyecto

Stack: Playwright, Next.js 14, Express.js backend.
Repo: frontend-ruta.
Working dir: /mnt/c/Dropbox/Alex/DEV/ruta/frontend-ruta
Branch de trabajo: crea y trabaja en `feat/qa-3-1` desde main actualizado.

Lee primero:
- docs-ruta/estrategia_testing.md — estrategia de testing del proyecto
- docs-ruta/flujos/flujo_2_ship_completo.txt — flujo completo a testear
- frontend-ruta/playwright.config.ts (si existe) — configuración actual

## Flujo a cubrir (happy path)

1. ADMIN_CLIENT crea pedido y valida → READY_TO_SHIP
2. ADMIN_CLIENT abre mapa de asignación → asigna courier
3. COURIER inicia envío → SHIPPED → IN_TRANSIT → ARRIVED_AT_CUSTOMER
4. COURIER registra cobro (si COD) → mark-delivered → DELIVERED
5. BUYER confirma recepción → CONFIRMED_BY_CUSTOMER → COMPLETED_SUCCESSFULLY

## Flujos alternativos a cubrir

- Cancelación post-despacho: BUYER solicita → ADMIN aprueba → RETURN_TO_ORIGIN
- Intento fallido: COURIER marca DELIVERY_ATTEMPTED → 2do intento → DELIVERED
- Auto-confirmación: pedido en DELIVERED sin acción del buyer → CONFIRMED_BY_SYSTEM
  (mockear el job o avanzar tiempo)

## Definition of Done

- Suite Playwright pasa en CI
- Cubre happy path + 2 flujos alternativos mínimo
- PR "[3.QA-1] E2E flujo SHIP completo"
```

---

## Tabla de seguimiento

| Tarea | Agente | Rama | Estado | PR | Mergeado |
|---|---|---|---|---|---|
| Pre-wave: verificar shared | — | — | ✅ 2026-05-28 | — | — |
| `3.SHARED-1` recordCollectionSchema | — | `packages-ruta` | ✅ 2026-05-28 | — | publicado 1.3.0 |
| `3.BACK-1` Asignación courier | BACK-A | `feat/back-3-1` | ✅ 2026-05-29 | #9 | ✅ main |
| `3.BACK-2+3` Endpoints + cobro | BACK-B | `feat/back-3-2-3` | ✅ 2026-05-29 | #12 | ✅ main |
| `3.BACK-4+5` Cancel + RETURN | BACK-C | `feat/back-3-4-5` | ✅ 2026-05-29 | #11 | ✅ main |
| `3.BACK-6` Auto-confirmación | BACK-D | `feat/back-3-6` | ✅ 2026-05-29 | #10 | ✅ main |
| `3.ADMIN-2` Vista courier | FRONT-A | `feat/admin-3-2` | ✅ 2026-05-29 | #14 | ✅ main |
| Integration: mounts app.ts | — | — | ✅ 2026-05-29 | — | ✅ main |
| `3.ADMIN-1` Mapa asignación | FRONT-B | `feat/admin-3-1` | ✅ 2026-05-29 | #15 | ✅ main |
| `3.QA-1` E2E SHIP | QA | `feat/qa-3-1-route-e2e` | ✅ 2026-05-29 | #16 | ✅ main |

Notas QA 2026-05-29:
- Se corrigió la ruta canónica del mapa a `/admin/orders/map`; `/admin/map` queda como redirect de compatibilidad.
- Se agregó Playwright en `frontend-ruta` con suite SHIP.
- PR mergeado: https://github.com/orkoruta/frontend-ruta/pull/16
- CI ejecuta `pnpm exec playwright test --project=chromium`.
- Cobertura E2E: asignación de courier, entrega COD, intento fallido y pedido confirmado por sistema.

---

## Criterio de cierre Sprint 3 ✅ COMPLETO 2026-05-29

Flujo SHIP cubierto en código y CI:
- Admin asigna courier a un pedido.
- Courier completa entrega COD.
- Courier registra intento fallido.
- Sistema muestra pedido confirmado por sistema.
- Cancelación post-despacho y return-to-origin están cubiertos por backend y tests.
- Suite E2E Playwright verde en CI.
- No quedan PRs abiertos del Sprint 3.
