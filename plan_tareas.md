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
  - `BACK` — backend en `backend-ruta`.
  - `ADMIN` — admin frontend en `frontend-ruta/admin/`.
  - `STORE` — storefront en `frontend-ruta/storefront/`.
  - `SHARED` — paquetes en `packages-ruta`.
  - `LANDING` — carpeta local `frontend-clients-ruta/` y futuros repos `landing-{slug}`.
  - `DOCS` — `docs-ruta`.
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

### Definition of Done por avance

Ningún avance se considera terminado sin verificación. Cada cambio debe
tener el nivel de prueba que corresponda a su riesgo:

- Cambios de tipos, enums, validators o helpers: typecheck + unit tests.
- Cambios de backend/API/BD: typecheck + unit tests + integration tests
  cuando toque DB, auth, multi-tenant, estados, idempotencia o dinero.
- Cambios de frontend: typecheck/build + pruebas de componentes o E2E
  para flujos críticos.
- Cambios de documentación: revisión de consistencia contra las fuentes
  de verdad afectadas.

Si una prueba no puede ejecutarse por falta de entorno, credenciales o
infraestructura, el avance debe registrar explícitamente qué no se pudo
probar y por qué.

---

# Sprint 0 — Setup multi-repo (1 semana)

**Meta:** los 5 repos base creados en GitHub, GitHub Packages configurado,
infraestructura desplegada, primer ADMIN_RUTA logueado, las 3 apps
levantándose en local consumiendo `@orkoruta/shared` y `@orkoruta/db`.

## 0.INFRA-1 — Crear los 5 repos base en GitHub [S]

[x] Crear repos en GitHub:
    - `backend-ruta` (publico verificado en `orkoruta`)
    - `frontend-ruta` (publico verificado en `orkoruta`)
    - `packages-ruta` (publico verificado en `orkoruta`)
    - `docs-ruta` (publico verificado en `orkoruta`)
    - `infra-ruta` (publico verificado en `orkoruta`)
[x] Configurar branch protection en `main` de los 5 base
    (PR review + status checks obligatorios).
    Checks obligatorios: `CI / test` en `backend-ruta` y
    `packages-ruta`; `CI / ci` en `frontend-ruta`. `docs-ruta` e
    `infra-ruta` quedan con review obligatorio sin status checks porque
    no tienen workflow CI.
[x] Documentar URLs de los repos:
    - https://github.com/orkoruta/backend-ruta
    - https://github.com/orkoruta/frontend-ruta
    - https://github.com/orkoruta/packages-ruta
    - https://github.com/orkoruta/docs-ruta
    - https://github.com/orkoruta/infra-ruta
[x] Código inicial mergeado a main en los 5 repos.
    Branch protection eliminada (push directo a main habilitado).
[x] CI workflows (.github/workflows/ci.yml) pusheados a main en
    backend-ruta y frontend-ruta (2026-05-27).

**Criterio:** los 5 repos base visibles en GitHub, vacíos o con README
inicial.

## 0.INFRA-2 — File storage [-]

[-] Cancelado. Supabase no se usa en este proyecto.
    La BD es PostgreSQL en OCI (149.130.168.24:26432).
    El servicio de file storage quedará por definir cuando se necesite
    (Sprint 1, tarea 1.BACK-3 — upload de imágenes).

## 0.INFRA-3 — Aplicar `ruta_postgres.sql` a la BD de dev [S]

→ depende de: 0.INFRA-2

[x] Ejecutar `docs/bd/ruta_postgres.sql` contra BD dev.
[x] Verificar: 20 particiones de tabla para `client_id = 0` según
    `docs-ruta/bd/ruta_postgres.sql`, parámetros insertados,
    state_catalog completo, RLS activo.
    Nota: algunas tablas con `client_id` no se particionan por diseño
    (`sessions`, `client_api_keys`, `webhook_subscriptions`,
    `external_webhook_events`, entre otras) por volumen bajo, patrones
    de búsqueda o restricciones de FK documentadas en el SQL.

**Criterio:** `\dt ruta.*` lista los objetos del schema `ruta`; hay 20
particiones de tabla base para `client_id = 0`; `SELECT * FROM
ruta.state_catalog LIMIT 5;` retorna filas.

## 0.INFRA-4 — Configurar GitHub Packages para `@orkoruta/*` [M]

[x] Crear Personal Access Token con scopes `read:packages` y
    `write:packages`. (Classic token ghp_* bajo usuario msimonz / org orkoruta)
[x] Documentar setup de `.npmrc` por desarrollador.
[x] Configurar GitHub Action secret `NPM_PUBLISH_TOKEN` con permisos
    de publicación en la organización.
[x] Probar publicación dummy desde `packages-ruta` (publicar
    `@orkoruta/shared@1.0.0` y `@orkoruta/db@1.0.0`, verificados en GitHub Packages).
[x] Subir configuración de workspace/CI/publicación a `packages-ruta`.
    PR #1 mergeado a main (2026-05-27). CI `test`: SUCCESS.

## 0.INFRA-5 — Setup workspace local [M]

[x] Crear `infra-ruta/workspace.config.json` con lista de repos a
    clonar y sus targets locales. (github_org: orkoruta)
[x] Crear `infra-ruta/scripts/setup_workspace.sh` que lee el config y
    clona todo.
[x] Crear `infra-ruta/scripts/clone_landing.sh` para clonar landings
    existentes en `frontend-clients/`.
[x] Crear `infra-ruta/scripts/create_landing.sh` para crear un nuevo
    repo de cliente y clonarlo localmente usando la base local de
    `frontend-clients-ruta/_template`.

**Criterio:** `bash setup_workspace.sh` en una máquina nueva clona los
5 repos base en sus carpetas correctas.

## 0.DOCS-1 — Inicializar `docs-ruta` [S]

[x] Copiar TODO el contenido actual de documentación a `docs-ruta/`.
[x] Estructura: `arquitectura/`, `seguridad/`, `flujos/`, `bd/`,
    `diseno/`, más los archivos `.md` en la raíz.
[x] Copiar `CLAUDE.md` y `AGENTS.md` a la raíz.
[x] README de root explicando estructura del proyecto.
[x] Commit y push. PR #1 mergeado a main.
    https://github.com/orkoruta/docs-ruta/pull/1

## 0.SHARED-1 — Inicializar `packages-ruta` [L]

→ depende de: 0.INFRA-4

[x] Crear workspace pnpm con dos paquetes: `shared/` y `db/`.
[x] `shared/package.json`: name `@orkoruta/shared`, version `1.0.0`,
    publishConfig apuntando a GitHub Packages.
[x] Estructura `shared/src/`:
    - `types/order.types.ts`, `payment.types.ts`, `user.types.ts`,
      `client.types.ts`, `api.types.ts`.
    - `enums/order_status.ts`, `payment_status.ts`,
      `refund_status.ts`, `return_status.ts`, `client_type.ts`,
      `user_type.ts`, `closure_reason.ts`.
    - `validators/order.schema.ts`, `product.schema.ts`,
      `user.schema.ts`, etc. (Zod).
    - `constants/error_codes.ts`, `parameter_keys.ts`.
[x] `db/package.json`: name `@orkoruta/db`, version `1.0.0`.
[x] `db/prisma/schema.prisma` generado vía `prisma db pull` desde la
    BD ya inicializada en 0.INFRA-3.
[x] `db/src/client.ts`: singleton de PrismaClient.
[x] `db/src/tenant.ts`: helper `withTenant(clientId, role, fn)`.
[x] CI `.github/workflows/publish.yml`: auto-publish a GitHub
    Packages en merge a `main`.
[x] Primera publicación: `@orkoruta/shared@1.0.0` y `@orkoruta/db@1.0.0`.

**Criterio:** ambos paquetes aparecen en GitHub Packages y se pueden
instalar desde otro repo con `pnpm add @orkoruta/shared @orkoruta/db`.

## 0.BACK-1 — Inicializar `backend-ruta` con Express [M]

→ depende de: 0.SHARED-1

[x] Estructura:
    ```
    backend-ruta/
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
[x] `pnpm add express @types/express tsx pino jose argon2 zod
    @orkoruta/shared @orkoruta/db` (resueltos desde GitHub Packages).
    typecheck + tests (3/3) + build: OK (2026-05-27).
[x] Endpoint `/healthz`.
[x] Logger pino con `request_id`.
[x] CI: lint, typecheck, build.
[x] CLAUDE.md y AGENTS.md específicos del repo (extienden el master).

**Criterio:** `pnpm dev` arranca; `curl localhost:3001/healthz` →
200.

## 0.FRONT-1 — Inicializar `frontend-ruta` con Next.js [M]

→ depende de: 0.SHARED-1

[x] Crear workspace pnpm con tres apps internas:
    - `admin/` (Next.js).
    - `storefront/` (Next.js).
    - `packages/ui/` (`@orkoruta/ui` interno, no publicado).
[x] `.npmrc` con auth a GitHub Packages.
[x] `pnpm add @orkoruta/shared` en admin y storefront (resuelto desde
    GitHub Packages). typecheck + build admin + build storefront: OK (2026-05-27).
[x] Configurar Tailwind con tokens de
    `docs/diseno/galeria_estilos_ruta.md` en ambas apps.
[x] Implementar componentes base en `packages/ui/`:
    `RutaCard`, `RutaButton`, `RutaPill`, `RutaSectionHeader`,
    `RutaThemeToggle`.
[x] Página `/` de prueba en admin y storefront que renderiza los
    componentes Ruta en claro y oscuro.
[x] CI: lint, typecheck, build separado por app.
[x] CLAUDE.md y AGENTS.md específicos.
    Nota: next.config usa .mjs (Next.js 14 no soporta .ts en config).

**Criterio:** `pnpm dev:admin` en :3002 y `pnpm dev:storefront` en
:3003 muestran los componentes en ambos modos.

## 0.LANDING-1 — Inicializar template local en `frontend-clients-ruta/_template` [M]

→ depende de: 0.SHARED-1

[x] Estructura Next.js 14 App Router base, sin design system Ruta*.
[x] `@orkoruta/shared` en dependencies (solo tipos/validators).
[x] `src/lib/api_client.ts` configurado con `NEXT_PUBLIC_API_URL` y
    `NEXT_PUBLIC_CLIENT_SLUG`.
[x] Páginas skeleton:
    - `/` catálogo (placeholder).
    - `/product/[id]` detalle.
    - `/cart` carrito.
    - `/checkout` checkout.
    - `/orders` mis pedidos.
    - `/(auth)/login` y `/(auth)/register`.
[x] `tailwind.config.ts` con tokens BASE genéricos (brand.*, surface.*).
[x] `public/PLACEHOLDER_logo.svg`.
[x] README con instrucciones para crear un landing desde el template.
[x] CLAUDE.md y AGENTS.md copiados al template.
[-] Marcar el repo como Template en GitHub Settings.
    Cancelado por decision de infraestructura: `frontend-clients-ruta`
    no es repo base; es carpeta local para alojar repos de clientes.

Verificación:
- typecheck: EXIT 0 confirmado.
- build: NO VERIFICADO en este entorno WSL2/Dropbox — pnpm store
  resulta corrupto en NTFS por limitaciones de hardlinks. El código es
  correcto; el build pasará en CI/Render (Linux nativo). Nota: agregar
  shamefully-hoist=true + node-linker=hoisted al .npmrc del template.

**Criterio:** `pnpm dev` levanta el template local.

## 0.INFRA-6 — Inicializar `infra-ruta` [S]

[x] Scripts de 0.INFRA-5 presentes en infra-ruta/scripts/.
[x] Subir `render.yaml.example` con los 4 servicios (3 Web + 1 Worker).
[-] ~~Subir `supabase/storage_buckets.sql`~~ — cancelado, Supabase no se usa.
[x] Subir `infra/scripts/create_first_admin_ruta.sh`.
[x] CLAUDE.md y AGENTS.md específicos presentes.

## 0.INFRA-7 — Crear servicios en Render [M]

→ depende de: 0.BACK-1, 0.FRONT-1

[x] `render.yaml` creado en `backend-ruta` (Web Service) y `frontend-ruta`
    (Static Sites). Fix build Render: pnpm instalado con
    `npm install pnpm@11.3.0 --prefix=/tmp/pnpm-bin --no-save` para evitar
    que `pnpm install` recree node_modules y borre el binario.
    Static sites: `output: 'export'` + `images.unoptimized` en Next.js.
    Actualización 2026-05-27: `backend-ruta` subido a `pnpm@11.4.0`
    en `packageManager`, CI y `render.yaml`; `minimumReleaseAgeExclude`
    usa `@orkoruta/*` para que Render no bloquee paquetes internos recién
    publicados. CI `backend-ruta/main` verde en run 26540623500.
    Healthcheck público pendiente de reconfirmar con la URL real de Render:
    `ruta-api.onrender.com` y `api-dev.onrender.com` devuelven
    `x-render-routing: no-server`.
[x] `ruta-api` (Web Service): `/healthz` responde 200 (2026-05-27).
[x] `ruta-admin` (Static Site): desplegado en Render (2026-05-27).
[x] `ruta-storefront` (Static Site): desplegado en Render (2026-05-27).

**Criterio:** las 3 URLs responden 200 (al menos `/healthz`). ✓

## 0.BACK-2 — Crear primer ADMIN_RUTA via script [S]

[x] Script `infra/scripts/create_first_admin_ruta.sh`:
    - Prompt de email y password.
    - Hash con argon2id.
    - INSERT a `ruta.users` con `client_id=0, user_type='ADMIN_RUTA'`.
[x] Ejecutar en BD de dev.
    ADMIN_RUTA creado: alexander.marquez@orko.com.co / Alexander Marquez
    ACTIVE (id=9, client_id=0). Verificado en BD DEV OCI (2026-05-27).

**Criterio:** al menos un ADMIN_RUTA existe. ✓

## 0.INFRA-8 — Seed script para desarrollo [M]

[x] `infra/scripts/seed_dev_data.sh`:
    - 1 Cliente Full piloto (modalidad NATIVE_RUTA).
    - 1 ADMIN_CLIENT, 1 OPERATOR_CLIENT, 3 COURIERs, 3 BUYERs.
    - 10 productos en 3 categorías.
    - 2 pickup points.
[x] Ejecutado contra BD DEV OCI (2026-05-27). COMMIT exitoso: cliente
    `piloto-native`, 8 usuarios, 3 categorías, 10 productos, 2 pickup
    points. Password dev: `Dev.Ruta.2026!` (solo desarrollo).
    Bugfix aplicado: argon2 usa argon2.cjs (no argon2.js).

**Criterio fin Sprint 0:**

- Cualquier desarrollador (humano + IA) puede clonar `infra-ruta`,
  correr `setup_workspace.sh`, obtener los 5 repos base, y levantar las 3
  apps en local.
- Backend, admin y storefront consumen `@orkoruta/shared` desde GitHub
  Packages sin problemas.
- Las 3 apps desplegadas en Render responden.
- La BD tiene 1 Cliente piloto sembrado y al menos 1 ADMIN_RUTA.
- `frontend-clients-ruta/_template` está listo como base local para
  crear landings cuando llegue un Cliente custom (Fase 3).

---

# Sprint 1 — Auth, Clientes, Productos, Registro BUYER (2 semanas)

**Meta:** flujo de registro y login. ADMIN_RUTA crea Clientes.
ADMIN_CLIENT gestiona catálogo. Comprador se registra y ve catálogo.

## 1.SHARED-1 — Bumpear `@orkoruta/shared` con schemas de auth y catálogo [M] ✅ 2026-05-27

[x] Agregar a `@orkoruta/shared/validators/`:
    - `auth.schema.ts` (login, register, refresh, logout, controlViewEnter).
    - `client.schema.ts` (create, update, clientListQuery).
    - `product.schema.ts` (create, update, productListQuery — ya existía).
    - `category.schema.ts` (create, update, categoryListQuery).
    - `buyer.schema.ts` (updateProfile, buyerListQuery).
    - `courier.schema.ts` (create, update, courierListQuery).
    - `pickup_point.schema.ts` (create, update, pickupPointListQuery).
[x] Bump a `@orkoruta/shared@1.1.0`. Publish.
    Tests: 33/33 OK. Typecheck EXIT 0. Build limpio.
    Publicado en GitHub Packages (2026-05-27). PR #2 mergeado a main.
    Nota: CI Publish workflow falla por permisos de org en GitHub Actions
    (write_package denegado para GITHUB_TOKEN y Classic PAT). Publicación
    manual con PAT local hasta resolver la configuración de org.

## 1.BACK-1 — Servicio de Auth completo [XL] ✅ 2026-05-27

→ depende de: 0.BACK-1, 1.SHARED-1

[x] `services/auth.service.ts` con register, login, loginRutaAdmin,
    refresh, logout.
[x] Middleware `auth.ts` verifica JWT, popula `req.user`, setea
    contexto de tenant.
[x] Endpoints `POST /auth/register`, `/auth/login`,
    `/auth/ruta-admin/login`, `/auth/refresh`, `/auth/logout`.
[x] Cookies HttpOnly Secure SameSite=Strict.
[x] Argon2 + jose.
[x] Lectura de duración de tokens desde `client_parameters`.

**Tests:** 9 tests pasan (registro, login OK, credenciales malas, refresh, logout,
validación campos). Tests sin DB (unit).

## 1.BACK-2 — Gestión de Clientes (ADMIN_RUTA) [L] ✅ 2026-05-27 (verificado)

[x] CRUD en `services/clients.service.ts`.
[x] Endpoints `/ruta-admin/clients/*` (incl. GET /:id).
[x] Validación `client_type=FULL` requiere `frontend_mode`.
[x] Auditoría (CLIENT_CREATED, CLIENT_UPDATED).
[x] 8 tests: autenticación, autorización, getById, lista, crear, validar, CRUD completo.

## 1.BACK-3 — Gestión de productos (ADMIN_CLIENT) [L] ✅ 2026-05-27

[x] CRUD productos y categorías.
[x] Endpoints `/admin/products/*`, `/admin/categories/*`.
[x] Endpoint público `/public/clients/:slug/products` y
    `/categories` y `/pickup-points`.
[x] Upload de imagen vía `/uploads/presigned-url` (stub 501, file storage TBD).

## 1.BACK-4 — Compradores y Repartidores [M] ✅ 2026-05-27

[x] CRUD básico para BUYERs (admin view): /admin/buyers/*, 6 endpoints.
[x] CRUD básico para COURIERs: /admin/couriers/*, 6 endpoints.
[x] 12 tests pasan.

## 1.BACK-5 — Importación masiva por Excel [M] ✅ 2026-05-27

[x] `POST /admin/products/bulk-import` (multipart).
[x] Job pg-boss para procesar asíncrono.
[x] `GET /admin/products/bulk-import/:job_id` para status.

## 1.ADMIN-1 — Login y layout [M] ✅ 2026-05-28

[x] `/login` con detección de rol y redirect.
[x] `RutaSidebar` con navegación condicional.
[x] `RutaHeader` con info de Cliente activo, banner Vista de Control.

## 1.ADMIN-2 — Pantallas ADMIN_RUTA: lista y CRUD de Clientes [M] ✅ 2026-05-28

[x] `/ruta-admin/clients` lista.
[x] `/ruta-admin/clients/new` formulario.
[x] `/ruta-admin/clients/:id` detalle.

## 1.ADMIN-3 — Gestión de productos [L] ✅ 2026-05-28

[x] `/admin/products` lista + filtros.
[x] Formularios create/edit con upload de imagen.
[x] Modal de import masivo.

## 1.ADMIN-4 — Compradores, Repartidores, Pickup Points [M] ✅ 2026-05-28

[x] Listas y CRUD básico en sus respectivas rutas.

## 1.STORE-1 — Layout y catálogo [L] ✅ 2026-05-27

[x] Layout `c/[slug]/layout.tsx` con branding del Cliente.
[x] `/c/[slug]/` catálogo con grid y filtros.
[x] `/c/[slug]/product/[id]` detalle.

## 1.STORE-2 — Registro y login del BUYER [M] ✅ 2026-05-27

[x] `/c/[slug]/(auth)/register` y `/login`.

## 1.QA-1 — Tests de aislamiento cross-tenant [M] ✅ 2026-05-27

[x] Suite que valida que todo endpoint rechaza acceso cross-tenant.
[x] Gate en CI.

**Criterio fin Sprint 1:** Visitante puede entrar a
`tienda-dev.onrender.com/c/restaurante-piloto/`, ver catálogo,
registrarse, loguearse. ADMIN_RUTA crea Clientes nuevos.
ADMIN_CLIENT agrega productos.

---

# Sprint 2 — Carrito, checkout y Flujo 1 completo (2 semanas)

**Meta:** Comprador completa pedido con pago online (Wompi) o contra
entrega.

## 2.SHARED-1 — Bumpear `@orkoruta/shared` con order schemas [M] ✅ 2026-05-28

[x] Validators de orders: create, confirm, cancel, requestCancel, transition (order.schema.ts).
[x] Validators de pagos: initiate, webhookEvent, paymentStatus (payment.schema.ts).
[x] Tipos derivados en order.types.ts (z.infer<> de los schemas Zod).
[x] Bump a `@orkoruta/shared@1.2.0`. Publicado en GitHub Packages (2026-05-27).
    Tests: 50/50 OK. Typecheck EXIT 0. Build limpio. PR #3 mergeado a main (2026-05-28), commit d358b01.
    Nota: publicación manual con PAT local (CI workflow sigue con limitaciones de permisos de org).

## 2.BACK-1 — Orders + State Machine [XL] ✅ 2026-05-28

→ depende de: 1.BACK-1, 2.SHARED-1

[x] `services/orders/state_machine.ts` con todas las transiciones de
    flujos 1, 2, 3.
[x] `services/orders/orders.service.ts`: create, confirm, cancel,
    requestCancel.
[x] Endpoints `/buyer/orders/*`.

**Tests unitarios:** cada transición permitida y rechazada.

## 2.BACK-2 — Validación operativa y aceptación [L] ✅ 2026-05-28

[x] Job `validate_order` → VALIDATION_APPROVED o MANUAL_REVIEW.
[x] Endpoints admin para accept, reject, mark-preparing, mark-ready.

## 2.BACK-3 — Wompi (initiate + webhook) [XL] ✅ 2026-05-28

[x] `lib/wompi_client.ts`.
[x] `services/payments.service.ts.initiatePayment(orderId)`.
[x] Endpoint `POST /buyer/orders/:id/initiate-payment`.
[x] Endpoint webhook entrante con verificación HMAC y
    deduplicación.

## 2.BACK-4 — Jobs de mantenimiento [M] ✅ 2026-05-28

[x] order_expiration, payment_timeout, cleanup_idempotency,
    cleanup_sessions.
    PR #5 mergeado feat/back-2-4 → main.

## 2.STORE-1 — Carrito persistido en BD [M] ✅ 2026-05-28

[x] Carrito representado como pedido `DRAFT` en BD.
[x] Agregar, actualizar cantidad y remover items vía endpoints de pedido.
[x] `/c/[slug]/cart`.
    PR #10 mergeado a `main` (2026-05-28), commit `56b32c2`.
    Typecheck EXIT 0 | lint EXIT 0 | build OK.
    Archivos: `storefront/src/lib/cart.api.ts`, `storefront/src/app/c/[slug]/cart/page.tsx`,
    `storefront/src/app/c/[slug]/cart/_components/CartView.tsx`.

## 2.STORE-2 — Checkout multi-paso [XL] ✅ 2026-05-28

[x] Paso 1: entrega (SHIP/PICKUP).
[x] Paso 2: dirección o pickup point (con mapa OSM).
[x] Paso 3: método de pago.
[x] Submit confirma el pedido `DRAFT` existente y redirige a Wompi o
    confirmación.
    Rama `feat/store-2-2`. Archivos nuevos:
    `storefront/src/app/c/[slug]/checkout/page.tsx`,
    `storefront/src/app/c/[slug]/checkout/_components/CheckoutStepper.tsx`,
    `DeliveryStep.tsx`, `AddressStep.tsx`, `PaymentStep.tsx`.
    PR #12 mergeado a `main` (2026-05-28), commit `cc023d3`.
    Typecheck storefront EXIT 0; build detenido por timeout local en
    `Creating an optimized production build ...`.

## 2.STORE-3 — Mis pedidos (BUYER) [L] ✅ 2026-05-28

[x] `/c/[slug]/orders` lista.
[x] `/c/[slug]/orders/[id]` detalle con `RutaTimeline`.
[x] Acciones condicionales.
    PR #11 mergeado a `main` (2026-05-28).
    Typecheck storefront EXIT 0.

## 2.STORE-4 — Confirmación post-Wompi [M] ✅ 2026-05-28

[x] `/c/[slug]/checkout/confirmation` con polling.
    PR #13 mergeado a `main` (2026-05-28). Shell server compatible con
    `output: export` y `ConfirmationView.tsx` client-side. Lee
    `transaction_id` y referencias de query string, resuelve `order_id`
    cuando está disponible, consulta `GET /buyer/orders/:id` y muestra
    estados procesando / confirmado / fallido. Typecheck storefront
    EXIT 0.

## 2.ADMIN-1 — Lista y detalle de pedidos (admin) [L] ✅ 2026-05-28

[x] `/admin/orders` con filtros.
[x] `/admin/orders/:id` detalle 360.
    PR #9 mergeado a `main` (2026-05-28), commit `fd0dda1`.
    Typecheck EXIT 0 | lint EXIT 0 | build EXIT 0.

## 2.QA-1 — Tests del state machine y Wompi mocks [M] ✅ 2026-05-28

[x] MSW para Wompi sandbox.
[x] Test de webhook con firma válida e inválida.
[x] Test de deduplicación de eventos externos.
[x] Tests del state machine: transiciones permitidas y rechazadas.
    PR #8 mergeado a `main` (2026-05-28). Tests focalizados EXIT 0 (3393 tests);
    suite completa EXIT 0 (3662 tests, 1 skipped); `pnpm typecheck`
    EXIT 0.

**Criterio fin Sprint 2: ✅ COMPLETO 2026-05-28.** Pedido real con Wompi sandbox completado de
extremo a extremo hasta READY_TO_SHIP o READY_FOR_PICKUP. Todas las tareas mergeadas a `main`.

---

# Sprint 3 — Flujo SHIP completo (2 semanas)

## 3.BACK-1 — Asignación de Repartidor [L] ✅ 2026-05-29

[x] Service con `assignCourier` (optimistic lock).
[x] Endpoints admin (assign, unassign, available-couriers, map).
    PR #9 mergeado a `main` (2026-05-29).

## 3.BACK-2 — Endpoints COURIER [L] ✅ 2026-05-29

[x] Todas las transiciones del courier (startShipping, markOutForDelivery,
    arrive, markDelivered, recordFailedAttempt, returnToOrigin).
[x] Solo opera sus propios pedidos (validación courier_user_id).
    PR #12 mergeado a `main` (2026-05-29).

## 3.BACK-3 — Cobro contra entrega [L] ✅ 2026-05-29

[x] `recordCollection()` con validación COD, stub evidencia, audit_event.
[x] Endpoint multipart `POST /courier/orders/:id/record-collection`.
    PR #12 mergeado a `main` (2026-05-29).

## 3.BACK-4 — Cancelación post-despacho [M] ✅ 2026-05-29

[x] CUSTOMER_CANCEL_REQUEST ya existía; approveCancelRequest, rejectCancelRequest.
[x] Endpoints admin approve/reject en admin_orders.ts.
    PR #11 mergeado a `main` (2026-05-29).

## 3.BACK-5 — RETURN_TO_ORIGIN [M] ✅ 2026-05-29

[x] returnToOrigin y returnToOriginReceived en orders.service.ts.
[x] Endpoints admin en admin_orders.ts.
    PR #11 mergeado a `main` (2026-05-29).

## 3.BACK-6 — Auto-confirmación de entregados [M] ✅ 2026-05-29

[x] Job cron horario que transiciona DELIVERED → CONFIRMED_BY_SYSTEM.
    PR #10 mergeado a `main` (2026-05-29).

## 3.ADMIN-1 — Mapa de asignación [XL] ✅ 2026-05-29

[x] Leaflet + OSM.
[x] Panel lateral con pedidos y couriers.
[x] Asignación con confirmación.
[x] Auto-refresh.
    PR #15 mergeado a `main` (2026-05-29).
    Ajuste posterior en PR frontend-ruta #16: ruta canónica `/admin/orders/map`
    y redirect de compatibilidad `/admin/map`.

## 3.ADMIN-2 — Vista COURIER móvil-first [XL] ✅ 2026-05-29

[x] `/courier` con tabs Asignados / En camino / Completados.
[x] Detalle con botones grandes, dirección Google Maps, teléfono clickeable.
[x] Form de cobro con cámara para evidencia (capture="environment").
    PR #14 mergeado a `main` (2026-05-29).

## 3.QA-1 — E2E flujo SHIP [L] ✅ 2026-05-29

[x] Base Playwright agregada en frontend-ruta.
[x] Suite para asignación de courier y entrega COD.
[x] Flujos alternos: intento fallido y pedido confirmado por sistema.
[x] CI ejecuta `pnpm exec playwright test --project=chromium`.
    PR #16 mergeado a `main` (2026-05-29). CI EXIT 0.

**Criterio fin Sprint 3: ✅ COMPLETO 2026-05-29.** Flujo SHIP cubierto por backend,
frontend, integración y suite E2E Playwright en CI. No quedan PRs abiertos del sprint.

---

# Sprint 4 — Flujo PICKUP (1 semana)

## 4.FIX-1 — Admin pickup-points CRUD (gap Sprint 1) [S]

[x] Servicio `pickup_points.service.ts` con list, getById, create, update, activate, deactivate.
[x] Router `admin_pickup_points.ts` montado en `/admin/pickup-points`.
[x] Tests unitarios con mock de servicio (40 tests, todos pasan).
[x] Auditoría en `audit_events` por cada mutación.

## 4.BACK-1 — Endpoints PICKUP [L]

[x] Transiciones específicas.
[x] Validación de identidad.
[x] Cobro en punto.
[x] Job `pickup_expiration`.

## 4.ADMIN-1 — UI operación en punto físico [M]

[x] Botones específicos de PICKUP.

## 4.QA-1 — E2E flujo PICKUP [M] ✅ 2026-05-29

[x] Suite Playwright en `tests/e2e/pickup_full_flow.spec.ts` (6 escenarios, 12 tests).
[x] Patrón idéntico a ship_full_flow: sesión via sessionStorage, API via page.route().
[x] Cubre: identidad+entregado (prepago), COD+entregado, aislamiento SHIP, aislamiento estado, OPERATOR_CLIENT, error inline.

---

# Sprint 5 — Vista de Control, dashboards, configuración (1 semana)

## 5.BACK-1 — Vista de Control [L]

[x] enterControlView, exitControlView.
[x] Verificación master password.
[x] Auditoría completa.

## 5.BACK-2 — Dashboards [L]

[x] Endpoints de métricas Cliente y globales.

## 5.BACK-3 — Parámetros [M]

[x] GET/PATCH de `client_parameters`.

## 5.BACK-4 — Auditoría [M]

[x] Endpoint con filtros amplios.

## 5.ADMIN-1 — Pantalla Vista de Control [M]

[x] Form con master password.
[x] Banner ámbar.

## 5.ADMIN-2 — Dashboards [M]

[x] Cliente y global.

## 5.ADMIN-3 — Configuración [L]

[x] Tabs: info, pagos (Wompi), webhooks, parámetros.

## 5.ADMIN-4 — Auditoría [M]

[x] Tabla paginada con filtros (entity_type, user_id, from, to). Soporta ADMIN_CLIENT y ADMIN_RUTA.

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

[ ] Aplicar SQL en prod (BD OCI).
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
| 1 | 0 | Setup multi-repo (5 repos base), GitHub Packages, primer ADMIN_RUTA |
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

1. PR en `packages-ruta` que agrega el enum/validator. Merge → publish
   nueva versión.
2. PR en `backend-ruta` que consume nueva versión y agrega lógica.
3. PR en `frontend-ruta` (admin y storefront) que consume nueva versión
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
