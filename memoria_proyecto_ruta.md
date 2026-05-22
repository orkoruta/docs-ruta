# MEMORIA DEL PROYECTO RUTA

> Documento de **memoria operativa** para Claude Cowork (y cualquier
> agente IA o persona) que necesite tomar los archivos generados en la
> sesión de planning de RUTA y distribuirlos en su estructura local
> de repositorios.
>
> Si eres Cowork: **lee primero la PARTE 1 (contexto), luego ejecuta
> las instrucciones operativas de la PARTE 3** sobre los archivos
> ubicados en el directorio fuente.

---

# PARTE 1 — Contexto del proyecto (lectura obligada)

## 1.1 Qué es RUTA

Plataforma SaaS **multi-tenant** para administrar operaciones
comerciales entre Clientes (negocios) y Compradores. Mercado:
Colombia. Moneda: COP. UI en español, código en inglés.

**Dos tipos de Cliente:**

- **Cliente API** (`client_type='API'`): tiene su propia plataforma de
  venta; RUTA solo le hace logística.
- **Cliente Full** (`client_type='FULL'`): RUTA le provee todo
  (catálogo, checkout, pagos, logística). Modalidad de frontend:
  - `NATIVE_RUTA`: usa el storefront genérico.
  - `CUSTOM_LANDING_BY_RUTA`: tiene landing custom con branding propio.

**5 roles:** ADMIN_RUTA, ADMIN_CLIENT, OPERATOR_CLIENT, BUYER, COURIER.

## 1.2 Stack técnico (no negociable)

| Capa | Tecnología |
|---|---|
| Backend | Express.js + TypeScript |
| Frontend (admin, storefront, landings) | Next.js 14+ App Router + Tailwind |
| ORM | Prisma (con SQL crudo para particiones/RLS) |
| Auth | `jose` (JWT) + `argon2` |
| Jobs | `pg-boss` (Postgres) |
| File storage | Supabase Storage |
| Pasarela de pagos | Wompi |
| Mapas | OpenStreetMap + Leaflet |
| Hosting | Render + Supabase |
| Migraciones BD | `node-pg-migrate` + estado autoritativo en SQL |
| Testing | Vitest + Supertest + Playwright + MSW |
| Logger | `pino` |
| Validación | Zod (compartido) |
| Workspaces internos | pnpm |
| Distribución de paquetes | GitHub Packages (npm privado) |

## 1.3 Estructura multi-repo (lo que Cowork debe construir)

**Carpeta local raíz:** `~/projects/ruta/` (puede variar por máquina).

```
ruta/                                    ← carpeta local (NO es un repo)
│
├── backend-ruta/                        ← repo Git: ruta-backend
│   └── api/                             ← Express API
│
├── frontend-ruta/                       ← repo Git: ruta-frontend
│   ├── admin/                           ← Next.js panel admin
│   └── storefront/                      ← Next.js storefront nativo
│
├── frontend-clients-ruta/               ← carpeta local (NO es un repo)
│   ├── _template/                       ← repo Git: landing-template
│   ├── cliente-1/                       ← repo Git: landing-{slug-cliente-1}
│   ├── cliente-2/                       ← repo Git: landing-{slug-cliente-2}
│   └── cliente-n/                       ← repo Git: landing-{slug-cliente-n}
│
├── packages-ruta/                       ← repo Git: ruta-shared
│   ├── shared/                          ← @ruta/shared (npm)
│   └── db/                              ← @ruta/db (npm)
│
├── docs-ruta/                           ← repo Git: ruta-docs
└── infra-ruta/                          ← repo Git: ruta-infra
```

**Reglas importantes:**

- `ruta/` y `frontend-clients-ruta/` son carpetas locales puras. No
  son repos Git, no se publican a GitHub.
- Los **6 repos base** son: `ruta-backend`, `ruta-frontend`,
  `ruta-shared`, `ruta-docs`, `ruta-infra`, `landing-template`.
- Los **repos de landings custom** se llaman `landing-{slug}` (SIN
  prefijo `ruta-`). Ej: `landing-restaurante-el-prado`. Estos no
  existen al inicio; se crean conforme aparezcan Clientes Full con
  modalidad CUSTOM_LANDING_BY_RUTA (Fase 3 del MVP).

## 1.4 Decisiones críticas (referenciar siempre)

1. **RUTA no custodia dinero.** Pagos online van directo a la
   pasarela del Cliente. Pagos contra entrega: el Repartidor registra
   evidencia. Reembolsos los ejecuta el Cliente. RUTA solo registra
   estados.
2. **BIGINT, nunca UUID** en IDs.
3. **Particionamiento LIST por `client_id`** en todas las tablas
   operativas. Auto-creado al crear Cliente.
4. **RLS activo** con `app.current_client_id` y `app.current_user_role`
   por sesión.
5. **Tablas append-only:** `audit_events`, `order_state_history`,
   `external_webhook_events`, `webhook_deliveries`.
6. **Idempotencia obligatoria** en POST/PUT/PATCH/DELETE con header
   `X-Idempotency-Key`.
7. **Auth propia con `jose` + `argon2`.** No usar Supabase Auth.
8. **Tokens en cookies HttpOnly Secure SameSite=Strict.** Nunca
   localStorage.
9. **Design system `@ruta/ui` vive solo en `ruta-frontend`.** Las
   landings custom NO heredan el design system; tienen branding
   propio.
10. **Vista de Control:** ADMIN_RUTA impersona ADMIN_CLIENT con master
    password, auditado.

## 1.5 Alcance del MVP

**Fase 1 (10 semanas):** Cliente Full + frontend + checkout. Flujos
1, 2, 3 completos. Sin reembolsos, returns, recurrencia,
corporativos, disputas, Cliente API ni landings custom.

**Fase 2:** Cliente API (logística-as-a-service).

**Fase 3:** Reembolsos, returns post-cierre, disputas, recurrencia,
corporativos, landings custom.

---

# PARTE 2 — Inventario de archivos generados

Hay **22 archivos finales** en el directorio fuente (`/mnt/user-data/outputs/`
o equivalente). Más 1 archivo de memoria (este).

## 2.1 Documentación principal (.md)

| # | Archivo | Tamaño | Función |
|---|---|---|---|
| 1 | `all_ruta.md` | 41 KB | README funcional general del proyecto |
| 2 | `mvp_alcance.md` | 10 KB | MVP en 3 fases con cronograma |
| 3 | `estructura_proyecto.md` | 16 KB | Estructura multi-repo detallada |
| 4 | `plan_tareas.md` | 21 KB | Plan operativo 7 sprints / 10 semanas |
| 5 | `matriz_permisos.md` | 17 KB | Permisos por rol en 15 dominios |
| 6 | `parametros_negocio.md` | 13 KB | ~70 parámetros operativos con valores default |
| 7 | `contrato_api.md` | 16 KB | ~70 endpoints REST documentados |
| 8 | `wireframes_mvp.md` | 28 KB | 28 pantallas descritas funcionalmente |
| 9 | `estrategia_testing.md` | 14 KB | Estrategia de tests Vitest + Playwright |
| 10 | `estrategia_multi_tenant_ruta.md` | 66 KB | Decisiones de arquitectura multi-tenant |
| 11 | `galeria_estilos_ruta.md` | 12 KB | Design system y componentes Ruta* |

## 2.2 Flujos de estados de pedido (.txt)

| # | Archivo | Tamaño | Aplica a |
|---|---|---|---|
| 12 | `reglas_para_diagramar_flujos.txt` | (original) | Convenciones para todos los flujos |
| 13 | `flujo_1_comun_y_decision_de_pago.txt` | 13 KB | Solo Cliente Full |
| 14 | `flujo_2_ship_completo.txt` | 22 KB | Cliente API y Full |
| 15 | `flujo_3_pickup_completo.txt` | 15 KB | Cliente API y Full |
| 16 | `flujo_4_refund_completo.txt` | 13 KB | Solo Cliente Full |
| 17 | `flujo_5_recurrencia.txt` | 9 KB | Solo Cliente Full |
| 18 | `flujo_6_pedidos_corporativos.txt` | 7 KB | Solo Cliente Full |
| 19 | `flujo_7_devoluciones_post_cierre.txt` | 10 KB | Solo Cliente Full |

## 2.3 Seguridad (.txt)

| # | Archivo | Origen |
|---|---|---|
| 20 | `ciclo_vida_token.txt` | Original del usuario (de uploads) |

## 2.4 Modelo de datos (.sql)

| # | Archivo | Tamaño | Uso |
|---|---|---|---|
| 21 | `ruta_postgres.sql` | 84 KB | Schema completo ejecutable en PostgreSQL/Supabase |
| 22 | `ruta_oracle.sql` | 80 KB | Schema equivalente para Oracle Data Modeler (DER) |

## 2.5 Manifiestos de agentes (.md)

| # | Archivo | Tamaño | Uso |
|---|---|---|---|
| 23 | `CLAUDE.md` | 14 KB | Manifiesto master para Claude Code |
| 24 | `AGENTS.md` | 8 KB | Manifiesto master para otros agentes IA |

---

# PARTE 3 — Instrucciones operativas para Cowork

Estas instrucciones asumen que:

- Los archivos fuente están en `/mnt/user-data/outputs/` (o donde el
  usuario los haya descargado).
- Cowork tiene permisos de creación de carpetas y copia/movimiento de
  archivos en el sistema local del usuario.

## 3.1 Crear la estructura de carpetas locales

Crear esta jerarquía partiendo del directorio que el usuario indique
como raíz (típicamente `~/projects/ruta/` o equivalente):

```
ruta/
├── backend-ruta/
├── frontend-ruta/
├── frontend-clients-ruta/
│   └── _template/
├── packages-ruta/
├── docs-ruta/
│   ├── arquitectura/
│   ├── seguridad/
│   ├── flujos/
│   ├── bd/
│   │   └── migrations/
│   └── diseno/
└── infra-ruta/
```

**Notas:**
- `ruta/` y `frontend-clients-ruta/` son contenedores puros (sin
  `.git`).
- `bd/migrations/` queda vacío inicialmente; se llenará durante el
  desarrollo.
- No crear `cliente-1/`, `cliente-2/`, etc. en
  `frontend-clients-ruta/`. Esas carpetas se materializan después
  (Fase 3) cuando aparezcan Clientes Full con landings custom.

## 3.2 Mapeo de archivos a destinos

### 3.2.1 Documentación → `docs-ruta/`

Estos archivos van a la raíz de `docs-ruta/` (que cuando se inicialice
como repo Git será `ruta-docs`):

| Archivo fuente | Destino local |
|---|---|
| `all_ruta.md` | `ruta/docs-ruta/all_ruta.md` |
| `mvp_alcance.md` | `ruta/docs-ruta/mvp_alcance.md` |
| `estructura_proyecto.md` | `ruta/docs-ruta/estructura_proyecto.md` |
| `plan_tareas.md` | `ruta/docs-ruta/plan_tareas.md` |
| `matriz_permisos.md` | `ruta/docs-ruta/matriz_permisos.md` |
| `parametros_negocio.md` | `ruta/docs-ruta/parametros_negocio.md` |
| `contrato_api.md` | `ruta/docs-ruta/contrato_api.md` |
| `wireframes_mvp.md` | `ruta/docs-ruta/wireframes_mvp.md` |
| `estrategia_testing.md` | `ruta/docs-ruta/estrategia_testing.md` |

### 3.2.2 Documentación → `docs-ruta/arquitectura/`

| Archivo fuente | Destino local |
|---|---|
| `estrategia_multi_tenant_ruta.md` | `ruta/docs-ruta/arquitectura/estrategia_multi_tenant_ruta.md` |

### 3.2.3 Documentación → `docs-ruta/seguridad/`

| Archivo fuente | Destino local |
|---|---|
| `ciclo_vida_token.txt` | `ruta/docs-ruta/seguridad/ciclo_vida_token.txt` |

### 3.2.4 Flujos → `docs-ruta/flujos/`

| Archivo fuente | Destino local |
|---|---|
| `reglas_para_diagramar_flujos.txt` | `ruta/docs-ruta/flujos/reglas_para_diagramar_flujos.txt` |
| `flujo_1_comun_y_decision_de_pago.txt` | `ruta/docs-ruta/flujos/flujo_1_comun_y_decision_de_pago.txt` |
| `flujo_2_ship_completo.txt` | `ruta/docs-ruta/flujos/flujo_2_ship_completo.txt` |
| `flujo_3_pickup_completo.txt` | `ruta/docs-ruta/flujos/flujo_3_pickup_completo.txt` |
| `flujo_4_refund_completo.txt` | `ruta/docs-ruta/flujos/flujo_4_refund_completo.txt` |
| `flujo_5_recurrencia.txt` | `ruta/docs-ruta/flujos/flujo_5_recurrencia.txt` |
| `flujo_6_pedidos_corporativos.txt` | `ruta/docs-ruta/flujos/flujo_6_pedidos_corporativos.txt` |
| `flujo_7_devoluciones_post_cierre.txt` | `ruta/docs-ruta/flujos/flujo_7_devoluciones_post_cierre.txt` |

### 3.2.5 Modelo de datos → `docs-ruta/bd/`

| Archivo fuente | Destino local |
|---|---|
| `ruta_postgres.sql` | `ruta/docs-ruta/bd/ruta_postgres.sql` |
| `ruta_oracle.sql` | `ruta/docs-ruta/bd/ruta_oracle.sql` |

### 3.2.6 Diseño → `docs-ruta/diseno/`

| Archivo fuente | Destino local |
|---|---|
| `galeria_estilos_ruta.md` | `ruta/docs-ruta/diseno/galeria_estilos_ruta.md` |

### 3.2.7 Manifiestos de agentes → COPIAR a cada repo

`CLAUDE.md` y `AGENTS.md` son **manifiestos master** que deben
**copiarse (no moverse) a la raíz de cada repo**. El contenido es el
mismo para todos los repos; el manifiesto ya está diseñado para
funcionar en cualquier contexto (su sección 0 dice cómo identificar
en qué repo estás).

Copiar `CLAUDE.md` a:

| Destino |
|---|
| `ruta/docs-ruta/CLAUDE.md` (versión master) |
| `ruta/backend-ruta/CLAUDE.md` |
| `ruta/frontend-ruta/CLAUDE.md` |
| `ruta/packages-ruta/CLAUDE.md` |
| `ruta/infra-ruta/CLAUDE.md` |
| `ruta/frontend-clients-ruta/_template/CLAUDE.md` |

Copiar `AGENTS.md` a las mismas ubicaciones:

| Destino |
|---|
| `ruta/docs-ruta/AGENTS.md` (versión master) |
| `ruta/backend-ruta/AGENTS.md` |
| `ruta/frontend-ruta/AGENTS.md` |
| `ruta/packages-ruta/AGENTS.md` |
| `ruta/infra-ruta/AGENTS.md` |
| `ruta/frontend-clients-ruta/_template/AGENTS.md` |

Total: **6 copias de CLAUDE.md** y **6 copias de AGENTS.md**.

### 3.2.8 Memoria → `docs-ruta/`

Este archivo (`memoria_proyecto_ruta.md`) debe quedar también en
`docs-ruta/` para que cualquier agente futuro pueda cargarlo como
contexto:

| Archivo fuente | Destino local |
|---|---|
| `memoria_proyecto_ruta.md` | `ruta/docs-ruta/memoria_proyecto_ruta.md` |

## 3.3 Crear README mínimo en cada repo (opcional pero recomendado)

En cada uno de los siguientes destinos, crear un `README.md` con el
contenido placeholder indicado:

### `ruta/backend-ruta/README.md`

```markdown
# ruta-backend

Backend Express + TypeScript de RUTA.

- Manifiesto: ver `CLAUDE.md` y `AGENTS.md` en este repo.
- Documentación completa: ver repo `ruta-docs` (carpeta local `docs-ruta/`).
- Plan de tareas: ver `docs-ruta/plan_tareas.md`.
```

### `ruta/frontend-ruta/README.md`

```markdown
# ruta-frontend

Frontend Next.js de RUTA: panel admin + storefront nativo.

- Manifiesto: ver `CLAUDE.md` y `AGENTS.md` en este repo.
- Documentación: ver repo `ruta-docs` (carpeta local `docs-ruta/`).
- Design system: vive en `packages/ui/` (no se publica externamente).
```

### `ruta/packages-ruta/README.md`

```markdown
# ruta-shared

Paquetes compartidos: `@ruta/shared` (tipos, validators, enums) y
`@ruta/db` (Prisma client).

- Distribución: GitHub Packages.
- Manifiesto: ver `CLAUDE.md` y `AGENTS.md`.
- Documentación: ver repo `ruta-docs`.
```

### `ruta/docs-ruta/README.md`

```markdown
# ruta-docs

Documentación completa del proyecto RUTA.

**Lee primero:** `all_ruta.md` (descripción funcional general).

**Para agentes IA:** carga `memoria_proyecto_ruta.md` como contexto
o lee `CLAUDE.md` / `AGENTS.md`.

**Para arrancar desarrollo:** ver `plan_tareas.md` (sprints).
```

### `ruta/infra-ruta/README.md`

```markdown
# ruta-infra

Scripts de despliegue y setup de RUTA.

- `scripts/setup_workspace.sh` — clona los 6 repos base.
- `scripts/create_landing.sh` — crea un landing custom desde
  `landing-template`.
- Manifiesto: ver `CLAUDE.md` y `AGENTS.md`.
```

### `ruta/frontend-clients-ruta/_template/README.md`

```markdown
# landing-template

Template base para crear landings custom de Clientes Full (modalidad
CUSTOM_LANDING_BY_RUTA).

Para crear un landing nuevo desde este template, usar
"Use this template" en GitHub o ejecutar
`bash ~/projects/ruta/infra-ruta/scripts/create_landing.sh {slug}`.
```

## 3.4 Resumen del estado final esperado

Después de ejecutar todas las instrucciones, el sistema de archivos
local debe verse así:

```
~/projects/ruta/
├── backend-ruta/
│   ├── CLAUDE.md
│   ├── AGENTS.md
│   └── README.md
│
├── frontend-ruta/
│   ├── CLAUDE.md
│   ├── AGENTS.md
│   └── README.md
│
├── frontend-clients-ruta/
│   └── _template/
│       ├── CLAUDE.md
│       ├── AGENTS.md
│       └── README.md
│
├── packages-ruta/
│   ├── CLAUDE.md
│   ├── AGENTS.md
│   └── README.md
│
├── docs-ruta/
│   ├── CLAUDE.md
│   ├── AGENTS.md
│   ├── README.md
│   ├── memoria_proyecto_ruta.md
│   ├── all_ruta.md
│   ├── mvp_alcance.md
│   ├── estructura_proyecto.md
│   ├── plan_tareas.md
│   ├── matriz_permisos.md
│   ├── parametros_negocio.md
│   ├── contrato_api.md
│   ├── wireframes_mvp.md
│   ├── estrategia_testing.md
│   │
│   ├── arquitectura/
│   │   └── estrategia_multi_tenant_ruta.md
│   │
│   ├── seguridad/
│   │   └── ciclo_vida_token.txt
│   │
│   ├── flujos/
│   │   ├── reglas_para_diagramar_flujos.txt
│   │   ├── flujo_1_comun_y_decision_de_pago.txt
│   │   ├── flujo_2_ship_completo.txt
│   │   ├── flujo_3_pickup_completo.txt
│   │   ├── flujo_4_refund_completo.txt
│   │   ├── flujo_5_recurrencia.txt
│   │   ├── flujo_6_pedidos_corporativos.txt
│   │   └── flujo_7_devoluciones_post_cierre.txt
│   │
│   ├── bd/
│   │   ├── ruta_postgres.sql
│   │   ├── ruta_oracle.sql
│   │   └── migrations/      (vacío)
│   │
│   └── diseno/
│       └── galeria_estilos_ruta.md
│
└── infra-ruta/
    ├── CLAUDE.md
    ├── AGENTS.md
    └── README.md
```

**Conteo de archivos:**
- 6 copias de `CLAUDE.md` (en 6 repos)
- 6 copias de `AGENTS.md` (en 6 repos)
- 6 archivos `README.md` (uno por repo, creados nuevos)
- 22 archivos de contenido únicos en `docs-ruta/` (incluye esta
  memoria)

---

# PARTE 4 — Lo que falta después de que Cowork distribuya los archivos

Una vez Cowork termine la distribución de archivos, lo siguiente NO es
automático y requiere acciones del equipo humano:

## 4.1 Inicializar repos Git

Para cada carpeta que debe ser un repo (no `ruta/` ni
`frontend-clients-ruta/`):

```bash
cd ~/projects/ruta/backend-ruta && git init && git add . && git commit -m "Initial documentation and manifests"
cd ~/projects/ruta/frontend-ruta && git init && git add . && git commit -m "Initial documentation and manifests"
cd ~/projects/ruta/packages-ruta && git init && git add . && git commit -m "Initial documentation and manifests"
cd ~/projects/ruta/docs-ruta && git init && git add . && git commit -m "Initial documentation"
cd ~/projects/ruta/infra-ruta && git init && git add . && git commit -m "Initial documentation and manifests"
cd ~/projects/ruta/frontend-clients-ruta/_template && git init && git add . && git commit -m "Initial template"
```

## 4.2 Crear repos en GitHub

Crear los 6 repos privados en GitHub:
- `ruta-backend`
- `ruta-frontend`
- `ruta-shared`
- `ruta-docs`
- `ruta-infra`
- `landing-template` (marcar como Template Repository)

## 4.3 Conectar y pushear

```bash
cd ~/projects/ruta/backend-ruta
git remote add origin git@github.com:<org>/ruta-backend.git
git branch -M main && git push -u origin main

# Repetir para los 5 restantes
```

## 4.4 Continuar con Sprint 0

Seguir el `plan_tareas.md` desde la tarea `0.INFRA-2` (crear cuenta
Supabase) en adelante. Las tareas `0.INFRA-1` y `0.DOCS-1` se
consideran completadas con este setup inicial.

---

# PARTE 5 — Referencias rápidas

## 5.1 Glosario

- **Cliente** = tenant. Cliente API o Cliente Full.
- **Comprador / BUYER** = consumidor final.
- **Repartidor / COURIER** = persona que entrega.
- **ADMIN_RUTA** = equipo de RUTA.
- **ADMIN_CLIENT** = administrador del Cliente.
- **OPERATOR_CLIENT** = staff operativo del Cliente.
- **Vista de Control** = impersonación auditada.
- **SHIP** = entrega a domicilio.
- **PICKUP** = recogida en punto físico.
- **NATIVE_RUTA** = Cliente Full usa storefront genérico.
- **CUSTOM_LANDING_BY_RUTA** = Cliente Full con landing propio.
- **Cliente plataforma** = `client_id = 0`, registro especial donde
  viven los ADMIN_RUTA y parámetros globales.

## 5.2 Rutas DNS planeadas

| URL | Servicio |
|---|---|
| `api.ruta.com` | Backend Express |
| `app.ruta.com` | Frontend admin |
| `tienda.ruta.com` | Storefront nativo |
| Dominio propio del Cliente | Landings custom (ej. `elprado.com.co`) |

## 5.3 Documentos a consultar por dominio

| Tarea | Documento prioritario |
|---|---|
| Cualquier cosa | `all_ruta.md` |
| Arquitectura técnica | `arquitectura/estrategia_multi_tenant_ruta.md` |
| Estados de pedido | `flujos/flujo_1.txt` a `flujo_7.txt` |
| Convenciones diagramas | `flujos/reglas_para_diagramar_flujos.txt` |
| Endpoints HTTP | `contrato_api.md` |
| Pantallas UI | `wireframes_mvp.md` |
| Estilos y componentes | `diseno/galeria_estilos_ruta.md` |
| Permisos por rol | `matriz_permisos.md` |
| Plazos y parámetros | `parametros_negocio.md` |
| Estructura del proyecto | `estructura_proyecto.md` |
| Testing | `estrategia_testing.md` |
| Auth detallada | `seguridad/ciclo_vida_token.txt` |
| Modelo de datos | `bd/ruta_postgres.sql` |
| Plan operativo | `plan_tareas.md` |
| Alcance MVP | `mvp_alcance.md` |

## 5.4 Tabla resumen de aplicabilidad de flujos

| Flujo | Cliente API | Cliente Full | Notas |
|---|---|---|---|
| 1 — Común y decisión de pago | NO | SÍ | Cliente API entra directo en preparación |
| 2 — SHIP completo | SÍ | SÍ | Con notas específicas por tipo |
| 3 — PICKUP completo | SÍ | SÍ | Con notas específicas por tipo |
| 4 — Refund completo | NO | SÍ | Solo Cliente Full |
| 5 — Recurrencia | NO | SÍ | Solo Cliente Full |
| 6 — Pedidos corporativos | NO | SÍ | Solo Cliente Full |
| 7 — Devoluciones post-cierre | NO | SÍ | Solo Cliente Full |

## 5.5 Naming conventions

- **Código:** inglés. **UI y docs:** español.
- **Services y routes:** `snake_case` para alinear con BD.
- **Variables/funciones:** `camelCase`.
- **Tipos/clases:** `PascalCase`.
- **Constantes:** `SCREAMING_SNAKE_CASE`.
- **Repos base:** prefijo `ruta-` (ej. `ruta-backend`).
- **Repos de landing:** `landing-{slug}` SIN prefijo `ruta-`.
- **Carpetas locales:** sufijo `-ruta` (ej. `backend-ruta`,
  `docs-ruta`).

---

# FIN DE MEMORIA

Última actualización: sesión de planning de RUTA, decisiones
confirmadas por el equipo. Total de archivos generados: 22 + esta
memoria = 23.

Si encuentras inconsistencias o falta información, consulta
`docs-ruta/all_ruta.md` o `docs-ruta/estructura_proyecto.md` (que son
las fuentes de verdad más actuales) y reporta al equipo.
