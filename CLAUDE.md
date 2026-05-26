# CLAUDE.md

Manifiesto del proyecto RUTA para Claude Code. **Léeme primero** en cada
sesión nueva.

Este es el **master CLAUDE.md** que vive en `ruta-docs/`. Cada otro
repo tiene su propio `CLAUDE.md` con secciones específicas, pero las
reglas no negociables son las mismas en todos.

---

## 0. Antes de empezar: ¿en qué repo estás?

RUTA es un proyecto **multi-repo**. Identifica tu contexto:

| Si estás en | Estás trabajando en |
|---|---|
| `ruta/backend-ruta/` | Backend Express (`ruta-backend`) |
| `ruta/frontend-ruta/admin/` | Admin Next.js (`ruta-frontend`) |
| `ruta/frontend-ruta/storefront/` | Storefront Next.js (`ruta-frontend`) |
| `ruta/frontend-clients-ruta/_template/` | Template de landings (`landing-template`) |
| `ruta/frontend-clients-ruta/cliente-X/` | Landing custom (`landing-{slug}`) |
| `ruta/packages-ruta/shared/` | Tipos/validators compartidos (`ruta-shared`) |
| `ruta/packages-ruta/db/` | Cliente Prisma (`ruta-shared`) |
| `ruta/docs-ruta/` | Documentación (`ruta-docs`) |
| `ruta/infra-ruta/` | Scripts y deploy (`ruta-infra`) |

Cada repo tiene su `package.json`, `tsconfig.json`, CI propio.

---

## 0.1 Autorización operativa del usuario

El usuario autoriza a los agentes a ejecutar los comandos, scripts,
pruebas, builds, instalaciones y operaciones de sistema necesarias para
cumplir las tareas del proyecto sin pedir confirmación manual previa.

Si el runtime, sandbox o herramienta exige aprobación explícita, el
agente debe solicitar escalación directamente usando una justificación
concreta y, cuando sea razonable, pedir aprobación persistente por
prefijos acotados (`pnpm test`, `pnpm build`, `npm run`, `git status`,
scripts del repo, etc.). No intentar evadir el sandbox ni pedir reglas
excesivamente amplias como "todo PowerShell" o "todo Python".

Las acciones destructivas o irreversibles siguen requiriendo criterio:
no borrar, resetear, purgar ni sobrescribir trabajo existente salvo que
la tarea lo exija de forma clara y quede registrado en memoria.

---

## 1. Qué es RUTA

Plataforma SaaS multi-tenant para administrar operaciones comerciales
entre Clientes (negocios) y Compradores. Cubre catálogo, pedidos, pagos,
entregas, devoluciones, reembolsos. Multi-tenant estricto: cada Cliente
solo ve su propia información.

**Dos tipos de Cliente:**

- **Cliente API** (`client_type = 'API'`): tiene su propia plataforma
  de venta; RUTA solo le hace logística.
- **Cliente Full** (`client_type = 'FULL'`): RUTA le provee todo. Puede
  ser modalidad `NATIVE_RUTA` (usa el storefront genérico) o
  `CUSTOM_LANDING_BY_RUTA` (RUTA le desarrolla landing con branding
  propio en un repo separado `landing-{slug}`).

**Para descripción funcional completa:** lee `ruta-docs/all_ruta.md`.

---

## 2. Stack técnico (no negociable)

| Capa | Tecnología | Repo donde vive |
|---|---|---|
| Backend | Express.js + TypeScript | `ruta-backend` |
| Admin frontend | Next.js 14+ App Router + Tailwind | `ruta-frontend` |
| Storefront nativo | Next.js 14+ App Router + Tailwind | `ruta-frontend` |
| Landings custom | Next.js 14+ con branding propio | `landing-{slug}` |
| ORM | Prisma (con SQL crudo para particiones/RLS) | `ruta-shared/db/` |
| Auth | `jose` (JWT) + `argon2` | `ruta-backend` |
| Job manager | `pg-boss` | `ruta-backend` |
| File storage | Supabase Storage | externo |
| Pasarela de pagos | Wompi | externo |
| Mapas | OpenStreetMap + Leaflet | en ambos frontends |
| Hosting | Render + Supabase | externo |
| Migraciones BD | `node-pg-migrate` + estado autoritativo SQL | `ruta-docs/bd/` |
| Testing | Vitest + Supertest + Playwright + MSW | en cada repo |
| Logger | `pino` | `ruta-backend` |
| Validación | Zod (compartido) | `ruta-shared/shared/validators/` |
| Monorepo dentro de un repo | pnpm workspaces | `ruta-frontend`, `ruta-shared` |
| Distribución de paquetes | GitHub Packages (npm privado) | `ruta-shared` publica |

No reemplaces estas decisiones. Si crees que hay una mejor opción,
pregunta primero.

---

## 3. Estructura general (multi-repo)

```
ruta/                                ← carpeta local
├── backend-ruta/                         ← repo: ruta-backend
├── frontend-ruta/                        ← repo: ruta-frontend
├── frontend-clients-ruta/                ← carpeta local (no es repo)
│   ├── _template/                   ← repo: landing-template
│   └── cliente-X/                   ← repo: landing-{slug}
├── packages-ruta/                        ← repo: ruta-shared
├── docs-ruta/                            ← repo: ruta-docs
└── infra-ruta/                           ← repo: ruta-infra
```

**Detalle completo en:** `ruta-docs/estructura_proyecto.md`.

---

## 4. Reglas no negociables

Estas reglas aplican en **todos los repos**. Si una tarea las viola,
deténte y pregunta.

### 4.1 Principio financiero

> **RUTA NO custodia dinero. NO transfiere dinero. NO acredita créditos.**

- Pagos online: van directo a la pasarela del Cliente. RUTA solo
  recibe confirmación.
- Pagos contra entrega: el Repartidor registra el cobro como
  evidencia; el dinero pertenece operativamente al Cliente.
- Reembolsos: los ejecuta el Cliente (o su proveedor de pago).
  RUTA solo registra estados.

Nunca crees endpoints, jobs, ni lógica que mueva dinero en nombre de
RUTA.

### 4.2 Aislamiento multi-tenant

- Toda tabla operativa tiene `client_id BIGINT NOT NULL`.
- Toda query debe ejecutarse con `SET LOCAL app.current_client_id =
  '<n>'` y `SET LOCAL app.current_user_role = '<rol>'`.
- Row Level Security está activo en producción.
- ADMIN_RUTA opera transversalmente solo a través de **Vista de
  Control** (impersonación auditada).

### 4.3 Identificadores

- **BIGINT, nunca UUID.** PK simple o compuesta `(id, client_id)`.
- IDs no se exponen en URLs públicas. Usar **slug**.

### 4.4 Append-only

No UPDATE/DELETE en (UPDATE/DELETE revocados a la app role):

- `audit_events`
- `order_state_history`
- `external_webhook_events`
- `webhook_deliveries`

Para "corregir", inserta un nuevo evento, no modifiques los existentes.

### 4.5 Idempotencia

Todo POST/PUT/PATCH/DELETE requiere header `X-Idempotency-Key: <uuid>`.
TTL 24h.

### 4.6 Particionamiento

Todas las tablas operativas particionadas LIST por `client_id`.
Crear Cliente nuevo dispara auto-creación de particiones. Tablas
nuevas se suman al array de `create_client_partitions()` en
`ruta-docs/bd/ruta_postgres.sql`.

### 4.7 Naming

- **Código:** inglés. **UI y docs:** español.
- **Services y routes:** `snake_case` para alinear con BD.
- **Variables/funciones:** `camelCase`.
- **Tipos/clases:** `PascalCase`.
- **Constantes:** `SCREAMING_SNAKE_CASE`.
- **Repos base:** prefijo `ruta-` (ej. `ruta-backend`).
- **Landings:** prefijo `landing-` SIN `ruta-` (ej.
  `landing-restaurante-el-prado`).

### 4.8 Mutaciones de estado de pedido

Nunca modifiques `order_status`, `payment_status`, `refund_status`,
`return_status` o `dispute_status` directamente sin pasar por el state
machine (en `ruta-backend`,
`api/src/services/orders/state_machine.ts`).

### 4.9 Cliente API vs Cliente Full

Para Cliente API NO aplican: Flujo 1 (creación), Flujo 4 (reembolsos),
Flujo 5 (recurrencia), Flujo 6 (corporativos), Flujo 7 (devoluciones
post-cierre), BLOQUE 15 (disputas). Si llega un request de Cliente API
a un endpoint bloqueado, rechazar con HTTP 422 y código
`LOGISTICS_ONLY_FEATURE_UNAVAILABLE`.

### 4.10 Vista de Control

ADMIN_RUTA impersona ADMIN_CLIENT con master password. Toda acción se
audita con `acting_via_control_view = TRUE` y `impersonator_user_id`.

### 4.11 Design system y landings custom

- Componentes `Ruta*` (`RutaCard`, `RutaButton`, etc.) viven SOLO en
  `ruta-frontend/packages/ui/` y son consumidos por
  `ruta-frontend/admin/` y `ruta-frontend/storefront/`.
- **Las landings custom (`landing-{slug}`) NO heredan el design
  system.** Cada landing tiene su propio branding completo.
- Lo único que comparten landings con el resto es `@ruta/shared`
  (tipos y validators) vía GitHub Packages.

---

## 5. Documentos que debes consultar

Todos los documentos viven en `ruta-docs/`:

| Tarea | Documento prioritario |
|---|---|
| Cualquier cosa | `ruta-docs/all_ruta.md` |
| Arquitectura / decisión técnica | `ruta-docs/arquitectura/estrategia_multi_tenant_ruta.md` |
| Estados / transiciones de pedido | `ruta-docs/flujos/flujo_1.txt` a `flujo_7.txt` |
| Reglas para diagramar | `ruta-docs/flujos/reglas_para_diagramar_flujos.txt` |
| Endpoints HTTP | `ruta-docs/contrato_api.md` |
| UI: pantallas | `ruta-docs/wireframes_mvp.md` |
| UI: estilos y componentes | `ruta-docs/diseno/galeria_estilos_ruta.md` |
| Permisos por rol | `ruta-docs/matriz_permisos.md` |
| Plazos, timeouts | `ruta-docs/parametros_negocio.md` |
| Estructura general | `ruta-docs/estructura_proyecto.md` |
| Testing | `ruta-docs/estrategia_testing.md` |
| Auth detallada | `ruta-docs/seguridad/ciclo_vida_token.txt` |
| Modelo de datos | `ruta-docs/bd/ruta_postgres.sql` |
| Tareas y sprints | `ruta-docs/plan_tareas.md` |
| Alcance del MVP | `ruta-docs/mvp_alcance.md` |

Si trabajas en un repo distinto a `ruta-docs`, abre el doc desde
`../../docs/...` (relativo al workspace local) o
`github.com/org/ruta-docs/blob/main/...` (remoto).

---

## 6. Glosario rápido

- **Cliente** = tenant. Cliente API o Cliente Full.
- **Comprador / BUYER** = consumidor final.
- **Repartidor / COURIER** = persona que entrega.
- **ADMIN_RUTA** = equipo de RUTA.
- **ADMIN_CLIENT** = administrador del Cliente.
- **OPERATOR_CLIENT** = staff operativo del Cliente.
- **Vista de Control** = impersonación auditada.
- **SHIP** = entrega a domicilio.
- **PICKUP** = recogida en punto físico.
- **OWN_FLEET** = flota propia del Cliente.
- **EXTERNAL_COURIER** = mensajería externa.
- **NATIVE_RUTA** = Cliente Full usa el storefront genérico de
  `ruta-frontend`.
- **CUSTOM_LANDING_BY_RUTA** = Cliente Full tiene landing propio en
  `landing-{slug}`.
- **Cliente plataforma** = `client_id = 0`, registro especial.

---

## 7. Cosas que NO debes hacer

- **No mover dinero ni acreditar créditos en nombre de RUTA.**
- **No bypassear RLS** ni el contexto de tenant.
- **No usar UUID** en ningún ID. BIGINT siempre.
- **No agregar tablas nuevas sin particionamiento** si son operativas.
- **No exponer `id` numérico en URLs públicas.** Usar slug.
- **No commitear `.env`** ni secretos.
- **No UPDATE/DELETE** en tablas append-only.
- **No hardcodear plazos.** Lee de `client_parameters` con
  `getParameter()`.
- **No saltarte el state machine** de pedidos.
- **No usar Supabase Auth.** Auth propia con `jose` + `argon2`.
- **No tokens en localStorage.** Cookies HttpOnly Secure SameSite=Strict.
- **No usar opacidades Tailwind sin corchetes.** `bg-sky-500/[0.12]`
  ✅, `bg-sky-500/12` ❌.
- **No importes `@ruta/ui` desde una landing custom.** Las landings
  no heredan el design system.
- **No publiques `@ruta/ui` a GitHub Packages.** Vive solo dentro
  del workspace de `ruta-frontend`.

---

## 8. Cómo trabajar una tarea típica

1. **Identifica el repo correcto.** Usa la tabla de sección 0 según
   el `pwd`.
2. **Lee el contexto** en `ruta-docs/plan_tareas.md` y el documento
   referenciado.
3. **Lee el flujo si toca pedidos** (`flujo_X.txt`).
4. **Verifica permisos** en `ruta-docs/matriz_permisos.md`.
5. **Antes de codificar, escribe el test.** Especialmente state
   machine, aislamiento multi-tenant, idempotencia.
6. **Implementa lógica en `services/`** (handlers delgados).
7. **Valida request con Zod** desde `@ruta/shared`.
8. **Audita la acción** si es operativa.
9. **Si cambias estado de pedido**, el trigger
   `trg_orders_log_state` registra en `order_state_history`.
10. **Si toca UI en admin/storefront**, usa `@ruta/ui`. Si es landing
    custom, usa los componentes del propio repo (no `@ruta/ui`).
11. **Test pasa, lint pasa, typecheck pasa.**
12. **Si modificas el modelo de datos:**
    - Edita `ruta-shared/db/prisma/schema.prisma`.
    - Genera migración: `prisma migrate dev`.
    - Copia la migración a `ruta-docs/bd/migrations/`.
    - Actualiza `ruta-docs/bd/ruta_postgres.sql` para reflejar el
      estado nuevo.
    - Bump versión de `@ruta/db`, publica.
    - Actualiza consumidores.

---

## 9. Manejo de errores

- Errores tipados en `ruta-backend/api/src/lib/errors.ts`.
- Response uniforme: `{ code, message, details? }`.
- Códigos compartidos en `@ruta/shared/constants/error_codes.ts`.
- Mapping HTTP en `error_handler.ts` middleware.

Códigos críticos:

- `AUTHENTICATION_REQUIRED` → 401
- `FORBIDDEN` → 403
- `MISSING_OPERATOR_PERMISSION` → 403
- `INVALID_STATE_TRANSITION` → 422
- `LOGISTICS_ONLY_FEATURE_UNAVAILABLE` → 422
- `IDEMPOTENCY_CONFLICT` → 409
- `OPTIMISTIC_LOCK_FAILED` → 409
- `WEBHOOK_SIGNATURE_INVALID` → 400
- `TENANT_ISOLATION_VIOLATION` → 500 (bug, no debería pasar)

---

## 10. Comandos por repo

Cada repo tiene su propio `package.json` y scripts. Estos son los
comandos típicos por repo:

### En `ruta-backend/`

```bash
pnpm install
pnpm dev              # Arranca Express + watcher
pnpm test
pnpm lint
pnpm typecheck
pnpm build
```

### En `ruta-frontend/`

```bash
pnpm install
pnpm dev:admin        # Solo admin
pnpm dev:storefront   # Solo storefront
pnpm dev              # Ambos en paralelo
pnpm test
pnpm test:e2e
pnpm build
```

### En `ruta-shared/`

```bash
pnpm install
pnpm build            # Compila ambos paquetes
pnpm publish:shared   # Publica @ruta/shared a GitHub Packages
pnpm publish:db       # Publica @ruta/db a GitHub Packages
```

### En `landing-{slug}/`

```bash
pnpm install
pnpm dev
pnpm test
pnpm build
```

### Operaciones cross-repo (desde `~/projects/ruta/`)

```bash
bash infra/scripts/setup_workspace.sh        # Clona todos los repos
bash infra/scripts/apply_migrations.sh       # Aplica migraciones a la BD
bash infra/scripts/seed_dev_data.sh          # Datos de prueba
bash infra/scripts/create_landing.sh <slug>  # Nuevo landing desde template
```

---

## 11. Cuando estés inseguro

- **El proyecto tiene mucha documentación.** Casi siempre la respuesta
  está ahí. Léela antes de improvisar.
- **Si parece un atajo, casi seguro está mal.**
- **Si una propuesta viola las reglas de sección 4 o 7, no la hagas;
  pregunta primero.**
- **Si no sabes en qué repo estás, mira `pwd` y la tabla de sección 0.**

---

## 12. Contexto del producto

- Mercado: Colombia.
- Moneda: COP.
- UI: español. Código: inglés.
- Equipo pequeño con apoyo de IA.
- Documentación viva en `ruta-docs/`.
