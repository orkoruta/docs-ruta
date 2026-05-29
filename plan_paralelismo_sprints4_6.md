# Plan de Paralelismo — Sprints 4, 5 y 6

Documento autoritativo para coordinar agentes IA trabajando en paralelo.
Cada agente debe leer este archivo completo antes de empezar su tarea.

**Fuente de verdad de tareas:** `docs-ruta/plan_tareas.md`
**Plan Sprints 1–3:** `docs-ruta/plan_paralelismo_sprints2_6.md`
**Estado de entrada:** Sprints 0–3 ✅ CERRADOS (2026-05-29). CI `backend-ruta/main` y `frontend-ruta/main` verde.

---

## Correcciones post-validación (ya aplicadas — NO re-implementar)

- **CORS:** middleware `cors` en `app.ts` (origins `localhost:3002`, `localhost:3003`, `credentials: true`)
- **Prefijo `/v1`:** eliminado de todos los `*.api.ts` del frontend (`http://localhost:3001` sin prefijo)
- **Idempotencia en auth:** `requireIdempotencyKey` eliminado de `/auth/login`, `/auth/ruta-admin/login`, `/auth/register`
- **Login redirect:** apunta temporalmente a `/admin/orders` (dashboard no existe aún — lo crea Sprint 5, Agente H)
- **Gap identificado:** `/admin/pickup-points` backend CRUD no existe → corregido en Wave 4.0

---

## Reglas de coordinación (OBLIGATORIO leer antes de empezar)

1. **Cada agente tiene jurisdicción exclusiva** sobre los archivos listados en su sección.
2. **Nunca modificar archivos de otro agente.** Si necesitas algo de otro agente, impórtalo, no lo edites.
3. **`app.ts` del backend es el único archivo compartido permitido:** cada agente solo agrega su bloque con comentario `// N.TASK-N`. Nada más.
4. **Branch propia por tarea:** `fix/admin-pickup-points`, `feat/back-4-1`, etc.
5. **No crear `index.ts` barrels compartidos** — pueden causar conflictos de merge.
6. **Una Wave no arranca hasta que la anterior esté mergeada a `main`** (o la dependencia específica indicada).
7. **Definition of Done:** typecheck limpio + tests pasando + lint OK + build OK antes de hacer PR.

---

## Protocolo de agente especializado (TODOS los agentes, OBLIGATORIO antes de codificar)

### Paso 1 — Análisis previo (entregar ANTES de escribir código)

1. **Documentación revisada:** lista de `.md` y `.txt` revisados.
2. **Reglas detectadas:** reglas de negocio aplicables, restricciones técnicas, decisiones ya definidas.
3. **Flujos obligatorios:** flujos que gobiernan esta tarea y el orden de pasos.
4. **Vacíos o contradicciones:** puntos no documentados. Si falta info, no inventes — bloquea y reporta.
5. **Plan de ejecución:** cómo vas a resolver la tarea respetando la documentación.

### Paso 2 — Prioridad durante la ejecución

1. Instrucciones directas del usuario.
2. Reglas de negocio documentadas (`docs-ruta/CLAUDE.md`, `docs-ruta/all_ruta.md`).
3. Flujos funcionales (`docs-ruta/flujos/`).
4. Documentación técnica (`contrato_api.md`, `estrategia_testing.md`, `matriz_permisos.md`).
5. Convenciones de código existentes en el repo.
6. Buenas prácticas generales — solo si no contradicen la documentación.

### Paso 3 — Bloqueo ante información faltante

> "No encontré una definición documentada para: [descripción]. No voy a inventar una solución. Se requiere confirmación del usuario."

### Paso 4 — Validación de cumplimiento (antes del PR)

| Criterio | Estado |
|----------|--------|
| Documentación revisada y respetada | Sí / No |
| Flujos existentes respetados | Sí / No |
| Reglas nuevas inventadas | No (siempre No) |
| Cambios fuera del alcance | No (siempre No) |
| Contradicciones encontradas | Sí / No (detallar) |
| Resumen de cambios realizados | [descripción] |

---

## Protocolo de calidad — cero errores, cero warnings (TODOS los agentes)

```
[ ] pnpm typecheck   → EXIT 0
[ ] pnpm lint        → EXIT 0
[ ] pnpm test        → todos pasan, cero fallos, cero skipped sin justificación
[ ] pnpm build       → EXIT 0
[ ] Sin console.log de depuración en producción
[ ] Sin `any` implícito
[ ] Sin TODO sin ticket del plan
[ ] Sin secrets hardcodeados
```

**Backend además:**
```
[ ] Nuevos endpoints con validación Zod del body/query
[ ] POST/PUT/PATCH/DELETE verifican X-Idempotency-Key (excepto /auth/*)
[ ] Endpoints protegidos verifican auth (requireAuth / requireAdminClient / requireAdminRuta)
[ ] Queries a BD usan withTenant() — nunca queries globales sin contexto de tenant
[ ] Tests cubren: caso feliz + error de validación + autenticación fallida
```

**Frontend además:**
```
[ ] Build sin warnings de Next.js
[ ] Formularios muestran errores de validación visibles al usuario
[ ] Llamadas a API manejan 401, 403, 500 con feedback al usuario
[ ] Páginas protegidas redirigen a /login si no hay sesión
[ ] No hay tokens en localStorage — solo cookies HttpOnly
```

---

## Registro de avance obligatorio (TODOS los agentes, sin excepción)

Al terminar — antes de abrir el PR — actualizar:

1. `docs-ruta/plan_tareas.md` → cambiar `[ ]` a `[x]` en la tarea completada.
2. `docs-ruta/plan_paralelismo_sprints4_6.md` (este archivo) → marcar tarea con ✅ y fecha en la tabla de avance.
3. Archivo de memoria del sprint en `/home/alexander_marquez/.claude/projects/-mnt-c-Dropbox-Alex-DEV-ruta/memory/project_sprint[N]_status.md` → mover tarea de PENDIENTE a HECHO.
4. `MEMORY.md` → actualizar línea del sprint si cambió significativamente.

**Orden:** código → plan_tareas → este archivo → memory/sprint → PR.

---

## Contexto técnico para todos los agentes

- **BD:** PostgreSQL OCI `149.130.168.24:26432/rutadb` (no Supabase)
- **API base URL local:** `http://localhost:3001` (sin prefijo `/v1`)
- **Admin:** `:3002` · **Storefront:** `:3003`
- **Paquetes:** `@orkoruta/shared@1.2.0`, `@orkoruta/db@1.0.0`
- **Design system:** `Ruta*` solo en `frontend-ruta/packages/ui/` — admin lo importa, storefront NO
- **Auth cookies:** HttpOnly, Secure, SameSite=Strict — no localStorage
- **Multi-tenant:** toda query backend usa `withTenant(clientId, role, fn)`
- **Idempotencia:** todo POST/PUT/PATCH/DELETE requiere `X-Idempotency-Key` (excepto `/auth/*`)
- **Tests backend:** `export NPM_TOKEN=$(grep "_authToken=" ~/.npmrc | cut -d= -f2)` antes de `pnpm test`
- **`state_machine.ts`** ya existe con Flujos 1+2+3 — Sprint 4 agrega PICKUP, no tocar lo existente
- **`app.ts`** ya tiene mounts hasta Sprint 3 — cada agente agrega solo su bloque con comentario

---

## Documentación mínima a revisar por track

| Track | Documentos obligatorios |
|-------|------------------------|
| Backend | `CLAUDE.md`, `contrato_api.md`, `all_ruta.md`, `matriz_permisos.md`, `parametros_negocio.md`, `estrategia_testing.md`, `bd/ruta_postgres.sql`, flujos relevantes en `flujos/` |
| Admin frontend | `CLAUDE.md`, `wireframes_mvp.md`, `diseno/galeria_estilos_ruta.md`, `matriz_permisos.md`, `contrato_api.md`, `all_ruta.md` |
| QA / E2E | `CLAUDE.md`, `estrategia_testing.md`, `arquitectura/estrategia_multi_tenant_ruta.md`, `matriz_permisos.md`, `contrato_api.md` |
| INFRA | `CLAUDE.md`, `all_ruta.md`, docs relevantes de `arquitectura/` |
| DOCS | `wireframes_mvp.md`, `all_ruta.md`, `matriz_permisos.md` |

---

---

# SPRINT 4 — Flujo PICKUP

**Dependencia de entrada:** Sprint 3 completamente mergeado. CI verde.
**Duración estimada:** 1 semana.

## Diagrama de waves Sprint 4

```
Wave 4.0 — 1 agente (BLOQUEANTE — ejecutar primero, solo):
  Agente ZERO → 4.FIX-1  ── /admin/pickup-points CRUD (gap Sprint 1)

Wave 4.1 — 2 agentes en paralelo (tras merge 4.FIX-1 a main):
  Agente A → 4.BACK-1 (L)  ── Transiciones PICKUP + service + job expiración
  Agente B → 4.ADMIN-1 (M) ── UI botones PICKUP en detalle de pedido

Wave 4.2 — 1 agente (tras merge Wave 4.1):
  Agente C → 4.QA-1 (M)   ── E2E flujo PICKUP completo
```

---

### AGENTE ZERO — 4.FIX-1 (Wave 4.0 — SOLO, BLOQUEANTE)

**Repo:** `backend-ruta/`
**Rama:** `fix/admin-pickup-points`
**Tamaño:** S

**Contexto:** `1.BACK-4` (Sprint 1) implementó buyers y couriers pero omitió el CRUD admin de pickup_points. El frontend (`lib/users.api.ts`) llama `/admin/pickup-points/*` y recibe 404. El patrón a seguir es idéntico a `admin_buyers.ts` + `buyers.service.ts`.

**Qué hacer:**
- `src/services/pickup_points.service.ts` — list (paginado, filtros status/search), getById, create, update, activate, deactivate
- `src/routes/admin_pickup_points.ts` — GET /, POST /, GET /:id, PATCH /:id, POST /:id/activate, POST /:id/deactivate; `requireAdminClient` + `requireIdempotencyKey` en mutaciones
- Tests: auth, lista, getById, crear, actualizar, activar/desactivar (seguir patrón de `buyers.test.ts`)

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/
│   └── pickup_points.service.ts    ← NUEVO
├── routes/
│   └── admin_pickup_points.ts      ← NUEVO
└── __tests__/
    └── pickup_points.test.ts       ← NUEVO
```

**Archivo compartido:** `src/app.ts` — solo agregar:
```typescript
app.use('/admin/pickup-points', adminPickupPointsRouter); // 4.FIX-1
```
Montar ANTES de `/admin/orders` para evitar conflicto de rutas.

**Solo lectura (referencia de patrón):** `services/buyers.service.ts`, `routes/admin_buyers.ts`, `__tests__/buyers.test.ts`

**Documentos obligatorios:** `contrato_api.md`, `matriz_permisos.md`, `estrategia_testing.md`

**Criterio:** typecheck ✓ · tests ✓ · build ✓ · PR `[4.FIX-1] Admin pickup-points CRUD (gap Sprint 1)` mergeado a main antes de Wave 4.1.

---

### AGENTE A — 4.BACK-1 (Wave 4.1)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-4-1`
**Tamaño:** L

**Contexto relevante del código existente:**
- `state_machine.ts` tiene transiciones Flujos 1+2+3. Sprint 4 agrega la rama PICKUP.
- `admin_orders.ts` tiene `accept`, `reject`, `mark-preparing`, `mark-ready`, `approve-cancel`, `return-to-origin` — no se toca.
- El flujo PICKUP: `READY_FOR_PICKUP → DELIVERED` (actor: OPERATOR_CLIENT); `READY_FOR_PICKUP → EXPIRED` (actor: SYSTEM).

**Qué hacer:**
- `src/services/orders/pickup_ops.service.ts`:
  - `verifyBuyerIdentity(orderId, clientId, actingUser, documentData)` — verifica documento del comprador; audita en `audit_events`
  - `recordPickupCollection(orderId, clientId, actingUser, amount)` — registra cobro COD en punto; crea registro en `payments`
  - `markPickupDelivered(orderId, clientId, actingUser)` — transiciona READY_FOR_PICKUP → DELIVERED; optimistic lock
- `src/routes/admin_pickup_ops.ts`:
  - `POST /admin/orders/:id/verify-pickup-identity` — requireAdminClient | requireOperatorClient + requireIdempotencyKey
  - `POST /admin/orders/:id/pickup-collection` — requireAdminClient | requireOperatorClient + requireIdempotencyKey
  - `POST /admin/orders/:id/mark-pickup-delivered` — requireAdminClient | requireOperatorClient + requireIdempotencyKey
- `src/jobs/pickup_expiration.job.ts` — cron `*/5 * * * *`; expira READY_FOR_PICKUP por `order.pickup_expiration_minutes`; transiciona → EXPIRED
- Modificar `state_machine.ts` — agregar al objeto de transiciones:
  - `READY_FOR_PICKUP → DELIVERED` (actor: OPERATOR_CLIENT)
  - `READY_FOR_PICKUP → EXPIRED` (actor: SYSTEM)
  - No tocar ninguna transición existente

**Archivos exclusivos (NUEVOS):**
```
backend-ruta/api/src/
├── services/orders/
│   └── pickup_ops.service.ts         ← NUEVO
├── routes/
│   └── admin_pickup_ops.ts           ← NUEVO
└── jobs/
    └── pickup_expiration.job.ts      ← NUEVO
```

**Archivo a MODIFICAR (con cuidado):**
```
backend-ruta/api/src/services/orders/
└── state_machine.ts    ← MODIFICAR — solo agregar entradas PICKUP, sin tocar transiciones existentes
```

**Archivo compartido:** `src/app.ts` — solo agregar:
```typescript
app.use('/admin', adminPickupOpsRouter); // 4.BACK-1
```

**Solo lectura:** `state_machine.ts` (existente), `orders.service.ts`, `courier_ops.service.ts`, `collection.service.ts`
**NO tocar:** `admin_orders.ts`, `admin_order_assignment.ts`, `courier_orders.ts`

**Documentos obligatorios:** `flujos/flujo_1.txt` (sección PICKUP), `contrato_api.md`, `parametros_negocio.md`, `matriz_permisos.md`

**Criterio:** typecheck ✓ · tests (happy path + 422 transición inválida + 403 buyer intentando acceder) ✓ · PR `[4.BACK-1] Flujo PICKUP backend`

---

### AGENTE B — 4.ADMIN-1 (Wave 4.1 — paralelo con A)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-4-1`
**Tamaño:** M

**Contexto:**
- `admin/orders/[id]/OrderDetailClient.tsx` ya existe con acciones condicionales por `order.status` y `order.delivery_type`
- Los endpoints PICKUP están en `contrato_api.md` — implementar aunque el backend no esté mergeado aún
- Patrón: crear componente separado e importarlo condicionalmente en `OrderDetailClient.tsx`

**Qué hacer:**
- `_components/PickupActions.tsx` — componente `'use client'` con 3 botones condicionales:
  - "Verificar identidad" → `POST /admin/orders/:id/verify-pickup-identity`
  - "Registrar cobro" (campo monto COP) → `POST /admin/orders/:id/pickup-collection`
  - "Marcar entregado" → `POST /admin/orders/:id/mark-pickup-delivered`
  - Cada botón con loading + error inline + `X-Idempotency-Key` (uuid v4) automático
- `lib/pickup_ops.api.ts` — `verifyPickupIdentity()`, `recordPickupCollection()`, `markPickupDelivered()` con `credentials: 'include'`
- Modificar `OrderDetailClient.tsx` — agregar al bloque de acciones:
  ```tsx
  {order.delivery_type === 'PICKUP' && order.status === 'READY_FOR_PICKUP' && (
    <PickupActions order={order} onAction={refetch} />
  )}
  ```
  Solo esta adición — no refactorizar ni tocar ninguna otra lógica existente

**Archivos exclusivos (NUEVOS):**
```
frontend-ruta/admin/src/
├── app/(protected)/admin/orders/[id]/_components/
│   └── PickupActions.tsx                     ← NUEVO
└── lib/
    └── pickup_ops.api.ts                     ← NUEVO
```

**Archivo a MODIFICAR:**
```
frontend-ruta/admin/src/app/(protected)/admin/orders/[id]/
└── OrderDetailClient.tsx    ← MODIFICAR — solo agregar import + render condicional PickupActions
```

**Solo lectura:** `orders/[id]/page.tsx`, `lib/orders.api.ts`, `components/RutaSidebar.tsx`, `components/RutaHeader.tsx`
**NO tocar:** ningún archivo fuera de los listados; no tocar `app.ts` del backend

**Documentos obligatorios:** `wireframes_mvp.md` (pantallas PICKUP), `contrato_api.md`, `matriz_permisos.md`

**Criterio:** typecheck ✓ · lint ✓ · build ✓ · PR `[4.ADMIN-1] UI operación punto físico`

---

### AGENTE C — 4.QA-1 (Wave 4.2 — tras merge completo Wave 4.1)

**Repo:** `frontend-ruta/`
**Rama:** `feat/qa-4-1`
**Tamaño:** M

**Contexto:** Suite Playwright configurada desde Sprint 3; `e2e/ship_flow.spec.ts` y `e2e/fixtures/ship_flow.fixture.ts` como referencia de patrón.

**Qué hacer:**
- `e2e/pickup_flow.spec.ts` — spec completo:
  1. BUYER hace checkout con `delivery_type: PICKUP`, selecciona pickup point
  2. ADMIN_CLIENT acepta → mark-preparing → mark-ready (READY_FOR_PICKUP)
  3. OPERATOR_CLIENT verifica identidad del comprador
  4. OPERATOR_CLIENT registra cobro o marca entregado directamente
  5. Pedido llega a DELIVERED → confirmación
- `e2e/fixtures/pickup_flow.fixture.ts` — setup de datos (buyer, pedido PICKUP, pickup point activo)

**Archivos exclusivos (NUEVOS):**
```
frontend-ruta/e2e/
├── pickup_flow.spec.ts              ← NUEVO
└── fixtures/
    └── pickup_flow.fixture.ts       ← NUEVO
```

**Solo lectura:** `e2e/ship_flow.spec.ts`, `e2e/fixtures/ship_flow.fixture.ts` (referencia)
**NO tocar:** specs o fixtures existentes de Sprint 3

**Criterio:** `pnpm test:e2e --project=chromium` pasa ✓ · CI verde ✓ · PR `[4.QA-1] E2E flujo PICKUP`

---

---

# SPRINT 5 — Vista de Control, dashboards, configuración

**Dependencia de entrada:** Sprint 4 completamente mergeado. CI verde.
**Duración estimada:** 1 semana.

## Diagrama de waves Sprint 5

```
Wave 5.1 — 3 agentes en paralelo (backend — ramas separadas de backend-ruta):
  Agente D → 5.BACK-1 (L)      ── Vista de Control (enter/exit + auditoría)
  Agente E → 5.BACK-2 (L)      ── Métricas (Cliente + global ADMIN_RUTA)
  Agente F → 5.BACK-3+4 (M+M)  ── Parámetros GET/PATCH + Auditoría endpoint

Wave 5.2 — 3 agentes en paralelo (frontend — tras merge COMPLETO Wave 5.1):
  Agente G → 5.ADMIN-1 (M)     ── Pantalla Vista de Control + activar banner RutaHeader
  Agente H → 5.ADMIN-2 (M)     ── Dashboards admin (Cliente + ADMIN_RUTA)
  Agente I → 5.ADMIN-3+4 (L+M) ── Configuración tabs + Auditoría UI
```

**Separación de zonas Wave 5.2 (sin solapamiento):**
- G: `ruta-admin/control-view/`, `lib/control_view.api.ts`, `components/RutaHeader.tsx`, `lib/session.ts`
- H: `admin/dashboard/`, `ruta-admin/dashboard/`, `lib/metrics.api.ts`, `app/login/page.tsx`
- I: `admin/settings/`, `admin/audit/`, `lib/parameters.api.ts`, `lib/audit.api.ts`

---

### AGENTE D — 5.BACK-1 (Wave 5.1)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5-1`
**Tamaño:** L

**Contexto:**
- `middleware/auth.ts` ya tiene `requireAdminRuta`; el JWT payload tiene `sub`, `client_id`, `user_type`, `session_id`
- El JWT de impersonación necesita campos adicionales: `impersonating: true` + `target_client_id`
- Toda acción en Vista de Control audita con `acting_via_control_view = TRUE` e `impersonator_user_id`

**Qué hacer:**
- `src/services/control_view.service.ts`:
  - `enterControlView(adminRutaUser, targetClientId, masterPassword, ctx)`: verifica master password (argon2.verify contra parámetro `control_view_master_password` de client_id=0), emite JWT con `impersonating: true`, audita `CONTROL_VIEW_ENTERED`
  - `exitControlView(session, ctx)`: revoca sesión de impersonación, audita `CONTROL_VIEW_EXITED`
- `src/middleware/control_view.ts` — `requireControlView`: verifica `impersonating === true` en JWT; 403 si no
- `src/routes/ruta_admin_control_view.ts`:
  - `POST /ruta-admin/control-view/enter` — `requireAdminRuta`
  - `POST /ruta-admin/control-view/exit` — `requireAdminRuta`
- Modificar `lib/token.ts` — extender `TokenPayload` con campos opcionales `impersonating?` y `target_client_id?`

**Archivos exclusivos (NUEVOS):**
```
backend-ruta/api/src/
├── services/
│   └── control_view.service.ts          ← NUEVO
├── middleware/
│   └── control_view.ts                  ← NUEVO
└── routes/
    └── ruta_admin_control_view.ts       ← NUEVO
```

**Archivo a MODIFICAR (mínimo):**
```
backend-ruta/api/src/lib/
└── token.ts    ← MODIFICAR — agregar campos opcionales a TokenPayload, sin romper firma existente
```

**Archivo compartido:** `src/app.ts` — solo agregar mount con comentario `// 5.BACK-1`
**Solo lectura:** `middleware/auth.ts`, `services/auth.service.ts`, `lib/password.ts`
**NO tocar:** archivos de Agentes E o F

**Documentos obligatorios:** `seguridad/ciclo_vida_token.txt`, `all_ruta.md` (Vista de Control), `matriz_permisos.md`, `contrato_api.md`

**Criterio:** typecheck ✓ · tests (enter OK, master pw incorrecta → 401, exit, auditoría registrada) ✓ · PR `[5.BACK-1] Vista de Control backend`

---

### AGENTE E — 5.BACK-2 (Wave 5.1 — paralelo con D)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5-2`
**Tamaño:** L

**Qué hacer:**
- `src/services/metrics.service.ts` — `getClientMetrics(clientId)`:
  - Pedidos hoy / semana / mes por estado
  - Ingresos: suma `total_amount` de pedidos CLOSED/CONFIRMED
  - Couriers activos en el tenant
  - Tasa de éxito: DELIVERED / total despachados
- `src/services/global_metrics.service.ts` — `getGlobalMetrics()`:
  - Clientes activos por tipo
  - Pedidos totales por estado (cross-tenant, client_id=0)
  - Ingresos globales
- `src/routes/admin_metrics.ts` — `GET /admin/metrics` (requireAdminClient)
- `src/routes/ruta_admin_metrics.ts` — `GET /ruta-admin/metrics` (requireAdminRuta)

**Archivos exclusivos (NUEVOS):**
```
backend-ruta/api/src/
├── services/
│   ├── metrics.service.ts               ← NUEVO
│   └── global_metrics.service.ts        ← NUEVO
└── routes/
    ├── admin_metrics.ts                 ← NUEVO
    └── ruta_admin_metrics.ts            ← NUEVO
```

**Archivo compartido:** `src/app.ts` — solo agregar mounts con comentario `// 5.BACK-2`
**NO tocar:** archivos de Agentes D o F

**Documentos obligatorios:** `contrato_api.md`, `matriz_permisos.md`, `all_ruta.md`

**Criterio:** typecheck ✓ · tests (autenticación, estructura de respuesta correcta) ✓ · PR `[5.BACK-2] Dashboards métricas backend`

---

### AGENTE F — 5.BACK-3+4 (Wave 5.1 — paralelo con D y E)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5-34`
**Tamaño:** M+M

**Contexto:**
- `lib/parameter.ts` ya tiene `getParameter()` y `getParameterInt()` — no modificar
- `client_parameters` ya tiene filas sembradas — solo exponer endpoints para leer/actualizar
- `audit_events` es append-only — solo SELECT, nunca UPDATE/DELETE

**Qué hacer:**
- **5.BACK-3 — Parámetros:**
  - `src/routes/admin_parameters.ts`:
    - `GET /admin/parameters` — lista todos los `client_parameters` del tenant (paginado, filtrable por `group`)
    - `PATCH /admin/parameters/:key` — actualiza `parameter_value`; `requireAdminClient`; `requireIdempotencyKey`
- **5.BACK-4 — Auditoría:**
  - `src/routes/admin_audit.ts`:
    - `GET /admin/audit-events` — filtros: `entity_type`, `user_id`, `from`, `to`, `page`, `page_size`; solo eventos del propio tenant
    - `GET /ruta-admin/audit-events` — cross-tenant para ADMIN_RUTA; filtro adicional `client_id`

**Archivos exclusivos (NUEVOS):**
```
backend-ruta/api/src/routes/
├── admin_parameters.ts                  ← NUEVO
└── admin_audit.ts                       ← NUEVO
```

**Archivo compartido:** `src/app.ts` — solo agregar mounts con comentario `// 5.BACK-34`
**Solo lectura:** `lib/parameter.ts`
**NO tocar:** archivos de Agentes D o E

**Criterio:** typecheck ✓ · tests (lista params, actualizar param, listar audit con filtros, autorización) ✓ · PR `[5.BACK-3+4] Parámetros y auditoría backend`

---

### AGENTE G — 5.ADMIN-1 (Wave 5.2 — tras merge completo Wave 5.1)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-5-1`
**Tamaño:** M

**Contexto:**
- `RutaHeader.tsx` ya renderiza el banner ámbar visualmente pero la condición `impersonating` no está conectada a la sesión real
- `lib/session.ts` define `RutaSession` — extender con `impersonating?: boolean` y `target_client_id?: number`

**Qué hacer:**
- `src/app/(protected)/ruta-admin/control-view/page.tsx`:
  - Selector de cliente + campo `master_password`
  - Submit → `POST /ruta-admin/control-view/enter` → guarda sesión con `impersonating: true` → redirect a `/admin/orders`
  - Botón "Salir de Vista de Control" → `POST /ruta-admin/control-view/exit` → limpia sesión → redirect `/ruta-admin/clients`
- `src/lib/control_view.api.ts` — `enterControlView(targetClientId, masterPassword)`, `exitControlView()`
- Modificar `src/components/RutaHeader.tsx` — conectar banner ámbar a `session?.impersonating === true`; mostrar nombre del cliente impersonado y botón "Salir"
- Modificar `src/lib/session.ts` — agregar `impersonating?: boolean` y `target_client_id?: number` a `RutaSession`

**Archivos exclusivos (NUEVOS):**
```
frontend-ruta/admin/src/
├── app/(protected)/ruta-admin/control-view/
│   └── page.tsx                              ← NUEVO
└── lib/
    └── control_view.api.ts                   ← NUEVO
```

**Archivos a MODIFICAR:**
```
frontend-ruta/admin/src/
├── components/RutaHeader.tsx    ← MODIFICAR — conectar banner ámbar a session.impersonating
└── lib/session.ts               ← MODIFICAR — agregar campos impersonating y target_client_id
```

**Solo lectura:** `lib/session-context.tsx`, `lib/clients.api.ts`, `app/(protected)/layout.tsx`
**NO tocar:** archivos de Agentes H o I — específicamente `login/page.tsx` (H), `admin/settings/` (I), `admin/audit/` (I), `admin/dashboard/` (H)

**Documentos obligatorios:** `all_ruta.md` (Vista de Control), `wireframes_mvp.md`, `seguridad/ciclo_vida_token.txt`

**Criterio:** typecheck ✓ · lint ✓ · build ✓ · PR `[5.ADMIN-1] Vista de Control frontend`

---

### AGENTE H — 5.ADMIN-2 (Wave 5.2 — paralelo con G e I)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-5-2`
**Tamaño:** M

**Contexto:**
- `/admin/dashboard` y `/ruta-admin/dashboard` no existen — redirect post-login apunta temporalmente a `/admin/orders`
- Al crear las páginas, restaurar el redirect en `login/page.tsx` a `/admin/dashboard`

**Qué hacer:**
- `src/app/(protected)/admin/dashboard/page.tsx` — dashboard ADMIN_CLIENT:
  - Cards: pedidos hoy, en proceso, ingresos semana, couriers activos
  - Tabla de últimos 5 pedidos con link a detalle
- `src/app/(protected)/ruta-admin/dashboard/page.tsx` — dashboard ADMIN_RUTA:
  - Cards: clientes activos, pedidos globales hoy, ingresos globales
- `src/lib/metrics.api.ts` — `getClientMetrics()`, `getGlobalMetrics()`
- Modificar `src/app/login/page.tsx` — restaurar:
  ```typescript
  ADMIN_CLIENT: '/admin/dashboard',
  OPERATOR_CLIENT: '/admin/dashboard',
  ```

**Archivos exclusivos (NUEVOS):**
```
frontend-ruta/admin/src/
├── app/(protected)/admin/dashboard/
│   └── page.tsx                              ← NUEVO
├── app/(protected)/ruta-admin/dashboard/
│   └── page.tsx                              ← NUEVO
└── lib/
    └── metrics.api.ts                        ← NUEVO
```

**Archivo a MODIFICAR:**
```
frontend-ruta/admin/src/app/login/
└── page.tsx    ← MODIFICAR — restaurar ADMIN_CLIENT/OPERATOR_CLIENT → /admin/dashboard
```

**Solo lectura:** `components/RutaSidebar.tsx`, `app/(protected)/layout.tsx`
**NO tocar:** archivos de Agentes G o I — específicamente `RutaHeader.tsx` (G), `lib/session.ts` (G), `admin/settings/` (I), `admin/audit/` (I)

**Criterio:** typecheck ✓ · lint ✓ · build ✓ · PR `[5.ADMIN-2] Dashboards admin`

---

### AGENTE I — 5.ADMIN-3+4 (Wave 5.2 — paralelo con G y H)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-5-34`
**Tamaño:** L+M

**Qué hacer:**
- **5.ADMIN-3 — Configuración:**
  - `src/app/(protected)/admin/settings/page.tsx` — layout con 4 tabs
  - `_components/BusinessInfoTab.tsx` — info básica del tenant (solo lectura)
  - `_components/WompiTab.tsx` — display de public key configurada (solo lectura)
  - `_components/WebhooksTab.tsx` — lista de webhook_subscriptions + historial placeholder (**Sprint 6 lo completa**)
  - `_components/ParametersTab.tsx` — tabla de parámetros con edición inline; llama `PATCH /admin/parameters/:key`
  - `src/lib/parameters.api.ts` — `getParameters()`, `updateParameter(key, value)`
- **5.ADMIN-4 — Auditoría:**
  - `src/app/(protected)/admin/audit/page.tsx` — tabla paginada; filtros: rango fechas, tipo entidad, usuario
  - `src/lib/audit.api.ts` — `getAuditEvents(filters)`

**Archivos exclusivos (todos NUEVOS):**
```
frontend-ruta/admin/src/
├── app/(protected)/admin/
│   ├── settings/
│   │   ├── page.tsx                          ← NUEVO
│   │   └── _components/
│   │       ├── BusinessInfoTab.tsx            ← NUEVO
│   │       ├── WompiTab.tsx                   ← NUEVO
│   │       ├── WebhooksTab.tsx                ← NUEVO (stub — Sprint 6 lo enriquece)
│   │       └── ParametersTab.tsx              ← NUEVO
│   └── audit/
│       └── page.tsx                           ← NUEVO
└── lib/
    ├── parameters.api.ts                      ← NUEVO
    └── audit.api.ts                           ← NUEVO
```

**Solo lectura:** `components/RutaSidebar.tsx`, `lib/orders.api.ts` (referencia de patrón)
**NO tocar:** archivos de Agentes G o H — específicamente `RutaHeader.tsx` (G), `lib/session.ts` (G), `login/page.tsx` (H), `admin/dashboard/` (H)

**Criterio:** typecheck ✓ · lint ✓ · build ✓ · PR `[5.ADMIN-3+4] Configuración y auditoría frontend`

---

---

# SPRINT 6 — Hardening, observabilidad, deploy, piloto

**Dependencia de entrada:** Sprint 5 completamente mergeado. CI verde.
**Duración estimada:** 1 semana.

## Diagrama de waves Sprint 6

```
Wave 6.1 — 3 agentes en paralelo:
  Agente J → 6.INFRA-1 (L)    ── Observabilidad (pino enriquecido + Logtail/Axiom)
  Agente K → 6.INFRA-3 (L)    ── Webhooks salientes (pg-boss + WebhooksTab real)
  Agente L → 6.QA-1+2 (L+M)  ── Suite E2E completa + cobertura backend >85%

Wave 6.2 — 2 agentes en paralelo (puede correr simultáneamente con Wave 6.1):
  Agente M → 6.INFRA-2 (M)    ── Backups BD + restore probado
  Agente N → 6.QA-4 (M)       ── Documentación de usuario por rol

Wave 6.3 — coordinada con humano (ÚLTIMO — requiere aprobación explícita):
  Agente O → 6.INFRA-4 + 6.QA-3 ── Deploy a producción + onboarding piloto
```

**Separación de zonas Wave 6.1:**
- J: `middleware/logger.ts` (MOD), `lib/logger.ts` (NEW) en backend + `infra-ruta/docs/`
- K: `services/webhooks_outgoing.service.ts` (NEW), `jobs/webhook_sender.job.ts` (NEW), `settings/_components/WebhooksTab.tsx` (MOD en admin)
- L: `e2e/full_suite.spec.ts` (NEW), `e2e/helpers/` (NEW), `vitest.config.ts` (MOD en backend)

Wave 6.2: repos completamente distintos (`infra-ruta/` y `docs-ruta/`) — sin solapamiento posible.

---

### AGENTE J — 6.INFRA-1 (Wave 6.1)

**Repo:** `backend-ruta/` + `infra-ruta/`
**Rama:** `feat/infra-6-1`
**Tamaño:** L

**Contexto:** `src/middleware/logger.ts` existe con pino básico. Se enriquece con correlación y se agrega transporte externo solo en producción.

**Qué hacer:**
- Modificar `src/middleware/logger.ts` — agregar al log de cada request: `client_id`, `user_id`, `user_type` (desde `req.user`), `trace_id` (UUID por request)
- `src/lib/logger.ts` (NUEVO) — singleton pino exportable:
  - `NODE_ENV=production`: transport a Logtail (`@logtail/pino`) o Axiom
  - Dev: pretty print a consola
- Agregar `LOGTAIL_TOKEN=` a `.env.example`
- `infra-ruta/docs/observability_setup.md` — procedimiento de configuración, umbrales de alerta, dashboards recomendados

**Archivo a MODIFICAR:**
```
backend-ruta/api/src/middleware/
└── logger.ts    ← MODIFICAR — enriquecer con client_id, user_id, trace_id
```

**Archivos exclusivos (NUEVOS):**
```
backend-ruta/api/src/lib/
└── logger.ts                              ← NUEVO (singleton pino con transport prod)
infra-ruta/docs/
└── observability_setup.md                 ← NUEVO
```

**Solo lectura:** `src/config/env.ts`, `src/middleware/auth.ts`
**NO tocar:** archivos de Agentes K o L; no requiere cambio en `app.ts`

**Criterio:** typecheck ✓ · PR `[6.INFRA-1] Observabilidad pino + Logtail`

---

### AGENTE K — 6.INFRA-3 (Wave 6.1 — paralelo con J)

**Repo:** `backend-ruta/` + `frontend-ruta/admin/`
**Rama:** `feat/infra-6-3`
**Tamaño:** L

**Contexto:**
- `webhook_subscriptions` y `webhook_deliveries` existen en BD; `webhook_deliveries` es append-only
- `WebhooksTab.tsx` fue creado en Sprint 5 como stub — este agente lo completa con historial real
- `pg-boss` ya está configurado y en uso desde Sprint 1

**Qué hacer:**
- `src/services/webhooks_outgoing.service.ts`:
  - `processWebhookEvent(eventType, payload, clientId)`: lee `webhook_subscriptions`, invoca URL (fetch, timeout 10s), INSERT en `webhook_deliveries` (append-only)
  - `retryWebhookDelivery(deliveryId, clientId)`: reencola delivery fallido
- `src/jobs/webhook_sender.job.ts` — pg-boss con reintentos exponenciales: 1m, 5m, 15m, 60m, 240m
- `admin/src/lib/webhooks.api.ts` (NUEVO) — `getWebhookDeliveries(filters)`, `retryDelivery(deliveryId)`
- Modificar `admin/src/app/(protected)/admin/settings/_components/WebhooksTab.tsx` — reemplazar stub por tabla de entregas con estado (DELIVERED/FAILED/RETRYING) y botón "Reintentar"

**Archivos exclusivos (NUEVOS):**
```
backend-ruta/api/src/
├── services/
│   └── webhooks_outgoing.service.ts       ← NUEVO
└── jobs/
    └── webhook_sender.job.ts              ← NUEVO

frontend-ruta/admin/src/lib/
└── webhooks.api.ts                        ← NUEVO
```

**Archivo a MODIFICAR:**
```
frontend-ruta/admin/src/app/(protected)/admin/settings/_components/
└── WebhooksTab.tsx    ← MODIFICAR — reemplazar stub por historial real
```

**Archivo compartido:** `backend-ruta/api/src/app.ts` — solo agregar init del job con comentario `// 6.INFRA-3`
**NO tocar:** otros tabs de settings (BusinessInfoTab, WompiTab, ParametersTab); archivos de Agentes J o L

**Criterio:** typecheck ✓ · tests (envío, reintento, fallo max-retries) ✓ · PR `[6.INFRA-3] Webhooks salientes + UI historial`

---

### AGENTE L — 6.QA-1+2 (Wave 6.1 — paralelo con J y K)

**Repo:** `frontend-ruta/` + `backend-ruta/`
**Rama:** `feat/qa-6-12`
**Tamaño:** L+M

**Contexto:** `e2e/ship_flow.spec.ts` y `e2e/pickup_flow.spec.ts` ya existen.

**Qué hacer:**
- **6.QA-1 — Suite E2E completa:**
  - `e2e/helpers/auth.helper.ts` — `loginAs(role, slug?)`, `logout()`, `clearSession()`
  - `e2e/full_suite.spec.ts` — orquesta ship + pickup + agrega:
    - Cancelación por buyer (requestCancel → admin aprueba)
    - Acceso a auditoría como ADMIN_CLIENT
    - Flujo de Vista de Control (ADMIN_RUTA entra, opera, sale)
- **6.QA-2 — Cobertura backend >85%:**
  - Modificar `backend-ruta/api/vitest.config.ts` — `coverage: { provider: 'v8', thresholds: { lines: 85 } }`
  - Agregar tests unitarios donde la cobertura esté baja; **no modificar código de producción**

**Archivos exclusivos (NUEVOS):**
```
frontend-ruta/e2e/
├── full_suite.spec.ts                     ← NUEVO
└── helpers/
    └── auth.helper.ts                     ← NUEVO
```

**Archivo a MODIFICAR:**
```
backend-ruta/api/vitest.config.ts          ← MODIFICAR — agregar coverage config
```

**Solo lectura:** todos los archivos de producción de todos los sprints
**NO tocar:** código de producción de backend o frontend; archivos de Agentes J o K

**Criterio:** `pnpm test:e2e` pasa en CI ✓ · `pnpm test -- --coverage` >85% líneas ✓ · PR `[6.QA-1+2] Suite E2E + cobertura`

---

### AGENTE M — 6.INFRA-2 (Wave 6.2 — puede correr con Wave 6.1)

**Repo:** `infra-ruta/`
**Rama:** `feat/infra-6-2`
**Tamaño:** M

**Qué hacer:**
- `scripts/backup_db.sh` — `pg_dump --format=custom`, nombre con timestamp, upload a OCI Object Storage o S3-compatible
- `scripts/restore_db.sh` — restore desde archivo especificado; verificación post-restore (cuenta tablas y filas clave); probado contra BD staging
- `docs/backups.md` — procedimiento, frecuencia, retención, restore de emergencia

**Archivos exclusivos (todos NUEVOS):**
```
infra-ruta/
├── scripts/
│   ├── backup_db.sh                       ← NUEVO
│   └── restore_db.sh                      ← NUEVO
└── docs/
    └── backups.md                         ← NUEVO
```

**NO tocar:** ningún archivo de `backend-ruta/` ni `frontend-ruta/`

**Criterio:** `bash -n` EXIT 0 en ambos scripts · restore probado con verificación de integridad · PR `[6.INFRA-2] Backups BD`

---

### AGENTE N — 6.QA-4 (Wave 6.2 — paralelo con M)

**Repo:** `docs-ruta/`
**Rama:** `feat/docs-6-4`
**Tamaño:** M

**Qué hacer:**
- `guias/admin_client.md` — catálogo, pedidos, mapa, compradores/repartidores/pickup points, configuración, auditoría
- `guias/operator_client.md` — operación diaria, aceptar/rechazar pedidos, preparación y despacho
- `guias/courier.md` — app móvil: mis pedidos, iniciar entrega, cobro COD, evidencia, pedido entregado
- `guias/buyer.md` — catálogo, carrito, checkout SHIP/PICKUP, pago Wompi/COD, mis pedidos, cancelar

**Archivos exclusivos (todos NUEVOS):**
```
docs-ruta/guias/
├── admin_client.md                        ← NUEVO
├── operator_client.md                     ← NUEVO
├── courier.md                             ← NUEVO
└── buyer.md                               ← NUEVO
```

**NO tocar:** ningún archivo de código; verificar consistencia contra `wireframes_mvp.md` y `all_ruta.md`

**Criterio:** consistencia con wireframes ✓ · PR `[6.QA-4] Documentación de usuario por rol`

---

### AGENTE O — 6.INFRA-4 + 6.QA-3 (Wave 6.3 — REQUIERE APROBACIÓN HUMANA EXPLÍCITA)

**Repo:** cross-repo + producción
**Rama:** `feat/infra-6-4`
**Tamaño:** M+M

> ⚠️ **ESTE AGENTE NO PUEDE ARRANCAR SIN CONFIRMACIÓN EXPLÍCITA DEL USUARIO.**
> Las acciones son irreversibles y afectan el entorno de producción.

**Qué hacer:**
- **6.INFRA-4 — Deploy producción:**
  1. Verificar que no hay migraciones de BD pendientes (schema aplicado en Sprint 0)
  2. Configurar DNS: `api.ruta.com` → Render Web Service, `app.ruta.com` → ruta-admin, `tienda.ruta.com` → ruta-storefront
  3. Activar env vars prod en Render: `WOMPI_PUBLIC_KEY`, `WOMPI_PRIVATE_KEY`, `WOMPI_WEBHOOK_SECRET`, `LOGTAIL_TOKEN`
  4. Verificar `/healthz` en todos los servicios en producción
  5. Smoke test básico post-deploy (login + catálogo + healthz)
- **6.QA-3 — Onboarding piloto (coordinado con humano):**
  1. ADMIN_RUTA crea Cliente piloto real
  2. Configura catálogo (categorías + productos), Wompi, repartidores, pickup points
  3. Primer pedido real de extremo a extremo verificado

---

---

## Tabla resumen de paralelismo

| Sprint | Wave | Agente | Tarea | Repos afectados | Estado |
|--------|------|--------|-------|-----------------|--------|
| S4 | 4.0 | ZERO | 4.FIX-1 Admin pickup-points CRUD | backend-ruta | ✅ COMPLETADO 2026-05-29 |
| S4 | 4.1 | A | 4.BACK-1 Flujo PICKUP backend | backend-ruta | ⬜ PENDIENTE |
| S4 | 4.1 | B | 4.ADMIN-1 UI punto físico | frontend-ruta/admin | ⬜ PENDIENTE |
| S4 | 4.2 | C | 4.QA-1 E2E PICKUP | frontend-ruta/e2e | ⬜ PENDIENTE |
| S5 | 5.1 | D | 5.BACK-1 Vista de Control | backend-ruta | ⬜ PENDIENTE |
| S5 | 5.1 | E | 5.BACK-2 Métricas | backend-ruta | ⬜ PENDIENTE |
| S5 | 5.1 | F | 5.BACK-3+4 Parámetros + Auditoría | backend-ruta | ⬜ PENDIENTE |
| S5 | 5.2 | G | 5.ADMIN-1 Vista de Control frontend | frontend-ruta/admin | ⬜ PENDIENTE |
| S5 | 5.2 | H | 5.ADMIN-2 Dashboards | frontend-ruta/admin | ⬜ PENDIENTE |
| S5 | 5.2 | I | 5.ADMIN-3+4 Config + Auditoría UI | frontend-ruta/admin | ⬜ PENDIENTE |
| S6 | 6.1 | J | 6.INFRA-1 Observabilidad | backend-ruta, infra-ruta | ⬜ PENDIENTE |
| S6 | 6.1 | K | 6.INFRA-3 Webhooks salientes | backend-ruta, admin | ⬜ PENDIENTE |
| S6 | 6.1 | L | 6.QA-1+2 Suite E2E + cobertura | frontend-ruta/e2e, backend-ruta | ⬜ PENDIENTE |
| S6 | 6.2 | M | 6.INFRA-2 Backups BD | infra-ruta | ⬜ PENDIENTE |
| S6 | 6.2 | N | 6.QA-4 Docs usuario | docs-ruta | ⬜ PENDIENTE |
| S6 | 6.3 | O | 6.INFRA-4 + 6.QA-3 Deploy + piloto | cross-repo + prod | ⬜ PENDIENTE |

**Leyenda:** ⬜ PENDIENTE · 🔄 EN CURSO · ✅ COMPLETADO · ❌ BLOQUEADO

**Máximo de agentes simultáneos:** 3 (Waves 5.1, 5.2 y 6.1)
**Total de agentes:** 15 (ZERO, A–N, O)

---

## Diagrama de dependencias entre waves

```
Sprint 3 cerrado ✅
       │
       ▼
[Wave 4.0] ZERO — 4.FIX-1 (BLOQUEANTE)
       │
       ├─────────────────────┐
       ▼                     ▼
[Wave 4.1] A — 4.BACK-1   B — 4.ADMIN-1  (paralelo)
       │
       ▼
[Wave 4.2] C — 4.QA-1
       │
Sprint 4 cerrado
       │
       ├──────────────────────┬──────────────────────┐
       ▼                      ▼                      ▼
[Wave 5.1] D — 5.BACK-1   E — 5.BACK-2   F — 5.BACK-3+4  (paralelo)
       │
       ├──────────────────────┬──────────────────────┐
       ▼                      ▼                      ▼
[Wave 5.2] G — 5.ADMIN-1  H — 5.ADMIN-2  I — 5.ADMIN-3+4  (paralelo)
       │
Sprint 5 cerrado
       │
       ├──────────────────────┬──────────────────────┐
       ▼                      ▼                      ▼
[Wave 6.1] J — 6.INFRA-1  K — 6.INFRA-3  L — 6.QA-1+2    (paralelo)
[Wave 6.2] M — 6.INFRA-2  N — 6.QA-4                     (paralelo, puede correr con 6.1)
       │
       ▼
[Wave 6.3] O — 6.INFRA-4 + 6.QA-3  ← REQUIERE APROBACIÓN HUMANA
```

---

## Registro de avance por sprint

### Sprint 4

| Fecha | Evento |
|-------|--------|
| — | Wave 4.0 arranca |
| — | 4.FIX-1 mergeado — Wave 4.1 desbloqueada |
| — | 4.BACK-1 mergeado |
| — | 4.ADMIN-1 mergeado |
| — | Wave 4.2 arranca |
| — | 4.QA-1 mergeado — **Sprint 4 CERRADO** |

### Sprint 5

| Fecha | Evento |
|-------|--------|
| — | Wave 5.1 arranca (3 agentes) |
| — | 5.BACK-1 mergeado |
| — | 5.BACK-2 mergeado |
| — | 5.BACK-3+4 mergeado — Wave 5.2 desbloqueada |
| — | Wave 5.2 arranca (3 agentes) |
| — | 5.ADMIN-1 mergeado |
| — | 5.ADMIN-2 mergeado |
| — | 5.ADMIN-3+4 mergeado — **Sprint 5 CERRADO** |

### Sprint 6

| Fecha | Evento |
|-------|--------|
| — | Wave 6.1 + 6.2 arrancan (5 agentes) |
| — | 6.INFRA-1 mergeado |
| — | 6.INFRA-3 mergeado |
| — | 6.QA-1+2 mergeado |
| — | 6.INFRA-2 mergeado |
| — | 6.QA-4 mergeado |
| — | ✋ APROBACIÓN HUMANA para Wave 6.3 |
| — | 6.INFRA-4 — Deploy prod completado |
| — | 6.QA-3 — Primer pedido real ✓ — **Sprint 6 CERRADO** |
