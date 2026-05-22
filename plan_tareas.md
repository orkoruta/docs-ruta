# RUTA — Plan de Tareas para Desarrollo

Plan operativo para construir el **MVP Fase 1** (Cliente Full + frontend
+ checkout). Alineado al cronograma de 10 semanas definido en
`docs/mvp_alcance.md` y a la estructura multi-repo de
`docs/estructura_proyecto.md`.

---

## Convenciones del plan

### Notación de tareas

`[SPRINT].[TRACK]-[N] — Título`

- **SPRINT:** 0 a 6.
- **TRACK:**
  - `INFRA` — infraestructura, deploy, scripts.
  - `BACK` — backend en `ruta-backend`.
  - `ADMIN` — admin frontend en `ruta-frontend/admin/`.
  - `STORE` — storefront en `ruta-frontend/storefront/`.
  - `SHARED` — paquetes en `ruta-shared`.
  - `LANDING` — `landing-template` y futuros `landing-{slug}`.
  - `DOCS` — `ruta-docs`.
  - `QA` — testing, observabilidad, hardening.

### Estimaciones

- **S** = Small (1-4h)
- **M** = Medium (4h-1d)
- **L** = Large (1-3d)
- **XL** = Extra Large (3-5d)

### Estados

`[ ]` pendiente · `[/]` en progreso · `[x]` completado · `[-]` cancelado

### Dependencias

`→ depende de: A.B-N, X.Y-M`

---

# Sprint 0 — Setup multi-repo (1 semana)

**Meta:** los 6 repos creados en GitHub, GitHub Packages configurado,
infraestructura desplegada, primer ADMIN_RUTA logueado, las 3 apps
levantándose en local consumiendo `@ruta/shared` y `@ruta/db`.

## 0.INFRA-1 — Crear los 6 repos en GitHub [S]

[ ] Crear repos privados en GitHub:
    - `ruta-backend`
    - `ruta-frontend`
    - `ruta-shared`
    - `ruta-docs`
    - `ruta-infra`
    - `landing-template` (marcado como Template Repository en
      Settings → Template repository ✓)
[ ] Configurar branch protection en `main` de los 5 base
    (PR review + status checks obligatorios).
[ ] Configurar `landing-template` igual.
[ ] Documentar URLs de los repos en gestor de secretos del equipo.

**Criterio:** los 6 repos visibles en GitHub, vacíos o con README
inicial.

## 0.INFRA-2 — Crear cuenta Supabase y proyecto [S]

[ ] Crear proyecto Supabase (región sa-east-1 si disponible, sino
    us-east).
[ ] Anotar `DATABASE_URL`, anon key, service role key.
[ ] Crear buckets de storage:
    - `product-images` (public read).
    - `evidence` (private; URLs firmadas).
    - `logos` (public read).
[ ] Crear proyecto separado para producción (no usar el mismo de dev).

## 0.INFRA-3 — Aplicar `ruta_postgres.sql` a la BD de dev [S]

→ depende de: 0.INFRA-2

[ ] Ejecutar `docs/bd/ruta_postgres.sql` contra Supabase dev.
[ ] Verificar: 21+ particiones para `client_id = 0`, parámetros
    insertados, state_catalog completo, RLS activo.

**Criterio:** `\dt ruta.*` lista las 21 tablas; `SELECT * FROM
ruta.state_catalog LIMIT 5;` retorna filas.

## 0.INFRA-4 — Configurar GitHub Packages para `@ruta/*` [M]

[ ] Crear Personal Access Token con scopes `read:packages` y
    `write:packages`.
[ ] Documentar setup de `.npmrc` por desarrollador.
[ ] Configurar GitHub Action secret `NPM_PUBLISH_TOKEN` con permisos
    de publicación en la organización.
[ ] Probar publicación dummy desde `ruta-shared` (publicar
    `@ruta/shared@0.0.1` placeholder, verificar que aparece en
    GitHub Packages).

## 0.INFRA-5 — Setup workspace local [M]

[ ] Crear `ruta-infra/workspace.config.json` con lista de repos a
    clonar y sus targets locales:
    ```json
    {
      "base_dir": "~/projects/ruta",
      "repos": [
        {"name": "ruta-backend", "target": "backend-ruta"},
        {"name": "ruta-frontend", "target": "frontend-ruta"},
        {"name": "ruta-shared", "target": "packages-ruta"},
        {"name": "ruta-docs", "target": "docs-ruta"},
        {"name": "ruta-infra", "target": "infra-ruta"},
        {"name": "landing-template", "target": "frontend-clients-ruta/_template"}
      ]
    }
    ```
[ ] Crear `ruta-infra/scripts/setup_workspace.sh` que lee el config y
    clona todo.
[ ] Crear `ruta-infra/scripts/clone_landing.sh` para clonar landings
    existentes en `frontend-clients/`.
[ ] Crear `ruta-infra/scripts/create_landing.sh` que usa la API de
    GitHub para crear un nuevo repo desde `landing-template` y
    clonarlo localmente.

**Criterio:** `bash setup_workspace.sh` en una máquina nueva clona los
6 repos en sus carpetas correctas.

## 0.DOCS-1 — Inicializar `ruta-docs` [S]

[ ] Copiar TODO el contenido actual de documentación a `ruta-docs/`.
[ ] Estructura: `arquitectura/`, `seguridad/`, `flujos/`, `bd/`,
    `diseno/`, más los archivos `.md` en la raíz.
[ ] Copiar `CLAUDE.md` y `AGENTS.md` a la raíz.
[ ] README de root explicando estructura del proyecto.
[ ] Commit y push.

## 0.SHARED-1 — Inicializar `ruta-shared` [L]

→ depende de: 0.INFRA-4

[ ] Crear workspace pnpm con dos paquetes: `shared/` y `db/`.
[ ] `shared/package.json`: name `@ruta/shared`, version `1.0.0`,
    publishConfig apuntando a GitHub Packages.
[ ] Estructura `shared/src/`:
    - `types/order.types.ts`, `payment.types.ts`, `user.types.ts`,
      `client.types.ts`, `api.types.ts`.
    - `enums/order_status.ts`, `payment_status.ts`,
      `refund_status.ts`, `return_status.ts`, `client_type.ts`,
      `user_type.ts`, `closure_reason.ts`.
    - `validators/order.schema.ts`, `product.schema.ts`,
      `user.schema.ts`, etc. (Zod).
    - `constants/error_codes.ts`, `parameter_keys.ts`.
[ ] `db/package.json`: name `@ruta/db`, version `1.0.0`.
[ ] `db/prisma/schema.prisma` generado vía `prisma db pull` desde la
    BD ya inicializada en 0.INFRA-3.
[ ] `db/src/client.ts`: singleton de PrismaClient.
[ ] `db/src/tenant.ts`: helper `withTenant(clientId, role, fn)`.
[ ] CI `.github/workflows/publish.yml`: auto-publish a GitHub
    Packages en merge a `main`.
[ ] Primera publicación: `@ruta/shared@1.0.0` y `@ruta/db@1.0.0`.

**Criterio:** ambos paquetes aparecen en GitHub Packages y se pueden
instalar desde otro repo con `pnpm add @ruta/shared @ruta/db`.

## 0.BACK-1 — Inicializar `ruta-backend` con Express [M]

→ depende de: 0.SHARED-1

[ ] Estructura:
    ```
    ruta-backend/
    ├── .npmrc
    ├── package.json
    ├── tsconfig.json
    ├── .env.example
    ├── render.yaml
    └── api/
        ├── package.json
        ├── tsconfig.json
        └── src/
            ├── index.ts
            ├── app.ts
            ├── config/env.ts
            ├── middleware/
            ├── routes/
            ├── services/
            ├── jobs/
            └── lib/
    ```
[ ] `pnpm add express @types/express tsx pino jose argon2 zod
    @ruta/shared @ruta/db` (las @ruta/* desde GitHub Packages).
[ ] Endpoint `/healthz`.
[ ] Logger pino con `request_id`.
[ ] CI: lint, typecheck, build.
[ ] CLAUDE.md y AGENTS.md específicos del repo (extienden el master).

**Criterio:** `pnpm dev` arranca; `curl localhost:3001/healthz` →
200.

## 0.FRONT-1 — Inicializar `ruta-frontend` con Next.js [M]

→ depende de: 0.SHARED-1

[ ] Crear workspace pnpm con tres apps internas:
    - `admin/` (Next.js).
    - `storefront/` (Next.js).
    - `packages/ui/` (`@ruta/ui` interno, no publicado).
[ ] `.npmrc` con auth a GitHub Packages.
[ ] `pnpm add @ruta/shared` en admin y storefront (shared, no db).
[ ] Configurar Tailwind con tokens de
    `docs/diseno/galeria_estilos_ruta.md` en ambas apps.
[ ] Implementar componentes base en `packages/ui/`:
    `RutaCard`, `RutaButton`, `RutaPill`, `RutaSectionHeader`,
    `RutaThemeToggle`.
[ ] Página `/` de prueba en admin y storefront que renderiza los
    componentes Ruta en claro y oscuro.
[ ] CI: lint, typecheck, build separado por app.
[ ] CLAUDE.md y AGENTS.md específicos.

**Criterio:** `pnpm dev:admin` en :3002 y `pnpm dev:storefront` en
:3003 muestran los componentes en ambos modos.

## 0.LANDING-1 — Inicializar `landing-template` [M]

→ depende de: 0.SHARED-1

[ ] Estructura Next.js base, sin design system Ruta*.
[ ] `pnpm add @ruta/shared` (solo tipos/validators).
[ ] `src/lib/api_client.ts` configurado con `process.env.API_URL` y
    `process.env.CLIENT_SLUG`.
[ ] Páginas skeleton:
    - `/` catálogo (placeholder).
    - `/product/[id]` detalle.
    - `/cart` carrito.
    - `/checkout` checkout.
    - `/orders` mis pedidos.
    - `/(auth)/login` y `/(auth)/register`.
[ ] `tailwind.config.ts` con tokens BASE genéricos (a sobrescribir
    por cada landing).
[ ] `public/PLACEHOLDER_logo.svg`.
[ ] README con instrucciones para crear un landing desde el template.
[ ] CLAUDE.md y AGENTS.md específicos para landings custom.
[ ] Marcar el repo como Template en GitHub Settings (ya hecho en
    0.INFRA-1).

**Criterio:** `pnpm dev` levanta el template; en GitHub el botón "Use
this template" está disponible.

## 0.INFRA-6 — Inicializar `ruta-infra` [S]

[ ] Subir los scripts ya descritos en 0.INFRA-5.
[ ] Subir `render.yaml.example` con los 3 servicios principales.
[ ] Subir `supabase/storage_buckets.sql` con la configuración de
    buckets.
[ ] Subir `infra/scripts/create_first_admin_ruta.sh`.
[ ] CLAUDE.md y AGENTS.md específicos.

## 0.INFRA-7 — Crear servicios en Render [M]

→ depende de: 0.BACK-1, 0.FRONT-1

[ ] Crear 3 Web Services apuntando a sus repos:
    - `ruta-api` → repo `ruta-backend`, build path `api/`,
      build command `pnpm install && pnpm --filter @ruta/api build`,
      start `pnpm --filter @ruta/api start`.
    - `ruta-admin` → repo `ruta-frontend`, build path `admin/`.
    - `ruta-storefront` → repo `ruta-frontend`, build path
      `storefront/`.
[ ] Configurar Background Worker `ruta-api-worker` para pg-boss.
[ ] Inyectar `DATABASE_URL`, `NPM_TOKEN` (para auth a GitHub
    Packages), Wompi keys (placeholders).
[ ] Dominios provisionales: `api-dev.onrender.com`,
    `app-dev.onrender.com`, `tienda-dev.onrender.com`.

**Criterio:** las 3 URLs responden 200 (al menos `/healthz`).

## 0.BACK-2 — Crear primer ADMIN_RUTA via script [S]

[ ] Script `infra/scripts/create_first_admin_ruta.sh`:
    - Prompt de email y password.
    - Hash con argon2id.
    - INSERT a `ruta.users` con `client_id=0, user_type='ADMIN_RUTA'`.
[ ] Ejecutar en BD de dev.

**Criterio:** al menos un ADMIN_RUTA existe.

## 0.INFRA-8 — Seed script para desarrollo [M]

[ ] `infra/scripts/seed_dev_data.sh`:
    - 1 Cliente Full piloto (modalidad NATIVE_RUTA).
    - 1 ADMIN_CLIENT, 1 OPERATOR_CLIENT, 3 COURIERs, 3 BUYERs.
    - 10 productos en 3 categorías.
    - 2 pickup points.

**Criterio fin Sprint 0:**

- Cualquier desarrollador (humano + IA) puede clonar `ruta-infra`,
  correr `setup_workspace.sh`, obtener los 6 repos, y levantar las 3
  apps en local.
- Backend, admin y storefront consumen `@ruta/shared` desde GitHub
  Packages sin problemas.
- Las 3 apps desplegadas en Render responden.
- La BD tiene 1 Cliente piloto sembrado y al menos 1 ADMIN_RUTA.
- `landing-template` está listo para clonar como nuevo landing
  cuando llegue un Cliente custom (Fase 3).

---

# Sprint 1 — Auth, Clientes, Productos, Registro BUYER (2 semanas)

**Meta:** flujo de registro y login. ADMIN_RUTA crea Clientes.
ADMIN_CLIENT gestiona catálogo. Comprador se registra y ve catálogo.

## 1.SHARED-1 — Bumpear `@ruta/shared` con schemas de auth y catálogo [M]

[ ] Agregar a `@ruta/shared/validators/`:
    - `auth.schema.ts` (login, register, refresh).
    - `client.schema.ts` (create, update).
    - `product.schema.ts`, `category.schema.ts`.
    - `buyer.schema.ts`, `courier.schema.ts`, `pickup_point.schema.ts`.
[ ] Bump a `@ruta/shared@1.1.0`. Publish.

## 1.BACK-1 — Servicio de Auth completo [XL]

→ depende de: 0.BACK-1, 1.SHARED-1

[ ] `services/auth.service.ts` con register, login, loginRutaAdmin,
    refresh, logout.
[ ] Middleware `auth.ts` verifica JWT, popula `req.user`, setea
    contexto de tenant.
[ ] Endpoints `POST /auth/register`, `/auth/login`,
    `/auth/ruta-admin/login`, `/auth/refresh`, `/auth/logout`.
[ ] Cookies HttpOnly Secure SameSite=Strict.
[ ] Argon2 + jose.
[ ] Lectura de duración de tokens desde `client_parameters`.

**Tests:** registro, login OK, credenciales malas, refresh, logout,
cross-tenant rechazado.

## 1.BACK-2 — Gestión de Clientes (ADMIN_RUTA) [L]

[ ] CRUD en `services/clients.service.ts`.
[ ] Endpoints `/ruta-admin/clients/*`.
[ ] Validación `client_type=FULL` requiere `frontend_mode`.
[ ] Auditoría.

## 1.BACK-3 — Gestión de productos (ADMIN_CLIENT) [L]

[ ] CRUD productos y categorías.
[ ] Endpoints `/admin/products/*`, `/admin/categories/*`.
[ ] Endpoint público `/public/clients/:slug/products` y
    `/categories`.
[ ] Upload de imagen vía `/uploads/presigned-url` (Supabase Storage).

## 1.BACK-4 — Compradores y Repartidores [M]

[ ] CRUD básico para BUYERs (admin view) y COURIERs.

## 1.BACK-5 — Importación masiva por Excel [M]

[ ] `POST /admin/products/bulk-import` (multipart).
[ ] Job pg-boss para procesar asíncrono.
[ ] `GET /admin/products/bulk-import/:job_id` para status.

## 1.ADMIN-1 — Login y layout [M]

[ ] `/login` con detección de rol y redirect.
[ ] `RutaSidebar` con navegación condicional.
[ ] `RutaHeader` con info de Cliente activo, banner Vista de Control.

## 1.ADMIN-2 — Pantallas ADMIN_RUTA: lista y CRUD de Clientes [M]

[ ] `/ruta-admin/clients` lista.
[ ] `/ruta-admin/clients/new` formulario.
[ ] `/ruta-admin/clients/:id` detalle.

## 1.ADMIN-3 — Gestión de productos [L]

[ ] `/admin/products` lista + filtros.
[ ] Formularios create/edit con upload de imagen.
[ ] Modal de import masivo.

## 1.ADMIN-4 — Compradores, Repartidores, Pickup Points [M]

[ ] Listas y CRUD básico en sus respectivas rutas.

## 1.STORE-1 — Layout y catálogo [L]

[ ] Layout `c/[slug]/layout.tsx` con branding del Cliente.
[ ] `/c/[slug]/` catálogo con grid y filtros.
[ ] `/c/[slug]/product/[id]` detalle.

## 1.STORE-2 — Registro y login del BUYER [M]

[ ] `/c/[slug]/(auth)/register` y `/login`.

## 1.QA-1 — Tests de aislamiento cross-tenant [M]

[ ] Suite que valida que todo endpoint rechaza acceso cross-tenant.
[ ] Gate en CI.

**Criterio fin Sprint 1:** Visitante puede entrar a
`tienda-dev.onrender.com/c/restaurante-piloto/`, ver catálogo,
registrarse, loguearse. ADMIN_RUTA crea Clientes nuevos.
ADMIN_CLIENT agrega productos.

---

# Sprint 2 — Carrito, checkout y Flujo 1 completo (2 semanas)

**Meta:** Comprador completa pedido con pago online (Wompi) o contra
entrega.

## 2.SHARED-1 — Bumpear `@ruta/shared` con order schemas [M]

[ ] Validators de orders: create, confirm, transition, payments.
[ ] Tipos derivados.
[ ] Bump a `@ruta/shared@1.2.0`.

## 2.BACK-1 — Orders + State Machine [XL]

→ depende de: 1.BACK-1, 2.SHARED-1

[ ] `services/orders/state_machine.ts` con todas las transiciones de
    flujos 1, 2, 3.
[ ] `services/orders/orders.service.ts`: create, confirm, cancel,
    requestCancel.
[ ] Endpoints `/buyer/orders/*`.

**Tests unitarios:** cada transición permitida y rechazada.

## 2.BACK-2 — Validación operativa y aceptación [L]

[ ] Job `validate_order` → VALIDATION_APPROVED o MANUAL_REVIEW.
[ ] Endpoints admin para accept, reject, mark-preparing, mark-ready.

## 2.BACK-3 — Wompi (initiate + webhook) [XL]

[ ] `lib/wompi_client.ts`.
[ ] `services/payments.service.ts.initiatePayment(orderId)`.
[ ] Endpoint `POST /buyer/orders/:id/initiate-payment`.
[ ] Endpoint webhook entrante con verificación HMAC y
    deduplicación.

## 2.BACK-4 — Jobs de mantenimiento [M]

[ ] order_expiration, payment_timeout, cleanup_idempotency,
    cleanup_sessions.

## 2.STORE-1 — Carrito local con Zustand [M]

[ ] Zustand + persist en localStorage.
[ ] `/c/[slug]/cart`.

## 2.STORE-2 — Checkout multi-paso [XL]

[ ] Paso 1: entrega (SHIP/PICKUP).
[ ] Paso 2: dirección o pickup point (con mapa OSM).
[ ] Paso 3: método de pago.
[ ] Submit con creación de pedido y redirección a Wompi o
    confirmación.

## 2.STORE-3 — Mis pedidos (BUYER) [L]

[ ] `/c/[slug]/orders` lista.
[ ] `/c/[slug]/orders/[id]` detalle con `RutaTimeline`.
[ ] Acciones condicionales.

## 2.STORE-4 — Confirmación post-Wompi [M]

[ ] `/c/[slug]/checkout/confirmation` con polling.

## 2.ADMIN-1 — Lista y detalle de pedidos (admin) [L]

[ ] `/admin/orders` con filtros.
[ ] `/admin/orders/:id` detalle 360.

## 2.QA-1 — Tests del state machine y Wompi mocks [M]

[ ] MSW para Wompi sandbox.
[ ] Test de webhook con firma válida e inválida.

**Criterio fin Sprint 2:** Pedido real con Wompi sandbox completado de
extremo a extremo hasta READY_TO_SHIP o READY_FOR_PICKUP.

---

# Sprint 3 — Flujo SHIP completo (2 semanas)

## 3.BACK-1 — Asignación de Repartidor [L]

[ ] Service con `assignCourier` (optimistic lock).
[ ] Endpoints admin.

## 3.BACK-2 — Endpoints COURIER [L]

[ ] Todas las transiciones del courier.
[ ] Solo opera sus propios pedidos.

## 3.BACK-3 — Cobro contra entrega [L]

[ ] `recordCollection()` con upload de evidencia.
[ ] Endpoint multipart.

## 3.BACK-4 — Cancelación post-despacho [M]

[ ] CUSTOMER_CANCEL_REQUEST y review por admin.

## 3.BACK-5 — RETURN_TO_ORIGIN [M]

[ ] Transición tras intentos agotados.

## 3.BACK-6 — Auto-confirmación de entregados [M]

[ ] Job que transiciona DELIVERED → CONFIRMED_BY_SYSTEM.

## 3.ADMIN-1 — Mapa de asignación [XL]

[ ] Leaflet + OSM.
[ ] Panel lateral con pedidos y couriers.
[ ] Asignación con confirmación.
[ ] Auto-refresh.

## 3.ADMIN-2 — Vista COURIER móvil-first [XL]

[ ] `/courier` con tabs.
[ ] Detalle con botones grandes.
[ ] Form de cobro con cámara para evidencia.

## 3.QA-1 — E2E flujo SHIP [L]

[ ] Playwright spec end-to-end.

---

# Sprint 4 — Flujo PICKUP (1 semana)

## 4.BACK-1 — Endpoints PICKUP [L]

[ ] Transiciones específicas.
[ ] Validación de identidad.
[ ] Cobro en punto.
[ ] Job `pickup_expiration`.

## 4.ADMIN-1 — UI operación en punto físico [M]

[ ] Botones específicos de PICKUP.

## 4.QA-1 — E2E flujo PICKUP [M]

---

# Sprint 5 — Vista de Control, dashboards, configuración (1 semana)

## 5.BACK-1 — Vista de Control [L]

[ ] enterControlView, exitControlView.
[ ] Verificación master password.
[ ] Auditoría completa.

## 5.BACK-2 — Dashboards [L]

[ ] Endpoints de métricas Cliente y globales.

## 5.BACK-3 — Parámetros [M]

[ ] GET/PATCH de `client_parameters`.

## 5.BACK-4 — Auditoría [M]

[ ] Endpoint con filtros amplios.

## 5.ADMIN-1 — Pantalla Vista de Control [M]

[ ] Form con master password.
[ ] Banner ámbar.

## 5.ADMIN-2 — Dashboards [M]

[ ] Cliente y global.

## 5.ADMIN-3 — Configuración [L]

[ ] Tabs: info, pagos (Wompi), webhooks, parámetros.

## 5.ADMIN-4 — Auditoría [M]

---

# Sprint 6 — Hardening, observabilidad, deploy, piloto (1 semana)

## 6.QA-1 — Suite E2E completa [L]

## 6.QA-2 — Cobertura >85% [M]

## 6.INFRA-1 — Observabilidad [L]

[ ] Logs JSON estructurados.
[ ] Integración con Logtail / Better Stack / Axiom.
[ ] Métricas y alerting básico.

## 6.INFRA-2 — Backups y restore probado [M]

## 6.INFRA-3 — Webhooks salientes [L]

[ ] Cola pg-boss con reintentos.
[ ] UI para historial y reintentos.

## 6.INFRA-4 — Deploy a producción [M]

[ ] Supabase prod.
[ ] Aplicar SQL en prod.
[ ] DNS reales (api.ruta.com, app.ruta.com, tienda.ruta.com).
[ ] Wompi prod.
[ ] Deploy las 3 apps en Render prod.

## 6.QA-3 — Onboarding Cliente piloto [M]

[ ] ADMIN_RUTA crea Cliente piloto.
[ ] Soporte vivo configurando catálogo, Wompi, repartidores.
[ ] Primer pedido real.

## 6.QA-4 — Documentación de usuario [M]

[ ] Guía ADMIN_CLIENT, OPERATOR_CLIENT, COURIER, BUYER.

---

## Cronograma compacto

| Semana | Sprint | Foco principal |
|---|---|---|
| 1 | 0 | Setup multi-repo (6 repos), Supabase, GitHub Packages, primer ADMIN_RUTA |
| 2-3 | 1 | Auth, Clientes, Productos, Registro BUYER |
| 4-5 | 2 | Carrito, checkout, Wompi, Flujo 1 |
| 6-7 | 3 | Flujo SHIP, mapa, courier |
| 8 | 4 | Flujo PICKUP |
| 9 | 5 | Vista Control, dashboards, config |
| 10 | 6 | Hardening, deploy, piloto |

---

## Tracks paralelos

Cuando hay capacidad simultánea (humano + IA en paralelo), las tareas
pueden ejecutarse en paralelo si no comparten dependencias críticas.
Tracks principales: BACK, ADMIN, STORE, INFRA, SHARED, LANDING, DOCS,
QA.

Dependencias documentadas (`→ depende de:`) son barreras reales.

---

## Convención de Pull Requests por repo

- Una tarea = un PR (idealmente).
- Título: `[TASK-X.Y-N] Título`.
- Descripción: link a la sección del plan + criterios + notas.
- Checks obligatorios: lint, typecheck, tests, aislamiento cross-tenant
  (cuando aplica).
- Review obligatorio: 1 review humano mínimo.
- Squash & merge.

Cuando un cambio cruza varios repos (ej. agregar un nuevo estado de
pedido):

1. PR en `ruta-shared` que agrega el enum/validator. Merge → publish
   nueva versión.
2. PR en `ruta-backend` que consume nueva versión y agrega lógica.
3. PR en `ruta-frontend` (admin y storefront) que consume nueva versión
   y agrega UI.
4. PRs en landings activas si aplican.

Documentar cross-repo PRs con el mismo TASK ID en todos los títulos.

---

## Métricas de progreso

Actualizar este documento cada viernes con:

- Tareas completadas esta semana.
- Tareas en progreso.
- Bloqueos.
- Ajustes al plan.

El plan es guía, no contrato. Ajustes documentados.
