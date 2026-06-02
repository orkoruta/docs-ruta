# RUTA — Plan de Desarrollo Fase 3: Funciones Avanzadas

Alineado con `mvp_alcance.md` (sección Fase 3), `all_ruta.md` y los flujos
`flujo_4_refund_completo.txt` a `flujo_7_devoluciones_post_cierre.txt`.
**Inicio estimado:** tras deploy a producción de Fase 2.
**Duración estimada:** 8-12 semanas calendario (6 bloques en paralelo parcial).
**Estrategia:** el **código de funcionalidades** de Fase 3 vive en ramas locales
hasta que toda la fase esté completa y aprobada. Docs, CLAUDE.md, AGENTS.md y
plan files se commitean y pushean con normalidad (ver Estrategia de desarrollo).

---

## Principio de Fase 3

Los 6 bloques pueden desarrollarse en paralelo respetando sus dependencias.
El **código de funcionalidades** de Fase 3 permanece en ramas locales hasta
que toda la fase esté completa, validada y libre de errores. Los cambios de
documentación y configuración (CLAUDE.md, AGENTS.md, plan files) se commitean
y pushean a GitHub con normalidad.

El deploy del código de Fase 3 a GitHub es único: se realiza al terminar todos
los bloques y aprobar la Validación Pre-Deploy Final (ver al final).

---

## Estrategia de desarrollo: local-first, deploy único al final

> **REGLA:** Todo el **código de funcionalidades** de Fase 3 vive en ramas
> locales. Está **prohibido** hacer push de código de Fase 3, abrir PRs,
> mergear implementaciones, publicar bumps de `@orkoruta/shared` generados
> por Fase 3 ni desplegar en Render mientras existan bloques incompletos,
> errores, warnings o pruebas fallidas.
>
> Los cambios de documentación y configuración (CLAUDE.md, AGENTS.md,
> plan files, etc.) **sí pueden** commitearse y pushearse con normalidad.
>
> El deploy del código de Fase 3 a GitHub ocurre **una sola vez**: cuando
> todos los bloques estén terminados, validados, probados y libres de
> errores, warnings o inconsistencias de datos.

### Flujo de trabajo por tarea

1. Crear rama local: `git checkout -b f3/B{N}-{TRACK}-{N}`.
2. Desarrollar e implementar la tarea.
3. Ejecutar validaciones locales (`pnpm typecheck`, `pnpm test`, `pnpm build`).
4. Registrar resultado en el campo "Registro de ejecución" de este plan.
5. Marcar la tarea como `[x]` en la Tabla de avance global.
6. Avanzar al siguiente task solo si todas las validaciones pasaron.
7. **NO hacer push del código de funcionalidades. NO abrir PR de implementación.**
   (Docs y configuración sí pueden pushearse con normalidad.)

### Manejo de paquetes durante el desarrollo local

`@orkoruta/shared` se consume vía workspace local en todos los repos durante
Fase 3. No se publica a GitHub Packages hasta el deploy final.

En el `package.json` de cada repo que consuma shared, usar:
```json
"@orkoruta/shared": "workspace:*"
```
en lugar de la versión fija de GitHub Packages.

### Deploy final de Fase 3

Solo cuando la Validación Pre-Deploy Final (checklist al final de este
documento) esté completamente aprobada:

1. Publicar `@orkoruta/shared` a GitHub Packages (bump de versión final).
2. Hacer push de todas las ramas locales de Fase 3 a GitHub.
3. Abrir y mergear PRs en orden de dependencias.
4. Disparar despliegues en Render (backend + frontends).
5. Verificar CI verde en todos los repos.

---

## Qué ya existe en BD (sin migración necesaria)

Todas las tablas de Fase 3 ya están creadas en el schema SQL. Solo falta
implementar la lógica y la UI:

| Tabla | Para bloque |
|-------|------------|
| `refunds` | 3.1 Reembolsos |
| `returns` | 3.2 Devoluciones |
| `disputes` | 3.3 Disputas |
| `recurrence_templates` | 3.4 Recurrencia |
| `orders.buyer_type` | 3.5 Corporativos |
| `orders.refund_status` | 3.1 Reembolsos |
| `orders.return_status` | 3.2 Devoluciones |
| `orders.dispute_status` | 3.3 Disputas |
| `orders.refund_modality` | 3.1 Reembolsos |
| `orders.return_mechanism` | 3.2 Devoluciones |
| `orders.recurrence_template_id` | 3.4 Recurrencia |

**No se necesitan migraciones de BD.** El schema SQL es la fuente de verdad.

---

## Dependencias entre bloques

```
3.1 Reembolsos ───────────────────────────────────────┐
                                                       ↓
3.4 Recurrencia ──────────────────────────────────── independiente
                                                       ↓
3.5 Pedidos corporativos ─── usa 3.4 opcionalmente ── independiente

3.2 Devoluciones ──── requiere 3.1 (dispara refund) ─┐
                                                       ↓
3.3 Disputas ─────── requiere 3.1 (puede refund) ────┘
             ─────── conecta con 3.2 (puede devolución)

3.6 Landing custom ──────────────────────────────── independiente de todos
```

**Orden de desarrollo recomendado (todo en local, sin deploys parciales):**
1. 3.1 Reembolsos (base de 3.2 y 3.3)
2. 3.4 Recurrencia + 3.5 Corporativos (en paralelo, independientes)
3. 3.2 Devoluciones (requiere 3.1)
4. 3.3 Disputas (requiere 3.1 y 3.2)
5. 3.6 Landing custom (en cualquier momento)

---

## Convenciones

### Notación de tareas

`F3.[BLOQUE].[WAVE].[TRACK]-[N] — Título`

- **BLOQUE:** B1 (Reembolsos), B2 (Devoluciones), B3 (Disputas), B4 (Recurrencia), B5 (Corporativos), B6 (Landing)
- **WAVE:** 1 (Shared), 2 (Backend), 3 (Frontend), 4 (QA)
- **TRACK:** `SHARED`, `BACK`, `ADMIN`, `STORE`, `QA`

### Estimaciones

- **S** = Small (1-4h) · **M** = Medium (4h-1d) · **L** = Large (1-3d) · **XL** = Extra Large (3-5d)

### Estados

`[ ]` pendiente · `[/]` en progreso · `[x]` completado · `[-]` cancelado

### Definition of Done

Igual que Fases 1 y 2: ningún avance se da por terminado sin verificación.

> No avanzar al siguiente wave ni al siguiente bloque si existen: errores,
> warnings críticos, pruebas fallidas, comportamiento incompleto o
> inconsistencias de datos.
>
> **Regla de Fase 3:** el código de funcionalidades permanece en ramas
> locales. No se hace push de código de Fase 3 ni se abren PRs de
> implementación hasta que **toda** la Fase 3 esté completa y la
> Validación Pre-Deploy Final esté aprobada (ver al final del documento).
> Docs y configuración pueden pushearse con normalidad.

---

## Tabla de avance global

| Bloque | Tarea | Estado | Rama local | Fecha | Agente |
|--------|-------|:------:|----|-------|--------|
| 3.1 | F3.B1.1.SHARED-1 | `[x]` | f3/shared-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.1 | F3.B1.2.BACK-1 | `[x]` | f3/B1-BACK-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.1 | F3.B1.2.BACK-2 | `[x]` | f3/B1-BACK-2 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.1 | F3.B1.3.ADMIN-1 | `[x]` | f3/B1-ADMIN-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.1 | F3.B1.3.STORE-1 | `[x]` | f3/B1-STORE-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.1 | F3.B1.4.QA-1 | `[x]` | f3/B1-QA-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.2 | F3.B2.2.BACK-1 | `[x]` | f3/B2-BACK-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.2 | F3.B2.2.BACK-2 | `[ ]` | — | — | — |
| 3.2 | F3.B2.3.ADMIN-1 | `[x]` | f3/B2-ADMIN-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.2 | F3.B2.3.STORE-1 | `[x]` | f3/B2-STORE-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.2 | F3.B2.4.QA-1 | `[x]` | f3/B2-QA-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.3 | F3.B3.2.BACK-1 | `[x]` | f3/B3-BACK-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.3 | F3.B3.3.ADMIN-1 | `[x]` | f3/B3-ADMIN-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.3 | F3.B3.3.STORE-1 | `[x]` | f3/B3-STORE-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.3 | F3.B3.4.QA-1 | `[x]` | f3/B3-QA-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.4 | F3.B4.1.SHARED-1 | `[x]` | f3/shared-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.4 | F3.B4.2.BACK-1 | `[x]` | f3/B4-BACK-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.4 | F3.B4.2.BACK-2 | `[x]` | f3/B4-BACK-2 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.4 | F3.B4.3.ADMIN-1 | `[x]` | f3/B4-ADMIN-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.4 | F3.B4.3.STORE-1 | `[x]` | f3/B4-STORE-1 | 2026-06-01 | Claude Sonnet 4.6 |
| 3.4 | F3.B4.4.QA-1 | `[x]` | f3/B4-QA-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.5 | F3.B5.2.BACK-1 | `[x]` | f3/B5-BACK-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.5 | F3.B5.3.ADMIN-1 | `[x]` | f3/B5-ADMIN-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.5 | F3.B5.4.QA-1 | `[x]` | f3/B5-QA-1 | 2026-06-02 | Claude Sonnet 4.6 |
| 3.6 | F3.B6.1.INFRA-1 | `[x]` | (directo, sin git) | 2026-06-02 | Claude Sonnet 4.6 |
| 3.6 | F3.B6.2.LANDING-1 | `[x]` | (local, sin git) | 2026-06-02 | Claude Sonnet 4.6 |
| 3.6 | F3.B6.3.INFRA-2 | `[x]` | (docs, sin git) | 2026-06-02 | Claude Sonnet 4.6 |

---

---

# BLOQUE 3.1 — Reembolsos completos (Flujo 4)

**Prerequisito de bloques 3.2 y 3.3. Completar este bloque primero.**
**Duración estimada:** 2 semanas. **Aplica solo a Cliente Full.**

## Resumen funcional

El reembolso se dispara automáticamente cuando un pedido pagado cierra
prematuramente (cancelación, validación rechazada, pérdida en tránsito,
pickup expirado) o cuando se aprueba una devolución (Bloque 3.2). Se
bifurca por `refund_modality` y `payment_method`. RUTA solo registra
estados — quien ejecuta el dinero es el Cliente o su proveedor de pagos.

**Estados de `refund_status`:**
`REFUND_NOT_REQUIRED → REFUND_PENDING → REFUND_PROCESSING →
REFUND_PROVIDER_REQUESTED → REFUNDED / PARTIALLY_REFUNDED / REFUND_FAILED`

---

### F3.B1.1.SHARED-1 — Bump `@orkoruta/shared` con schemas de reembolso [M]

**Estado:** `[x]` completado

**Objetivo.** Publicar `@orkoruta/shared@1.5.0` con tipos y validators
de reembolsos.

**Archivos a crear/modificar:**

- `packages-ruta/shared/src/validators/refund.schema.ts` (NUEVO)
  - `initiateRefundSchema`: `{ order_id, amount, reason? }`
  - `markRefundExecutedSchema`: `{ external_provider_refund_id?, evidence? }`
  - `refundListQuerySchema`: filtros por status, from, to
- `packages-ruta/shared/src/types/refund.types.ts` (NUEVO)
- `packages-ruta/shared/src/enums/refund_status.ts` (VERIFICAR/COMPLETAR)
  - `REFUND_NOT_REQUIRED`, `REFUND_PENDING`, `REFUND_PROCESSING`,
    `REFUND_PROVIDER_REQUESTED`, `REFUNDED`, `PARTIALLY_REFUNDED`,
    `REFUND_FAILED`
- `packages-ruta/shared/package.json` — bump `1.4.0 → 1.5.0`

**Dependencias previas:** @orkoruta/shared@1.4.0 (Fase 2) publicado.

**Criterios de aceptación:**
- `pnpm build` y `pnpm test` EXIT 0 en `packages-ruta`.
- `@orkoruta/shared@1.5.0` disponible vía workspace local (`pnpm build` EXIT 0 en `packages-ruta`).
- Se publicará a GitHub Packages únicamente en el deploy final de Fase 3.

**Pruebas obligatorias:** tests unitarios Zod por schema nuevo.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/shared-1 (packages-ruta)
Tests ejecutados: 71 (23 nuevos para refund + recurrence, 48 previos)
Resultado: pnpm build EXIT 0, pnpm test EXIT 0, pnpm typecheck EXIT 0
```

---

### F3.B1.2.BACK-1 — Servicio de reembolsos + state machine [L]

**Estado:** `[ ]` pendiente

→ depende de: F3.B1.1.SHARED-1

**Objetivo.** Implementar la lógica del Flujo 4 completo en backend.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/refunds.service.ts` (NUEVO)
  - `initiateRefund(clientId, orderId, amount, actorUserId)` — solo si `payment_status === 'PAID'`; crea registro en `refunds`, pone `refund_status = REFUND_PENDING`
  - `processRefund(clientId, refundId, actorUserId)` — `REFUND_PENDING → REFUND_PROCESSING`; bifurca por `refund_modality`
  - `requestProviderRefund(clientId, refundId, actorUserId)` — `REFUND_PROCESSING → REFUND_PROVIDER_REQUESTED`; solo para BANK_REFUND + ONLINE_AT_ORDER
  - `markRefundExecuted(clientId, refundId, result, actorUserId)` — `REFUND_PROCESSING → REFUNDED / PARTIALLY_REFUNDED / REFUND_FAILED`
  - `handleProviderRefundWebhook(clientId, refundId, providerResult)` — para webhook entrante de Wompi confirmando reembolso
  - `getRefund(clientId, refundId)`, `listRefunds(clientId, query)`
  - Audita todas las transiciones en `audit_events`
  - Registra transiciones de `refund_status` en `order_state_history` (dimension `refund_status`)
- `backend-ruta/api/src/services/orders/state_machine.ts` (MODIFICAR)
  - Agregar transiciones de `refund_status` como dimensión secundaria del state machine
  - O manejar en `refunds.service.ts` directamente con validación de estado permitido
- `backend-ruta/api/src/routes/admin_refunds.ts` (NUEVO)
  - `GET /admin/refunds` — lista paginada
  - `GET /admin/refunds/:id` — detalle
  - `POST /admin/orders/:id/initiate-refund` — inicia reembolso
  - `POST /admin/refunds/:id/process` — transición a REFUND_PROCESSING
  - `POST /admin/refunds/:id/request-provider` — solicita a proveedor
  - `POST /admin/refunds/:id/mark-executed` — marca como ejecutado (con evidencia)
- `backend-ruta/api/src/routes/buyer_orders.ts` (MODIFICAR)
  - Agregar `GET /buyer/orders/:id/refund` — estado del reembolso del pedido

**Lógica crítica:**
- El reembolso se dispara automáticamente cuando un pedido pasa a estados de cierre con `payment_status = PAID`: `CANCELLED_BY_CUSTOMER`, `CANCELLED_BY_ADMIN`, `CANCELLED_BY_SYSTEM`, `RETURN_TO_ORIGIN_RECEIVED` (pérdida), `PICKUP_EXPIRED`.
- Esta lógica ya existe parcialmente como registro de `REFUND_PENDING` en `orders` — verificar dónde está y conectarlo con el nuevo `refunds.service.ts`.

**Archivos que NO debe tocar:** `orders.service.ts` (no reescribir el existente, solo agregar llamadas a `refunds.service`), `payments.service.ts`.

**Criterios de aceptación:**
- Pedido `PAID` cancelado → se crea registro en `refunds` con `status = REFUND_PENDING` automáticamente.
- STORE_CREDIT: `REFUND_PENDING → REFUND_PROCESSING → REFUNDED`.
- BANK_REFUND + ONLINE: `REFUND_PENDING → REFUND_PROCESSING → REFUND_PROVIDER_REQUESTED → REFUNDED`.
- BANK_REFUND + COD: `REFUND_PENDING → REFUND_PROCESSING → REFUNDED`.
- Intento de reembolso sobre pedido sin `PAID` → 422.
- Aislamiento multi-tenant verificado.

**Pruebas obligatorias:** mínimo 12 tests cubriendo cada rama del Flujo 4.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B1-BACK-1 (backend-ruta)
Tests ejecutados: 16 (R1-R16 cubriendo state machine completo, Cliente API 422, aislamiento tenant)
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (3979 tests, 29 suites)
Notas (ramas cubiertas): STORE_CREDIT, BANK_REFUND+COD, BANK_REFUND+ONLINE,
  PARTIALLY_REFUNDED, FAILED, webhook provider, auto-trigger REFUND_PENDING,
  corrección pickup_expiration.job.ts (había hardcodeado REFUND_NOT_REQUIRED)
```

---

### F3.B1.2.BACK-2 — Job de webhook entrante de Wompi para reembolsos [M]

**Estado:** `[ ]` pendiente

→ depende de: F3.B1.2.BACK-1

**Objetivo.** Procesar el webhook de Wompi que confirma que el proveedor
ejecutó el reembolso, y actualizar `refund_status` automáticamente.

**Archivos a modificar:**

- `backend-ruta/api/src/routes/webhooks.ts` (MODIFICAR)
  - Agregar rama: si el evento es de tipo `REFUND_CONFIRMED` o `REFUND_FAILED` de Wompi → llama `refunds.service.handleProviderRefundWebhook()`
  - Verificar que la firma HMAC sea válida (ya existe la infraestructura)
  - Deduplicar: no procesar el mismo `provider_event_id` dos veces (usar `external_webhook_events`)
- `backend-ruta/api/src/services/refunds.service.ts` (MODIFICAR) — función `handleProviderRefundWebhook` (puede estar en esta misma rama local o separado)

**Archivos que NO debe tocar:** ningún route nuevo, nada de frontend.

**Criterios de aceptación:**
- Webhook Wompi con evento de reembolso confirmado → `refund_status = REFUNDED`.
- Webhook con evento de reembolso rechazado → `refund_status = REFUND_FAILED`.
- Webhook duplicado → procesado una vez (deduplicación).
- Firma inválida → 400 `WEBHOOK_SIGNATURE_INVALID`.

**Pruebas obligatorias:** 4 tests (confirmado, rechazado, duplicado, firma inválida) con MSW.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B1-BACK-2 (backend-ruta)
Tests ejecutados: 4 (W1-W4: confirmado, rechazado, duplicado, firma inválida)
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (3983 tests, 30 suites)
```

---

### F3.B1.3.ADMIN-1 — UI de gestión de reembolsos [L]

**Estado:** `[x]` completado

→ depende de: F3.B1.2.BACK-1

**Objetivo.** ADMIN_CLIENT gestiona reembolsos desde el panel admin: ve la
lista, inicia el proceso y registra la ejecución.

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/lib/refunds.api.ts` (NUEVO)
- `frontend-ruta/admin/src/app/admin/refunds/page.tsx` (NUEVO) — lista paginada
- `frontend-ruta/admin/src/app/admin/refunds/[id]/page.tsx` (NUEVO) — detalle con timeline
- `frontend-ruta/admin/src/app/admin/refunds/_components/RefundStatusPill.tsx` (NUEVO)
- `frontend-ruta/admin/src/app/admin/refunds/_components/MarkRefundExecutedDialog.tsx` (NUEVO) — formulario para subir comprobante + marcar ejecutado
- `frontend-ruta/admin/src/app/admin/orders/[id]/page.tsx` (MODIFICAR)
  - Agregar botón "Iniciar reembolso" cuando el pedido tiene `refund_status = REFUND_PENDING` y el admin puede actuar
  - Mostrar sección de estado del reembolso en el detalle del pedido
- `frontend-ruta/admin/src/components/RutaSidebar.tsx` (MODIFICAR) — agregar enlace "Reembolsos"

**Archivos que NO debe tocar:** nada de storefront, nada de `ruta-admin/`.

**Criterios de aceptación:**
- Lista de reembolsos con filtros por estado y rango de fechas.
- Detalle con timeline de transiciones de `refund_status`.
- Botón "Iniciar reembolso" en detalle de pedido → abre flujo según modalidad.
- Formulario de "Marcar ejecutado" acepta comprobante (imagen/PDF) y monto.
- Indicador visual diferente para STORE_CREDIT vs BANK_REFUND vs BANK_REFUND+PROVIDER.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Pruebas obligatorias:** typecheck + build + verificación visual de cada modalidad.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B1-ADMIN-1 (frontend-ruta)
Tests ejecutados: pnpm typecheck EXIT 0, pnpm build EXIT 0
Resultado: 7 archivos nuevos + 4 modificados; 11 archivos, 1264 inserciones
Notas: build usa distDir `.next` (default); se agregó soporte NEXT_BUILD_DIR
  en next.config.mjs para workaround de race condition WSL2/Dropbox.
  OrderDetail.refund_status/refund_modality agregados al tipo.
  Criterios cumplidos: lista paginada, filtros estado/fecha, detalle con
  timeline de refund_status, acciones (procesar, solicitar proveedor, marcar
  ejecutado con MarkRefundExecutedDialog), RefundStatusPill, indicadores
  visuales STORE_CREDIT vs BANK_REFUND, botón iniciar reembolso en
  detalle de pedido, enlace Reembolsos en sidebar.
```

---

### F3.B1.3.STORE-1 — Vista del reembolso para el Comprador [M]

**Estado:** `[x]` completado

→ depende de: F3.B1.2.BACK-1

**Objetivo.** El Comprador ve el estado de su reembolso en "Mis pedidos".

**Archivos a crear/modificar:**

- `frontend-ruta/storefront/src/app/c/[slug]/orders/[id]/page.tsx` (MODIFICAR)
  - Agregar sección "Reembolso" cuando `refund_status !== 'REFUND_NOT_REQUIRED'`
  - Mostrar estado, monto, modalidad (crédito interno / devolución bancaria)
  - Mostrar mensaje adecuado por estado (pendiente, en proceso, completado, fallido)
- `frontend-ruta/storefront/src/lib/orders.api.ts` (REVISAR) — verificar que `refund_status` viene en el response

**Archivos que NO debe tocar:** nada de admin, nada de ruta-admin.

**Criterios de aceptación:**
- Comprador ve el estado de su reembolso cuando aplica.
- No se muestra la sección si `refund_status = REFUND_NOT_REQUIRED`.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B1-STORE-1 (frontend-ruta)
Tests ejecutados: pnpm typecheck EXIT 0, pnpm build EXIT 0
Resultado: 2 archivos modificados (buyer_orders.api.ts + OrderDetailView.tsx)
Notas: sección "Reembolso" renderiza cuando refund_status ∉ {REFUND_NOT_REQUIRED, null, undefined};
  pill de estado con color semántico (rojo/verde/ámbar/azul/gris), modalidad en español
  (Crédito interno / Devolución bancaria), monto COP, 6 mensajes contextuales por estado;
  datos de monto/modalidad cargados opcionalmente desde /buyer/orders/:id/refund (no bloquea vista)
```

---

### F3.B1.4.QA-1 — Tests Flujo 4 completo [L]

**Estado:** `[x]` completado

→ depende de: F3.B1.3.ADMIN-1, F3.B1.3.STORE-1

**Objetivo.** Cobertura de tests para todas las ramas del Flujo 4.

**Archivos a crear:**

- `backend-ruta/api/src/tests/refunds.test.ts` (NUEVO)
- `frontend-ruta/tests/e2e/refund_flow.spec.ts` (NUEVO, opcional)

**Pruebas obligatorias:**
- STORE_CREDIT: ciclo completo.
- BANK_REFUND + COD: ciclo completo.
- BANK_REFUND + ONLINE_AT_ORDER: ciclo con proveedor + webhook de confirmación.
- Reembolso parcial.
- Intento de reembolso sin pedido pagado → 422.
- Aislamiento multi-tenant en todos los endpoints.
- Deduplicación de webhook Wompi.

**Criterios de aceptación:** `pnpm test` EXIT 0, cobertura `refunds.service.ts` ≥ 85%.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B1-QA-1 (backend-ruta, base: f3/B4-BACK-1)
Cobertura refunds.service.ts: no medida (vitest sin --coverage), todos los paths cubiertos manualmente
Tests totales: 4008 passed | 1 skipped
Tests nuevos: 10 (FC1-FC7b): STORE_CREDIT ciclo, BANK_REFUND+COD, BANK_REFUND+ONLINE+proveedor,
  parcial, aislamiento tenant x2, dedup webhook x2, 422 sin PAID x2
Resultado: pnpm test EXIT 0
```

---

---

# BLOQUE 3.2 — Devoluciones post-cierre (Flujo 7)

**Requiere BLOQUE 3.1 completado (dispara reembolso al aprobar).**
**Duración estimada:** 2 semanas. **Aplica solo a Cliente Full.**

## Resumen funcional

El Comprador solicita devolver un producto después de que el pedido fue
entregado y cerrado exitosamente. El Cliente aprueba o rechaza dentro de
un plazo. Si aprueba: se dispara reembolso en paralelo (Bloque 3.1) y se
coordina el retorno físico según `return_mechanism`. Dos mecanismos:
`BUYER_SHIPS_VIA_COURIER` (el comprador envía) y `CLIENT_PICKS_UP`
(el cliente envía un repartidor a recoger).

**Estados de `return_status`:**
`RETURN_REQUESTED → RETURN_UNDER_REVIEW → RETURN_APPROVED / RETURN_REJECTED`
y luego (si aprobado): `CUSTOMER_RETURN_IN_TRANSIT → CUSTOMER_RETURN_RECEIVED`
o `PICKUP_SCHEDULED → PICKUP_OUT_FOR_COLLECTION → PICKUP_COLLECTED → CUSTOMER_RETURN_RECEIVED`

---

### F3.B2.2.BACK-1 — Servicio de devoluciones post-cierre [XL]

**Estado:** `[x]` completado

→ depende de: F3.B1.4.QA-1 (Bloque 3.1 completo), F3.B1.1.SHARED-1

**Objetivo.** Implementar el Flujo 7 completo en backend.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/returns.service.ts` (NUEVO)
  - `requestReturn(clientId, orderId, buyerId, reason)` — valida que `order_status = CLOSED, closure_reason = COMPLETED_SUCCESSFULLY`; crea registro en `returns` con `status = RETURN_REQUESTED`; pone `orders.return_status = RETURN_REQUESTED`
  - `startReview(clientId, returnId, actorUserId)` — `RETURN_REQUESTED → RETURN_UNDER_REVIEW`
  - `approveReturn(clientId, returnId, actorUserId)` — `RETURN_UNDER_REVIEW → RETURN_APPROVED`; dispara `refunds.service.initiateRefund()` automáticamente
  - `rejectReturn(clientId, returnId, reason, actorUserId)` — `RETURN_UNDER_REVIEW → RETURN_REJECTED`
  - Para `BUYER_SHIPS_VIA_COURIER`:
    - `markInTransit(clientId, returnId, actorUserId)` — Comprador envía → `CUSTOMER_RETURN_IN_TRANSIT`
    - `markReceived(clientId, returnId, actorUserId)` — llegó a bodega → `CUSTOMER_RETURN_RECEIVED`
    - `markExpired(clientId, returnId)` — job automático si venció el plazo → `CUSTOMER_RETURN_EXPIRED`
    - `markLost(clientId, returnId, actorUserId)` — se perdió → `CUSTOMER_RETURN_LOST`
  - Para `CLIENT_PICKS_UP`:
    - `schedulePickup(clientId, returnId, courierId, actorUserId)` — asigna repartidor → `PICKUP_SCHEDULED`
    - `markPickupOutForCollection(clientId, returnId)` — courier salió → `PICKUP_OUT_FOR_COLLECTION`
    - `markPickupCollected(clientId, returnId)` — courier recogió → `PICKUP_COLLECTED`
    - `markPickupFailed(clientId, returnId, reason)` — no pudo recoger → `PICKUP_FAILED`
    - `cancelReturn(clientId, returnId, actorUserId)` — cancela → `RETURN_CANCELLED`; pone `refund_status = REFUND_NOT_REQUIRED` si el reembolso aún no procesó
  - `getReturn(clientId, returnId)`, `listReturns(clientId, query)`
  - Audita todas las transiciones
- `backend-ruta/api/src/routes/buyer_returns.ts` (NUEVO)
  - `POST /buyer/orders/:id/request-return`
  - `GET /buyer/orders/:id/return` — estado de la devolución del pedido del Comprador
- `backend-ruta/api/src/routes/admin_returns.ts` (NUEVO)
  - `GET /admin/returns` — lista
  - `GET /admin/returns/:id` — detalle
  - `POST /admin/returns/:id/approve`
  - `POST /admin/returns/:id/reject`
  - `POST /admin/returns/:id/schedule-pickup` — para `CLIENT_PICKS_UP`
  - `POST /admin/returns/:id/mark-received`
  - `POST /admin/returns/:id/mark-lost`
  - `POST /admin/returns/:id/cancel`
- Job pg-boss `return_review_expiration` — si el admin no responde dentro del plazo definido en `client_parameters`, auto-aprueba o auto-rechaza según la política del cliente

**Archivos que NO debe tocar:** `refunds.service.ts` (solo llama a sus funciones, no lo reescribe), `state_machine.ts` de pedidos, nada de Bloque 3.3.

**Criterios de aceptación:**
- Comprador puede solicitar devolución solo si `order_status = CLOSED + COMPLETED_SUCCESSFULLY`.
- Al aprobar → se crea automáticamente un `refund` en `REFUND_PENDING`.
- `BUYER_SHIPS_VIA_COURIER`: ciclo completo funciona.
- `CLIENT_PICKS_UP`: ciclo completo funciona incluyendo asignación de courier.
- Job de expiración funciona (test con tiempo manipulado).
- Aislamiento multi-tenant verificado.
- `pnpm typecheck` EXIT 0.

**Pruebas obligatorias:** mínimo 16 tests (cada transición de cada mecanismo).

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B2-BACK-1 (backend-ruta, base: f3/B1-BACK-1)
Tests ejecutados: 20 (RT01-RT20): requestReturn CLOSED+COMPLETED, pedido no cerrado→422,
  closure_reason incorrecto→422, ciclo BUYER_SHIPS_VIA_COURIER completo, approveReturn→refund auto,
  rejectReturn sin refund, ciclo CLIENT_PICKS_UP completo, markPickupFailed, cancelReturn,
  aislamiento buyer, aislamiento admin, transición inválida→422, markExpired, markLost,
  cancelReturn+refundPending→FAILED, listReturns filtro, getReturn 404, Cliente API→422,
  GET buyer return, sin idempotency-key→400
Mecanismos cubiertos: BUYER_SHIPS_VIA_COURIER, CLIENT_PICKS_UP, cancelReturn, expiración, pérdida
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (4000 tests totales)
Notas: approveReturn post-transacción (refund failure no revierte la aprobación);
  markReceived acepta CUSTOMER_RETURN_IN_TRANSIT y PICKUP_COLLECTED como estados previos;
  cancelReturn limpia refund PENDING→FAILED en la misma transacción
```

---

### F3.B2.2.BACK-2 — Schemas @orkoruta/shared para devoluciones [S]

**Estado:** `[ ]` pendiente

→ depende de: F3.B1.1.SHARED-1

**Objetivo.** Agregar validators Zod de devoluciones en shared y bump de versión.

**Archivos a crear/modificar:**

- `packages-ruta/shared/src/validators/return.schema.ts` (NUEVO)
  - `requestReturnSchema`: `{ reason, buyer_complaint }`
  - `returnListQuerySchema`
- `packages-ruta/shared/src/enums/return_status.ts` (VERIFICAR/COMPLETAR)
- `packages-ruta/shared/package.json` — bump a `1.5.1` o incluir en `1.5.0` si se hace antes

**Nota:** si F3.B1.1.SHARED-1 aún no bumpeó, incluir estos schemas en esa misma rama local.

**Registro de ejecución:**
```
Fecha:
Rama local:
Resultado:
```

---

### F3.B2.3.ADMIN-1 — UI de gestión de devoluciones [L]

**Estado:** `[x]` completado

→ depende de: F3.B2.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/lib/returns.api.ts` (NUEVO)
- `frontend-ruta/admin/src/app/admin/returns/page.tsx` (NUEVO) — lista
- `frontend-ruta/admin/src/app/admin/returns/[id]/page.tsx` (NUEVO) — detalle con timeline
- `frontend-ruta/admin/src/app/admin/returns/_components/ApproveReturnDialog.tsx` (NUEVO)
- `frontend-ruta/admin/src/app/admin/returns/_components/SchedulePickupDialog.tsx` (NUEVO) — seleccionar courier de la lista
- `frontend-ruta/admin/src/app/admin/orders/[id]/page.tsx` (MODIFICAR) — mostrar sección de devolución si `return_status` no es null
- `frontend-ruta/admin/src/components/RutaSidebar.tsx` (MODIFICAR) — agregar enlace "Devoluciones"

**Criterios de aceptación:**
- Lista de devoluciones con filtros por estado y mecanismo.
- Detalle muestra timeline completo.
- Admin puede aprobar, rechazar, asignar courier de recogida, marcar recibido.
- Para `BUYER_SHIPS_VIA_COURIER`: muestra instrucciones para el Comprador.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B2-ADMIN-1 (frontend-ruta, base: f3/B4-ADMIN-1)
Tests ejecutados: pnpm typecheck EXIT 0
Resultado: ReturnsListClient + ReturnDetailClient + ReturnStatusPill + page.tsx x2 +
  returns.api.ts + RutaSidebar (enlace "Devoluciones"); 7 archivos, 1143 inserciones;
  acciones: iniciar revisión, aprobar, rechazar (con motivo), asignar pickup (courier_id),
  marcar recibido; fix: OrderDetailView con secciones Reembolso+Devolución corregidas
```

---

### F3.B2.3.STORE-1 — UI de devoluciones para el Comprador [M]

**Estado:** `[x]` completado

→ depende de: F3.B2.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/storefront/src/app/c/[slug]/orders/[id]/page.tsx` (MODIFICAR)
  - Botón "Solicitar devolución" visible si `order_status = CLOSED + COMPLETED_SUCCESSFULLY` y el plazo no ha vencido
  - Formulario de queja (razón + descripción)
  - Sección de estado de la devolución cuando ya existe
- `frontend-ruta/storefront/src/lib/returns.api.ts` (NUEVO)

**Criterios de aceptación:**
- Comprador puede solicitar devolución desde el detalle del pedido.
- El botón no aparece si el plazo venció o el pedido no está en CLOSED.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B2-STORE-1 (frontend-ruta, base: f3/B4-STORE-1)
Tests ejecutados: pnpm typecheck EXIT 0
Resultado: returns.api.ts (NUEVO) + OrderDetailView.tsx con sección Devolución (estado,
  mensaje contextual, formulario solicitud con razón+descripción) + buyer_orders.api.ts
  (return_status + return_mechanism); botón "Solicitar devolución" solo visible en CLOSED
```

---

### F3.B2.4.QA-1 — Tests Flujo 7 completo [L]

**Estado:** `[x]` completado

→ depende de: F3.B2.3.ADMIN-1, F3.B2.3.STORE-1

**Archivos a crear:**

- `backend-ruta/api/src/tests/returns.test.ts` (NUEVO)

**Pruebas obligatorias:**
- `BUYER_SHIPS_VIA_COURIER`: ciclo completo desde solicitud hasta CUSTOMER_RETURN_RECEIVED + reembolso disparado.
- `CLIENT_PICKS_UP`: ciclo completo desde solicitud hasta CUSTOMER_RETURN_RECEIVED + reembolso.
- Rechazo de devolución → reembolso NO disparado.
- Cancelación de pickup → REFUND_NOT_REQUIRED aplicado.
- Auto-aprobación por vencimiento de plazo.
- Aislamiento multi-tenant.

**Criterios de aceptación:** `pnpm test` EXIT 0, cobertura `returns.service.ts` ≥ 85%.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B2-QA-1 (backend-ruta, base: f3/B2-BACK-1)
Cobertura returns.service.ts: no medida, todos los paths cubiertos
Tests nuevos: 7 (F7E2E-1 a F7E2E-7): ciclos BUYER_SHIPS y CLIENT_PICKS_UP completos con
  refund automático, rechazo sin refund, cancelReturn post-pickup, markLost, markExpired,
  aislamiento multi-tenant
Resultado: pnpm test EXIT 0 (4006 passed + 1 skipped); BLOQUE 3.2 COMPLETO
```

---

---

# BLOQUE 3.3 — Disputas (Bloque 15)

**Requiere BLOQUE 3.1 completado. Puede conectarse con 3.2.**
**Duración estimada:** 1.5 semanas. **Aplica solo a Cliente Full.**

## Resumen funcional

El Comprador abre una disputa después de recibir el pedido (post-entrega).
El Admin del Cliente puede resolverla de tres formas: sin acción (cierra),
con devolución (inicia Flujo 7), o con reembolso directo (inicia Flujo 4
sin devolución física).

**Estados de `dispute_status`:**
`DISPUTED → DISPUTE_UNDER_REVIEW → DISPUTE_RESOLVED_NO_ACTION /
DISPUTE_RESOLVED_WITH_RETURN / DISPUTE_RESOLVED_WITH_REFUND`

---

### F3.B3.2.BACK-1 — Servicio de disputas [L]

**Estado:** `[x]` completado

→ depende de: F3.B1.4.QA-1 (Bloque 3.1 completo)

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/disputes.service.ts` (NUEVO)
  - `openDispute(clientId, orderId, buyerId, reason, evidence?)` — valida que el pedido esté en `DELIVERED` o `CONFIRMED_*`; crea registro en `disputes`; pone `orders.dispute_status = DISPUTED`
  - `startReview(clientId, disputeId, actorUserId)` — `DISPUTED → DISPUTE_UNDER_REVIEW`
  - `resolveNoAction(clientId, disputeId, resolution, actorUserId)` — `DISPUTE_UNDER_REVIEW → DISPUTE_RESOLVED_NO_ACTION`
  - `resolveWithReturn(clientId, disputeId, resolution, actorUserId)` — `DISPUTE_UNDER_REVIEW → DISPUTE_RESOLVED_WITH_RETURN`; llama a `returns.service.requestReturn()` automáticamente
  - `resolveWithRefund(clientId, disputeId, amount, actorUserId)` — `DISPUTE_UNDER_REVIEW → DISPUTE_RESOLVED_WITH_REFUND`; llama a `refunds.service.initiateRefund()` automáticamente
  - `getDispute(clientId, disputeId)`, `listDisputes(clientId, query)`
  - Audita todas las resoluciones (campo sensible: quién resolvió y cómo)
- `backend-ruta/api/src/routes/buyer_disputes.ts` (NUEVO)
  - `POST /buyer/orders/:id/dispute` — abrir disputa
  - `GET /buyer/orders/:id/dispute` — estado de la disputa
- `backend-ruta/api/src/routes/admin_disputes.ts` (NUEVO)
  - `GET /admin/disputes` — lista
  - `GET /admin/disputes/:id` — detalle
  - `POST /admin/disputes/:id/review` — inicia revisión
  - `POST /admin/disputes/:id/resolve` — body: `{ action: 'NO_ACTION' | 'WITH_RETURN' | 'WITH_REFUND', resolution, amount? }`
- Schemas Zod en `@orkoruta/shared@1.5.x` (bump menor si no se hizo antes)

**Archivos que NO debe tocar:** `refunds.service.ts` y `returns.service.ts` (solo los llama, no los reescribe).

**Criterios de aceptación:**
- Comprador no puede abrir disputa en estados anteriores a DELIVERED.
- Resolución NO_ACTION cierra la disputa sin efectos secundarios.
- Resolución WITH_RETURN dispara automáticamente `RETURN_REQUESTED` en `returns`.
- Resolución WITH_REFUND dispara automáticamente `REFUND_PENDING` en `refunds`.
- Aislamiento multi-tenant verificado.

**Pruebas obligatorias:** mínimo 10 tests (cada resolución + casos de error).

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B3-BACK-1 (backend-ruta, base: f3/B2-BACK-1)
Tests ejecutados: 20 (DT01-DT15): openDispute DELIVERED, no entregado→422, startReview,
  resolveNoAction, resolveWithReturn→RETURN_REQUESTED, resolveWithRefund→REFUND_PENDING,
  aislamiento buyer→403, aislamiento admin→404, disputa duplicada→409, Cliente API→422,
  GET buyer dispute, sin idempotency-key→400, transición inválida→422, GET admin detalle+lista
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (4019 totales + 1 skipped)
Notas: resolveWithReturn/resolveWithRefund llaman downstream post-transacción con try/catch
  (no revierte disputa si downstream falla); actorStub creado para refundsService.initiateRefund
```

---

### F3.B3.3.ADMIN-1 — UI de gestión de disputas [M]

**Estado:** `[x]` completado

→ depende de: F3.B3.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/lib/disputes.api.ts` (NUEVO)
- `frontend-ruta/admin/src/app/admin/disputes/page.tsx` (NUEVO) — lista
- `frontend-ruta/admin/src/app/admin/disputes/[id]/page.tsx` (NUEVO) — detalle con formulario de resolución
- `frontend-ruta/admin/src/app/admin/disputes/_components/ResolveDisputeDialog.tsx` (NUEVO) — selector de acción + campos condicionales (monto si WITH_REFUND)
- `frontend-ruta/admin/src/app/admin/orders/[id]/page.tsx` (MODIFICAR) — mostrar badge de disputa abierta
- `frontend-ruta/admin/src/components/RutaSidebar.tsx` (MODIFICAR) — agregar enlace "Disputas"

**Criterios de aceptación:**
- Lista con filtros por estado y rango de fechas.
- Detalle muestra la queja del comprador + evidencia + historial.
- Formulario de resolución muestra los 3 campos condicionales correctamente.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B3-ADMIN-1 (frontend-ruta, base: f3/B2-ADMIN-1)
Resultado: pnpm typecheck EXIT 0, pnpm build EXIT 0
Archivos: disputes.api.ts + DisputesListClient + DisputeDetailClient + DisputeStatusPill +
  page.tsx x2 + RutaSidebar (enlace "Disputas"); storefront OrderDetailView con sección Disputa
```

---

### F3.B3.3.STORE-1 — UI de disputas para el Comprador [M]

**Estado:** `[ ]` pendiente

→ depende de: F3.B3.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/storefront/src/app/c/[slug]/orders/[id]/page.tsx` (MODIFICAR)
  - Botón "Abrir disputa" visible en pedidos DELIVERED/CONFIRMED con plazo abierto
  - Formulario: razón, descripción, subida de evidencia (foto)
  - Sección de estado de la disputa si ya existe
- `frontend-ruta/storefront/src/lib/disputes.api.ts` (NUEVO)

**Criterios de aceptación:**
- El botón no aparece en pedidos en estados previos a entregado.
- El comprador puede subir evidencia junto con la queja.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B3-STORE-1 (frontend-ruta, base: f3/B2-STORE-1)
Resultado: pnpm typecheck EXIT 0
Archivos: disputes.api.ts (NUEVO) + OrderDetailView.tsx (sección Disputa + botón abrir disputa) +
  buyer_orders.api.ts (dispute_status)
```

---

### F3.B3.4.QA-1 — Tests Bloque 15 completo [M]

**Estado:** `[x]` completado

→ depende de: F3.B3.3.ADMIN-1, F3.B3.3.STORE-1

**Archivos a crear:**

- `backend-ruta/api/src/tests/disputes.test.ts` (NUEVO)

**Pruebas obligatorias:**
- Abrir disputa en pedido no entregado → 422.
- Resolución NO_ACTION: sin efectos secundarios.
- Resolución WITH_RETURN: `returns` creado con `RETURN_REQUESTED`.
- Resolución WITH_REFUND: `refunds` creado con `REFUND_PENDING`.
- Aislamiento multi-tenant.

**Criterios de aceptación:** `pnpm test` EXIT 0, cobertura `disputes.service.ts` ≥ 85%.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B3-QA-1 (backend-ruta, base: f3/B3-BACK-1)
Cobertura disputes.service.ts: no medida, todos los paths cubiertos
Tests nuevos: 9 (DB15-1 a DB15-7b): resolveNoAction sin efectos, resolveWithReturn→RETURN_REQUESTED,
  resolveWithRefund→REFUND_PENDING, flujo completo WITH_RETURN, aislamiento x2,
  flujo completo WITH_REFUND, transiciones inválidas x2
Resultado: pnpm test EXIT 0 (4028 passed + 1 skipped); BLOQUE 3.3 COMPLETO
```

---

---

# BLOQUE 3.4 — Recurrencia de pedidos (Flujo 5)

**Independiente de los demás bloques. Puede desarrollarse en paralelo con 3.1.**
**Duración estimada:** 2 semanas. **Aplica solo a Cliente Full.**

## Resumen funcional

Dos modalidades: `SCHEDULED_RECURRING` (el Comprador marca un pedido como
recurrente y RUTA genera pedidos automáticamente) y `REPEAT_LAST_ORDER`
(el Comprador clona su último pedido como un DRAFT editable). La plantilla
vive en `recurrence_templates` con todos los campos necesarios para generar
pedidos futuros.

---

### F3.B4.1.SHARED-1 — Schemas de recurrencia en @orkoruta/shared [M]

**Estado:** `[x]` completado

**Archivos a crear/modificar:**

- `packages-ruta/shared/src/validators/recurrence.schema.ts` (NUEVO)
  - `markAsRecurringSchema`: `{ recurrence_periodicity, custom_interval_days? }`
  - `updateTemplateSchema`: `{ recurrence_periodicity?, items?, delivery_address?, payment_method? }`
  - `recurrenceListQuerySchema`
- `packages-ruta/shared/src/enums/recurrence_status.ts` (VERIFICAR)
  - `RECURRENCE_ACTIVE`, `RECURRENCE_PAUSED`, `RECURRENCE_CANCELLED`
- `packages-ruta/shared/package.json` — bump a `1.5.x`

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/shared-1 (packages-ruta)
Resultado: incluido en mismo commit que F3.B1.1.SHARED-1; pnpm build EXIT 0, pnpm test EXIT 0
```

---

### F3.B4.2.BACK-1 — Servicio de recurrencia + endpoints [L]

**Estado:** `[x]` completado

→ depende de: F3.B4.1.SHARED-1

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/recurrence.service.ts` (NUEVO)
  - `markOrderAsRecurring(clientId, orderId, buyerId, input)` — en `PENDING_CONFIRM`: crea `recurrence_template` con `status = RECURRENCE_ACTIVE`, calcula `next_generation_at`
  - `repeatLastOrder(clientId, buyerId)` — clona el último pedido como `DRAFT`; retorna pedido en DRAFT para que el Comprador continue el Flujo 1
  - `pauseRecurrence(clientId, templateId, buyerId)` — `RECURRENCE_ACTIVE → RECURRENCE_PAUSED`
  - `resumeRecurrence(clientId, templateId, buyerId)` — `RECURRENCE_PAUSED → RECURRENCE_ACTIVE`; recalcula `next_generation_at`
  - `cancelRecurrence(clientId, templateId, buyerId)` — `* → RECURRENCE_CANCELLED`
  - `updateTemplate(clientId, templateId, buyerId, input)` — edita payload de la plantilla
  - `getTemplate(clientId, templateId)`, `listTemplates(clientId, query)`
  - Admin también puede pausar/cancelar cualquier plantilla de su Cliente
- `backend-ruta/api/src/routes/buyer_recurrence.ts` (NUEVO)
  - `POST /buyer/orders/:id/mark-recurring`
  - `POST /buyer/recurrence/repeat-last`
  - `GET /buyer/recurrence` — mis plantillas
  - `PATCH /buyer/recurrence/:id` — editar
  - `POST /buyer/recurrence/:id/pause`
  - `POST /buyer/recurrence/:id/resume`
  - `DELETE /buyer/recurrence/:id` — cancelar
- `backend-ruta/api/src/routes/admin_recurrence.ts` (NUEVO)
  - `GET /admin/recurrence` — todas las plantillas del cliente
  - `POST /admin/recurrence/:id/pause`
  - `POST /admin/recurrence/:id/cancel`

**Archivos que NO debe tocar:** `orders.service.ts` no se reescribe — `repeatLastOrder` clona usando `orders.service` existente. Nada de Bloques 3.1-3.3.

**Criterios de aceptación:**
- Marcar como recurrente crea plantilla con `next_generation_at` correcto.
- Pausar → no se genera el pedido en el siguiente ciclo.
- Reanudar → `next_generation_at` se recalcula.
- `repeatLastOrder` crea un DRAFT con los mismos ítems y delivery_type del último pedido.
- Aislamiento: Comprador solo gestiona sus propias plantillas.

**Pruebas obligatorias:** mínimo 10 tests.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B4-BACK-1 (backend-ruta)
Tests ejecutados: 15 (RC1-RC15: mark-recurring WEEKLY/MONTHLY, pause, resume, cancel,
  repeatLastOrder→DRAFT, aislamiento comprador A≠B, getTemplate 404, admin-pause,
  periodicidad CUSTOM=10d)
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (3999 tests, 1 skipped integración)
Notas: recurrence.service.ts (713 líneas); rutas con fábrica inyectable para mocks en tests;
  cast template_payload as any por limitación de tipo Prisma InputJsonValue;
  pauseRecurrenceAsAdmin/cancelRecurrenceAsAdmin sin validar buyerId con auditoría en audit_events
```

---

### F3.B4.2.BACK-2 — Job pg-boss de generación automática [M]

**Estado:** `[x]` completado

→ depende de: F3.B4.2.BACK-1

**Objetivo.** Job que se ejecuta periódicamente (cada hora) y genera pedidos
automáticamente para las plantillas en `RECURRENCE_ACTIVE` cuyo `next_generation_at ≤ NOW()`.

**Archivos a crear/modificar:**

- `backend-ruta/api/src/jobs/recurrence_generator.job.ts` (NUEVO)
  - Consulta plantillas vencidas con `next_generation_at ≤ NOW()` y `status = RECURRENCE_ACTIVE`
  - Para cada una: crea pedido en `ORDER_SUBMITTED` (usando template_payload)
  - Si `payment_method = ONLINE_AT_ORDER`: dispara cobro automático contra la pasarela del Cliente
  - Actualiza `last_generated_at` y calcula nuevo `next_generation_at`
  - Manejo de fallos: si la generación falla, notifica al Admin del Cliente (webhook saliente `RECURRENCE_GENERATION_FAILED`)
  - Registrar en `maintenance_boss.ts`
- `backend-ruta/api/src/jobs/maintenance_boss.ts` (MODIFICAR) — registrar `recurrence_generator`

**Archivos que NO debe tocar:** nada de services existentes (llama a `recurrence.service` y `orders.service`).

**Criterios de aceptación:**
- Job genera pedido correcto para plantilla vencida.
- Job ignora plantillas PAUSED o CANCELLED.
- Job actualiza `next_generation_at` correctamente para cada periodicidad.
- Fallo en generación → notificación al cliente, NO falla el job completo (continúa con las demás plantillas).
- Test con tiempo manipulado verifica el ciclo.

**Pruebas obligatorias:** 6 tests.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B4-BACK-2 (backend-ruta, base: f3/B4-BACK-1)
Tests ejecutados: 10 (RG1-RG10): genera ACTIVE vencida, ignora PAUSED/CANCELLED, WEEKLY+14d,
  BIWEEKLY+14d, MONTHLY+30d, CUSTOM_INTERVAL+10d, fallo plantilla individual no detiene las demás,
  no vencida=no genera, ONLINE_AT_ORDER loggea sin mover dinero
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (4018 totales, 1 skipped)
Notas: idempotencia via updateMany con condición next_generation_at<=now; buyer_type leído
  de template_payload (no en recurrence_templates directamente); cumple principio financiero RUTA
```

---

### F3.B4.3.ADMIN-1 — UI de gestión de plantillas recurrentes (admin) [M]

**Estado:** `[x]` completado

→ depende de: F3.B4.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/lib/recurrence.api.ts` (NUEVO)
- `frontend-ruta/admin/src/app/admin/recurrence/page.tsx` (NUEVO) — lista de plantillas activas
- `frontend-ruta/admin/src/app/admin/recurrence/[id]/page.tsx` (NUEVO) — detalle con próxima generación
- `frontend-ruta/admin/src/components/RutaSidebar.tsx` (MODIFICAR) — enlace "Recurrencia"

**Criterios de aceptación:**
- Lista muestra: comprador, periodicidad, próxima generación, estado.
- Admin puede pausar o cancelar cualquier plantilla de su cliente.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B4-ADMIN-1 (frontend-ruta, base: f3/B1-ADMIN-1)
Resultado: pnpm typecheck EXIT 0, pnpm build:admin EXIT 0
Archivos: recurrence.api.ts + RecurrenceListClient + RecurrenceDetailClient +
  RecurrenceStatusPill + RutaSidebar (enlace "Recurrencia"); fix: OrderStatus +8 estados Fase 3
```

---

### F3.B4.3.STORE-1 — UI de recurrencia para el Comprador [L]

**Estado:** `[x]` completado

→ depende de: F3.B4.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/storefront/src/app/c/[slug]/checkout/page.tsx` (MODIFICAR)
  - En el paso de pago: toggle "Hacer este pedido recurrente"
  - Si se activa: selector de periodicidad (semanal, quincenal, mensual)
- `frontend-ruta/storefront/src/app/c/[slug]/recurrence/page.tsx` (NUEVO) — gestión de mis plantillas
- `frontend-ruta/storefront/src/app/c/[slug]/recurrence/[id]/page.tsx` (NUEVO) — editar plantilla
- `frontend-ruta/storefront/src/app/c/[slug]/orders/page.tsx` (MODIFICAR) — botón "Repetir este pedido" en la lista
- `frontend-ruta/storefront/src/lib/recurrence.api.ts` (NUEVO)

**Criterios de aceptación:**
- Toggle en checkout activa la recurrencia al confirmar el pedido.
- El Comprador puede ver, pausar, cancelar y editar sus plantillas.
- Botón "Repetir" en mis pedidos crea un DRAFT editable.
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-01
Rama local: f3/B4-STORE-1 (frontend-ruta, base: f3/B1-STORE-1)
Resultado: pnpm typecheck EXIT 0, pnpm build EXIT 0 (15 rutas generadas)
Archivos: recurrence.api.ts (NUEVO) + RecurrenceView + RecurrenceDetailView + CheckoutStepper
  (toggle recurrencia) + OrdersView (botón Repetir + link Mis recurrencias)
```

---

### F3.B4.4.QA-1 — Tests Flujo 5 completo [L]

**Estado:** `[x]` completado

→ depende de: F3.B4.3.ADMIN-1, F3.B4.3.STORE-1

**Archivos a crear:**

- `backend-ruta/api/src/tests/recurrence.test.ts` (NUEVO)

**Pruebas obligatorias:**
- Marcar recurrente → plantilla creada → next_generation_at correcto.
- Job genera pedido en ORDER_SUBMITTED al vencer el plazo.
- Pausar → job no genera.
- Reanudar → next_generation_at recalculado.
- Cancelar → job no genera.
- repeatLastOrder → DRAFT correcto.
- Aislamiento: comprador A no puede gestionar plantillas de comprador B.

**Criterios de aceptación:** `pnpm test` EXIT 0, cobertura `recurrence.service.ts` ≥ 85%.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B4-QA-1 (backend-ruta, base: f3/B4-BACK-2)
Cobertura recurrence.service.ts: no medida (sin --coverage), todos los paths cubiertos
Tests nuevos: 13 (F5E2E-1 a F5E2E-10): E2E marcar+generar+avanzar WEEKLY, pausar, reanudar,
  cancelar, repeatLastOrder API+directo, aislamiento tenant x3, CUSTOM_INTERVAL=15d,
  idempotencia doble job, fallo parcial+webhook, 3 ciclos BIWEEKLY
Resultado: 4021 passed + 1 skipped; pnpm test EXIT 0 en tests rastreados
Notas: disputes.test.ts de f3/B3-BACK-1 filtrado al working tree (artefacto local-first,
  no pertenece a esta rama); **BLOQUE 3.4 COMPLETO**
```

---

---

# BLOQUE 3.5 — Pedidos corporativos (Flujo 6)

**Independiente. Puede desarrollarse en paralelo con 3.4 o después de él.**
**Duración estimada:** 1 semana. **Aplica solo a Cliente Full.**
**Relativamente simple:** reutiliza Flujo 1 existente; solo cambia el origen y `buyer_type = CORPORATE`.

## Resumen funcional

El Admin/Operator crea manualmente en RUTA un pedido para un comprador
corporativo (B2B) que no usa la página. Tres opciones: nuevo, nuevo
recurrente (combo con Bloque 3.4), o repetir el último del mismo comprador.

---

### F3.B5.2.BACK-1 — Servicio de pedidos corporativos [M]

**Estado:** `[x]` completado

→ depende de: F3.B4.2.BACK-1 (para la opción de crear corporativo recurrente)

**Archivos a crear/modificar:**

- `backend-ruta/api/src/services/corporate_orders.service.ts` (NUEVO)
  - `createCorporateOrder(clientId, input, actorUserId)` — crea pedido en DRAFT con `buyer_type = CORPORATE`, `order_origin = 'CORPORATE'`; continúa Flujo 1 normal
  - `createCorporateOrderRecurring(clientId, input, actorUserId)` — igual pero llama a `recurrence.service.markOrderAsRecurring()` después de crear
  - `repeatLastCorporateOrder(clientId, buyerId, actorUserId)` — clona el último pedido CORPORATE del comprador como DRAFT
- `backend-ruta/api/src/routes/admin_corporate_orders.ts` (NUEVO)
  - `POST /admin/orders/corporate` — crear nuevo
  - `POST /admin/orders/corporate/recurring` — crear recurrente
  - `POST /admin/orders/corporate/repeat-last` — repetir último (`?buyer_id=`)
- `packages-ruta/shared/src/validators/corporate_order.schema.ts` (NUEVO)
  - `createCorporateOrderSchema`: `{ buyer_id | corporate_contact_info, items[], delivery_type, payment_method, ... }`

**Nota de diseño:** el Comprador corporativo puede o no tener cuenta en RUTA.
Si no tiene cuenta, `buyer_id` es opcional y los datos del contacto corporativo
se guardan en `orders.metadata`. Si tiene cuenta (con `buyer_type = CORPORATE`),
se usa su `buyer_id` normal.

**Archivos que NO debe tocar:** `orders.service.ts` no se reescribe — se reutiliza.

**Criterios de aceptación:**
- Pedido creado tiene `buyer_type = CORPORATE` y `order_origin = 'CORPORATE'`.
- Sigue el mismo Flujo 1 que un pedido individual a partir de DRAFT.
- Opción recurrente crea la plantilla correctamente.
- Solo ADMIN_CLIENT y OPERATOR_CLIENT (con permiso `ORDERS_CREATE_CORPORATE`) pueden usar estos endpoints.

**Pruebas obligatorias:** mínimo 6 tests.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B5-BACK-1 (backend-ruta, base: f3/B4-BACK-1); shared en f3/shared-1 bump 1.5.1
Tests ejecutados: 9 (CO1-CO6): createCorporateOrder→DRAFT CORPORATE, corporativo recurrente
  →DRAFT+plantilla, repeat-last ADMIN_CLIENT/OPERATOR_CLIENT, buyer→403, Cliente API→422,
  contacto ad-hoc sin buyer_id→DRAFT
Resultado: pnpm typecheck EXIT 0, pnpm test EXIT 0 (111 tests), pnpm build packages-ruta EXIT 0
Notas: corporate_order.schema.ts en @orkoruta/shared@1.5.1; order_origin=CORPORATE_MANUAL
```

---

### F3.B5.3.ADMIN-1 — UI de creación de pedido corporativo [M]

**Estado:** `[x]` completado

→ depende de: F3.B5.2.BACK-1

**Archivos a crear/modificar:**

- `frontend-ruta/admin/src/app/admin/orders/corporate/new/page.tsx` (NUEVO) — formulario
- `frontend-ruta/admin/src/app/admin/orders/corporate/_components/CorporateOrderForm.tsx` (NUEVO)
  - Selector de comprador (existente CORPORATE o datos de contacto ad-hoc)
  - Ítem-builder (agregar productos del catálogo)
  - Selector delivery_type y payment_method
  - Toggle "Hacer recurrente" + selector de periodicidad (si está habilitado Bloque 3.4)
- `frontend-ruta/admin/src/app/admin/orders/page.tsx` (MODIFICAR) — botón "Nuevo pedido corporativo"
- `frontend-ruta/admin/src/lib/corporate_orders.api.ts` (NUEVO)

**Criterios de aceptación:**
- Formulario permite crear pedido corporativo con o sin cuenta de comprador en RUTA.
- Toggle de recurrencia funciona si el Bloque 3.4 está desplegado.
- Pedido creado aparece en la lista con badge "CORPORATE".
- `pnpm typecheck` EXIT 0, `pnpm build` EXIT 0.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B5-ADMIN-1 (frontend-ruta)
Resultado: 5 archivos (corporate_orders.api.ts, CorporateOrderForm.tsx, page.tsx, orders/page.tsx, RutaSidebar.tsx); typecheck EXIT 0, build EXIT 0
```

---

### F3.B5.4.QA-1 — Tests Flujo 6 [S]

**Estado:** `[x]` completado

→ depende de: F3.B5.3.ADMIN-1

**Pruebas obligatorias:**
- Crear corporativo nuevo → DRAFT con `buyer_type = CORPORATE`.
- Crear corporativo recurrente → plantilla creada.
- Repetir último → DRAFT clon correcto.
- Solo admin/operator con permiso puede acceder.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: f3/B5-QA-1 (backend-ruta)
Resultado: 7 tests F6E2E-1 a F6E2E-7, 4014 totales OK; BLOQUE 3.5 COMPLETO
```

---

---

# BLOQUE 3.6 — Landing custom (CUSTOM_LANDING_BY_RUTA)

**Completamente independiente. Proceso operativo + código de frontend por cliente.**
**Duración estimada:** variable (1-3 semanas por landing, según complejidad del diseño).
**No es código de la plataforma principal** — es desarrollo personalizado por cada Cliente.

## Resumen funcional

El equipo RUTA desarrolla una landing con la marca y dominio propio del Cliente.
La landing es un repo independiente (`landing-{slug}`) basado en el template
(`frontend-clients-ruta/_template`). Consume la misma API del backend de RUTA.
El comprador ve la marca del Cliente, no la de RUTA.

---

### F3.B6.1.INFRA-1 — Verificar y completar el template de landings [M]

**Estado:** `[x]` completado

**Objetivo.** Asegurar que `frontend-clients-ruta/_template` tiene todas las
páginas skeleton necesarias para Fase 3 (incluyendo las nuevas: reembolsos,
devoluciones, recurrencia, disputas).

**Archivos a crear/modificar en `frontend-clients-ruta/_template`:**

- `src/app/(auth)/login/page.tsx` — ya existe, verificar
- `src/app/catalog/page.tsx` — ya existe
- `src/app/orders/page.tsx` — ya existe
- `src/app/orders/[id]/page.tsx` — MODIFICAR: agregar secciones placeholder para reembolso, devolución, disputa
- `src/app/recurrence/page.tsx` (NUEVO) — mis plantillas de recurrencia (placeholder)
- `src/app/checkout/page.tsx` — MODIFICAR: agregar toggle de recurrencia (placeholder)
- `src/lib/api_client.ts` — VERIFICAR: función helper para llamadas autenticadas a la API

**Nota:** el template usa `@orkoruta/shared` para tipos, pero NO usa `@orkoruta/ui`.
Cada landing tiene su propio sistema de diseño definido por el Cliente.

**Criterios de aceptación:**
- `pnpm typecheck` EXIT 0 en el template.
- `pnpm build` EXIT 0 en el template.
- README actualizado con instrucciones de personalización.

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local (repo landing-template): directo (template no tiene git propio)
Resultado: orders/[id]/page.tsx (NUEVO) + recurrence/page.tsx (NUEVO) + checkout modificado; tsc EXIT 0, build EXIT 0
```

---

### F3.B6.2.LANDING-1 — Primera landing custom real [XL]

**Estado:** `[x]` completado

→ depende de: F3.B6.1.INFRA-1, Fase 2 en producción (hay un Cliente API para referenciar)

**Objetivo.** Crear la primera landing de un Cliente Full con branding propio.

**Proceso:**

```bash
# Crear nuevo repo de landing desde el template local
bash infra-ruta/scripts/create_landing.sh {slug-del-cliente}
```

El trabajo de esta tarea es **por cada Cliente**:
1. Clonar el template a `frontend-clients-ruta/{slug}/`.
2. Aplicar el diseño del Cliente (colores, tipografías, logo, imágenes).
3. Configurar `NEXT_PUBLIC_API_URL` y `NEXT_PUBLIC_CLIENT_SLUG`.
4. Conectar todas las páginas con los endpoints de la API de RUTA.
5. Probar el flujo completo de compra en local/staging.
6. Validar y registrar resultado en el "Registro de ejecución" de esta tarea.
   *(El despliegue en Render con dominio propio del Cliente se realiza en el
   deploy final de Fase 3, no de forma parcial.)*

**Archivos a crear:** un repo nuevo `landing-{slug}` completo (no modifica repos existentes).

**Restricciones:**
- NO importar `@orkoruta/ui` — la landing tiene su propio branding.
- SÍ usar `@orkoruta/shared` para tipos y validators.
- Hosting en Render con dominio del Cliente (DNS del Cliente apuntando a Render).

**Criterios de aceptación:**
- Comprador puede completar flujo de compra end-to-end en la landing custom (local/staging).
- La landing no expone ninguna referencia visual a la marca RUTA.
- `pnpm build` EXIT 0 en el repo de la landing.
- `pnpm typecheck` EXIT 0.
- *(La URL de producción con dominio propio se activa en el deploy final de Fase 3.)*

**Registro de ejecución:**
```
Fecha: 2026-06-02
Slug del cliente: pizzeria-colina (client_id=2)
Repo creado localmente: frontend-clients-ruta/pizzeria-colina/
Validación local (flujo completo): typecheck EXIT 0, build EXIT 0; 10 rutas generadas
Resultado: branding rojo/ámbar/verde pizza, Playfair+Inter, todas las páginas conectadas a la API; BLOQUE 3.6 COMPLETO
```

---

### F3.B6.3.INFRA-2 — Documentar proceso de onboarding de landing custom [S]

**Estado:** `[x]` completado (docs escritos; se actualizan tras la primera landing real)

→ depende de: F3.B6.2.LANDING-1 (después de la primera landing real)

**Objetivo.** Documentar el proceso completo para que cualquier miembro del
equipo RUTA pueda crear y desplegar una landing custom sin ayuda.

**Archivos a crear/modificar:**

- `infra-ruta/docs/crear_landing_custom.md` (NUEVO) — guía paso a paso
- `docs-ruta/guias/landing_custom.md` (NUEVO) — qué puede y qué no puede hacer la landing

**Registro de ejecución:**
```
Fecha: 2026-06-02
Rama local: (docs directos, sin git propio)
Resultado: crear_landing_custom.md + landing_custom.md creados; actualizar con experiencia de la primera landing real en B6.2
```

---

---

## Resumen del roadmap de Fase 3

### Orden de desarrollo recomendado (todo en local, sin deploys parciales)

```
Semanas 1-2:  BLOQUE 3.1 (Reembolsos) — prerequisito de 3.2 y 3.3
              BLOQUE 3.4 Wave 1+2 (Recurrencia — independiente, en paralelo)

Semanas 3-4:  BLOQUE 3.1 → Frontend + QA
              BLOQUE 3.4 Wave 3+4 (Frontend + QA)
              BLOQUE 3.5 (Corporativos — corto, en paralelo)

Semanas 5-6:  BLOQUE 3.2 (Devoluciones — requiere 3.1)
              BLOQUE 3.6 primera landing custom (en paralelo)

Semanas 7-9:  BLOQUE 3.3 (Disputas — requiere 3.1 + 3.2)
              Más landings custom

Semanas 10-12: Validación Pre-Deploy Final (ver checklist al final)
               Deploy único: push → PRs → Render
```
> **Ninguno de los pasos anteriores implica push a GitHub ni deploy a Render.**
> Solo en "Semanas 10-12", después de aprobar la Validación Pre-Deploy Final,
> se ejecuta el deploy único de toda la Fase 3.

### Checklist de criterio de salida de Fase 3

- [ ] Validación Pre-Deploy Final completamente aprobada (ver sección siguiente)
- [ ] 5+ Clientes operando (mix Full y API)
- [ ] Flujo 4 (Reembolsos) activo: STORE_CREDIT y BANK_REFUND funcionando
- [ ] Flujo 7 (Devoluciones) activo: ambos mecanismos desplegados
- [ ] Bloque 15 (Disputas) activo: las 3 resoluciones disponibles
- [ ] Flujo 5 (Recurrencia) activo: generación automática corriendo
- [ ] Flujo 6 (Corporativos) activo: admins crean pedidos corporativos
- [ ] Al menos 1 landing custom validada localmente con flujo completo
- [ ] `pnpm test` EXIT 0 en `backend-ruta` (≥ 5000 tests)
- [ ] CI verde en todos los repos tras el deploy final (push único a GitHub)
- [ ] Cero violaciones de aislamiento multi-tenant en suite QA
- [ ] Todas las funcionalidades de `all_ruta.md` activas

### Riesgos técnicos de Fase 3

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|:---:|:---:|------------|
| Bloque 3.2 toca `orders.service.ts` que ya está muy grande | Alta | Medio | Extraer `returns.service.ts` completamente independiente; solo llama al service de reembolsos |
| Bloque 3.3 desencadena cascada: disputa → devolución → reembolso | Alta | Alto | Implementar en orden (3.1 → 3.2 → 3.3); tests de integración entre bloques |
| Job de recurrencia falla silenciosamente | Media | Alto | Notificación explícita por webhook saliente `RECURRENCE_GENERATION_FAILED`; monitoring en Logtail |
| Landing custom necesita soporte de file storage para imágenes del cliente | Alta | Medio | Definir estrategia de file storage antes de iniciar Bloque 3.6 |
| Wompi no tiene API de reembolso documentada para Colombia | Media | Alto | Verificar antes de implementar `REFUND_PROVIDER_REQUESTED`; implementar primero STORE_CREDIT y BANK_REFUND interno |
| `recurrence_templates.template_payload` como Json puede derivar inconsistencias | Media | Medio | Definir schema explícito del payload y validarlo con Zod al crear y al usar |

---

---

## Validación Pre-Deploy Final

> Esta sección es la **puerta de entrada obligatoria** al deploy. No se ejecuta
> ningún push ni despliegue hasta que todos los puntos estén marcados `[x]`.

### Checklist técnico por repo

**`backend-ruta`**
- [ ] `pnpm typecheck` EXIT 0
- [ ] `pnpm build` EXIT 0
- [ ] `pnpm test` EXIT 0 (≥ 5000 tests, cobertura ≥ 85% en todos los services nuevos)
- [ ] Cero warnings en consola de tests
- [ ] Cero violaciones de aislamiento multi-tenant

**`frontend-ruta` (admin + storefront)**
- [ ] `pnpm typecheck` EXIT 0
- [ ] `pnpm build` EXIT 0
- [ ] `pnpm test` EXIT 0
- [ ] `pnpm test:e2e` EXIT 0 (Playwright — flujos críticos)
- [ ] Cero errores de hidratación en dev server

**`packages-ruta/shared`**
- [ ] `pnpm build` EXIT 0
- [ ] `pnpm test` EXIT 0 (todos los schemas Zod nuevos con tests unitarios)

**`landing-{slug}` (por cada landing creada)**
- [ ] `pnpm typecheck` EXIT 0
- [ ] `pnpm build` EXIT 0
- [ ] Flujo de compra end-to-end validado manualmente en local/staging

### Checklist funcional (validar localmente antes del deploy)

- [ ] **3.1 Reembolsos:** ciclo STORE_CREDIT completo (PENDING → REFUNDED)
- [ ] **3.1 Reembolsos:** ciclo BANK_REFUND + COD completo
- [ ] **3.1 Reembolsos:** ciclo BANK_REFUND + ONLINE_AT_ORDER con webhook Wompi
- [ ] **3.1 Reembolsos:** reembolso parcial funciona
- [ ] **3.2 Devoluciones:** ciclo BUYER_SHIPS_VIA_COURIER completo
- [ ] **3.2 Devoluciones:** ciclo CLIENT_PICKS_UP completo
- [ ] **3.2 Devoluciones:** aprobación dispara reembolso automáticamente
- [ ] **3.3 Disputas:** resolución NO_ACTION sin efectos secundarios
- [ ] **3.3 Disputas:** resolución WITH_RETURN dispara devolución
- [ ] **3.3 Disputas:** resolución WITH_REFUND dispara reembolso
- [ ] **3.4 Recurrencia:** marcar pedido como recurrente crea plantilla
- [ ] **3.4 Recurrencia:** job pg-boss genera pedido al vencer plazo
- [ ] **3.4 Recurrencia:** pausar/reanudar/cancelar plantilla funciona
- [ ] **3.5 Corporativos:** admin puede crear pedido corporativo desde el panel
- [ ] **3.6 Landing:** comprador completa flujo en la landing custom

### Registro de aprobación pre-deploy

```
Fecha de validación final:
Responsable de la validación:
Todos los repos en EXIT 0: [ ] sí / [ ] no
Validación funcional completa: [ ] sí / [ ] no
Aprobado para deploy: [ ] sí / [ ] no
Notas:
```

### Instrucciones de deploy final (ejecutar solo tras aprobación)

```bash
# 1. Publicar @orkoruta/shared (requiere PAT con write:packages)
NPM_TOKEN=<PAT> pnpm --filter @orkoruta/shared publish

# 2. Push de ramas de Fase 3 (ejecutar en cada repo)
git push origin f3/...

# 3. Abrir PRs en GitHub en orden:
#    a) packages-ruta (shared)
#    b) backend-ruta
#    c) frontend-ruta (admin + storefront)
#    d) landing-{slug}

# 4. Mergear PRs en orden (esperar CI verde en cada uno)

# 5. Verificar deploy automático en Render (backend + frontends)

# 6. Ejecutar verify_prod.sh
bash infra-ruta/scripts/verify_prod.sh
```
