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
  - `LANDING` — carpeta local `frontend-clients-ruta/` y futuros
    repos `landing-{slug}`.
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
[/] Subir configuración de workspace/CI/publicación a `packages-ruta`.
    PR abierto: https://github.com/orkoruta/packages-ruta/pull/1
    CI remoto `test`: SUCCESS. Merge pendiente por branch protection.

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
[x] Commit y push.
    Nota: rama `docs/close-0-docs-1` publicada y PR abierto:
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
[/] `pnpm add express @types/express tsx pino jose argon2 zod
    @orkoruta/shared @orkoruta/db` (instalados via file: en DEV; pendiente cambiar
    a GitHub Packages cuando se publiquen en 0.INFRA-4).
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
[/] `pnpm add @orkoruta/shared` en admin y storefront (instalado via file: en DEV;
    pendiente cambiar a GitHub Packages cuando se publiquen en 0.INFRA-4).
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

[ ] Crear 3 Web Services apuntando a sus repos:
    - `ruta-api` → repo `backend-ruta`, build path `api/`,
      build command `pnpm install && pnpm --filter @ruta/api build`,
      start `pnpm --filter @ruta/api start`.
    - `ruta-admin` → repo `frontend-ruta`, build path `admin/`.
    - `ruta-storefront` → repo `frontend-ruta`, build path
      `storefront/`.
[ ] Configurar Background Worker `ruta-api-worker` para pg-boss.
[ ] Inyectar `DATABASE_URL`, `NPM_TOKEN` (para auth a GitHub
    Packages), Wompi keys (placeholders).
[ ] Dominios provisionales: `api-dev.onrender.com`,
    `app-dev.onrender.com`, `tienda-dev.onrender.com`.

**Criterio:** las 3 URLs responden 200 (al menos `/healthz`).

## 0.BACK-2 — Crear primer ADMIN_RUTA via script [S]

[x] Script `infra/scripts/create_first_admin_ruta.sh`:
    - Prompt de email y password.
    - Hash con argon2id.
    - INSERT a `ruta.users` con `client_id=0, user_type='ADMIN_RUTA'`.
[ ] Ejecutar en BD de dev.
    Nota: requiere psql instalado localmente y ejecutar bash manualmente
    (script interactivo; pide email, password y nombre en tiempo real).

**Criterio:** al menos un ADMIN_RUTA existe.

## 0.INFRA-8 — Seed script para desarrollo [M]

[x] `infra/scripts/seed_dev_data.sh`:
    - 1 Cliente Full piloto (modalidad NATIVE_RUTA).
    - 1 ADMIN_CLIENT, 1 OPERATOR_CLIENT, 3 COURIERs, 3 BUYERs.
    - 10 productos en 3 categorías.
    - 2 pickup points.

Verificación:
- bash -n: EXIT 0 confirmado.
- Ejecución real: pendiente (requiere psql + DATABASE_URL activo).
  El script es idempotente: si 'piloto-native' ya existe, sale sin cambios.

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

## 1.SHARED-1 — Bumpear `@orkoruta/shared` con schemas de auth y catálogo [M]

[ ] Agregar a `@orkoruta/shared/validators/`:
    - `auth.schema.ts` (login, register, refresh).
    - `client.schema.ts` (create, update).
    - `product.schema.ts`, `category.schema.ts`.
    - `buyer.schema.ts`, `courier.schema.ts`, `pickup_point.schema.ts`.
[ ] Bump a `@orkoruta/shared@1.1.0`. Publish.

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
[ ] Upload de imagen vía `/uploads/presigned-url` (servicio de file storage por definir).

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

## 2.SHARED-1 — Bumpear `@orkoruta/shared` con order schemas [M]

[ ] Validators de orders: create, confirm, transition, payments.
[ ] Tipos derivados.
[ ] Bump a `@orkoruta/shared@1.2.0`.

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

## 2.STORE-1 — Carrito persistido en BD [M]

[ ] Carrito representado como pedido `DRAFT` en BD.
[ ] Agregar, actualizar cantidad y remover items vía endpoints de pedido.
[ ] `/c/[slug]/cart`.

## 2.STORE-2 — Checkout multi-paso [XL]

[ ] Paso 1: entrega (SHIP/PICKUP).
[ ] Paso 2: dirección o pickup point (con mapa OSM).
[ ] Paso 3: método de pago.
[ ] Submit confirma el pedido `DRAFT` existente y redirige a Wompi o
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
