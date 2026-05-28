# Plan de Paralelismo — Sprint 1 Fase 2

Documento autoritativo para coordinar agentes IA trabajando en paralelo.
Cada agente debe leer este archivo completo antes de empezar su tarea.

**Fuente de verdad de tareas:** `docs-ruta/plan_tareas.md`
**Estado actual:** Sprint 0 ✅ cerrado. Sprint 1 ✅ CERRADO 2026-05-28 — todas las tareas
completadas y mergeadas (BACK-5 #3, ADMIN-2 #7, ADMIN-3 #6, ADMIN-4 #8).
CI de `backend-ruta/main` y `frontend-ruta/main` verde.
Para Sprint 2 en adelante ver `docs-ruta/plan_paralelismo_sprints2_6.md`.

---

## Reglas de coordinación (OBLIGATORIO leer antes de empezar)

1. **Cada agente tiene jurisdicción exclusiva** sobre los archivos listados en su sección.
2. **Nunca modificar archivos de otro agente.** Si necesitas algo de otro agente, impórtalo, no lo edites.
3. **Archivos `lib/*.api.ts`:** cada agente crea el suyo propio con nombre único. Nunca dos agentes tocan el mismo archivo de lib.
4. **Branch propia por tarea:** `feat/back-5`, `feat/admin-1`, `feat/store-1`, `feat/store-2`, `feat/qa-1`, etc.
5. **No crear `index.ts` barrels compartidos** — pueden causar conflictos de merge.
6. **Wave 2 no arranca hasta que ADMIN-1 esté mergeado a main.**
7. **Definition of Done:** typecheck limpio + tests pasando + lint OK antes de hacer PR.

---

## Protocolo de agente especializado (TODOS los agentes, OBLIGATORIO antes de codificar)

Actúa como un agente especializado dentro de este proyecto.

Antes de realizar cualquier tarea, debes revisar obligatoriamente toda la documentación relacionada con tu trabajo, incluyendo archivos `.md`, `.txt`, flujos `.md`, flujos `.txt`, documentación funcional, documentación técnica, reglas de negocio y cualquier definición previa del proyecto.

**No puedes inventar** funcionalidades, reglas, nombres, entidades, rutas, pantallas, roles, permisos, validaciones, estados, procesos, textos de interfaz ni comportamientos que no estén definidos en la documentación existente.

Debes identificar qué documentos aplican específicamente a tu tarea y seguirlos de forma obligatoria. Si existe un flujo documentado, debes respetarlo exactamente. No puedes cambiar, simplificar, omitir ni reemplazar un flujo existente sin autorización explícita del usuario.

### Paso 1 — Análisis previo (entregar ANTES de escribir código)

Antes de implementar o modificar cualquier archivo, debes entregar este análisis:

**1. Documentación revisada:**
- Lista de archivos `.md` revisados.
- Lista de archivos `.txt` revisados.
- Lista de flujos revisados.

**2. Reglas detectadas:**
- Reglas de negocio aplicables.
- Restricciones técnicas aplicables.
- Decisiones ya definidas que no pueden modificarse.

**3. Flujos obligatorios:**
- Flujos que gobiernan esta tarea.
- Orden de pasos que debe respetarse.

**4. Vacíos o contradicciones:**
- Indica cualquier punto que no esté documentado.
- Indica cualquier contradicción entre documentos.
- Si falta información, no inventes una solución.

**5. Plan de ejecución:**
- Explica cómo vas a resolver la tarea respetando la documentación existente.

### Paso 2 — Prioridad durante la ejecución

Durante la ejecución, sigue este orden de prioridad estricto:

1. Instrucciones directas del usuario.
2. Reglas de negocio documentadas (`docs-ruta/CLAUDE.md`, `docs-ruta/all_ruta.md`).
3. Flujos funcionales `.md` o `.txt` (`docs-ruta/flujos/`).
4. Documentación técnica (`contrato_api.md`, `estrategia_testing.md`, `matriz_permisos.md`).
5. README principal del proyecto.
6. Convenciones de código existentes en el repo.
7. Buenas prácticas generales, **solo si no contradicen** la documentación.

### Paso 3 — Bloqueo ante información faltante

Si no encuentras una definición documentada para una decisión, **detente** y reporta exactamente así:

> "No encontré una definición documentada para: [descripción]. No voy a inventar una solución. Se requiere confirmación del usuario o agregar esta definición a la documentación del proyecto."

No continúes hasta recibir respuesta.

### Paso 4 — Validación de cumplimiento (entregar al terminar, antes del PR)

Al finalizar, entrega esta validación:

| Criterio | Estado |
|----------|--------|
| Documentación revisada y respetada | Sí / No |
| Flujos existentes respetados | Sí / No |
| Reglas nuevas inventadas | No (siempre No) |
| Cambios fuera del alcance | No (siempre No) |
| Contradicciones encontradas | Sí / No (detallar) |
| Resumen de cambios realizados | [descripción] |

Tu objetivo es ejecutar la tarea sin desviarte de las definiciones existentes del proyecto, evitando suposiciones y garantizando que todo lo implementado esté respaldado por la documentación oficial del proyecto.

### Documentación mínima a revisar por track

| Track | Documentos obligatorios |
|-------|------------------------|
| Backend (A, E) | `CLAUDE.md`, `contrato_api.md`, `all_ruta.md`, `matriz_permisos.md`, `parametros_negocio.md`, `estrategia_testing.md`, `bd/ruta_postgres.sql`, flujos relevantes en `flujos/` |
| Admin frontend (B, F, G, H) | `CLAUDE.md`, `wireframes_mvp.md`, `diseno/galeria_estilos_ruta.md`, `matriz_permisos.md`, `contrato_api.md`, `all_ruta.md` |
| Storefront (C, D) | `CLAUDE.md`, `wireframes_mvp.md`, `diseno/galeria_estilos_ruta.md`, `contrato_api.md`, `all_ruta.md`, `flujos/flujo_1.txt` |
| QA (E) | `CLAUDE.md`, `estrategia_testing.md`, `arquitectura/estrategia_multi_tenant_ruta.md`, `matriz_permisos.md`, `contrato_api.md` |

---

## Registro de avance obligatorio (TODOS los agentes, sin excepción)

Al terminar tu tarea — antes de abrir el PR — debes actualizar los cuatro archivos de memoria del proyecto. Sin este paso, la tarea **no está terminada**.

### Archivos a actualizar

#### 1. `docs-ruta/plan_tareas.md`
- Cambia el `[ ]` de tu tarea a `[x]`.
- Ejemplo: `## 1.BACK-5 — Importación masiva por Excel [M]` → marcar `[x]` en todos sus sub-items completados.

#### 2. `docs-ruta/memoria_proyecto_ruta.md`
- Agrega una entrada en la sección correspondiente al Sprint 1 con:
  - Qué tarea completaste (`1.BACK-5`, `1.ADMIN-1`, etc.)
  - Qué archivos nuevos creaste (lista breve)
  - Qué tests pasaron y cuántos
  - Cualquier decisión técnica no obvia que tomaste

#### 3. Archivo de memoria del sprint en `/home/alexander_marquez/.claude/projects/-mnt-c-Dropbox-Alex-DEV-ruta/memory/project_sprint1_status.md`
- Mueve tu tarea de la sección `## PENDIENTE [ ]` a `## HECHO [x]`.
- Describe brevemente lo implementado (misma estructura que las tareas ya completadas en ese archivo).
- Actualiza el contador de tests en `## ESTADO GENERAL DEL BACKEND` si aplica.

#### 4. `/home/alexander_marquez/.claude/projects/-mnt-c-Dropbox-Alex-DEV-ruta/memory/MEMORY.md`
- Si el estado del sprint cambió significativamente, actualiza la línea del sprint en el índice.
- Ejemplo: `— 1.SHARED-1 + 1.BACK-1/2/3/4 completos (42 tests); pendiente 1.BACK-5, frontend, QA` → ajustar con lo nuevo.

### Orden de actualización

```
1. Termina el código → pasa todos los checks (ver sección siguiente)
2. Actualiza plan_tareas.md         (marcar [x])
3. Actualiza memoria_proyecto_ruta.md
4. Actualiza project_sprint1_status.md
5. Actualiza MEMORY.md (si aplica)
6. Abre el PR
```

No abras el PR sin haber actualizado los 4 archivos.

---

## Protocolo de calidad — cero errores, cero warnings (TODOS los agentes)

Antes de actualizar la memoria y abrir el PR, ejecuta esta checklist completa. **Todos los ítems deben pasar en verde.** Si alguno falla, corrígelo antes de continuar — no dejes deuda técnica para el siguiente agente.

### Checklist universal (aplica a TODOS los agentes)

```
[ ] pnpm typecheck          → EXIT 0, cero errores de tipos
[ ] pnpm lint               → EXIT 0, cero warnings ni errores ESLint/Prettier
[ ] pnpm test               → todos los tests pasan, cero fallos, cero skipped sin justificación
[ ] pnpm build              → EXIT 0, sin errores de compilación (frontend) o tsc (backend)
[ ] Sin console.log         → no dejar console.log de depuración en archivos de producción
[ ] Sin any implícito       → no usar `any` sin justificación explícita en comentario
[ ] Sin TODO sin ticket     → no dejar comentarios TODO sin referenciar una tarea del plan
[ ] Sin secrets hardcodeados → URLs, tokens o passwords hardcodeados: prohibido
```

### Checklist backend (Agentes A y E)

```
[ ] Todos los nuevos endpoints tienen validación Zod del body/query
[ ] Todos los nuevos endpoints POST/PUT/PATCH/DELETE verifican X-Idempotency-Key
[ ] Todos los nuevos endpoints protegidos verifican autenticación con requireAuth / requireAdminClient / requireAdminRuta
[ ] Las queries a BD usan withTenant() — nunca queries globales sin contexto de tenant
[ ] Los nuevos tests cubren: caso feliz + caso de error de validación + caso de autenticación fallida
[ ] No hay warnings de Prisma en los logs de test
```

### Checklist frontend (Agentes B, C, D, F, G, H)

```
[ ] pnpm build no genera warnings de Next.js (Image sin alt, Link deprecated, etc.)
[ ] No hay errores de hidratación (SSR/CSR mismatch)
[ ] Los formularios muestran errores de validación al usuario (no fallan silenciosamente)
[ ] Las llamadas a la API manejan el caso de error (401, 403, 500) con feedback al usuario
[ ] No hay imports de @orkoruta/ui desde storefront o landing (solo desde admin)
[ ] No hay tokens en localStorage — solo cookies HttpOnly vía el backend
[ ] Las páginas protegidas redirigen a /login si no hay sesión activa
```

### Si encuentras un problema en código de otro agente

- **No lo corrijas** directamente (violarías jurisdicción).
- Documéntalo en el PR como comentario: `HALLAZGO: [descripción del problema] en [archivo:línea]`.
- El humano decide qué agente lo resuelve.

---

## WAVE 1 — Agentes que arrancan simultáneamente

### AGENTE A — Tarea: 1.BACK-5

**Repo:** `backend-ruta/`
**Rama:** `feat/back-5`

**Qué hacer:**
- `POST /admin/products/bulk-import` (multipart, sube archivo Excel)
- Job pg-boss `bulk_import` para procesar asíncrono
- `GET /admin/products/bulk-import/:job_id` para consultar status

**Archivos que SOLO este agente crea/modifica:**
```
backend-ruta/api/src/
├── services/bulk_import.service.ts    ← NUEVO
├── jobs/bulk_import.job.ts            ← NUEVO
└── routes/admin_products_bulk.ts      ← NUEVO
```

**Archivos compartidos (tocar con cuidado):**
- `backend-ruta/api/src/app.ts` — solo agregar el import y el `app.use()` de la nueva ruta bulk. No modificar nada más. Marcar el bloque con comentario `// 1.BACK-5`.

**NO tocar:** `routes/admin_products.ts`, ningún otro archivo existente.

**Tests:** crear `src/__tests__/bulk_import.test.ts` (nuevo archivo).

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.BACK-5] Importación masiva por Excel`.

---

### AGENTE B — Tarea: 1.ADMIN-1

**Repo:** `frontend-ruta/` (trabajar dentro de `admin/`)
**Rama:** `feat/admin-1`

**Qué hacer:**
- Página `/login` con detección de rol (ADMIN_RUTA vs ADMIN_CLIENT) y redirect
- `RutaSidebar` con navegación condicional por rol
- `RutaHeader` con info de cliente activo y banner Vista de Control
- Layout root del admin y layout `(protected)/` que envuelve sidebar

**Archivos que SOLO este agente crea/modifica:**
```
frontend-ruta/admin/
├── app/
│   ├── layout.tsx                     ← NUEVO (root layout)
│   ├── login/
│   │   └── page.tsx                   ← NUEVO
│   └── (protected)/
│       └── layout.tsx                 ← NUEVO (envuelve sidebar/header)
├── components/
│   ├── RutaSidebar.tsx                 ← NUEVO
│   └── RutaHeader.tsx                  ← NUEVO
└── lib/
    └── auth.api.ts                     ← NUEVO (fetch /auth/login, /auth/ruta-admin/login, /auth/refresh)
```

**NO tocar:** ningún otro archivo existente del repo.

**Nota importante:** El `(protected)/layout.tsx` es el que los Agentes F, G y H de Wave 2 van a usar como wrapper. Asegurarse de que quede bien definido antes de hacer merge.

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR **mergeado a main** (es prerrequisito de Wave 2 — avisar al humano para que mergee antes de lanzar F/G/H).

---

### AGENTE C — Tarea: 1.STORE-1

**Repo:** `frontend-ruta/` (trabajar dentro de `storefront/`)
**Rama:** `feat/store-1`

**Qué hacer:**
- Layout `c/[slug]/layout.tsx` con branding del cliente (lee slug, carga nombre/logo del cliente)
- `/c/[slug]/` catálogo con grid de productos y filtros
- `/c/[slug]/product/[id]` detalle de producto

**Archivos que SOLO este agente crea/modifica:**
```
frontend-ruta/storefront/
├── app/c/[slug]/
│   ├── layout.tsx                     ← NUEVO (branding, slug context)
│   ├── page.tsx                       ← NUEVO (catálogo)
│   └── product/
│       └── [id]/
│           └── page.tsx               ← NUEVO (detalle)
└── lib/
    └── catalog.api.ts                  ← NUEVO (fetch /public/clients/:slug/products, /categories, /pickup-points)
```

**NO tocar:** `app/c/[slug]/(auth)/` (es zona del Agente D) · ningún otro archivo existente.

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.STORE-1] Layout y catálogo storefront`.

---

### AGENTE D — Tarea: 1.STORE-2

**Repo:** `frontend-ruta/` (trabajar dentro de `storefront/`)
**Rama:** `feat/store-2`

**Qué hacer:**
- Página `/c/[slug]/(auth)/register` — formulario de registro de BUYER
- Página `/c/[slug]/(auth)/login` — login de BUYER
- Layout propio del grupo `(auth)`

**Archivos que SOLO este agente crea/modifica:**
```
frontend-ruta/storefront/
├── app/c/[slug]/(auth)/
│   ├── layout.tsx                     ← NUEVO (layout del grupo auth, independiente)
│   ├── login/
│   │   └── page.tsx                   ← NUEVO
│   └── register/
│       └── page.tsx                   ← NUEVO
└── lib/
    └── auth.api.ts                     ← NUEVO (fetch /auth/login, /auth/register — storefront)
```

**NO tocar:** `app/c/[slug]/layout.tsx` (es del Agente C) · `lib/catalog.api.ts` (es del Agente C) · ningún otro archivo.

**Nota:** `app/c/[slug]/layout.tsx` ya existe (creado por Agente C) — heredarlo automáticamente por jerarquía de Next.js App Router. El `(auth)/layout.tsx` de este agente es el layout del grupo de rutas de autenticación solamente.

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.STORE-2] Registro y login BUYER storefront`.

---

### AGENTE E — Tarea: 1.QA-1

**Repo:** `backend-ruta/`
**Rama:** `feat/qa-1`

**Qué hacer:**
- Suite de tests que valida aislamiento cross-tenant en todos los endpoints existentes
- Verificar que un token de client_id=1 no puede acceder a recursos de client_id=2
- Gate en CI

**Archivos que SOLO este agente crea/modifica:**
```
backend-ruta/api/src/
└── __tests__/
    └── isolation.test.ts               ← NUEVO
```

**Solo LECTURA de:** todos los demás archivos existentes (routes, services, middleware).

**NO modificar** ningún archivo de producción. Si encuentra un bug de aislamiento, registrarlo como test fallido con descripción clara, no corregirlo (eso es una tarea de backend separada).

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad", checklist backend) · todos los tests de aislamiento pasan o los fallos quedan documentados como hallazgos en el PR · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.QA-1] Tests de aislamiento cross-tenant`.

---

## WAVE 2 — Arrancan cuando AGENTE B (ADMIN-1) está mergeado a main

Los tres agentes de esta wave pueden correr simultáneamente entre sí.
Todos trabajan en `frontend-ruta/admin/` pero en rutas completamente separadas.

Estado 2026-05-27: ADMIN-1 ya está mergeado en `frontend-ruta/main`;
Wave 2 puede arrancar. Antes de empezar: hacer `git pull origin main`
para obtener el trabajo de ADMIN-1.

---

### AGENTE F — Tarea: 1.ADMIN-2

**Repo:** `frontend-ruta/` (trabajar dentro de `admin/`)
**Rama:** `feat/admin-2`

**Qué hacer:**
- `/ruta-admin/clients` — lista de clientes con filtros
- `/ruta-admin/clients/new` — formulario de creación
- `/ruta-admin/clients/[id]` — detalle y edición

**Archivos que SOLO este agente crea/modifica:**
```
frontend-ruta/admin/
├── app/(protected)/ruta-admin/clients/
│   ├── page.tsx                        ← NUEVO (lista)
│   ├── new/
│   │   └── page.tsx                    ← NUEVO (crear)
│   └── [id]/
│       └── page.tsx                    ← NUEVO (detalle/editar)
└── lib/
    └── clients.api.ts                   ← NUEVO (fetch /ruta-admin/clients/*)
```

**Solo LECTURA de:** `components/RutaSidebar.tsx`, `components/RutaHeader.tsx`, `app/(protected)/layout.tsx` (creados por ADMIN-1).

**NO tocar:** ningún archivo de los Agentes G o H.

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.ADMIN-2] Pantallas ADMIN_RUTA — Clientes`.

---

### AGENTE G — Tarea: 1.ADMIN-3

**Repo:** `frontend-ruta/` (trabajar dentro de `admin/`)
**Rama:** `feat/admin-3`

**Qué hacer:**
- `/admin/products` — lista con filtros
- `/admin/products/new` y `/admin/products/[id]` — formularios con upload de imagen
- Modal de import masivo (conecta con el job de 1.BACK-5)

**Archivos que SOLO este agente crea/modifica:**
```
frontend-ruta/admin/
├── app/(protected)/admin/products/
│   ├── page.tsx                        ← NUEVO (lista + filtros)
│   ├── new/
│   │   └── page.tsx                    ← NUEVO
│   ├── [id]/
│   │   └── page.tsx                    ← NUEVO
│   └── _components/
│       └── BulkImportModal.tsx         ← NUEVO
└── lib/
    └── products.api.ts                  ← NUEVO (fetch /admin/products/*, /admin/categories/*, bulk-import)
```

**Solo LECTURA de:** `components/RutaSidebar.tsx`, `components/RutaHeader.tsx`, `app/(protected)/layout.tsx`.

**NO tocar:** ningún archivo de los Agentes F o H.

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.ADMIN-3] Gestión de productos frontend`.

---

### AGENTE H — Tarea: 1.ADMIN-4

**Repo:** `frontend-ruta/` (trabajar dentro de `admin/`)
**Rama:** `feat/admin-4`

**Qué hacer:**
- `/admin/buyers` — lista + detalle/edición
- `/admin/couriers` — lista + detalle/edición
- `/admin/pickup-points` — lista + detalle/edición

**Archivos que SOLO este agente crea/modifica:**
```
frontend-ruta/admin/
├── app/(protected)/admin/
│   ├── buyers/
│   │   ├── page.tsx                    ← NUEVO (lista)
│   │   └── [id]/
│   │       └── page.tsx               ← NUEVO (detalle)
│   ├── couriers/
│   │   ├── page.tsx                    ← NUEVO
│   │   └── [id]/
│   │       └── page.tsx               ← NUEVO
│   └── pickup-points/
│       ├── page.tsx                    ← NUEVO
│       └── [id]/
│           └── page.tsx               ← NUEVO
└── lib/
    └── users.api.ts                     ← NUEVO (fetch /admin/buyers/*, /admin/couriers/*, /admin/pickup-points/*)
```

**Solo LECTURA de:** `components/RutaSidebar.tsx`, `components/RutaHeader.tsx`, `app/(protected)/layout.tsx`.

**NO tocar:** ningún archivo de los Agentes F o G.

**Criterio de done:** checklist de calidad completa (ver sección "Protocolo de calidad") · plan_tareas.md actualizado `[x]` · memoria del sprint actualizada · PR abierto con título `[1.ADMIN-4] Compradores, Repartidores y Pickup Points frontend`.

---

## Diagrama de dependencias

```
WAVE 1 (todos simultáneos):
  Agente A → 1.BACK-5   (backend-ruta, bulk import)
  Agente B → 1.ADMIN-1  (admin layout + login)  ← prerrequisito de Wave 2
  Agente C → 1.STORE-1  (storefront catálogo)
  Agente D → 1.STORE-2  (storefront auth)
  Agente E → 1.QA-1     (backend tests isolation)

WAVE 2 (después de que ADMIN-1 esté en main):
  Agente F → 1.ADMIN-2  (admin clientes)
  Agente G → 1.ADMIN-3  (admin productos)
  Agente H → 1.ADMIN-4  (admin buyers/couriers/pickup-points)
```

---

## Contexto técnico para todos los agentes

- **BD:** PostgreSQL OCI `149.130.168.24:26432/rutadb` (no Supabase)
- **Auth backend ya implementado** en `backend-ruta/api/src/` — 1.BACK-1/2/3/4 completos con 42 tests
- **API base URL dev:** `http://localhost:3001` (backend Express)
- **Admin corre en:** `:3002` · **Storefront en:** `:3003`
- **Paquetes compartidos:** `@orkoruta/shared@1.1.0` y `@orkoruta/db@1.0.0` en GitHub Packages
- **Design system:** componentes `Ruta*` solo en `frontend-ruta/packages/ui/` — **las landings custom NO lo usan**
- **Auth cookies:** HttpOnly, Secure, SameSite=Strict — no localStorage
- **Multi-tenant:** toda query backend usa `withTenant(clientId, role, fn)`
- **Idempotencia:** todo POST/PUT/PATCH/DELETE requiere header `X-Idempotency-Key`
- **Tests backend:** `export NPM_TOKEN=$(grep "_authToken=" ~/.npmrc | cut -d= -f2)` antes de `pnpm test`
- **CI/Render backend:** `backend-ruta` usa `pnpm@11.4.0`; `minimumReleaseAgeExclude`
  debe mantenerse en `@orkoruta/*` para paquetes internos de GitHub Packages.
  `backend-ruta/main` verde en GitHub Actions run 26540623500.

**Documentos a consultar (en `docs-ruta/`):**
- `contrato_api.md` — endpoints disponibles
- `wireframes_mvp.md` — diseño de pantallas
- `diseno/galeria_estilos_ruta.md` — tokens de diseño y componentes
- `matriz_permisos.md` — permisos por rol
- `plan_tareas.md` — descripción detallada de cada tarea
- `CLAUDE.md` — reglas no negociables del proyecto
