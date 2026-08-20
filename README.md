# RUTA — Workspace del proyecto

Plataforma SaaS **multi-tenant** para administrar operaciones comerciales entre
**Clientes** (negocios) y **Compradores**: catálogo, pedidos, entregas (SHIP /
PICKUP), pagos, reembolsos, devoluciones, disputas, recurrencia y pedidos
corporativos. Mercado Colombia, moneda COP. UI en español, código en inglés.

> Esta carpeta (`ruta/`) **no es un repositorio**: es el workspace local que
> agrupa 5 repos Git independientes + una carpeta con las landings de clientes.

---

## 1. Índice rápido

| Quiero… | Ir a |
|---|---|
| Entender qué hay en cada carpeta | [§2 Estructura](#2-estructura-del-workspace) |
| Ver qué está construido | [§4 Qué se ha hecho](#4-qué-se-ha-hecho) |
| Levantar todo en local | [§6 Cómo correr el proyecto](#6-cómo-correr-el-proyecto) |
| Correr tests | [§7 Tests](#7-tests-y-calidad) |
| Desplegar | [§8 Despliegue](#8-despliegue-render) |
| Reglas que no puedo romper | [§9 Reglas no negociables](#9-reglas-no-negociables) |
| La documentación funcional | [§10 Mapa de documentación](#10-mapa-de-documentación) |

---

## 2. Estructura del workspace

```
ruta/                              ← carpeta local (NO es un repo)
├── backend-ruta/                  ← repo: backend-ruta   (Express + TypeScript)
│   └── api/                       ← la API (@ruta/api)
├── frontend-ruta/                 ← repo: frontend-ruta  (pnpm workspace)
│   ├── admin/                     ← Next.js panel administrativo (@ruta/admin)
│   ├── storefront/                ← Next.js tienda nativa    (@ruta/storefront)
│   ├── packages/ui/               ← design system @orkoruta/ui (NO se publica)
│   └── tests/                     ← E2E Playwright
├── packages-ruta/                 ← repo: packages-ruta  (paquetes publicables)
│   ├── shared/                    ← @orkoruta/shared — tipos, enums, validators Zod
│   └── db/                        ← @orkoruta/db — Prisma client + withTenant()
├── frontend-clients-ruta/         ← carpeta local (NO es un repo)
│   ├── _template/                 ← repo: landing-template
│   └── landignpage-avocadodelivery/ ← landing custom de cliente
├── docs-ruta/                     ← repo: docs-ruta — documentación viva del producto
├── infra-ruta/                    ← repo: infra-ruta — scripts, render.yaml, runbooks
└── README_RUTA_PLAN_11_SEMANAS.md ← plan de trabajo de 2 devs / 11 semanas
```

Detalle ampliado en `docs-ruta/estructura_proyecto.md`.

---

## 3. Stack técnico

| Capa | Tecnología |
|---|---|
| Backend | Express 5 + TypeScript (ESM), Node 22 |
| Frontends | Next.js (App Router) + Tailwind |
| ORM / BD | Prisma sobre PostgreSQL (schema SQL como fuente de verdad) |
| Auth | `jose` (JWT) + `argon2`, cookies HttpOnly, refresh tokens |
| Jobs | `pg-boss` (worker separado) |
| Validación | Zod, compartida vía `@orkoruta/shared` |
| Pasarela de pago | Wompi (online) + contra entrega (COD) |
| Mapas | OpenStreetMap + Leaflet |
| Logging | `pino` + `@logtail/pino` (Better Stack en prod), `trace_id` por request |
| Tests | Vitest + Supertest (backend), Playwright + MSW (frontend) |
| Paquetes | GitHub Packages, org `orkoruta` |
| Hosting | Render |
| Gestor | pnpm 11.4.0 (workspaces por repo) |

### Modelo de negocio en dos sabores de Cliente

- **Cliente API** (`client_type = 'API'`): tiene su propia plataforma; RUTA solo
  hace la logística. No tiene reembolsos, devoluciones, disputas, recurrencia ni
  pedidos corporativos (se rechazan con `422 LOGISTICS_ONLY_FEATURE_UNAVAILABLE`).
- **Cliente Full** (`client_type = 'FULL'`): RUTA provee todo.
  - `NATIVE_RUTA` → usa el storefront genérico (`/c/{slug}`).
  - `CUSTOM_LANDING_BY_RUTA` → landing propio en `frontend-clients-ruta/`.

---

## 4. Qué se ha hecho

### Fases 1 y 2 (MVP) — **completas y desplegadas**

Sprints 0–6 cerrados. Lo que existe y funciona:

- **Auth y sesiones**: JWT `jose` + `argon2`, refresh tokens, cookies HttpOnly,
  5 roles (`ADMIN_RUTA`, `ADMIN_CLIENT`, `OPERATOR_CLIENT`, `COURIER`, `BUYER`).
- **Multi-tenant**: `client_id` en toda tabla operativa, RLS activo,
  particionamiento LIST por cliente, helper `withTenant(clientId, role, fn)`.
- **Catálogo**: productos, categorías, importación masiva desde Excel.
- **Pedidos**: state machine con 20+ estados (`services/orders/state_machine.ts`),
  validación, aceptación, historial append-only.
- **Flujo SHIP**: asignación de repartidor con mapa (Leaflet+OSM), entrega,
  cobro COD, cancelación post-despacho, return-to-origin, auto-confirmación.
- **Flujo PICKUP**: puntos físicos, verificación de identidad, cobro, entrega.
- **Pagos**: Wompi (online) + COD, webhook con firma HMAC.
- **Vista de Control**: `ADMIN_RUTA` impersona a `ADMIN_CLIENT` con master
  password; todo auditado con `acting_via_control_view = TRUE`.
- **Dashboards y métricas** para `ADMIN_CLIENT` y `ADMIN_RUTA`.
- **Configuración del cliente**: 4 tabs (info, Wompi, webhooks salientes, parámetros).
- **Auditoría**: log completo por cliente (`audit_events`, append-only).
- **Webhooks salientes**: cola pg-boss con reintentos (1m, 5m, 15m, 60m, 4h).
- **API pública para Clientes API**: API keys + endpoints de pedidos.
- **Observabilidad**: pino JSON → Better Stack, `trace_id`/`requestId`/`client_id`/
  `user_id` en cada request.
- **Backups**: scripts de backup/restore en `infra-ruta/scripts/`.

### Fase 3 (funciones avanzadas) — **implementada, pendiente de deploy final**

Seis bloques, casi todos cerrados (ver tabla de avance en `docs-ruta/plan_fase3.md`):

| Bloque | Alcance | Estado |
|---|---|---|
| 3.1 Reembolsos | STORE_CREDIT / BANK_REFUND, parciales, webhook Wompi | ✅ |
| 3.2 Devoluciones | `BUYER_SHIPS_VIA_COURIER` y `CLIENT_PICKS_UP`, dispara reembolso | ✅ (falta `F3.B2.2.BACK-2`) |
| 3.3 Disputas | resolución `NO_ACTION` / `WITH_RETURN` / `WITH_REFUND` | ✅ |
| 3.4 Recurrencia | plantillas + job pg-boss generador, pausar/reanudar/cancelar | ✅ |
| 3.5 Pedidos corporativos | `buyer_type` corporativo, alta desde el panel | ✅ |
| 3.6 Landing custom | template + landing `avocado-delivery` + guía de creación | ✅ |

> ⚠️ **Régimen local-first de Fase 3 (vigente).** El *código de funcionalidades*
> de Fase 3 vive en ramas locales: está prohibido pushear, abrir PRs, publicar
> bumps de `@orkoruta/shared` o desplegar en Render hasta que la Validación
> Pre-Deploy Final esté aprobada. Docs, `CLAUDE.md`, `AGENTS.md` y plan files sí
> se pushean con normalidad. Checklist completo al final de `docs-ruta/plan_fase3.md`.
>
> Consecuencia práctica: los `pnpm-workspace.yaml` de `backend-ruta` y
> `frontend-ruta` tienen `overrides` que enlazan `@orkoruta/shared` y
> `@orkoruta/db` a `link:../packages-ruta/*` en vez de a GitHub Packages.

### Pendiente (acción humana)

- Validación Pre-Deploy Final de Fase 3 + deploy único (`docs-ruta/plan_fase3.md`).
- Ejecución del plan de pruebas completo (`docs-ruta/plan_pruebas_completo.md`).
- Onboarding del cliente piloto (`infra-ruta/docs/deploy_produccion.md`).

---

## 5. Mapa de la aplicación

### Backend — `backend-ruta/api/src/`

```
app.ts / index.ts          arranque HTTP;  index.worker.ts → worker pg-boss
config/env.ts              carga y valida variables de entorno
middleware/                auth, api_key_auth, control_view, idempotency, logger
routes/                    35 routers HTTP (ver abajo)
services/                  lógica de negocio (handlers delgados)
services/orders/           state machine de pedidos
jobs/                      12 jobs pg-boss
lib/                       errors, logger, token, password, parameter, wompi_client
tests/ + __tests__/        Vitest + Supertest
```

Routers por audiencia:

- **Staff de cliente** (`admin_*`): products, categories, products_bulk, orders,
  order_assignment, buyers, couriers, pickup_points, pickup_ops, parameters,
  audit, metrics, webhooks, api_keys, refunds, returns, disputes, recurrence,
  corporate_orders.
- **RUTA** (`ruta_admin_*`): clients, control_view, metrics.
- **Comprador** (`buyer_*`): orders, payment, returns, disputes, recurrence.
- **Repartidor** (`courier_*`): orders, collection.
- **Público / integración**: `public_catalog`, `api_client_orders`, `webhooks`
  (entrantes Wompi), `uploads`, `openapi`, `healthz`.

Jobs pg-boss: `validate_order`, `order_expiration`, `payment_timeout`,
`pickup_expiration`, `at_pickup_expiration`, `auto_confirm_delivered`,
`recurrence_generator`, `webhook_sender`, `bulk_import`, `cleanup_idempotency`,
`cleanup_sessions`, `maintenance_boss`.

### Admin — `frontend-ruta/admin/src/app/`

`/login` y, dentro de `(protected)`:

- `admin/`: dashboard, orders (+ `[id]`, `map`, `corporate`), products, buyers,
  couriers, pickup-points, refunds, returns, disputes, recurrence, api-keys,
  audit, map, settings.
- `ruta-admin/`: clients (+ `new`, `[id]`), dashboard, control-view.
- `courier/`: vista móvil-first del repartidor (+ `[id]`).

### Storefront — `frontend-ruta/storefront/src/app/`

Todo cuelga de `/c/[slug]` (slug del Cliente): catálogo, `product/[id]`, `cart`,
`checkout` (+ `confirmation`), `orders` (+ `[id]`), `recurrence` (+ `[id]`),
`(auth)/login` y `(auth)/register`.

### Paquetes compartidos — `packages-ruta/`

- `@orkoruta/shared` — types, enums, validators Zod, constants (`error_codes`).
- `@orkoruta/db` — cliente Prisma generado + `withTenant()` (aplica
  `SET LOCAL app.current_client_id`).

---

## 6. Cómo correr el proyecto

### 6.1 Requisitos

- Node **22** y **pnpm 11.4.0** (`corepack enable` o `npm i -g pnpm@11.4.0`).
- `psql` en el PATH (solo para seeds/migraciones).
- Acceso a GitHub Packages para `@orkoruta/*` **si no usas los overrides locales**:

  ```bash
  echo "@orkoruta:registry=https://npm.pkg.github.com" >> ~/.npmrc
  echo "//npm.pkg.github.com/:_authToken=<PAT>" >> ~/.npmrc
  ```

### 6.2 Base de datos

> ℹ️ En este workspace el backend local apunta a la **BD remota compartida** del
> proyecto (definida en `backend-ruta/.env`), no a un Postgres local. Ten cuidado
> con los datos que creas o borras.

Si necesitas montar una BD desde cero, la fuente de verdad del esquema es
`docs-ruta/bd/ruta_postgres.sql` (no `prisma migrate`):

```bash
psql "$DATABASE_URL" -f docs-ruta/bd/ruta_postgres.sql
bash infra-ruta/scripts/seed_dev_data.sh      # cliente piloto + usuarios + catálogo
bash infra-ruta/scripts/create_first_admin_ruta.sh
```

### 6.3 Variables de entorno

`backend-ruta/.env` (plantilla en `.env.example`):

```
NODE_ENV, PORT (3001), HOST
DATABASE_URL
JWT_SECRET, JWT_ACCESS_TOKEN_LIFETIME_MINUTES, JWT_REFRESH_TOKEN_LIFETIME_DAYS
WOMPI_PUBLIC_KEY, WOMPI_PRIVATE_KEY, WOMPI_WEBHOOK_SECRET
LOGTAIL_TOKEN            (solo prod)
CORS_ORIGINS             dev: http://localhost:3002,http://localhost:3003
NPM_TOKEN                PAT de GitHub Packages
```

`frontend-ruta/admin/.env.local` y `frontend-ruta/storefront/.env.local`:

```
NEXT_PUBLIC_API_URL=http://localhost:3001
```

### 6.4 Instalar y construir (en orden)

```bash
# 1) Paquetes compartidos — SIEMPRE primero (genera Prisma client)
cd packages-ruta && pnpm install && pnpm build

# 2) Backend
cd ../backend-ruta && pnpm install

# 3) Frontends
cd ../frontend-ruta && pnpm install
```

### 6.5 Levantar

| Servicio | Comando | Puerto |
|---|---|---|
| API | `cd backend-ruta && pnpm dev` | 3001 |
| Worker de jobs | `cd backend-ruta/api && pnpm tsx watch src/index.worker.ts` | — |
| Admin | `cd frontend-ruta && pnpm dev:admin` | 3002 |
| Storefront | `cd frontend-ruta && pnpm dev:storefront` | 3003 |
| Ambos frontends | `cd frontend-ruta && pnpm dev` | 3002 + 3003 |
| Landing custom | `cd frontend-clients-ruta/<landing> && pnpm dev` | 3000 |

Comprobación rápida: `curl http://localhost:3001/healthz`.

---

## 7. Tests y calidad

```bash
# Backend — Vitest + Supertest (unit + integración contra BD real)
cd backend-ruta && pnpm test
cd backend-ruta && pnpm typecheck && pnpm lint

# Paquetes compartidos
cd packages-ruta && pnpm ci        # generate + typecheck + test + build

# Frontends
cd frontend-ruta && pnpm typecheck && pnpm lint
cd frontend-ruta && pnpm build
cd frontend-ruta && pnpm test:e2e  # Playwright
```

Estado de referencia del MVP: ~3.900 tests de backend y 14 E2E de Playwright.
La filosofía y la pirámide de testing están en `docs-ruta/estrategia_testing.md`;
los casos manuales (PT-001 … PT-320) en `docs-ruta/plan_pruebas_completo.md`.

---

## 8. Despliegue (Render)

| Servicio Render | Repo | Sub-path |
|---|---|---|
| `ruta-api` | `backend-ruta` | `api/` |
| `ruta-api-worker` | `backend-ruta` | `api/` (background worker) |
| `ruta-admin` | `frontend-ruta` | `admin/` |
| `ruta-storefront` | `frontend-ruta` | `storefront/` |
| `landing-{slug}` | `landing-{slug}` | raíz, dominio propio del Cliente |

URLs actuales de producción (según el plan de pruebas):
API `https://ruta-orko-api.onrender.com`, Admin `https://ruta-admin.onrender.com`,
Storefront `https://ruta-by-orko.onrender.com`.

Scripts de operación (desde la raíz del workspace):

```bash
bash infra-ruta/scripts/setup_workspace.sh    # clona los repos base
bash infra-ruta/scripts/seed_dev_data.sh      # datos de desarrollo
bash infra-ruta/scripts/create_landing.sh <slug>
bash infra-ruta/scripts/migrate_prod.sh       # aplica el SQL en prod
bash infra-ruta/scripts/backup_db.sh          # backup manual
bash infra-ruta/scripts/restore_db.sh
bash infra-ruta/scripts/verify_prod.sh        # verifica estado de la BD de prod
```

Runbooks: `infra-ruta/docs/deploy_produccion.md`, `backups.md`,
`backup_restore.md`, `observability_setup.md`, `crear_landing_custom.md`.

Publicar paquetes (manual, con PAT; el CI no tiene permisos suficientes):

```bash
NPM_TOKEN=<PAT> pnpm --filter @orkoruta/shared publish
NPM_TOKEN=<PAT> pnpm --filter @orkoruta/db publish
```

---

## 9. Reglas no negociables

Resumen del manifiesto (`docs-ruta/CLAUDE.md`, sección 4 y 7):

1. **RUTA no custodia, no transfiere, no acredita dinero.** Los pagos van a
   cuentas del Cliente; los reembolsos los ejecuta el Cliente. RUTA solo registra
   estados.
2. **Aislamiento multi-tenant**: `client_id BIGINT NOT NULL` en toda tabla
   operativa; toda query con `SET LOCAL app.current_client_id`; RLS activo; nunca
   bypassear RLS.
3. **Identificadores BIGINT**, nunca UUID. Las URLs públicas usan slug, no IDs.
4. **Append-only**: `audit_events`, `order_state_history`,
   `external_webhook_events`, `webhook_deliveries` — sin UPDATE ni DELETE.
5. **Idempotencia**: `X-Idempotency-Key` obligatorio en mutaciones, TTL 24 h.
6. **Particionamiento** LIST por `client_id`; toda tabla operativa nueva se suma
   a `create_client_partitions()`.
7. **State machine**: los cambios de estado de pedido pasan *solo* por
   `services/orders/state_machine.ts`.
8. **Parámetros de negocio** vía `client_parameters` + `getParameter()`; nunca
   hardcodear plazos.
9. **Observabilidad**: nunca `console.log`; usar `logger` de `lib/logger.ts`.
10. **Auth propia** con `jose` + `argon2`; tokens nunca en `localStorage`.
11. **Design system**: `@orkoruta/ui` solo en admin/storefront; las landings
    custom tienen branding propio y no lo importan; `@orkoruta/ui` no se publica.
12. **Naming**: código en inglés, UI/docs en español; services y routes en
    `snake_case`, tipos en `PascalCase`, constantes en `SCREAMING_SNAKE_CASE`.
13. **Nunca commitear secretos.**

Errores: response uniforme `{ code, message, details? }`; tipos en
`api/src/lib/errors.ts`, códigos en `@orkoruta/shared/constants/error_codes.ts`.
Críticos: `AUTHENTICATION_REQUIRED` (401), `FORBIDDEN` (403),
`INVALID_STATE_TRANSITION` (422), `LOGISTICS_ONLY_FEATURE_UNAVAILABLE` (422),
`IDEMPOTENCY_CONFLICT` (409), `OPTIMISTIC_LOCK_FAILED` (409),
`TENANT_ISOLATION_VIOLATION` (500 — bug).

---

## 10. Mapa de documentación

Toda la documentación viva está en `docs-ruta/`:

| Tema | Documento |
|---|---|
| Manifiesto del proyecto (léelo primero) | `CLAUDE.md` / `AGENTS.md` |
| Especificación funcional completa | `all_ruta.md` |
| Alcance del MVP por fases | `mvp_alcance.md` |
| Estructura multi-repo | `estructura_proyecto.md` |
| Arquitectura multi-tenant | `arquitectura/estrategia_multi_tenant_ruta.md` |
| Modelo de datos (fuente de verdad) | `bd/ruta_postgres.sql`, `bd/ruta_oracle.sql`, `bd/ruta_DER` |
| Contrato de API HTTP | `contrato_api.md` |
| Flujos de pedido 1–7 + estados | `flujos/flujo_*.txt` |
| Matriz de permisos por rol | `matriz_permisos.md` |
| Parámetros de negocio | `parametros_negocio.md` |
| Wireframes | `wireframes_mvp.md` |
| Galería de estilos | `diseno/galeria_estilos_ruta.md` |
| Ciclo de vida del token | `seguridad/ciclo_vida_token.txt` |
| Estrategia de testing | `estrategia_testing.md` |
| Plan de pruebas manual (PT-001…PT-320) | `plan_pruebas_completo.md`, `plan_pruebas_mvp.md` |
| Planes de fase y sprints | `plan_tareas.md`, `plan_fase2.md`, `plan_fase3.md`, `plan_paralelismo*.md` |
| Estado por sprint | `project_sprint2_status.md`, `project_sprint3_status.md` |
| Guías por rol | `guias/{admin_client,operator_client,buyer,courier,landing_custom}.md` |
| Memoria del proyecto | `memoria_proyecto_ruta.md`, `MEMORY.md` |

Plan de trabajo del equipo (2 devs / 11 semanas, división por módulos
funcionales y no por tecnología): `README_RUTA_PLAN_11_SEMANAS.md`.

En la raíz del workspace hay además dos listas vivas, separadas a propósito:

| Archivo | Qué contiene |
|---|---|
| `COSAS POR HACER MAS ADELANTE.md` | Funcionalidades decididas y pendientes, con su bloqueante y paso a paso |
| `DEUDAS TECNICAS.md` | Atajos conscientes y problemas conocidos del código, con cómo se comprobó cada uno |

---

## 11. Glosario

- **Cliente** = tenant (negocio). **Cliente plataforma** = `client_id = 0`.
- **Comprador / BUYER** = consumidor final.
- **Repartidor / COURIER** = quien entrega.
- **ADMIN_RUTA / ADMIN_CLIENT / OPERATOR_CLIENT** = roles de staff.
- **SHIP / PICKUP** = tipo de entrega. **COD** = pago contra entrega.
- **OWN_FLEET / EXTERNAL_COURIER** = quién despacha.
- **NATIVE_RUTA / CUSTOM_LANDING_BY_RUTA** = modalidad de frontend del Cliente Full.
- **Vista de Control** = impersonación auditada de ADMIN_RUTA sobre un Cliente.
