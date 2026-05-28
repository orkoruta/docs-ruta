# Plan de Paralelismo — Sprints 2 a 6

Documento autoritativo para coordinar agentes IA trabajando en paralelo.
Cada agente debe leer este archivo completo antes de empezar su tarea.

**Fuente de verdad de tareas:** `docs-ruta/plan_tareas.md`
**Plan Sprint 1:** `docs-ruta/plan_paralelismo.md`
**Estado de entrada:** Sprint 0 ✅ cerrado. Sprint 1 ✅ CERRADO 2026-05-28 (todos los PRs mergeados: BACK-5 #3, ADMIN-2 #7, ADMIN-3 #6, ADMIN-4 #8).
CI de `backend-ruta/main` y `frontend-ruta/main` verde.

---

## Reglas de coordinación (OBLIGATORIO leer antes de empezar)

1. **Cada agente tiene jurisdicción exclusiva** sobre los archivos listados en su sección.
2. **Nunca modificar archivos de otro agente.** Si necesitas algo de otro agente, impórtalo, no lo edites.
3. **`app.ts` del backend es el único archivo compartido permitido:** cada agente solo agrega su bloque de rutas/jobs con comentario `// N.BACK-N`. Nada más.
4. **Branch propia por tarea:** `feat/back-2-1`, `feat/store-2-2`, etc.
5. **No crear `index.ts` barrels compartidos** — pueden causar conflictos de merge.
6. **Una Wave no arranca hasta que la anterior esté mergeada a `main`** (o la dependencia específica indicada).
7. **Definition of Done:** typecheck limpio + tests pasando + lint OK + build OK antes de hacer PR.
8. **SHARED siempre primero:** cuando un sprint requiere bump de `@orkoruta/shared`, ese agente es la primera wave completa, bloqueante.

---

## Protocolo de agente especializado (TODOS los agentes, OBLIGATORIO antes de codificar)

Antes de realizar cualquier tarea, revisa obligatoriamente toda la documentación relacionada.

**No puedes inventar** funcionalidades, reglas, nombres, entidades, rutas, pantallas, roles, permisos, validaciones, estados, procesos, textos de interfaz ni comportamientos que no estén definidos en la documentación existente.

### Paso 1 — Análisis previo (entregar ANTES de escribir código)

**1. Documentación revisada:** lista de `.md` y `.txt` revisados.
**2. Reglas detectadas:** reglas de negocio aplicables, restricciones técnicas, decisiones ya definidas.
**3. Flujos obligatorios:** flujos que gobiernan esta tarea y el orden de pasos.
**4. Vacíos o contradicciones:** puntos no documentados o contradicciones entre documentos. Si falta info, no inventes.
**5. Plan de ejecución:** cómo vas a resolver la tarea respetando la documentación.

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
[ ] POST/PUT/PATCH/DELETE verifican X-Idempotency-Key
[ ] Endpoints protegidos verifican auth (requireAuth / requireAdminClient / requireAdminRuta)
[ ] Queries a BD usan withTenant() — nunca queries globales sin contexto de tenant
[ ] Tests cubren: caso feliz + error de validación + autenticación fallida
```

**Frontend además:**
```
[ ] Build sin warnings de Next.js
[ ] Formularios muestran errores de validación visibles al usuario
[ ] Llamadas a API manejan 401, 403, 500 con feedback al usuario
[ ] No hay imports de @orkoruta/ui desde storefront o landing (solo admin)
[ ] No hay tokens en localStorage — solo cookies HttpOnly
[ ] Páginas protegidas redirigen a /login si no hay sesión
```

---

## Registro de avance obligatorio (TODOS los agentes, sin excepción)

Al terminar — antes de abrir el PR — actualizar:

1. `docs-ruta/plan_tareas.md` → cambiar `[ ]` a `[x]` en la tarea completada.
2. `docs-ruta/memoria_proyecto_ruta.md` → agregar entrada con: tarea, archivos nuevos, tests, decisiones técnicas.
3. Archivo de memoria del sprint en `/home/alexander_marquez/.claude/projects/-mnt-c-Dropbox-Alex-DEV-ruta/memory/project_sprint[N]_status.md` → mover tarea de PENDIENTE a HECHO.
4. `MEMORY.md` → actualizar línea del sprint si cambió significativamente.

**Orden:** código → plan_tareas → memoria_proyecto → memory/sprint → PR.

---

## Documentación mínima a revisar por track

| Track | Documentos obligatorios |
|-------|------------------------|
| SHARED | `CLAUDE.md`, `all_ruta.md`, flujos afectados |
| Backend | `CLAUDE.md`, `contrato_api.md`, `all_ruta.md`, `matriz_permisos.md`, `parametros_negocio.md`, `estrategia_testing.md`, `bd/ruta_postgres.sql`, flujos relevantes en `flujos/` |
| Admin frontend | `CLAUDE.md`, `wireframes_mvp.md`, `diseno/galeria_estilos_ruta.md`, `matriz_permisos.md`, `contrato_api.md`, `all_ruta.md` |
| Storefront | `CLAUDE.md`, `wireframes_mvp.md`, `diseno/galeria_estilos_ruta.md`, `contrato_api.md`, `all_ruta.md`, `flujos/flujo_1.txt` |
| QA / E2E | `CLAUDE.md`, `estrategia_testing.md`, `arquitectura/estrategia_multi_tenant_ruta.md`, `matriz_permisos.md`, `contrato_api.md` |
| INFRA | `CLAUDE.md`, `all_ruta.md`, docs relevantes de `arquitectura/` |

---

## Contexto técnico para todos los agentes

- **BD:** PostgreSQL OCI `149.130.168.24:26432/rutadb` (no Supabase)
- **Backend completo al iniciar Sprint 2:** Auth + Clientes + Productos + Buyers + Couriers + Bulk Import
- **API base URL dev:** `http://localhost:3001`
- **Admin:** `:3002` · **Storefront:** `:3003`
- **Paquetes:** `@orkoruta/shared@1.1.0` y `@orkoruta/db@1.0.0` en GitHub Packages
- **Design system:** `Ruta*` solo en `frontend-ruta/packages/ui/` — admin lo usa, storefront NO importa de ui, landings jamás
- **Auth cookies:** HttpOnly, Secure, SameSite=Strict — no localStorage
- **Multi-tenant:** toda query backend usa `withTenant(clientId, role, fn)`
- **Idempotencia:** todo POST/PUT/PATCH/DELETE requiere `X-Idempotency-Key`
- **Tests backend:** `export NPM_TOKEN=$(grep "_authToken=" ~/.npmrc | cut -d= -f2)` antes de `pnpm test`
- **CI/Render:** `pnpm@11.4.0`; `minimumReleaseAgeExclude: ['@orkoruta/*']` en `pnpm-workspace.yaml`

---

---

# SPRINT 1 — ✅ CERRADO 2026-05-28

Todas las tareas del Sprint 1 completadas y mergeadas a `main`.
Ver detalles en `memory/project_sprint1_status.md`.

---

---

# SPRINT 2 — Carrito, checkout y Flujo 1 completo

**Dependencia de entrada:** Sprint 1 completamente cerrado (incluido 1.ADMIN-4).
**Duración:** 2 semanas.

## Diagrama de waves Sprint 2

```
Wave 2.0 — 1 agente (BLOQUEANTE — publicar antes de continuar):
  Agente A → 2.SHARED-1 ─── publica @orkoruta/shared@1.2.0

Wave 2.1 — 3 agentes en paralelo (tras publish 2.SHARED-1):
  Agente B → 2.BACK-1 (XL) ─── State Machine + Orders service  ← BLOQUEANTE Wave 2.2+
  Agente C → 2.BACK-3 (XL) ─── Wompi + payments + webhook      ← BLOQUEANTE Wave 2.4
  Agente D → 2.BACK-4 (M)  ─── Jobs de mantenimiento

Wave 2.2 — 3 agentes en paralelo (tras merge 2.BACK-1):
  Agente E → 2.BACK-2 (L)  ─── Validación operativa y aceptación
  Agente F → 2.STORE-1 (M) ─── Carrito /c/[slug]/cart
  Agente G → 2.ADMIN-1 (L) ─── Lista y detalle de pedidos admin

Wave 2.3 — 2 agentes en paralelo (tras merge 2.BACK-1 + 2.STORE-1):
  Agente H → 2.STORE-2 (XL) ── Checkout multi-paso
  Agente I → 2.STORE-3 (L)  ── Mis pedidos BUYER

Wave 2.4 — 2 agentes en paralelo (tras merge 2.BACK-3 + 2.STORE-2):
  Agente J → 2.STORE-4 (M) ─── Confirmación post-Wompi
  Agente K → 2.QA-1 (M)   ─── Tests state machine + MSW Wompi
```

---

### AGENTE A — 2.SHARED-1 (Wave 2.0 — SOLO, BLOQUEANTE)

**Repo:** `packages-ruta/`
**Rama:** `feat/shared-2`
**Tamaño:** M

**Qué hacer:**
- `shared/src/validators/order.schema.ts` — create, confirm, cancel, requestCancel, transition
- `shared/src/validators/payment.schema.ts` — initiate, webhookEvent, paymentStatus
- `shared/src/types/order.types.ts` — tipos derivados
- Bump `@orkoruta/shared@1.2.0` → publicar en GitHub Packages

**Archivos exclusivos:**
```
packages-ruta/shared/src/
├── validators/
│   ├── order.schema.ts        ← NUEVO
│   └── payment.schema.ts      ← NUEVO
├── types/
│   └── order.types.ts         ← NUEVO o actualizado
└── package.json               ← bump version
```

**Criterio:** `@orkoruta/shared@1.2.0` visible en GitHub Packages e instalable → Wave 2.1 puede arrancar.

---

### AGENTE B — 2.BACK-1 (Wave 2.1 — BLOQUEANTE principal)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-2-1`
**Tamaño:** XL

**Qué hacer:**
- `src/services/orders/state_machine.ts` — todas las transiciones de Flujos 1, 2, 3
- `src/services/orders/orders.service.ts` — create, confirm, cancel, requestCancel
- `src/routes/buyer_orders.ts` — `/buyer/orders/*`
- Tests unitarios: cada transición permitida y rechazada

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   ├── state_machine.ts       ← NUEVO
│   └── orders.service.ts      ← NUEVO
├── routes/
│   └── buyer_orders.ts        ← NUEVO
└── __tests__/
    └── state_machine.test.ts  ← NUEVO
```

**Archivo compartido:** `src/app.ts` — solo agregar mount de `buyer_orders` con comentario `// 2.BACK-1`.

**Documentos obligatorios:** `flujos/flujo_1.txt`, `flujos/flujo_2.txt`, `flujos/flujo_3.txt`, `contrato_api.md`, `parametros_negocio.md`, `matriz_permisos.md`

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.BACK-1] Orders + State Machine`

---

### AGENTE C — 2.BACK-3 (Wave 2.1 — paralelo con B)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-2-3`
**Tamaño:** XL

**Qué hacer:**
- `src/lib/wompi_client.ts` — cliente Wompi (configurable sandbox/prod vía env)
- `src/services/payments.service.ts` — `initiatePayment(orderId)`
- `src/routes/buyer_payment.ts` — `POST /buyer/orders/:id/initiate-payment`
- `src/routes/webhooks.ts` — webhook entrante con verificación HMAC, deduplicación en `external_webhook_events` (append-only)

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── lib/
│   └── wompi_client.ts        ← NUEVO
├── services/
│   └── payments.service.ts    ← NUEVO
└── routes/
    ├── buyer_payment.ts       ← NUEVO
    └── webhooks.ts            ← NUEVO
```

**Archivo compartido:** `src/app.ts` — solo agregar mounts con comentario `// 2.BACK-3`.

**Documentos obligatorios:** `contrato_api.md` (sección Wompi), `flujos/flujo_1.txt`, `all_ruta.md` (sección pagos), `parametros_negocio.md`

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.BACK-3] Wompi + payments + webhook`

---

### AGENTE D — 2.BACK-4 (Wave 2.1 — paralelo con B y C)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-2-4`
**Tamaño:** M

**Qué hacer:**
- `src/jobs/order_expiration.job.ts` — transiciona pedidos expirados según `client_parameters`
- `src/jobs/payment_timeout.job.ts` — timeout de pagos pendientes
- `src/jobs/cleanup_idempotency.job.ts` — limpieza de idempotency_keys TTL>24h
- `src/jobs/cleanup_sessions.job.ts` — limpieza de sesiones expiradas

**Archivos exclusivos:**
```
backend-ruta/api/src/jobs/
├── order_expiration.job.ts    ← NUEVO
├── payment_timeout.job.ts     ← NUEVO
├── cleanup_idempotency.job.ts ← NUEVO
└── cleanup_sessions.job.ts    ← NUEVO
```

**Archivo compartido:** `src/app.ts` — solo agregar inicialización de jobs con comentario `// 2.BACK-4`.

**Documentos obligatorios:** `parametros_negocio.md`, `flujos/flujo_1.txt`

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.BACK-4] Jobs de mantenimiento`

---

### AGENTE E — 2.BACK-2 (Wave 2.2 — tras merge 2.BACK-1)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-2-2`
**Tamaño:** L

**Qué hacer:**
- `src/services/orders/validation.service.ts` — job `validate_order` → VALIDATION_APPROVED o MANUAL_REVIEW
- `src/jobs/validate_order.job.ts`
- `src/routes/admin_orders.ts` — endpoints admin: accept, reject, mark-preparing, mark-ready

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   └── validation.service.ts  ← NUEVO
├── jobs/
│   └── validate_order.job.ts  ← NUEVO
└── routes/
    └── admin_orders.ts        ← NUEVO
```

**Solo lectura:** `services/orders/state_machine.ts` (mergeado de BACK-1).
**Archivo compartido:** `src/app.ts` — solo agregar mount con comentario `// 2.BACK-2`.

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.BACK-2] Validación operativa y aceptación`

---

### AGENTE F — 2.STORE-1 (Wave 2.2 — paralelo con E y G)

**Repo:** `frontend-ruta/` → `storefront/`
**Rama:** `feat/store-2-1`
**Tamaño:** M

**Qué hacer:**
- Carrito representado como pedido DRAFT en BD
- Agregar, actualizar cantidad y remover items vía endpoints del pedido
- Página `/c/[slug]/cart`

**Archivos exclusivos:**
```
frontend-ruta/storefront/
├── app/c/[slug]/cart/
│   ├── page.tsx               ← NUEVO (shell server)
│   └── _components/
│       └── CartView.tsx       ← NUEVO ('use client')
└── lib/
    └── cart.api.ts            ← NUEVO
```

**Solo lectura:** `lib/catalog.api.ts`, `lib/auth.api.ts`, `app/c/[slug]/layout.tsx`
**NO tocar:** `app/c/[slug]/checkout/` (zona de STORE-2), `app/c/[slug]/orders/` (zona de STORE-3)

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.STORE-1] Carrito persistido en BD`

---

### AGENTE G — 2.ADMIN-1 (Wave 2.2 — paralelo con E y F)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-2-1`
**Tamaño:** L

**Qué hacer:**
- `/admin/orders` — lista con filtros (estado, fecha, courier)
- `/admin/orders/[id]` — detalle 360 del pedido (historial de estados, buyer, courier, items)

**Archivos exclusivos:**
```
frontend-ruta/admin/
├── app/(protected)/admin/orders/
│   ├── page.tsx               ← NUEVO (lista + filtros)
│   └── [id]/
│       └── page.tsx           ← NUEVO (detalle 360)
└── lib/
    └── orders.api.ts          ← NUEVO
```

**Solo lectura:** `components/RutaSidebar.tsx`, `components/RutaHeader.tsx`, `app/(protected)/layout.tsx`

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.ADMIN-1] Lista y detalle de pedidos admin`

---

### AGENTE H — 2.STORE-2 (Wave 2.3 — tras merge 2.BACK-1 + 2.STORE-1)

**Repo:** `frontend-ruta/` → `storefront/`
**Rama:** `feat/store-2-2`
**Tamaño:** XL

**Qué hacer:**
- Paso 1: tipo de entrega (SHIP / PICKUP)
- Paso 2: dirección o pickup point (mapa OSM + Leaflet)
- Paso 3: método de pago (Wompi / COD)
- Submit: confirma DRAFT → redirige a Wompi o confirmación COD

**Archivos exclusivos:**
```
frontend-ruta/storefront/
└── app/c/[slug]/checkout/
    ├── page.tsx               ← NUEVO (shell server)
    └── _components/
        ├── CheckoutStepper.tsx   ← NUEVO
        ├── DeliveryStep.tsx      ← NUEVO
        ├── AddressStep.tsx       ← NUEVO (OSM/Leaflet)
        └── PaymentStep.tsx       ← NUEVO
```

**Solo lectura:** `lib/cart.api.ts` (STORE-1).
**NO tocar:** `checkout/confirmation/` (zona de STORE-4).

**Documentos obligatorios:** `wireframes_mvp.md`, `flujos/flujo_1.txt`, `contrato_api.md`

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.STORE-2] Checkout multi-paso`

---

### AGENTE I — 2.STORE-3 (Wave 2.3 — paralelo con H)

**Repo:** `frontend-ruta/` → `storefront/`
**Rama:** `feat/store-2-3`
**Tamaño:** L

**Qué hacer:**
- `/c/[slug]/orders` — lista de mis pedidos con estado visual
- `/c/[slug]/orders/[id]` — detalle con `RutaTimeline` de historial y acciones condicionales

**Archivos exclusivos:**
```
frontend-ruta/storefront/
├── app/c/[slug]/orders/
│   ├── page.tsx               ← NUEVO
│   ├── _components/
│   │   └── OrdersView.tsx     ← NUEVO
│   └── [id]/
│       ├── page.tsx           ← NUEVO
│       └── _components/
│           └── OrderDetailView.tsx ← NUEVO
└── lib/
    └── buyer_orders.api.ts    ← NUEVO (distinto de orders.api.ts del admin)
```

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.STORE-3] Mis pedidos BUYER`

---

### AGENTE J — 2.STORE-4 (Wave 2.4 — tras merge 2.BACK-3 + 2.STORE-2)

**Repo:** `frontend-ruta/` → `storefront/`
**Rama:** `feat/store-2-4`
**Tamaño:** M

**Qué hacer:**
- `/c/[slug]/checkout/confirmation` — polling del estado del pago tras redirect de Wompi
- Leer `?transaction_id=` de la URL
- Estados: procesando / confirmado / fallido con feedback al usuario

**Archivos exclusivos:**
```
frontend-ruta/storefront/
└── app/c/[slug]/checkout/confirmation/
    ├── page.tsx               ← NUEVO (shell)
    └── _components/
        └── ConfirmationView.tsx ← NUEVO ('use client', polling)
```

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.STORE-4] Confirmación post-Wompi`

---

### AGENTE K — 2.QA-1 (Wave 2.4 — paralelo con J)

**Repo:** `backend-ruta/`
**Rama:** `feat/qa-2-1`
**Tamaño:** M

**Qué hacer:**
- MSW para mockear Wompi sandbox en tests
- Tests del state machine: cada transición permitida y rechazada
- Test del webhook entrante con firma HMAC válida e inválida
- Tests de deduplicación de eventos externos

**Archivos exclusivos:**
```
backend-ruta/api/src/__tests__/
├── payments.test.ts           ← NUEVO
├── wompi_webhook.test.ts      ← NUEVO
└── __mocks__/
    └── wompi_server.ts        ← NUEVO (MSW handler)
```

**Solo lectura:** todos los archivos de producción del sprint.

**Criterio:** checklist calidad · plan_tareas.md `[x]` · PR `[2.QA-1] Tests state machine y Wompi`

---

---

# SPRINT 3 — Flujo SHIP completo

**Dependencia de entrada:** Sprint 2 completamente mergeado.
**Duración:** 2 semanas.

## Diagrama de waves Sprint 3

```
Wave 3.1 — 4 agentes en paralelo (ramas separadas de backend-ruta):
  Agente A → 3.BACK-1 (L)     ── Asignación de Repartidor
  Agente B → 3.BACK-2 (L)     ── Endpoints COURIER (sus pedidos)
  Agente C → 3.BACK-3 (L)     ── Cobro contra entrega (multipart)
  Agente D → 3.BACK-4+5+6 (M) ── Cancelación + RETURN_TO_ORIGIN + auto-confirmación

Wave 3.2 — 2 agentes en paralelo (tras merge Wave 3.1):
  Agente E → 3.ADMIN-1 (XL)   ── Mapa asignación (Leaflet + OSM)
  Agente F → 3.ADMIN-2 (XL)   ── Vista COURIER móvil-first

Wave 3.3 — 1 agente (tras merge Wave 3.2):
  Agente G → 3.QA-1 (L)       ── Playwright E2E flujo SHIP completo
```

---

### AGENTE A — 3.BACK-1 (Wave 3.1)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-3-1`
**Tamaño:** L

**Qué hacer:**
- `src/services/orders/assignment.service.ts` — `assignCourier()` con optimistic lock (verifica `lock_version`)
- `src/routes/admin_assignment.ts` — endpoints admin para asignar/reasignar repartidor

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   └── assignment.service.ts  ← NUEVO
└── routes/
    └── admin_assignment.ts    ← NUEVO
```

**Solo lectura:** `services/orders/state_machine.ts`, `services/orders/orders.service.ts`
**Archivo compartido:** `src/app.ts` — comentario `// 3.BACK-1`

---

### AGENTE B — 3.BACK-2 (Wave 3.1 — paralelo con A)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-3-2`
**Tamaño:** L

**Qué hacer:**
- `src/services/orders/courier_ops.service.ts` — transiciones del courier (pickup → en camino → entregado → colección fallida)
- `src/routes/courier_orders.ts` — `/courier/orders/*` — courier solo ve sus pedidos asignados

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   └── courier_ops.service.ts ← NUEVO
└── routes/
    └── courier_orders.ts      ← NUEVO
```

**Solo lectura:** `services/orders/state_machine.ts`
**Archivo compartido:** `src/app.ts` — comentario `// 3.BACK-2`

**Documentos obligatorios:** `flujos/flujo_1.txt` (sección COURIER), `matriz_permisos.md`

---

### AGENTE C — 3.BACK-3 (Wave 3.1 — paralelo con A y B)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-3-3`
**Tamaño:** L

**Qué hacer:**
- `src/services/orders/collection.service.ts` — `recordCollection()` con upload de evidencia fotográfica
- `src/routes/courier_collection.ts` — `POST /courier/orders/:id/collection` (multipart)
- Integración con file storage (stub 501 si aún no definido, igual que BACK-3 del Sprint 1)

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   └── collection.service.ts  ← NUEVO
└── routes/
    └── courier_collection.ts  ← NUEVO
```

**Archivo compartido:** `src/app.ts` — comentario `// 3.BACK-3`

---

### AGENTE D — 3.BACK-4+5+6 (Wave 3.1 — paralelo con A, B y C)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-3-456`
**Tamaño:** M+M+M

**Qué hacer:**
- **3.BACK-4:** `src/services/orders/cancellation.service.ts` — CUSTOMER_CANCEL_REQUEST + revisión admin
- **3.BACK-5:** nuevas transiciones RETURN_TO_ORIGIN en state machine
- **3.BACK-6:** `src/jobs/auto_confirm.job.ts` — DELIVERED → CONFIRMED_BY_SYSTEM tras timeout

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   └── cancellation.service.ts ← NUEVO
└── jobs/
    └── auto_confirm.job.ts     ← NUEVO
```

**Archivo compartido con atención:** `src/services/orders/state_machine.ts` — solo agregar transiciones de RETURN_TO_ORIGIN. Resolver conflicto con BACK-1 al mergear.
**Archivo compartido:** `src/app.ts` — comentario `// 3.BACK-456`

---

### AGENTE E — 3.ADMIN-1 (Wave 3.2 — tras merge Wave 3.1)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-3-1`
**Tamaño:** XL

**Qué hacer:**
- `/admin/map` — mapa interactivo Leaflet + OSM
- Panel lateral: pedidos listos para despacho + couriers disponibles con posición
- Asignación de pedido a courier con confirmación modal
- Auto-refresh configurable desde `client_parameters`

**Archivos exclusivos:**
```
frontend-ruta/admin/
├── app/(protected)/admin/map/
│   ├── page.tsx                    ← NUEVO
│   └── _components/
│       ├── AssignmentMap.tsx        ← NUEVO ('use client', Leaflet)
│       ├── PendingOrdersPanel.tsx   ← NUEVO
│       └── CourierMarker.tsx        ← NUEVO
└── lib/
    └── assignment.api.ts            ← NUEVO
```

**NO tocar:** archivos de Agente F (courier/).

---

### AGENTE F — 3.ADMIN-2 (Wave 3.2 — paralelo con E)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-3-2`
**Tamaño:** XL

**Qué hacer:**
- `/courier` — tabs: Mis Pedidos Activos / Historial
- Detalle de pedido con botones grandes (táctil, móvil-first `h-16`)
- Formulario de cobro contra entrega con captura de foto (evidencia)
- Layout max-w-sm optimizado para teléfono

**Archivos exclusivos:**
```
frontend-ruta/admin/
├── app/(protected)/courier/
│   ├── page.tsx                       ← NUEVO
│   └── _components/
│       ├── CourierDashboard.tsx        ← NUEVO
│       ├── ActiveOrderCard.tsx         ← NUEVO
│       └── CollectionForm.tsx          ← NUEVO (foto + monto)
└── lib/
    └── courier_ops.api.ts              ← NUEVO
```

**NO tocar:** archivos de Agente E (map/).

---

### AGENTE G — 3.QA-1 (Wave 3.3 — tras merge Wave 3.2)

**Repo:** `frontend-ruta/`
**Rama:** `feat/qa-3-1`
**Tamaño:** L

**Qué hacer:**
- Playwright spec E2E flujo SHIP completo: registro buyer → pedido → validación → asignación → courier en camino → entregado → confirmado
- Fixtures de datos de prueba reutilizables

**Archivos exclusivos:**
```
frontend-ruta/e2e/
├── ship_flow.spec.ts          ← NUEVO
└── fixtures/
    └── ship_flow.fixture.ts   ← NUEVO
```

---

---

# SPRINT 4 — Flujo PICKUP

**Dependencia de entrada:** Sprint 3 completamente mergeado.
**Duración:** 1 semana.

## Diagrama de waves Sprint 4

```
Wave 4.1 — 2 agentes en paralelo:
  Agente A → 4.BACK-1 (L)   ── Endpoints PICKUP + job expiración
  Agente B → 4.ADMIN-1 (M)  ── UI operación punto físico

Wave 4.2 — 1 agente (tras merge Wave 4.1):
  Agente C → 4.QA-1 (M)     ── E2E flujo PICKUP completo
```

---

### AGENTE A — 4.BACK-1 (Wave 4.1)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-4-1`
**Tamaño:** L

**Qué hacer:**
- Transiciones específicas de PICKUP en state machine
- Validación de identidad del comprador en punto (documento)
- Cobro en punto (COD en pickup_point)
- `src/jobs/pickup_expiration.job.ts`

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/orders/
│   └── pickup_ops.service.ts   ← NUEVO
├── routes/
│   └── pickup_point_ops.ts     ← NUEVO
└── jobs/
    └── pickup_expiration.job.ts ← NUEVO
```

**Archivo compartido con atención:** `src/services/orders/state_machine.ts` — agregar transiciones PICKUP.
**Archivo compartido:** `src/app.ts` — comentario `// 4.BACK-1`

---

### AGENTE B — 4.ADMIN-1 (Wave 4.1 — paralelo con A)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-4-1`
**Tamaño:** M

**Qué hacer:**
- Componente de acciones PICKUP en la vista de detalle de pedido
- Botones: "Verificar identidad", "Registrar cobro en punto", "Marcar entregado"

**Archivos exclusivos:**
```
frontend-ruta/admin/
└── app/(protected)/admin/orders/[id]/_components/
    └── PickupActions.tsx       ← NUEVO
```

**Solo lectura:** `admin/orders/[id]/page.tsx` (existente de Sprint 2).

---

### AGENTE C — 4.QA-1 (Wave 4.2)

**Repo:** `frontend-ruta/`
**Rama:** `feat/qa-4-1`
**Tamaño:** M

**Archivos exclusivos:**
```
frontend-ruta/e2e/
├── pickup_flow.spec.ts         ← NUEVO
└── fixtures/
    └── pickup_flow.fixture.ts  ← NUEVO
```

---

---

# SPRINT 5 — Vista de Control, dashboards, configuración

**Dependencia de entrada:** Sprint 4 completamente mergeado.
**Duración:** 1 semana.

## Diagrama de waves Sprint 5

```
Wave 5.1 — 3 agentes en paralelo (backend):
  Agente A → 5.BACK-1 (L)      ── Vista de Control (enter/exit + auditoría)
  Agente B → 5.BACK-2 (L)      ── Dashboards (métricas Cliente y globales)
  Agente C → 5.BACK-3+4 (M+M) ── Parámetros + Auditoría endpoints

Wave 5.2 — 3 agentes en paralelo (frontend, tras merge Wave 5.1):
  Agente D → 5.ADMIN-1 (M)     ── Pantalla Vista de Control
  Agente E → 5.ADMIN-2 (M)     ── Dashboards admin
  Agente F → 5.ADMIN-3+4 (L+M) ── Configuración tabs + Auditoría UI
```

---

### AGENTE A — 5.BACK-1 (Wave 5.1)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5-1`
**Tamaño:** L

**Qué hacer:**
- `src/services/control_view.service.ts` — `enterControlView` (verifica master password argon2, emite token `impersonating=true`), `exitControlView` (revoca sesión)
- `src/middleware/control_view.ts` — `requireControlView` para rutas de impersonación
- `src/routes/ruta_admin_control_view.ts`
- Toda acción en Vista de Control registra `acting_via_control_view=TRUE` e `impersonator_user_id` en audit_events

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/
│   └── control_view.service.ts    ← NUEVO
├── middleware/
│   └── control_view.ts            ← NUEVO
└── routes/
    └── ruta_admin_control_view.ts ← NUEVO
```

**Archivo compartido:** `src/app.ts` — comentario `// 5.BACK-1`

**Documentos obligatorios:** `seguridad/ciclo_vida_token.txt`, `all_ruta.md` (Vista de Control), `matriz_permisos.md`

---

### AGENTE B — 5.BACK-2 (Wave 5.1 — paralelo con A)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5-2`
**Tamaño:** L

**Qué hacer:**
- `src/services/metrics.service.ts` — métricas del Cliente (pedidos por estado, ingresos, couriers activos)
- `src/services/global_metrics.service.ts` — métricas globales ADMIN_RUTA
- `src/routes/admin_metrics.ts` — `/admin/metrics` (ADMIN_CLIENT)
- `src/routes/ruta_admin_metrics.ts` — `/ruta-admin/metrics` (ADMIN_RUTA)

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/
│   ├── metrics.service.ts         ← NUEVO
│   └── global_metrics.service.ts  ← NUEVO
└── routes/
    ├── admin_metrics.ts           ← NUEVO
    └── ruta_admin_metrics.ts      ← NUEVO
```

**Archivo compartido:** `src/app.ts` — comentario `// 5.BACK-2`

---

### AGENTE C — 5.BACK-3+4 (Wave 5.1 — paralelo con A y B)

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5-34`
**Tamaño:** M+M

**Qué hacer:**
- **5.BACK-3:** `src/routes/admin_parameters.ts` — `GET /admin/parameters`, `PATCH /admin/parameters` (lee y actualiza `client_parameters` con `requireAdminClient`)
- **5.BACK-4:** `src/routes/admin_audit.ts` — `GET /admin/audit-events` con filtros amplios; `GET /ruta-admin/audit-events` para ADMIN_RUTA

**Archivos exclusivos:**
```
backend-ruta/api/src/routes/
├── admin_parameters.ts        ← NUEVO
└── admin_audit.ts             ← NUEVO
```

**Archivo compartido:** `src/app.ts` — comentario `// 5.BACK-34`

---

### AGENTE D — 5.ADMIN-1 (Wave 5.2 — paralelo con E y F)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-5-1`
**Tamaño:** M

**Qué hacer:**
- `/ruta-admin/control-view` — formulario de acceso con master password
- Al autenticar, activar banner ámbar de `RutaHeader` (`impersonating=true` en sesión)
- Botón "Salir de Vista de Control" en header cuando activo

**Archivos exclusivos:**
```
frontend-ruta/admin/
├── app/(protected)/ruta-admin/control-view/
│   └── page.tsx                   ← NUEVO
└── lib/
    └── control_view.api.ts         ← NUEVO
```

**Solo lectura:** `components/RutaHeader.tsx` (banner ámbar ya existe — activarlo lógicamente).

---

### AGENTE E — 5.ADMIN-2 (Wave 5.2 — paralelo con D y F)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-5-2`
**Tamaño:** M

**Qué hacer:**
- `/admin/dashboard` — métricas del Cliente (cards + gráficas simples)
- `/ruta-admin/dashboard` — métricas globales ADMIN_RUTA

**Archivos exclusivos:**
```
frontend-ruta/admin/
├── app/(protected)/admin/dashboard/
│   └── page.tsx                   ← NUEVO
├── app/(protected)/ruta-admin/dashboard/
│   └── page.tsx                   ← NUEVO
└── lib/
    └── metrics.api.ts              ← NUEVO
```

---

### AGENTE F — 5.ADMIN-3+4 (Wave 5.2 — paralelo con D y E)

**Repo:** `frontend-ruta/` → `admin/`
**Rama:** `feat/admin-5-34`
**Tamaño:** L+M

**Qué hacer:**
- **5.ADMIN-3:** `/admin/settings` con tabs: Información / Pagos Wompi / Webhooks / Parámetros operativos
- **5.ADMIN-4:** `/admin/audit` — tabla paginada de eventos de auditoría con filtros (fecha, tipo, usuario)

**Archivos exclusivos:**
```
frontend-ruta/admin/
├── app/(protected)/admin/
│   ├── settings/
│   │   ├── page.tsx               ← NUEVO
│   │   └── _components/
│   │       ├── BusinessInfoTab.tsx   ← NUEVO
│   │       ├── WompiTab.tsx          ← NUEVO
│   │       ├── WebhooksTab.tsx       ← NUEVO
│   │       └── ParametersTab.tsx     ← NUEVO
│   └── audit/
│       └── page.tsx               ← NUEVO
└── lib/
    ├── parameters.api.ts          ← NUEVO
    └── audit.api.ts               ← NUEVO
```

---

---

# SPRINT 6 — Hardening, observabilidad, deploy, piloto

**Dependencia de entrada:** Sprint 5 completamente mergeado.
**Duración:** 1 semana.

## Diagrama de waves Sprint 6

```
Wave 6.1 — 3 agentes en paralelo:
  Agente A → 6.INFRA-1 (L)    ── Observabilidad (logs JSON, Logtail/Axiom, alertas)
  Agente B → 6.INFRA-3 (L)    ── Webhooks salientes (pg-boss, reintentos, UI historial)
  Agente C → 6.QA-1+2 (L+M)  ── Suite E2E completa + cobertura >85%

Wave 6.2 — 2 agentes en paralelo (puede correr en paralelo con Wave 6.1):
  Agente D → 6.INFRA-2 (M)    ── Backups y restore probado
  Agente E → 6.QA-4 (M)       ── Documentación de usuario

Wave 6.3 — coordinada con humano (ÚLTIMO paso):
  Agente F → 6.INFRA-4 (M)    ── Deploy a producción
               + 6.QA-3 (M)   ── Onboarding Cliente piloto (humano + agente)
```

---

### AGENTE A — 6.INFRA-1 (Wave 6.1)

**Repo:** `backend-ruta/` + `infra-ruta/`
**Rama:** `feat/infra-6-1`
**Tamaño:** L

**Qué hacer:**
- Enriquecer logs pino con `client_id`, `user_id`, `request_id`, `trace_id`
- `src/middleware/request_logger.ts` — middleware que enriquece cada request
- Integración con Logtail / Better Stack / Axiom (elegir según cuenta disponible)
- Métricas básicas: latencia p50/p95, tasa 5xx por endpoint, eventos de pedido por estado
- Alerting: threshold de errores 5xx

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── lib/logger.ts              ← MODIFICAR (agregar campos correlación)
└── middleware/
    └── request_logger.ts      ← NUEVO
infra-ruta/docs/
└── observability_setup.md     ← NUEVO
```

---

### AGENTE B — 6.INFRA-3 (Wave 6.1 — paralelo con A)

**Repo:** `backend-ruta/`
**Rama:** `feat/infra-6-3`
**Tamaño:** L

**Qué hacer:**
- `src/services/webhooks_outgoing.service.ts` — lee `webhook_subscriptions` del tenant, invoca URL, registra resultado en `webhook_deliveries` (append-only)
- `src/jobs/webhook_sender.job.ts` — cola pg-boss con reintentos exponenciales
- UI: componente de historial de entregas en el tab de Webhooks de Settings (complementa ADMIN-3)

**Archivos exclusivos:**
```
backend-ruta/api/src/
├── services/
│   └── webhooks_outgoing.service.ts  ← NUEVO
└── jobs/
    └── webhook_sender.job.ts         ← NUEVO
```

**Archivo compartido:** `src/app.ts` — comentario `// 6.INFRA-3`

---

### AGENTE C — 6.QA-1+2 (Wave 6.1 — paralelo con A y B)

**Repo:** `frontend-ruta/` + `backend-ruta/`
**Rama:** `feat/qa-6-12`
**Tamaño:** L+M

**Qué hacer:**
- **6.QA-1:** Suite Playwright completa — registro, catálogo, checkout Wompi, checkout COD, SHIP, PICKUP, cancelación, Vista de Control
- **6.QA-2:** `vitest --coverage` en backend; llegar a >85% de líneas

**Archivos exclusivos:**
```
frontend-ruta/e2e/
├── full_suite.spec.ts         ← NUEVO (importa specs de Sprints 3 y 4)
└── helpers/
    └── auth.helper.ts         ← NUEVO
backend-ruta/api/
└── vitest.config.ts           ← MODIFICAR (agregar coverage config)
```

---

### AGENTE D — 6.INFRA-2 (Wave 6.2 — puede correr con Wave 6.1)

**Repo:** `infra-ruta/`
**Rama:** `feat/infra-6-2`
**Tamaño:** M

**Qué hacer:**
- `scripts/backup_db.sh` — `pg_dump` comprimido a OCI Object Storage o S3-compatible
- `scripts/restore_db.sh` — restore probado contra BD staging
- `docs/backups.md` — procedimiento y frecuencia documentados

**Archivos exclusivos:**
```
infra-ruta/
├── scripts/
│   ├── backup_db.sh           ← NUEVO
│   └── restore_db.sh          ← NUEVO
└── docs/
    └── backups.md             ← NUEVO
```

---

### AGENTE E — 6.QA-4 (Wave 6.2 — paralelo con D)

**Repo:** `docs-ruta/`
**Rama:** `feat/docs-6-4`
**Tamaño:** M

**Qué hacer:**
- Guía de usuario ADMIN_CLIENT (catálogo, pedidos, couriers, configuración)
- Guía de usuario OPERATOR_CLIENT (operación diaria)
- Guía de usuario COURIER (app móvil: pedidos, cobro, entrega)
- Guía de usuario BUYER (catálogo, carrito, checkout, mis pedidos)

**Archivos exclusivos:**
```
docs-ruta/guias/
├── admin_client.md            ← NUEVO
├── operator_client.md         ← NUEVO
├── courier.md                 ← NUEVO
└── buyer.md                   ← NUEVO
```

---

### AGENTE F — 6.INFRA-4 + 6.QA-3 (Wave 6.3 — ÚLTIMO, requiere aprobación humana)

**Repo:** `infra-ruta/` + coordinación cross-repo
**Rama:** `feat/infra-6-4`
**Tamaño:** M+M

> ⚠️ Este agente no puede arrancar sin confirmación explícita del usuario. Implica cambios en producción irreversibles.

**Qué hacer:**
- **6.INFRA-4:**
  1. Aplicar SQL de migraciones a BD OCI producción
  2. Configurar DNS reales (api.ruta.com, app.ruta.com, tienda.ruta.com)
  3. Activar credenciales Wompi producción en Render env vars
  4. Deploy las 3 apps en Render producción
  5. Verificar `/healthz` en todos los servicios
- **6.QA-3:** Acompañar al usuario en:
  - ADMIN_RUTA crea Cliente piloto real
  - Configuración de catálogo, Wompi, repartidores, pickup points
  - Primer pedido real de extremo a extremo

---

---

## Diagrama de dependencias entre sprints (secuencia completa)

```
[1.ADMIN-4] (cerrar Sprint 1)
      │
      ▼
[2.SHARED-1] publicar @orkoruta/shared@1.2.0
      │
      ├──────────────────────────────────────┐
      ▼                                      ▼
[2.BACK-1] state machine      [2.BACK-3] Wompi    [2.BACK-4] jobs
      │                              │
      ├──────────────┐               │
      ▼              ▼               │
[2.BACK-2]    [2.STORE-1]    [2.ADMIN-1]
              │
              ▼
         [2.STORE-2]    [2.STORE-3]
              │
              │  + [2.BACK-3]
              ▼
         [2.STORE-4]    [2.QA-1]
              │
         Sprint 2 cerrado
              │
              ▼
[3.BACK-1] [3.BACK-2] [3.BACK-3] [3.BACK-456]  (paralelo)
              │
              ▼
    [3.ADMIN-1]    [3.ADMIN-2]  (paralelo)
              │
              ▼
          [3.QA-1]
              │
         Sprint 3 cerrado
              │
              ▼
      [4.BACK-1]    [4.ADMIN-1]  (paralelo)
              │
              ▼
          [4.QA-1]
              │
         Sprint 4 cerrado
              │
              ▼
[5.BACK-1] [5.BACK-2] [5.BACK-34]  (paralelo)
              │
              ▼
[5.ADMIN-1] [5.ADMIN-2] [5.ADMIN-34]  (paralelo)
              │
         Sprint 5 cerrado
              │
              ▼
[6.INFRA-1] [6.INFRA-3] [6.QA-12]  (paralelo)
[6.INFRA-2] [6.QA-4]               (paralelo, puede correr con lo anterior)
              │
              ▼
  [6.INFRA-4 + 6.QA-3] ← REQUIERE APROBACIÓN HUMANA
```

---

## Tabla resumen de paralelismo

| Sprint | Wave | Agentes | Tareas | Repos afectados |
|--------|------|---------|--------|-----------------|
| S2 | 2.0 | 1 | 2.SHARED-1 | packages-ruta |
| S2 | 2.1 | 3 | 2.BACK-1, 2.BACK-3, 2.BACK-4 | backend-ruta |
| S2 | 2.2 | 3 | 2.BACK-2, 2.STORE-1, 2.ADMIN-1 | backend-ruta, storefront, admin |
| S2 | 2.3 | 2 | 2.STORE-2, 2.STORE-3 | storefront |
| S2 | 2.4 | 2 | 2.STORE-4, 2.QA-1 | storefront, backend-ruta |
| S3 | 3.1 | 4 | 3.BACK-1/2/3/456 | backend-ruta |
| S3 | 3.2 | 2 | 3.ADMIN-1, 3.ADMIN-2 | admin |
| S3 | 3.3 | 1 | 3.QA-1 | frontend-ruta e2e |
| S4 | 4.1 | 2 | 4.BACK-1, 4.ADMIN-1 | backend-ruta, admin |
| S4 | 4.2 | 1 | 4.QA-1 | frontend-ruta e2e |
| S5 | 5.1 | 3 | 5.BACK-1/2/34 | backend-ruta |
| S5 | 5.2 | 3 | 5.ADMIN-1/2/34 | admin |
| S6 | 6.1 | 3 | 6.INFRA-1/3, 6.QA-12 | backend-ruta, infra-ruta, e2e |
| S6 | 6.2 | 2 | 6.INFRA-2, 6.QA-4 | infra-ruta, docs-ruta |
| S6 | 6.3 | 1 | 6.INFRA-4 + 6.QA-3 | cross-repo + producción |

**Máximo de agentes simultáneos:** 4 (Wave 3.1 — backend SHIP)
**Total de agentes a lanzar:** ~32 a lo largo de los 5 sprints

---

## Estado del plan (actualizar al cerrar cada sprint)

| Sprint | Estado | Fecha cierre |
|--------|--------|--------------|
| Sprint 0 | ✅ CERRADO | 2026-05-27 |
| Sprint 1 | ✅ CERRADO | 2026-05-28 |
| Sprint 2 | 🔄 EN CURSO | — (Wave 2.1 ✅ cerrada 2026-05-28 · Wave 2.2 ✅ cerrada 2026-05-28: 2.BACK-2 PR #7, 2.ADMIN-1 PR #9 y 2.STORE-1 PR #10 mergeados · siguiente Wave 2.3: 2.STORE-2 + 2.STORE-3) |
| Sprint 3 | ⬜ PENDIENTE | — |
| Sprint 4 | ⬜ PENDIENTE | — |
| Sprint 5 | ⬜ PENDIENTE | — |
| Sprint 6 | ⬜ PENDIENTE | — |
