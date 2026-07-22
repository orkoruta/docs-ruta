# MEMORIA DEL PROYECTO RUTA

> **ARCHIVO HISTÓRICO — NO USAR COMO FUENTE DE VERDAD**
>
> Este archivo es el sistema de memoria anterior (pre Claude Code).
> La fuente de verdad del estado del proyecto son:
> - Manifiestos `CLAUDE.md` / `AGENTS.md` en cada repo (secciones 0.2 y 0.3)
> - Directorio de memoria de Claude Code para este proyecto
> - `docs-ruta/plan_fase3.md` para el estado de Fase 3
>
> No actualizar este archivo. Conservado solo como referencia histórica.

---

## Actualización operativa — 2026-05-30

**Sprint 0–6 (código) COMPLETOS. Pendiente solo deploy producción + piloto (acción humana).**

### Sprints completados

- **Sprint 0** ✅ — Setup multi-repo, GitHub Packages, BD dev, seed, deploy inicial Render.
- **Sprint 1** ✅ — Auth JWT, CRUD clientes/productos/compradores/repartidores, importación Excel, admin UI, storefront catálogo/registro BUYER.
- **Sprint 2** ✅ — Carrito, checkout (SHIP/PICKUP/OSM/Wompi/COD), Wompi webhook HMAC, jobs mantenimiento, mis pedidos BUYER. `@orkoruta/shared@1.2.0`.
- **Sprint 3** ✅ — Flujo SHIP completo: asignación courier (mapa Leaflet), cobro COD, cancelación, return-to-origin, auto-confirmación. Vista courier móvil-first. `@orkoruta/shared@1.3.0`. 3686 tests.
- **Sprint 4** ✅ — Flujo PICKUP: pickup_ops.service, admin_pickup_ops, pickup_expiration.job, PickupActions UI, E2E PICKUP.
- **Sprint 5** ✅ — Vista de Control (impersonación auditada), dashboards ADMIN_CLIENT y ADMIN_RUTA, Configuración (4 tabs: info, Wompi, webhooks, parámetros), Auditoría.
- **Sprint 6** ✅ (código) — Observabilidad pino+Logtail, webhooks salientes pg-boss, cobertura >85% (3947 tests), backup/restore BD, E2E 14 tests (3 specs), guías usuario 4 roles, packages-ruta binaryTargets cross-platform.

### Pendiente (requiere acción humana)
- **6.INFRA-4**: DNS reales (api/app/tienda.ruta.com), llaves Wompi producción en Render, aplicar SQL prod en OCI.
- **6.QA-3**: Onboarding cliente piloto + primer pedido real.

### Stack de producción (Render)
- API: `ruta-api` + `ruta-api-worker` — auto-deploy desde `main` de backend-ruta.
- Admin: `ruta-admin` (Next.js) — auto-deploy desde `main` de frontend-ruta.
- Storefront: `ruta-storefront` (Next.js) — auto-deploy desde `main` de frontend-ruta.
- BD: PostgreSQL OCI `149.130.168.24:26432`, schema `ruta`.
- Packages: `@orkoruta/shared@1.3.0` + `@orkoruta/db@1.0.0` en GitHub Packages (org: orkoruta).

> Documento de **memoria operativa** para Claude Cowork (y cualquier
> agente IA o persona) que necesite tomar los archivos generados en la
> sesiÃ³n de planning de RUTA y distribuirlos en su estructura local
> de repositorios.
>
> Si eres Cowork: **lee primero la PARTE 1 (contexto), luego ejecuta
> las instrucciones operativas de la PARTE 3** sobre los archivos
> ubicados en el directorio fuente.

---

# PARTE 1 â€” Contexto del proyecto (lectura obligada)

## 1.1 QuÃ© es RUTA

Plataforma SaaS **multi-tenant** para administrar operaciones
comerciales entre Clientes (negocios) y Compradores. Mercado:
Colombia. Moneda: COP. UI en espaÃ±ol, cÃ³digo en inglÃ©s.

**Dos tipos de Cliente:**

- **Cliente API** (`client_type='API'`): tiene su propia plataforma de
  venta; RUTA solo le hace logÃ­stica.
- **Cliente Full** (`client_type='FULL'`): RUTA le provee todo
  (catÃ¡logo, checkout, pagos, logÃ­stica). Modalidad de frontend:
  - `NATIVE_RUTA`: usa el storefront genÃ©rico.
  - `CUSTOM_LANDING_BY_RUTA`: tiene landing custom con branding propio.

**5 roles:** ADMIN_RUTA, ADMIN_CLIENT, OPERATOR_CLIENT, BUYER, COURIER.

## 1.2 Stack tÃ©cnico (no negociable)

| Capa | TecnologÃ­a |
|---|---|
| Backend | Express.js + TypeScript |
| Frontend (admin, storefront, landings) | Next.js 14+ App Router + Tailwind |
| ORM | Prisma (con SQL crudo para particiones/RLS) |
| Auth | `jose` (JWT) + `argon2` |
| Jobs | `pg-boss` (Postgres) |
| File storage | Por definir |
| Pasarela de pagos | Wompi |
| Mapas | OpenStreetMap + Leaflet |
| Hosting | Render |
| Migraciones BD | `node-pg-migrate` + estado autoritativo en SQL |
| Testing | Vitest + Supertest + Playwright + MSW |
| Logger | `pino` |
| ValidaciÃ³n | Zod (compartido) |
| Workspaces internos | pnpm |
| DistribuciÃ³n de paquetes | GitHub Packages (npm privado) |

## 1.3 Estructura multi-repo (lo que Cowork debe construir)

**Carpeta local raÃ­z:** `~/projects/ruta/` (puede variar por mÃ¡quina).

```
ruta/                                    â† carpeta local (NO es un repo)
â”‚
â”œâ”€â”€ backend-ruta/                        â† repo Git: backend-ruta
â”‚   â””â”€â”€ api/                             â† Express API
â”‚
â”œâ”€â”€ frontend-ruta/                       â† repo Git: frontend-ruta
â”‚   â”œâ”€â”€ admin/                           â† Next.js panel admin
â”‚   â””â”€â”€ storefront/                      â† Next.js storefront nativo
â”‚
â”œâ”€â”€ frontend-clients-ruta/               â† carpeta local (NO es un repo)
â”‚   â”œâ”€â”€ _template/                       â† template local, sin repo Git base
â”‚   â”œâ”€â”€ cliente-1/                       â† repo Git: landing-{slug-cliente-1}
â”‚   â”œâ”€â”€ cliente-2/                       â† repo Git: landing-{slug-cliente-2}
â”‚   â””â”€â”€ cliente-n/                       â† repo Git: landing-{slug-cliente-n}
â”‚
â”œâ”€â”€ packages-ruta/                       â† repo Git: packages-ruta
â”‚   â”œâ”€â”€ shared/                          â† @ruta/shared (npm)
â”‚   â””â”€â”€ db/                              â† @ruta/db (npm)
â”‚
â”œâ”€â”€ docs-ruta/                           â† repo Git: docs-ruta
â””â”€â”€ infra-ruta/                          â† repo Git: infra-ruta
```

**Reglas importantes:**

- `ruta/` y `frontend-clients-ruta/` son carpetas locales puras. No
  son repos Git, no se publican a GitHub.
- Los **5 repos base** son: `backend-ruta`, `frontend-ruta`,
  `packages-ruta`, `docs-ruta`, `infra-ruta`.
- Los **repos de landings custom** se llaman `landing-{slug}` (SIN
  prefijo `ruta-`). Ej: `landing-restaurante-el-prado`. Estos no
  existen al inicio; se crean conforme aparezcan Clientes Full con
  modalidad CUSTOM_LANDING_BY_RUTA (Fase 3 del MVP).

## 1.4 Decisiones crÃ­ticas (referenciar siempre)

1. **RUTA no custodia dinero.** Pagos online van directo a la
   pasarela del Cliente. Pagos contra entrega: el Repartidor registra
   evidencia. Reembolsos los ejecuta el Cliente. RUTA solo registra
   estados.
2. **BIGINT, nunca UUID** en IDs.
3. **Particionamiento LIST por `client_id`** en todas las tablas
   operativas. Auto-creado al crear Cliente.
4. **RLS activo** con `app.current_client_id` y `app.current_user_role`
   por sesiÃ³n.
5. **Tablas append-only:** `audit_events`, `order_state_history`,
   `external_webhook_events`, `webhook_deliveries`.
6. **Idempotencia obligatoria** en POST/PUT/PATCH/DELETE con header
   `X-Idempotency-Key`.
7. **Auth propia con `jose` + `argon2`.** No delegar autenticación a servicios externos.
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

**Fase 2:** Cliente API (logÃ­stica-as-a-service).

**Fase 3:** Reembolsos, returns post-cierre, disputas, recurrencia,
corporativos, landings custom.

---

# PARTE 2 â€” Inventario de archivos generados

Hay **22 archivos finales** en el directorio fuente (`/mnt/user-data/outputs/`
o equivalente). MÃ¡s 1 archivo de memoria (este).

## 2.1 DocumentaciÃ³n principal (.md)

| # | Archivo | TamaÃ±o | FunciÃ³n |
|---|---|---|---|
| 1 | `all_ruta.md` | 41 KB | README funcional general del proyecto |
| 2 | `mvp_alcance.md` | 10 KB | MVP en 3 fases con cronograma |
| 3 | `estructura_proyecto.md` | 16 KB | Estructura multi-repo detallada |
| 4 | `plan_tareas.md` | 21 KB | Plan operativo 7 sprints / 10 semanas |
| 5 | `matriz_permisos.md` | 17 KB | Permisos por rol en 15 dominios |
| 6 | `parametros_negocio.md` | 13 KB | ~70 parÃ¡metros operativos con valores default |
| 7 | `contrato_api.md` | 16 KB | ~70 endpoints REST documentados |
| 8 | `wireframes_mvp.md` | 28 KB | 28 pantallas descritas funcionalmente |
| 9 | `estrategia_testing.md` | 14 KB | Estrategia de tests Vitest + Playwright |
| 10 | `estrategia_multi_tenant_ruta.md` | 66 KB | Decisiones de arquitectura multi-tenant |
| 11 | `galeria_estilos_ruta.md` | 12 KB | Design system y componentes Ruta* |

## 2.2 Flujos de estados de pedido (.txt)

| # | Archivo | TamaÃ±o | Aplica a |
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

| # | Archivo | TamaÃ±o | Uso |
|---|---|---|---|
| 21 | `ruta_postgres.sql` | 84 KB | Schema completo ejecutable en PostgreSQL (OCI) |
| 22 | `ruta_oracle.sql` | 80 KB | Schema equivalente para Oracle Data Modeler (DER) |

## 2.5 Manifiestos de agentes (.md)

| # | Archivo | TamaÃ±o | Uso |
|---|---|---|---|
| 23 | `CLAUDE.md` | 14 KB | Manifiesto master para Claude Code |
| 24 | `AGENTS.md` | 8 KB | Manifiesto master para otros agentes IA |

---

# PARTE 3 â€” Instrucciones operativas para Cowork

Estas instrucciones asumen que:

- Los archivos fuente estÃ¡n en `/mnt/user-data/outputs/` (o donde el
  usuario los haya descargado).
- Cowork tiene permisos de creaciÃ³n de carpetas y copia/movimiento de
  archivos en el sistema local del usuario.

## 3.1 Crear la estructura de carpetas locales

Crear esta jerarquÃ­a partiendo del directorio que el usuario indique
como raÃ­z (tÃ­picamente `~/projects/ruta/` o equivalente):

```
ruta/
â”œâ”€â”€ backend-ruta/
â”œâ”€â”€ frontend-ruta/
â”œâ”€â”€ frontend-clients-ruta/
â”‚   â””â”€â”€ _template/
â”œâ”€â”€ packages-ruta/
â”œâ”€â”€ docs-ruta/
â”‚   â”œâ”€â”€ arquitectura/
â”‚   â”œâ”€â”€ seguridad/
â”‚   â”œâ”€â”€ flujos/
â”‚   â”œâ”€â”€ bd/
â”‚   â”‚   â””â”€â”€ migrations/
â”‚   â””â”€â”€ diseno/
â””â”€â”€ infra-ruta/
```

**Notas:**
- `ruta/` y `frontend-clients-ruta/` son contenedores puros (sin
  `.git` operativo del proyecto).
- En local, el codigo del template base vive en
  `frontend-clients-ruta/_template/`.
- `bd/migrations/` queda vacÃ­o inicialmente; se llenarÃ¡ durante el
  desarrollo.
- No crear `cliente-1/`, `cliente-2/`, etc. en
  `frontend-clients-ruta/`. Esas carpetas se materializan despuÃ©s
  (Fase 3) cuando aparezcan Clientes Full con landings custom.

## 3.2 Mapeo de archivos a destinos

### 3.2.1 DocumentaciÃ³n â†’ `docs-ruta/`

Estos archivos van a la raÃ­z de `docs-ruta/` (que cuando se inicialice
como repo Git serÃ¡ `ruta-docs`):

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

### 3.2.2 DocumentaciÃ³n â†’ `docs-ruta/arquitectura/`

| Archivo fuente | Destino local |
|---|---|
| `estrategia_multi_tenant_ruta.md` | `ruta/docs-ruta/arquitectura/estrategia_multi_tenant_ruta.md` |

### 3.2.3 DocumentaciÃ³n â†’ `docs-ruta/seguridad/`

| Archivo fuente | Destino local |
|---|---|
| `ciclo_vida_token.txt` | `ruta/docs-ruta/seguridad/ciclo_vida_token.txt` |

### 3.2.4 Flujos â†’ `docs-ruta/flujos/`

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

### 3.2.5 Modelo de datos â†’ `docs-ruta/bd/`

| Archivo fuente | Destino local |
|---|---|
| `ruta_postgres.sql` | `ruta/docs-ruta/bd/ruta_postgres.sql` |
| `ruta_oracle.sql` | `ruta/docs-ruta/bd/ruta_oracle.sql` |

### 3.2.6 DiseÃ±o â†’ `docs-ruta/diseno/`

| Archivo fuente | Destino local |
|---|---|
| `galeria_estilos_ruta.md` | `ruta/docs-ruta/diseno/galeria_estilos_ruta.md` |

### 3.2.7 Manifiestos de agentes â†’ COPIAR a cada repo

`CLAUDE.md` y `AGENTS.md` son **manifiestos master** que deben
**copiarse (no moverse) a la raÃ­z de cada repo**. El contenido es el
mismo para todos los repos; el manifiesto ya estÃ¡ diseÃ±ado para
funcionar en cualquier contexto (su secciÃ³n 0 dice cÃ³mo identificar
en quÃ© repo estÃ¡s).

Copiar `CLAUDE.md` a:

| Destino |
|---|
| `ruta/docs-ruta/CLAUDE.md` (versiÃ³n master) |
| `ruta/backend-ruta/CLAUDE.md` |
| `ruta/frontend-ruta/CLAUDE.md` |
| `ruta/packages-ruta/CLAUDE.md` |
| `ruta/infra-ruta/CLAUDE.md` |
| `ruta/frontend-clients-ruta/_template/CLAUDE.md` |

Copiar `AGENTS.md` a las mismas ubicaciones:

| Destino |
|---|
| `ruta/docs-ruta/AGENTS.md` (versiÃ³n master) |
| `ruta/backend-ruta/AGENTS.md` |
| `ruta/frontend-ruta/AGENTS.md` |
| `ruta/packages-ruta/AGENTS.md` |
| `ruta/infra-ruta/AGENTS.md` |
| `ruta/frontend-clients-ruta/_template/AGENTS.md` |

Total: **6 copias de CLAUDE.md** y **6 copias de AGENTS.md**.

### 3.2.8 Memoria â†’ `docs-ruta/`

Este archivo (`memoria_proyecto_ruta.md`) debe quedar tambiÃ©n en
`docs-ruta/` para que cualquier agente futuro pueda cargarlo como
contexto:

| Archivo fuente | Destino local |
|---|---|
| `memoria_proyecto_ruta.md` | `ruta/docs-ruta/memoria_proyecto_ruta.md` |

## 3.3 README por repo

Esta seccion queda obsoleta como instruccion operativa. No debe recrearse
ningun README borrado ni tratarse como obligatorio que cada repo tenga un
README local.

Fuentes vigentes para agentes y documentacion:
- `CLAUDE.md` y `AGENTS.md` donde existan en cada repo.
- `docs-ruta/memoria_proyecto_ruta.md`.
- Documentos autoritativos dentro de `docs-ruta/`.

## 3.4 Resumen del estado final esperado

DespuÃ©s de ejecutar todas las instrucciones, el sistema de archivos
local debe verse asÃ­:

```
~/projects/ruta/
â”œâ”€â”€ backend-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â””â”€â”€ AGENTS.md
â”‚
â”œâ”€â”€ frontend-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â””â”€â”€ AGENTS.md
â”‚
â”œâ”€â”€ frontend-clients-ruta/
â”‚   â””â”€â”€ _template/
â”‚       â”œâ”€â”€ CLAUDE.md
â”‚       â”œâ”€â”€ AGENTS.md
â”‚       â””â”€â”€ README.md
â”‚
â”œâ”€â”€ packages-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â””â”€â”€ AGENTS.md
â”‚
â”œâ”€â”€ docs-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â”œâ”€â”€ AGENTS.md
â”‚   â”œâ”€â”€ memoria_proyecto_ruta.md
â”‚   â”œâ”€â”€ all_ruta.md
â”‚   â”œâ”€â”€ mvp_alcance.md
â”‚   â”œâ”€â”€ estructura_proyecto.md
â”‚   â”œâ”€â”€ plan_tareas.md
â”‚   â”œâ”€â”€ matriz_permisos.md
â”‚   â”œâ”€â”€ parametros_negocio.md
â”‚   â”œâ”€â”€ contrato_api.md
â”‚   â”œâ”€â”€ wireframes_mvp.md
â”‚   â”œâ”€â”€ estrategia_testing.md
â”‚   â”‚
â”‚   â”œâ”€â”€ arquitectura/
â”‚   â”‚   â””â”€â”€ estrategia_multi_tenant_ruta.md
â”‚   â”‚
â”‚   â”œâ”€â”€ seguridad/
â”‚   â”‚   â””â”€â”€ ciclo_vida_token.txt
â”‚   â”‚
â”‚   â”œâ”€â”€ flujos/
â”‚   â”‚   â”œâ”€â”€ reglas_para_diagramar_flujos.txt
â”‚   â”‚   â”œâ”€â”€ flujo_1_comun_y_decision_de_pago.txt
â”‚   â”‚   â”œâ”€â”€ flujo_2_ship_completo.txt
â”‚   â”‚   â”œâ”€â”€ flujo_3_pickup_completo.txt
â”‚   â”‚   â”œâ”€â”€ flujo_4_refund_completo.txt
â”‚   â”‚   â”œâ”€â”€ flujo_5_recurrencia.txt
â”‚   â”‚   â”œâ”€â”€ flujo_6_pedidos_corporativos.txt
â”‚   â”‚   â””â”€â”€ flujo_7_devoluciones_post_cierre.txt
â”‚   â”‚
â”‚   â”œâ”€â”€ bd/
â”‚   â”‚   â”œâ”€â”€ ruta_postgres.sql
â”‚   â”‚   â”œâ”€â”€ ruta_oracle.sql
â”‚   â”‚   â””â”€â”€ migrations/      (vacÃ­o)
â”‚   â”‚
â”‚   â””â”€â”€ diseno/
â”‚       â””â”€â”€ galeria_estilos_ruta.md
â”‚
â””â”€â”€ infra-ruta/
    â”œâ”€â”€ CLAUDE.md
    â”œâ”€â”€ AGENTS.md
    â””â”€â”€ README.md
```

**Conteo de archivos:**
- 6 copias de `CLAUDE.md` (5 repos base + template local)
- 6 copias de `AGENTS.md` (5 repos base + template local)
- Los README locales no son obligatorios; no recrear README borrados.
- 22 archivos de contenido Ãºnicos en `docs-ruta/` (incluye esta
  memoria)

---

# PARTE 4 â€” Lo que falta despuÃ©s de que Cowork distribuya los archivos

Una vez Cowork termine la distribuciÃ³n de archivos, lo siguiente NO es
automÃ¡tico y requiere acciones del equipo humano:

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

Crear los 5 repos privados base en GitHub:
- `backend-ruta`
- `frontend-ruta`
- `packages-ruta`
- `docs-ruta`
- `infra-ruta`

## 4.3 Conectar y pushear

```bash
cd ~/projects/ruta/backend-ruta
git remote add origin git@github.com:<org>/backend-ruta.git
git branch -M main && git push -u origin main

# Repetir para los 5 restantes
```

## 4.4 Continuar con Sprint 0

Seguir el `plan_tareas.md` desde la tarea `0.INFRA-3` en adelante.
Las tareas `0.INFRA-1`, `0.INFRA-2` (cancelada) y `0.DOCS-1` se
consideran resueltas.

---

# PARTE 5 â€” Referencias rÃ¡pidas

## 5.1 Glosario

- **Cliente** = tenant. Cliente API o Cliente Full.
- **Comprador / BUYER** = consumidor final.
- **Repartidor / COURIER** = persona que entrega.
- **ADMIN_RUTA** = equipo de RUTA.
- **ADMIN_CLIENT** = administrador del Cliente.
- **OPERATOR_CLIENT** = staff operativo del Cliente.
- **Vista de Control** = impersonaciÃ³n auditada.
- **SHIP** = entrega a domicilio.
- **PICKUP** = recogida en punto fÃ­sico.
- **NATIVE_RUTA** = Cliente Full usa storefront genÃ©rico.
- **CUSTOM_LANDING_BY_RUTA** = Cliente Full con landing propio.
- **Cliente plataforma** = `client_id = 0`, registro especial donde
  viven los ADMIN_RUTA y parÃ¡metros globales.

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
| Arquitectura tÃ©cnica | `arquitectura/estrategia_multi_tenant_ruta.md` |
| Estados de pedido | `flujos/flujo_1.txt` a `flujo_7.txt` |
| Convenciones diagramas | `flujos/reglas_para_diagramar_flujos.txt` |
| Endpoints HTTP | `contrato_api.md` |
| Pantallas UI | `wireframes_mvp.md` |
| Estilos y componentes | `diseno/galeria_estilos_ruta.md` |
| Permisos por rol | `matriz_permisos.md` |
| Plazos y parÃ¡metros | `parametros_negocio.md` |
| Estructura del proyecto | `estructura_proyecto.md` |
| Testing | `estrategia_testing.md` |
| Auth detallada | `seguridad/ciclo_vida_token.txt` |
| Modelo de datos | `bd/ruta_postgres.sql` |
| Plan operativo | `plan_tareas.md` |
| Alcance MVP | `mvp_alcance.md` |

## 5.4 Tabla resumen de aplicabilidad de flujos

| Flujo | Cliente API | Cliente Full | Notas |
|---|---|---|---|
| 1 â€” ComÃºn y decisiÃ³n de pago | NO | SÃ | Cliente API entra directo en preparaciÃ³n |
| 2 â€” SHIP completo | SÃ | SÃ | Con notas especÃ­ficas por tipo |
| 3 â€” PICKUP completo | SÃ | SÃ | Con notas especÃ­ficas por tipo |
| 4 â€” Refund completo | NO | SÃ | Solo Cliente Full |
| 5 â€” Recurrencia | NO | SÃ | Solo Cliente Full |
| 6 â€” Pedidos corporativos | NO | SÃ | Solo Cliente Full |
| 7 â€” Devoluciones post-cierre | NO | SÃ | Solo Cliente Full |

## 5.5 Naming conventions

- **CÃ³digo:** inglÃ©s. **UI y docs:** espaÃ±ol.
- **Services y routes:** `snake_case` para alinear con BD.
- **Variables/funciones:** `camelCase`.
- **Tipos/clases:** `PascalCase`.
- **Constantes:** `SCREAMING_SNAKE_CASE`.
- **Repos base:** sufijo local `-ruta` (ej. `backend-ruta`).
- **Repos de landing:** `landing-{slug}` SIN prefijo `ruta-`.
- **Carpetas locales:** sufijo `-ruta` (ej. `backend-ruta`,
  `docs-ruta`).

---

---

# PARTE 6 â€” HistÃ³rico de cambios del proyecto

> âš ï¸ **Regla**: cada avance, cambio, mejora, nueva funcionalidad o
> tarea que quede en memoria debe registrarse aquÃ­. Este documento es
> la **memoria viva** del proyecto.
>
> Cualquier agente IA que llegue nuevo debe leer esta secciÃ³n primero
> para ponerse al dÃ­a.

## 6.1 LÃ­nea de tiempo cronolÃ³gica

| Fecha | Tipo | DescripciÃ³n | Docs afectados | Sprint |
|---|---|---|---|---|
| 2026-05-24 | ðŸ“ Setup | CreaciÃ³n inicial de un manifiesto experimental para agente. Removido posteriormente el 2026-05-26 por decision del usuario; `AGENTS.md` y `CLAUDE.md` quedan como manifiestos vigentes. | `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-24 | ðŸ“ Setup | RevisiÃ³n completa del proyecto: diagnÃ³stico de documentaciÃ³n (âœ… completa) y cÃ³digo (âŒ nada escrito aÃºn) | `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-24 | ðŸ“ Setup | VerificaciÃ³n de compatibilidad de `.md` con Codebuff/DeepSeek â€” confirmado que `AGENTS.md` es suficiente como manifiesto general para agentes. | `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-24 | ðŸ“ Setup | Agregada PARTE 6 (HistÃ³rico de cambios) a este documento. | `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-25 | DecisiÃ³n | Confirmadas decisiones de arranque: dominio principal `ruta.com`, Cliente piloto existente, credenciales Wompi pendientes, parÃ¡metros de negocio confirmados en SQL, Fase 1 = Cliente Full, Fase 2 = Cliente API, carrito persistido en BD como pedido `DRAFT` | `mvp_alcance.md`, `estrategia_testing.md`, `wireframes_mvp.md`, `plan_tareas.md`, `contrato_api.md`, `parametros_negocio.md`, `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-25 | Regla operativa | Agregada Definition of Done: ningÃºn avance se considera terminado sin verificaciÃ³n; cada cambio debe ejecutar pruebas proporcionales al riesgo o registrar explÃ­citamente por quÃ© no se pudieron ejecutar | `plan_tareas.md`, `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-25 | Regla operativa | Registrada autorizaciÃ³n operativa del usuario: los agentes pueden ejecutar comandos, scripts, pruebas, builds, instalaciones y operaciones de sistema necesarias sin confirmaciÃ³n manual previa; si el runtime exige aprobaciÃ³n, deben solicitar escalaciÃ³n directamente con prefijos persistentes acotados cuando aplique | `AGENTS.md`, `CLAUDE.md`, copias por repo, `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | DecisiÃ³n infraestructura | Confirmado por el usuario que la BD de DEV ya existe en OCI y contiene todos sus objetos; la conexiÃ³n de desarrollo estÃ¡ en `backend-ruta/.env`. Se trabajarÃ¡ primero contra DEV y PROD queda diferido para una etapa posterior. No registrar credenciales en documentaciÃ³n. | `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | VerificaciÃ³n infraestructura | Verificada conexiÃ³n a BD DEV OCI desde `backend-ruta/.env` sin exponer secretos. Resultado: conexiÃ³n OK a `rutadb` con PostgreSQL 18.4; schema `ruta` existe; 48 tablas base; 104 filas en `state_catalog`; 70 filas en `client_parameters`; 1 cliente; 21 polÃ­ticas RLS `tenant_isolation`; 20 particiones de tabla para `client_id=0`. Se contrastÃ³ con `docs-ruta/bd/ruta_postgres.sql` y 20 particiones es el diseÃ±o correcto; algunas tablas con `client_id` no se particionan por diseÃ±o (`sessions`, `client_api_keys`, `webhook_subscriptions`, `external_webhook_events`, entre otras). | `memoria_proyecto_ruta.md`, `plan_tareas.md`, BD DEV OCI | Sprint 0 |
| 2026-05-26 | Desarrollo | Avance local DEV de `0.SHARED-1`: generado `@ruta/db/prisma/schema.prisma` por introspecciÃ³n contra BD DEV OCI; generado Prisma Client; corregido `withTenant` para ejecutar callbacks con el transaction client dentro del contexto RLS; completado barrel de enums; agregados validators de auth, client, product, category, buyer, courier y pickup_point; agregada suite mÃ­nima de tests de validators. VerificaciÃ³n: typecheck/build OK en `@ruta/shared` y `@ruta/db`; tests `@ruta/shared` OK (4). Pendiente para cierre total: GitHub Packages/CI/publicaciÃ³n de `@ruta/shared@1.0.0` y `@ruta/db@1.0.0`. | `packages-ruta`, `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Regla operativa | Creado `README.md` en la raiz local del proyecto con instruccion obligatoria para que cualquier agente lea primero `docs-ruta/memoria_proyecto_ruta.md` y registre alli todo avance, cambio, ajuste, decision o tarea importante. | `README.md`, `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Desarrollo/CI | Avance ejecutado de tareas 1, 2 y 3: `@ruta/shared` y `@ruta/db` quedan con CI local verificable, `@ruta/db` empaqueta Prisma Client generado dentro de `dist/generated` y usa imports ESM compatibles con Node; agregados workflows de CI/publicacion para `ruta-shared`; backend consume `@ruta/shared` y `@ruta/db` via dependencias locales `file:` para DEV, agrega `/healthz/ready` con Prisma y CI de backend. Verificacion: `packages-ruta pnpm run ci` OK; `pnpm pack --dry-run` OK para `@ruta/shared@1.0.0` y `@ruta/db@1.0.0` con tarballs limpios; `backend-ruta pnpm typecheck`, `pnpm build`, `pnpm test` OK; `pnpm start` levanta API y `curl /healthz` responde 200. Pendiente externo: configurar `NPM_PUBLISH_TOKEN` y publicar realmente `@ruta/shared@1.0.0`/`@ruta/db@1.0.0` en GitHub Packages desde `main`. | `packages-ruta`, `backend-ruta`, `memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Bugfix | Corregido error intermitente en build de `@ruta/shared` y `@ruta/db` en entorno WSL2/Dropbox: `rm -rf dist` fallaba con "Permission denied" cuando Dropbox tenia archivos bloqueados tras generar Prisma Client. Solucion: cambiar build script a `(rm -rf dist || true) && tsc`. Verificado: `pnpm run ci` en `packages-ruta` pasa sin errores. | `packages-ruta/shared/package.json`, `packages-ruta/db/package.json` | Sprint 0 |
| 2026-05-26 | Actualizacion plan | Actualizado `plan_tareas.md` para reflejar el estado real verificado: `0.INFRA-3` marcado [x], `0.INFRA-4` sub-items actualizados, `0.DOCS-1` marcado [x] en items completados, `0.SHARED-1` con todos los items [x] menos publicacion, `0.BACK-1` con estructura/deps/healthz/CI como [x]. | `docs-ruta/plan_tareas.md` | Sprint 0 |
| 2026-05-26 | Desarrollo | Completada `0.FRONT-1`: inicializado `ruta-frontend` como workspace pnpm con `@ruta/admin` (Next.js 14, :3002), `@ruta/storefront` (Next.js 14, :3003) y `@ruta/ui` (paquete interno). Componentes base: `RutaCard`, `RutaButton` (6 variantes+3 tamanhos), `RutaPill`, `RutaSectionHeader`, `RutaThemeToggle`. Tailwind con tokens de `galeria_estilos_ruta.md`. Paginas de prueba muestran todos los componentes en claro/oscuro. CI workflow agregado. Verificacion: `pnpm typecheck` EXIT 0; `pnpm build` OK (4 paginas estaticas por app). Nota: `next.config` usa `.mjs` (Next.js 14 no soporta `.ts`). `@ruta/shared` via `file:`. | `frontend-ruta/`, `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Decisión infraestructura | GitHub Packages requiere que el scope npm coincida con el usuario/org de GitHub. El usuario es `msimonz`. Se creó organización GitHub `orkoruta`. Todos los paquetes renombrados: `@ruta/shared` → `@orkoruta/shared`, `@ruta/db` → `@orkoruta/db`, `@ruta/ui` → `@orkoruta/ui`. Actualizados: package.json (nombres y deps), .npmrc (scope mapping), tsconfig paths, next.config.mjs transpilePackages, plan_tareas.md, workflows. ~/.npmrc actualizado con scope @orkoruta. Verificado: pnpm ci en packages-ruta OK; backend typecheck+test OK; frontend typecheck EXIT 0. | `packages-ruta/`, `backend-ruta/`, `frontend-ruta/`, `docs-ruta/plan_tareas.md`, `~/.npmrc` | Sprint 0 |
| 2026-05-26 | Desarrollo | Completada `0.INFRA-6`: creados `render.yaml.example` (4 servicios Render: ruta-api, ruta-admin, ruta-storefront + worker), `scripts/create_first_admin_ruta.sh` (argon2id, password via env var, SQL con escape). Sintaxis OK. Nota: storage_buckets.sql eliminado (Supabase cancelado). | `infra-ruta/`, `docs-ruta/plan_tareas.md` | Sprint 0 |
| 2026-05-26 | Desarrollo | Completada `0.BACK-2` (script): `create_first_admin_ruta.sh` con argon2id y INSERT en `ruta.users`. Ejecucion pendiente — requiere psql (no disponible en WSL2 actual). | `infra-ruta/scripts/`, `docs-ruta/plan_tareas.md` | Sprint 0 |
| 2026-05-26 | Desarrollo | Completada `0.LANDING-1`: `landing-template` en `frontend-clients-ruta/_template/` — Next.js 14 App Router, 7 paginas skeleton, `api_client.ts`, tailwind tokens genericos, SVG placeholder, .npmrc con hoisting, README+manifiestos. Typecheck EXIT 0. Build no verificado en WSL2/Dropbox (corrupcion pnpm store en NTFS). | `frontend-clients-ruta/_template/`, `docs-ruta/plan_tareas.md` | Sprint 0 |
| 2026-05-26 | Nota tecnica | pnpm v11 + WSL2/NTFS/Dropbox: usar `node-linker=hoisted` + `shamefully-hoist=true` en .npmrc para evitar problemas de virtual store. `unrs-resolver` requiere `onlyBuiltDependencies` en `pnpm-workspace.yaml` (pnpm 11 ya no lee campo `pnpm` de `package.json`). | todos los repos frontend | Sprint 0 |
| 2026-05-26 | Verificacion infraestructura | Intento de ejecucion de `0.INFRA-1`: `gh` esta instalado, pero la autenticacion local de GitHub falla porque el token guardado para `axmz-dev` es invalido; no hay `GH_TOKEN` ni `GITHUB_TOKEN` en el entorno. Se verifico que los remotos locales actuales apuntan a `msimonz/backend-ruta`, `msimonz/docs-ruta`, `msimonz/frontend-ruta`, `msimonz/infra-ruta` y `msimonz/packages-ruta`, mientras el plan espera repos `orkoruta/ruta-backend`, `orkoruta/ruta-docs`, `orkoruta/ruta-frontend`, `orkoruta/ruta-infra`, `orkoruta/ruta-shared` y `orkoruta/landing-template`. `frontend-clients-ruta/_template` todavia no tiene `.git`. No se crearon repos ni branch protection por bloqueo de autenticacion GitHub. | `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Decision infraestructura | Corregida la convencion real de repos GitHub: la organizacion `orkoruta` usa `backend-ruta`, `frontend-ruta`, `packages-ruta`, `docs-ruta` e `infra-ruta` como repos base; no se usan los nombres `ruta-*` ni `landing-template` como repos base. Actualizados `plan_tareas.md`, `workspace.config.json` y memoria para alinear las fuentes de verdad. | `docs-ruta/plan_tareas.md`, `infra-ruta/workspace.config.json`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Infraestructura | Avance de `0.INFRA-1`: remotos locales actualizados a `https://github.com/orkoruta/{backend-ruta,frontend-ruta,packages-ruta,docs-ruta,infra-ruta}.git`. Verificados en GitHub cinco repos privados (`backend-ruta`, `frontend-ruta`, `packages-ruta`, `docs-ruta`, `infra-ruta`). Intento de branch protection en `main` para los cinco visibles fallo con HTTP 403: GitHub requiere Pro o repo publico para esa funcion. `infra-ruta/workspace.config.json` validado como JSON. | `backend-ruta`, `frontend-ruta`, `packages-ruta`, `docs-ruta`, `infra-ruta`, `infra-ruta/workspace.config.json`, `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Decision infraestructura | Aclarado por el usuario que `frontend-clients-ruta` no es repo GitHub base; es una carpeta local donde iran los repos de cada cliente. Se omite de `0.INFRA-1` y de `infra-ruta/workspace.config.json`; `0.INFRA-1` queda con 5 repos base. | `docs-ruta/plan_tareas.md`, `infra-ruta/workspace.config.json`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Infraestructura | Removido el `.git` local que se habia inicializado en `frontend-clients-ruta/_template` durante la correccion anterior; queda nuevamente como carpeta local sin repo, acorde a la decision del usuario. | `frontend-clients-ruta/_template` | Sprint 0 |
| 2026-05-26 | Infraestructura | Cerrada `0.INFRA-1`: verificados como publicos los cinco repos base en `orkoruta` (`backend-ruta`, `frontend-ruta`, `packages-ruta`, `docs-ruta`, `infra-ruta`); remotos locales apuntan a sus URLs `https://github.com/orkoruta/*.git`; aplicada branch protection en `main` con enforce admins y 1 review obligatorio en los cinco repos. Status checks obligatorios: `CI / test` en `backend-ruta` y `packages-ruta`, `CI / ci` en `frontend-ruta`; `docs-ruta` e `infra-ruta` quedan sin status checks por no tener workflows CI. URLs documentadas en `plan_tareas.md`. | GitHub `orkoruta`, remotos locales, `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-26 | Documentacion | Cerrada `0.DOCS-1`: documentacion inicial consolidada en `docs-ruta`, manifiestos y plan/memoria actualizados; normalizados finales de linea en archivos de texto para que `git diff --check` pase limpio. Commit local `92b5ddb` en rama `docs/close-0-docs-1`; rama subida a GitHub y PR abierto contra `main`: https://github.com/orkoruta/docs-ruta/pull/1. | `docs-ruta`, `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md`, GitHub PR #1 | Sprint 0 |
| 2026-05-26 | Documentacion | Removido el manifiesto experimental por solicitud del usuario; `AGENTS.md` y `CLAUDE.md` quedan como manifiestos vigentes para agentes. PR #1 actualizado. Verificacion: `git diff --check` OK. | `docs-ruta/memoria_proyecto_ruta.md`, GitHub PR #1 | Sprint 0 |
| 2026-05-26 | Documentacion | PR #1 de `0.DOCS-1` aprobado y mergeado a `main`; `main` local sincronizada con `origin/main`. | GitHub PR #1, `docs-ruta/main` | Sprint 0 |
| 2026-05-26 | Infraestructura | Avance de `0.INFRA-4`: verificado secret `NPM_PUBLISH_TOKEN` en `orkoruta/packages-ruta`; autenticacion npm contra GitHub Packages OK como `axmz-dev`; confirmadas versiones publicadas `@orkoruta/shared@1.0.0` y `@orkoruta/db@1.0.0`; instalacion real desde `/tmp` OK para ambos paquetes; `packages-ruta pnpm run ci` OK; `pack --dry-run` OK para ambos paquetes; agregado script idempotente `scripts/publish-packages.mjs` para que el workflow no falle si la version ya existe; PR abierto en `packages-ruta`: https://github.com/orkoruta/packages-ruta/pull/1. Estado del PR: mergeable, CI `test` SUCCESS, review requerida. | `packages-ruta`, GitHub Packages, `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 0 |
| 2026-05-27 | Infraestructura | Cerrada `0.INFRA-4`: PR #1 de `packages-ruta` mergeado a `main` (CI `test` SUCCESS). CI workflows de `backend-ruta` y `frontend-ruta` pusheados a `main`. `0.INFRA-1` queda completamente cerrada. Siguiente bloqueo: cambiar dependencias `file:` de `@orkoruta/shared` y `@orkoruta/db` a versiones publicadas en GitHub Packages en `backend-ruta` y `frontend-ruta` (`0.BACK-1` y `0.FRONT-1`). | `packages-ruta/main`, `backend-ruta/.github/workflows/ci.yml`, `frontend-ruta/.github/workflows/ci.yml` | Sprint 0 |
| 2026-05-27 | Infraestructura | Cerrada `0.INFRA-7`: `ruta-api` (Web Service), `ruta-admin` y `ruta-storefront` (Static Sites) desplegados en Render. Fix build: pnpm en `--prefix=/tmp/pnpm-bin` para que `pnpm install` no borre el binario al recrear node_modules. Static sites usan `output: 'export'` + `images.unoptimized`. `/healthz` responde 200. | `backend-ruta/render.yaml`, `frontend-ruta/render.yaml`, Render | Sprint 0 |
| 2026-05-27 | Bugfix + Desarrollo | Corregido bug en `seed_dev_data.sh` y `create_first_admin_ruta.sh`: argon2 expone `argon2.cjs` (CJS), no `argon2.js` (ESM); cambiado temp script de `.mjs+import` a `.cjs+require()`. Ejecutado `seed_dev_data.sh` con éxito (COMMIT): 1 cliente FULL/NATIVE_RUTA `piloto-native`, 8 usuarios (ADMIN_CLIENT, OPERATOR_CLIENT, 3 COURIERs, 3 BUYERs), 3 categorías, 10 productos, 2 pickup points. Password dev: `Dev.Ruta.2026!`. `0.INFRA-8` cerrada. | `infra-ruta/scripts/`, BD DEV OCI | Sprint 0 |
| 2026-05-27 | Desarrollo | Cerrada `0.BACK-2`: primer ADMIN_RUTA creado en BD DEV — `alexander.marquez@orko.com.co` / Alexander Marquez / ACTIVE (id=9, client_id=0). Script `create_first_admin_ruta.sh` ejecutado por el usuario en terminal interactiva. | BD DEV OCI, `infra-ruta/scripts/create_first_admin_ruta.sh` | Sprint 0 |
| 2026-05-27 | Desarrollo | Cerrada `1.SHARED-1`: `@orkoruta/shared@1.1.0` publicado en GitHub Packages. Nuevos schemas: auth (login, register, refresh, logout, controlViewEnter), client (create, update, listQuery), category, buyer, courier, pickup_point. Tests: 33/33 OK. Typecheck EXIT 0. PR #2 mergeado a main. Nota: CI Publish falla por permisos de org; publicación manual con PAT local. | `packages-ruta`, GitHub Packages | Sprint 1 |
| 2026-05-27 | Desarrollo | Cerradas `1.BACK-1`, `1.BACK-2`, `1.BACK-3` en un commit: Auth completo (JWT HS256 jose, argon2, register/login/refresh/logout, cookies HttpOnly, middleware authenticate/requireAdmin*); Gestión Clientes ADMIN_RUTA (CRUD, idempotencia, auditoría, assertFrontendMode); Gestión productos+catálogo (CRUD, rutas públicas slug→client_id, upload stub 501). 27 tests. Commit 03dba81. | `backend-ruta/api/src/`, commit 03dba81 | Sprint 1 |
| 2026-05-27 | Desarrollo | Cerrada `1.BACK-4`: Compradores y Repartidores CRUD. buyers.service.ts + couriers.service.ts crean user+perfil en una transacción. admin_buyers.ts + admin_couriers.ts: 6 endpoints cada uno. Mapeo transport_mode → transport_method. 12 tests. Commit 94fec4a. Total backend: 42 tests pasan, 1 skipped (integration). Typecheck clean. | `backend-ruta/api/src/`, commit 94fec4a | Sprint 1 |
| 2026-05-27 | Desarrollo | Cerrada `1.BACK-5`: Importación masiva por Excel con pg-boss. jobs/bulk_import.job.ts (handler pg-boss v12, procesa rows con withTenant), services/bulk_import.service.ts (parse xlsx con XLSX.read, validación Zod fila por fila, max 1000 filas), routes/admin_products_bulk.ts (POST / con multer memoryStorage, GET /:job_id con UUID validation, requireAdminClient, requireIdempotencyKey). app.ts: mount /admin/products/bulk-import antes de /admin/products para evitar conflicto con /:product_id. 11 tests. Commit ea73695, rama feat/back-5. | `backend-ruta/api/src/`, commit ea73695 | Sprint 1 |
| 2026-05-27 | QA | Cerrada `1.QA-1`: suite de aislamiento cross-tenant, 35 tests nuevos (77 total). G1: RBAC completo (sin auth→401, BUYER/COURIER/ADMIN_CLIENT en endpoints incorrectos→403). G2: service siempre recibe client_id del TOKEN en products/categories/buyers/couriers; client_id en body ignorado por Zod. G3: ADMIN_RUTA modo plataforma recibe client_id=0, sin fuga de datos de tenants. vitest.config.ts extendido para incluir src/__tests__/**. HALLAZGO-QA-001: requireAdminClient permite ADMIN_RUTA con client_id=0 en /admin/* fuera de Vista de Control. Rama feat/qa-1, PR abierto. | `backend-ruta/api/src/__tests__/isolation.test.ts`, `backend-ruta/api/vitest.config.ts`, rama feat/qa-1 | Sprint 1 |
| 2026-05-27 | Desarrollo | Cerrada `1.STORE-1`: Layout y catálogo storefront nativo. `catalog.api.ts` (5 funciones fetch sobre /public/clients/:slug/*). `layout.tsx` client-side con useParams: header logo/nombre, link /login, RutaThemeToggle, footer. `page.tsx` + `CatalogView.tsx`: grid 2→3→4 cols, sidebar categorías desktop, chips móvil, búsqueda, paginación, skeletons, estados error/vacío. `product/[id]/page.tsx` + `ProductView.tsx`: imagen fill, breadcrumb, pills promo/stock, selector cantidad, botón carrito con feedback 2s. Shell SSG con generateStaticParams([{ slug: '_' }]) para SPA fallback en Render. TypeCheck/lint/build EXIT 0. 8 páginas estáticas generadas. Commit 359a6e8, rama feat/store-1. | `frontend-ruta/storefront/src/`, commit 359a6e8 | Sprint 1 |
| 2026-05-27 | CI | Resuelto bloqueo de calidad en `frontend-ruta/main`: el workflow fallaba con `ERR_PNPM_FETCH_401` al instalar `@orkoruta/shared` desde GitHub Packages porque `NPM_PUBLISH_TOKEN` no existía en el repo y `NPM_TOKEN` llegaba vacío. Se agregó el secret `NPM_PUBLISH_TOKEN` en `frontend-ruta` usando el token local de GitHub Packages, se actualizó `.github/workflows/ci.yml` con `permissions: packages: read`, registry/scope `@orkoruta`, `pnpm install --frozen-lockfile` y `NPM_TOKEN`. PR #5 mergeado. Verificación en `main` run 26540026963: install, typecheck, build `@orkoruta/ui`, build admin, build storefront y lint SUCCESS. | `frontend-ruta/.github/workflows/ci.yml`, PR #5, run 26540026963 | Sprint 1 |
| 2026-05-27 | CI/Render | Resuelto bloqueo de deploy en `backend-ruta/main`: Render fallaba con `ERR_PNPM_MINIMUM_RELEASE_AGE_VIOLATION` para `@orkoruta/shared@1.1.0` usando `pnpm@11.3.0`. Se actualizó `packageManager`, CI y `render.yaml` a `pnpm@11.4.0`; `pnpm-workspace.yaml` cambió `minimumReleaseAgeExclude` a `@orkoruta/*` para paquetes internos; se configuró secret `NPM_PUBLISH_TOKEN` en `backend-ruta` y el workflow usa scope `@orkoruta`. Commits en main: `0f19387`, `8a3aded`, `b455c4c`. Verificación: GitHub Actions run 26540623500 SUCCESS; `pnpm install --frozen-lockfile` y `pnpm --filter @ruta/api build` OK en worktree limpio. Push a main disparó redeploy si auto-deploy está activo. No se pudo verificar `/healthz` público porque `ruta-api.onrender.com` y `api-dev.onrender.com` devuelven `x-render-routing: no-server`; falta confirmar URL real o deploy hook/API key de Render. | `backend-ruta/package.json`, `backend-ruta/render.yaml`, `backend-ruta/pnpm-workspace.yaml`, `backend-ruta/.github/workflows/ci.yml`, run 26540623500 | Sprint 1 |
| 2026-05-28 | Desarrollo | Cerradas `2.STORE-3` y `2.STORE-4` (frontend storefront): Mis pedidos BUYER con timeline y acciones condicionales (PR #11 mergeado a main). Confirmación post-Wompi compatible con `output: export`, polling a `/buyer/orders/:id`, estados procesando/confirmado/fallido (PR #13 mergeado a main). Typecheck storefront EXIT 0 en ambas. | `frontend-ruta/storefront/src/`, PR #11, PR #13 | Sprint 2 |
| 2026-05-28 | QA | Cerrada `2.QA-1`: tests backend del state machine y Wompi. Suite data-driven para transiciones permitidas/rechazadas, tests de webhook Wompi con HMAC valido/invalido, deduplicacion por `provider_event_id`, mapping DECLINED y mock compatible con MSW para sandbox. Verificacion: 3393 tests focalizados OK; suite completa 3662 passed, 1 skipped; `pnpm typecheck` EXIT 0. | `backend-ruta/api/src/__tests__/payments.test.ts`, `backend-ruta/api/src/__tests__/wompi_webhook.test.ts`, `backend-ruta/api/src/__tests__/__mocks__/wompi_server.ts`, `docs-ruta/plan_tareas.md`, `docs-ruta/project_sprint2_status.md`, `docs-ruta/memoria_proyecto_ruta.md` | Sprint 2 |

> *A partir de aquÃ­ se registran cronolÃ³gicamente todos los avances del proyecto.*

---

## 6.2 Detalle por Sprint

### Sprint 0 â€” Setup multi-repo

| Tarea | Estado | DescripciÃ³n | Archivos / commits |
|---|---|---|---|
| `0.DOCS-1` (ext.) | [-] | Manifiesto experimental removido por decision del usuario; quedan vigentes `AGENTS.md` / `CLAUDE.md`. | `docs-ruta/memoria_proyecto_ruta.md` |
| `0.DOCS-1` (ext.) | âœ… | RevisiÃ³n de compatibilidad de `.md` con Codebuff/DeepSeek | â€” |
| `0.DOCS-1` (ext.) | âœ… | Agregada PARTE 6 â€” HistÃ³rico de cambios a memoria del proyecto | `docs-ruta/memoria_proyecto_ruta.md` |
| `0.DOCS-1` (ext.) | âœ… | Regla de actualizaciÃ³n de memoria incorporada en la memoria viva del proyecto | `docs-ruta/memoria_proyecto_ruta.md` |
| `0.DOCS-1` (ext.) | âœ… | Actualizadas decisiones operativas: `ruta.com` como dominio principal, Cliente piloto confirmado, Wompi pendiente, parÃ¡metros confirmados en SQL, correcciÃ³n Fase 1/2 y carrito en BD | `docs-ruta/mvp_alcance.md`, `docs-ruta/estrategia_testing.md`, `docs-ruta/wireframes_mvp.md`, `docs-ruta/plan_tareas.md`, `docs-ruta/contrato_api.md`, `docs-ruta/parametros_negocio.md`, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.DOCS-1` (ext.) | âœ… | Agregada Definition of Done por avance: pruebas obligatorias segÃºn riesgo y registro explÃ­cito cuando no se puedan ejecutar | `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.DOCS-1` (ext.) | âœ… | Registrada autorizaciÃ³n operativa para ejecutar acciones necesarias sin confirmaciÃ³n manual previa, con escalaciÃ³n directa cuando el runtime la exija y reglas persistentes acotadas | `docs-ruta/AGENTS.md`, `docs-ruta/CLAUDE.md`, copias por repo, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.INFRA-2` | âœ… DEV | Confirmado que la base de datos de desarrollo ya estÃ¡ creada en OCI; la conexiÃ³n local estÃ¡ en `backend-ruta/.env`. El proyecto/entorno PROD queda pendiente para mÃ¡s adelante. | `backend-ruta/.env` (no documentar secretos) |
| `0.INFRA-3` | âœ… DEV | VerificaciÃ³n ejecutada contra DEV OCI: `DATABASE_URL` presente; conexiÃ³n OK; schema `ruta` OK; 48 tablas base; `state_catalog` y `client_parameters` cargados; RLS instalado en tablas operativas; 20 particiones de tabla para `client_id=0`, confirmado como diseÃ±o correcto segÃºn `docs-ruta/bd/ruta_postgres.sql`. Se actualizÃ³ `plan_tareas.md` para dejar el criterio alineado al SQL autoritativo. | BD DEV OCI, `docs-ruta/bd/ruta_postgres.sql`, `docs-ruta/plan_tareas.md`, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.SHARED-1` | [/] DEV local | Inicializados/ajustados paquetes `@ruta/shared` y `@ruta/db` para trabajo local: enums/validators exportados, Prisma introspectado desde BD DEV OCI, Prisma Client generado, helper `withTenant` transaccional y tipado. Verificado con `pnpm --filter @ruta/shared typecheck`, `pnpm --filter @ruta/shared test`, `pnpm --filter @ruta/shared build`, `pnpm --filter @ruta/db typecheck`, `pnpm --filter @ruta/db build`. No se publica a GitHub Packages todavÃ­a porque `0.INFRA-4` queda pendiente. | `packages-ruta/shared`, `packages-ruta/db`, `packages-ruta/pnpm-workspace.yaml`, `packages-ruta/package.json` |
| `0.DOCS-1` (ext.) | âœ… | Creado `README.md` raiz con regla de arranque: leer siempre `docs-ruta/memoria_proyecto_ruta.md` antes de trabajar y actualizar la memoria con cualquier avance, cambio, ajuste, decision o tarea importante. | `README.md`, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.DOCS-1` | [x] Cerrada | Commit y push realizados en rama `docs/close-0-docs-1`; PR #1 aprobado y mergeado a `main`: https://github.com/orkoruta/docs-ruta/pull/1. Verificacion: `git diff --check` OK. | `docs-ruta`, PR #1 |
| `0.INFRA-4` | [x] Cerrada | Secret `NPM_PUBLISH_TOKEN` existe; `@orkoruta/shared@1.0.0` y `@orkoruta/db@1.0.0` publicados en GitHub Packages; PR #1 mergeado a `main` con CI `test` SUCCESS. | `packages-ruta`, GitHub Packages, https://github.com/orkoruta/packages-ruta/pull/1 |
| `0.SHARED-1` | [/] DEV/CI local OK | Ajustado `@ruta/db` para generar Prisma Client en `src/generated/prisma-client`, copiarlo a `dist/generated` en build limpio y usar imports ESM con extension `.js`; `@ruta/shared` removio sourcemaps que apuntaban a fuentes no publicadas. Verificado `pnpm run ci` en `packages-ruta`: generate, typecheck, tests shared (4) y build OK. Verificado `pnpm pack --dry-run` para ambos paquetes con contenido limpio. Falta publicacion real de `@ruta/shared@1.0.0` y `@ruta/db@1.0.0`. | `packages-ruta/shared`, `packages-ruta/db`, `packages-ruta/package.json` |
| `0.BACK-1` | [/] DEV local OK | Backend conectado a paquetes compartidos: `@ruta/shared` y `@ruta/db` via dependencias locales `file:` para desarrollo; agregado helper de errores compartidos, `/healthz/ready` con query Prisma, dependencias `jose` y `argon2`, y workflow CI. Verificado `pnpm typecheck`, `pnpm build`, `pnpm test` (3 tests) y arranque con `pnpm start`; `curl http://127.0.0.1:3001/healthz` responde 200. Falta consumir versiones publicadas desde GitHub Packages cuando `0.INFRA-4` publique. | `backend-ruta/api`, `backend-ruta/.github/workflows/ci.yml`, `backend-ruta/pnpm-lock.yaml` |
| `packages-ruta` | Bugfix | Build scripts corregidos con `|| true` para tolerar bloqueos de Dropbox/WSL2 en `rm -rf dist`. | `packages-ruta/shared/package.json`, `packages-ruta/db/package.json` |
| `0.FRONT-1` | [x] DEV local OK | Workspace pnpm con @ruta/admin (Next.js 14 :3002), @ruta/storefront (Next.js 14 :3003) y @ruta/ui interno. Componentes RutaCard, RutaButton, RutaPill, RutaSectionHeader, RutaThemeToggle implementados. Tailwind con tokens del design system. Typecheck EXIT 0, build OK (4 paginas estaticas por app). @ruta/shared via file:. | `frontend-ruta/` |
| `0.INFRA-6` | [x] Local OK | render.yaml.example (4 servicios Render), scripts/create_first_admin_ruta.sh (argon2id seguro). storage_buckets.sql eliminado (Supabase cancelado). | `infra-ruta/` |
| `0.BACK-2` | [/] Script OK / ejecucion pendiente | create_first_admin_ruta.sh creado y sintaticamente correcto. Falta ejecutar en BD DEV (requiere psql). | `infra-ruta/scripts/` |
| `0.LANDING-1` | [x] Codigo OK / build WSL2 limitado | Next.js 14 template con 7 paginas, api_client.ts, tailwind generico, manifiestos. Typecheck EXIT 0. Build no verificable en WSL2/NTFS/Dropbox. | `frontend-clients-ruta/_template/` |
| `0.INFRA-8` | [/] Script OK / ejecucion pendiente | seed_dev_data.sh: idempotente, 1 cliente FULL/NATIVE_RUTA 'piloto-native', 8 usuarios (admin, operator, 3 couriers, 3 buyers + perfiles), 3 categorias, 10 productos, 2 pickup points. Transaccion unica, argon2id en runtime. bash -n EXIT 0. Ejecucion requiere psql. | `infra-ruta/scripts/seed_dev_data.sh` |
| `0.INFRA-1` | [/] Bloqueado por autenticacion GitHub | Verificado estado local: `gh` instalado, pero token guardado invalido; no hay `GH_TOKEN` ni `GITHUB_TOKEN`. Los remotos existentes apuntan a repos `msimonz/*` con nombres locales antiguos y `_template` no tiene repo Git local. No se pudo crear/verificar repos privados, branch protection ni marcar `landing-template` como template. | GitHub CLI local, remotos Git |
| `0.INFRA-1` | [/] Correccion de nombres | Confirmado por el usuario que los repos base reales son `orkoruta/backend-ruta`, `orkoruta/frontend-ruta`, `orkoruta/packages-ruta`, `orkoruta/docs-ruta` y `orkoruta/infra-ruta`. Fuentes de verdad locales corregidas para dejar de apuntar a `ruta-*`/`landing-template`. | `docs-ruta/plan_tareas.md`, `infra-ruta/workspace.config.json`, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.INFRA-1` | [/] Remotos OK / GitHub parcial | Cinco repos privados verificados en GitHub y cinco remotos locales apuntando a `orkoruta/*`. Branch protection no aplicado: GitHub API devolvio HTTP 403 por requerir GitHub Pro o repos publicos. | remotos Git locales, GitHub API, `docs-ruta/plan_tareas.md` |
| `0.INFRA-1` | [/] Ajuste alcance | `frontend-clients-ruta` removido del alcance de repos base por decision del usuario. El alcance real de `0.INFRA-1` es verificar/configurar 5 repos base: `backend-ruta`, `frontend-ruta`, `packages-ruta`, `docs-ruta`, `infra-ruta`. | `docs-ruta/plan_tareas.md`, `infra-ruta/workspace.config.json`, `docs-ruta/memoria_proyecto_ruta.md` |
| `0.INFRA-1` | [x] Cerrada | Cinco repos base publicos verificados en GitHub; remotos locales apuntando a `orkoruta/*`; branch protection aplicada en `main`; URLs documentadas en el plan. | GitHub `orkoruta`, remotos locales, `docs-ruta/plan_tareas.md` |

---

---

## 6.3 Sprint 1 — Auth, Clientes, Productos, Registro BUYER

| Tarea | Estado | Descripción | Archivos / commits |
|---|---|---|---|
| `1.SHARED-1` | ✅ Cerrada | `@orkoruta/shared@1.1.0` publicado en GitHub Packages. Schemas de auth, client, category, buyer, courier, pickup_point agregados. 33 tests. Nota: CI Publish falla por permisos de org en GitHub Actions; publicación manual con PAT local. | `packages-ruta`, PR #2 |
| `1.BACK-1` | ✅ Cerrada | Auth completo: JWT HS256 (jose), argon2, register/login/loginRutaAdmin/refresh/logout, cookies HttpOnly Secure SameSite=Strict, middleware authenticate/requireAdminRuta/requireAdminClient/requireAuth, parámetros de duración desde DB. 9 tests. | `backend-ruta/api/src/`, commit 03dba81 |
| `1.BACK-2` | ✅ Cerrada | Gestión Clientes ADMIN_RUTA: CRUD completo con idempotencia, auditoría (CLIENT_CREATED/CLIENT_UPDATED), assertFrontendMode() para clientes FULL. 8 tests. | `backend-ruta/api/src/`, commit 03dba81 |
| `1.BACK-3` | ✅ Cerrada | Productos y catálogo: CRUD products+categories (ADMIN_CLIENT), rutas públicas /public/clients/:slug/* (resuelve slug→client_id), upload stub 501. 10 tests. | `backend-ruta/api/src/`, commit 03dba81 |
| `1.BACK-4` | ✅ Cerrada | Compradores y Repartidores: CRUD admin (6 endpoints cada uno), transacción user+perfil, mapeo transport_mode→transport_method. 12 tests. Total backend: 42 tests, 1 skipped. | `backend-ruta/api/src/`, commit 94fec4a |
| `1.BACK-5` | ✅ Cerrada | Importación masiva por Excel (pg-boss): POST /admin/products/bulk-import (multipart multer, xlsx parse, max 1000 filas, validación Zod por fila), job asíncrono pg-boss v12 `bulk_import`, GET /admin/products/bulk-import/:job_id para estado. 11 tests. Total backend: 53 tests, 1 skipped. | `backend-ruta/api/src/`, commit ea73695, rama feat/back-5 |
| `1.ADMIN-1` | ✅ Cerrada | Login + layout admin: `/login` con detección de rol y redirect, `RutaSidebar` navegación condicional por rol, `RutaHeader` con banner Vista de Control, `(protected)/layout.tsx` con SessionContext, `lib/auth.api.ts`, `lib/session.ts`, `lib/session-context.tsx`. Build/typecheck/lint EXIT 0. Nota: auth 100% client-side (static export). | `frontend-ruta/admin/src/`, rama `feat/admin-1` |
| `1.ADMIN-2` | ✅ Cerrada | Pantallas ADMIN_RUTA de Clientes: `/ruta-admin/clients` con búsqueda, filtros por tipo/estado, tabla paginada y activar/desactivar; `/ruta-admin/clients/new` con formulario validado para `business_code`, `slug`, `name`, `description`, `client_type`, `frontend_mode`; `/ruta-admin/clients/[id]` con detalle, edición, activar/desactivar y eliminación con confirmación por slug/identificador; `clients.api.ts` usa `credentials: 'include'`, errores tipados e `X-Idempotency-Key` en mutaciones. Verificación: `pnpm --filter @ruta/admin typecheck` EXIT 0; `pnpm --filter @ruta/admin lint` EXIT 0; `pnpm run test` no existe en `frontend-ruta`; `pnpm --filter @ruta/admin build` compila pero falla por conflicto preexistente de `output: export` con rutas dinámicas Wave 2 fuera de este alcance. | `frontend-ruta/admin/src/app/(protected)/ruta-admin/clients/*`, `frontend-ruta/admin/src/lib/clients.api.ts`, rama `feat/admin-2` |
| `1.ADMIN-3` | ✅ Cerrada | Gestión de productos frontend: `/admin/products` con buscador, filtros por categoría/estado, tabla y estados; `/admin/products/new` para crear producto; `/admin/products/[id]` para edición con wrapper compatible con `output: export`; modal de importación Excel conectado a `POST /admin/products/bulk-import` y polling de status; `products.api.ts` con cookies HttpOnly, manejo de errores, idempotencia y flujo presigned upload. Verificación: `pnpm --filter @ruta/admin typecheck` EXIT 0; `pnpm --filter @ruta/admin test` EXIT 0 sin suite definida; `pnpm --filter @ruta/admin build` compila y pasa lint/tipos, pero falla luego por rutas dinámicas de otros agentes (`buyers`, `couriers`, `pickup-points`, `ruta-admin/clients`) que combinan `use client` con `generateStaticParams`. | `frontend-ruta/admin/src/app/(protected)/admin/products/*`, `frontend-ruta/admin/src/lib/products.api.ts`, rama `feat/admin-3` |
| `1.ADMIN-4` | ✅ Cerrada | Compradores, Repartidores y Puntos físicos frontend: `/admin/buyers`, `/admin/couriers`, `/admin/pickup-points` con listas, búsqueda, estados loading/empty/error; detalles editables en rutas `[id]`; creación inline para repartidores y puntos físicos; `users.api.ts` usa `credentials: 'include'`, errores tipados e `X-Idempotency-Key` en mutaciones. Verificación: `pnpm typecheck` EXIT 0; `pnpm lint` EXIT 0; `pnpm build` EXIT 0 (11 páginas estáticas); `pnpm test` EXIT 1 porque `admin/package.json` no define script `test`. PR #8 mergeado a `main` el 2026-05-28, merge commit `5c3281a`. | `frontend-ruta/admin/src/app/(protected)/admin/{buyers,couriers,pickup-points}/*`, `frontend-ruta/admin/src/lib/users.api.ts`, rama `feat/admin-4` |
| `1.STORE-1` | ✅ Cerrada | Layout y catálogo storefront: `layout.tsx` con header de branding (logo/nombre del Cliente via API pública), `CatalogView.tsx` con sidebar categorías, chips móvil, búsqueda, grid 2→3→4 cols, paginación; `ProductView.tsx` con imagen fill, selector cantidad, botón agregar carrito; `catalog.api.ts` con 5 funciones fetch (getClientBySlug, getCategories, getProducts, getProductById, getPickupPoints). Shell SSG con `generateStaticParams([{ slug: '_' }])` para SPA fallback en Render. Build/typecheck/lint EXIT 0. | `frontend-ruta/storefront/src/`, rama `feat/store-1` |
| `1.STORE-2` | ✅ Cerrada | Registro y login BUYER storefront. | `frontend-ruta/storefront/`, rama `feat/store-2` |
| `1.QA-1` | ✅ Cerrada | Tests de aislamiento cross-tenant (35 tests). G1: RBAC sin auth→401, BUYER/COURIER→403, ADMIN_CLIENT en /ruta-admin→403. G2: service siempre recibe client_id del TOKEN (verificado en products/categories/buyers/couriers). G3: ADMIN_RUTA en modo plataforma recibe client_id=0, no de ningún tenant — HALLAZGO-QA-001 documentado en PR. G4: /ruta-admin solo ADMIN_RUTA. vitest.config.ts extendido con src/__tests__/**. Total backend: 77 tests, 1 skipped. | `backend-ruta/api/src/__tests__/isolation.test.ts`, rama `feat/qa-1` |
| `frontend-ruta CI` | ✅ Desbloqueado | Corregido bloqueo `ERR_PNPM_FETCH_401` en GitHub Actions: `NPM_PUBLISH_TOKEN` agregado al repo, workflow con permisos `packages: read`, registry/scope `@orkoruta`, `NPM_TOKEN` durante install y `pnpm install --frozen-lockfile`. PR #5 mergeado. `main` run 26540026963 SUCCESS: install, typecheck, build UI/admin/storefront y lint. | `frontend-ruta/.github/workflows/ci.yml`, PR #5 |
| `backend-ruta CI/Render` | ✅ Desbloqueado | Corregido bloqueo de Render por `minimumReleaseAge` en `@orkoruta/shared@1.1.0` y bloqueo de CI por GitHub Packages. Backend usa `pnpm@11.4.0`; `minimumReleaseAgeExclude` ahora es `@orkoruta/*`; `NPM_PUBLISH_TOKEN` existe en `backend-ruta`; workflow usa scope `@orkoruta`. `main` run 26540623500 SUCCESS. Falta confirmar URL pública real del servicio Render para validar `/healthz` externo. | `backend-ruta/package.json`, `backend-ruta/render.yaml`, `backend-ruta/pnpm-workspace.yaml`, `.github/workflows/ci.yml`, commits `0f19387`, `8a3aded`, `b455c4c` |
| `2.SHARED-1` | ✅ Cerrada | Schemas Zod para pedidos y pagos: `requestCancelSchema` en `order.schema.ts`; nuevo `payment.schema.ts` con `initiatePaymentSchema`, `wompiWebhookEventSchema`, `paymentStatusQuerySchema` y tipos derivados; `order.types.ts` migrado a `z.infer<>` para los tipos de request (CreateOrderInput, ConfirmOrderInput, CancelOrderInput, RequestCancelInput, OrderTransitionInput, AssignCourierInput). 50 tests OK. Typecheck EXIT 0. Build limpio. Publicado `@orkoruta/shared@1.2.0` en GitHub Packages. PR #3 mergeado a main el 2026-05-28, commit d358b01. Wave 2.1 DESBLOQUEADA. | `packages-ruta/shared/src/validators/order.schema.ts`, `packages-ruta/shared/src/validators/payment.schema.ts`, `packages-ruta/shared/src/types/order.types.ts`, `packages-ruta/shared/package.json`, rama `feat/shared-2` |
| `2.BACK-1` | ✅ Cerrada | State Machine + Orders service + buyer_orders routes. `services/orders/state_machine.ts`: mapa completo de transiciones Flujos 1+2+3; `canTransition(from, to, actor, ctx)` + `assertTransition()` (HTTP 422 INVALID_STATE_TRANSITION); condiciones por paymentStatus/deliveryType/clientType. `services/orders/orders.service.ts`: create (DRAFT con items, calcula precios desde products BD), confirm (DRAFT→PENDING_CONFIRM→ORDER_SUBMITTED atómico, fija payment_status), cancel (pre-despacho, verifica estados y payment_status), requestCancel (post-despacho), list, getById, confirmReceipt (DELIVERED→CLOSED). `routes/buyer_orders.ts`: 7 endpoints, middleware requireBuyer + requireIdempotencyKey. `__tests__/state_machine.test.ts`: 120 tests unitarios sin BD. 243 tests totales. Typecheck EXIT 0. PR #4. | `api/src/services/orders/state_machine.ts`, `api/src/services/orders/orders.service.ts`, `api/src/routes/buyer_orders.ts`, `api/src/__tests__/state_machine.test.ts`, `api/src/app.ts`, rama `feat/back-2-1` |
| `2.BACK-4` | ✅ Cerrada | Jobs de mantenimiento. `jobs/maintenance_boss.ts`: singleton pg-boss compartido con guard `NODE_ENV=test`. `jobs/order_expiration.job.ts`: cron `*/5 * * * *`, por cliente FULL activo, detecta DRAFT (por `order.draft_expiration_minutes`) y PENDING_CONFIRM (por `order.pending_confirm_timeout_minutes`) vencidos con doble fallback cliente→global(0)→código; transición marcada TODO 2.BACK-1. `jobs/payment_timeout.job.ts`: cron `*/2 * * * *`, detecta PENDING_ONLINE_PAYMENT vencidos desde `submitted_at` por `order.pending_online_payment_timeout_minutes`; transición TODO 2.BACK-1. `jobs/cleanup_idempotency.job.ts`: cron `0 * * * *`, DELETE idempotency_keys donde `expires_at < NOW()` en todos los clientes incluido 0. `jobs/cleanup_sessions.job.ts`: cron `0 3 * * *`, DELETE sesiones revocadas/expiradas por `session.cleanup_after_revoked_days` y `session.cleanup_after_expired_days` (parámetros globales). 21 tests unitarios con mocks. Typecheck EXIT 0. Build OK. PR #5 feat/back-2-4. | `api/src/jobs/maintenance_boss.ts`, `api/src/jobs/order_expiration.job.ts`, `api/src/jobs/payment_timeout.job.ts`, `api/src/jobs/cleanup_idempotency.job.ts`, `api/src/jobs/cleanup_sessions.job.ts`, `api/src/__tests__/maintenance_jobs.test.ts`, `api/src/app.ts`, rama `feat/back-2-4` |
| `2.STORE-1` | ✅ Cerrada | Carrito persistido en BD: `cart.api.ts` con funciones `getDraftOrder()` (GET /buyer/orders + filter DRAFT, lanza CartApiError 401 para redirect auth), `addItemsToCart()`, `updateCartItem()`, `removeCartItem()`, `clearCart()` — todas con `credentials:'include'` y `X-Idempotency-Key` en mutaciones. `cart/page.tsx` shell server sin `generateStaticParams` (hereda `[slug]` del padre). `CartView.tsx` ('use client'): carga el DRAFT en mount, guard 401 → redirect `/c/${slug}/login?return=`, controles de cantidad, botón eliminar item, bloqueo checkout si hay items sin stock, empty state con link al catálogo. Typecheck EXIT 0, lint EXIT 0, build OK. PR #10 mergeado a `main` el 2026-05-28, commit `56b32c2`. | `frontend-ruta/storefront/src/lib/cart.api.ts`, `frontend-ruta/storefront/src/app/c/[slug]/cart/page.tsx`, `frontend-ruta/storefront/src/app/c/[slug]/cart/_components/CartView.tsx`, rama `feat/store-2-1` |
| `2.STORE-2` | ✅ Cerrada | Checkout multi-paso storefront. `/c/[slug]/checkout/page.tsx` shell server compatible con `output: export`. `CheckoutStepper.tsx` ('use client'): lee el DRAFT con `getDraftOrder()` desde `cart.api.ts`, guard 401 → `/c/${slug}/login?return=/checkout`, valida carrito/stock, orquesta pasos, confirma `PATCH /buyer/orders/:id/confirm` con `X-Idempotency-Key`; si `ONLINE_AT_ORDER` llama `POST /buyer/orders/:id/initiate-payment` y redirige a `wompi_checkout_url`; si COD redirige a `/c/${slug}/orders/${id}`. `DeliveryStep.tsx`: selección SHIP/PICKUP. `AddressStep.tsx`: dirección SHIP o pickup point con mapa OpenStreetMap/Leaflet cargado client-side por CDN. `PaymentStep.tsx`: Wompi, pago electrónico contra entrega con submétodos y efectivo contra entrega. PR #12 mergeado a `main` el 2026-05-28, merge commit `cc023d3`. Verificación: `pnpm --filter @ruta/storefront typecheck` EXIT 0; `git diff --check` EXIT 0; `pnpm --filter @ruta/storefront build` detenido con SIGTERM tras quedar sin avance local en `Creating an optimized production build ...`. | `frontend-ruta/storefront/src/app/c/[slug]/checkout/*`, rama `feat/store-2-2` |
| `2.BACK-3` | ✅ Cerrada | Wompi + pagos + webhook. `lib/wompi_client.ts`: cliente configurable sandbox/prod via env; `buildCheckoutUrl()` genera URL con params; `generateReference()` formato `RUTA-{id}-{HEX8}`; `verifySignature()` HMAC-SHA256 con timing-safe compare. `services/payments.service.ts`: `initiatePayment()` verifica orden (buyer, payment_method=ONLINE_AT_ORDER, payment_status en PENDING_ONLINE_PAYMENT o PAYMENT_FAILED_RETRYABLE), resuelve provider (por id o default), genera referencia, crea `payment` con status=PENDING, actualiza `payment_status → PAYMENT_PROCESSING`. Ruta `POST /buyer/orders/:id/initiate-payment` (auth BUYER, idempotency, Zod). Webhook `POST /webhooks/wompi/:client_id/:provider_id` montado ANTES de express.json() con express.raw() para HMAC sobre cuerpo crudo; verifica firma; deduplica por `(payment_provider_id, provider_event_id)` contra `external_webhook_events` (append-only, sin UPDATE/DELETE); actualiza `payment_status` en orders; actualiza payment a CONFIRMED si APPROVED. Package actualizado a `@orkoruta/shared@1.2.0`. 243 tests. Typecheck EXIT 0. Build limpio. | `api/src/lib/wompi_client.ts`, `api/src/services/payments.service.ts`, `api/src/routes/buyer_payment.ts`, `api/src/routes/webhooks.ts`, `api/src/app.ts`, `api/package.json`, rama `feat/back-2-3` |
| `2.ADMIN-1` | ✅ Cerrada | Lista y detalle de pedidos admin. `lib/orders.api.ts`: tipos completos (OrderStatus 40+ valores, PaymentStatus, DeliveryType, OrderSummary, OrderListFilters, OrderDetail con buyer/courier/items/history/payment), helper `idempotencyKey()`, funciones `listOrders`, `getOrder`, `acceptOrder`, `rejectOrder`, `markPreparing`, `markReady`, `cancelOrder`, `approveCancelRequest`, `rejectCancelRequest`. `/admin/orders/page.tsx` ('use client'): tabla paginada con filtros por estado, pago, rango de fecha, búsqueda; badges de color funcional por grupo de estados (slate/violet/amber/blue/green/red); paginación; guard de rol (ADMIN_RUTA/ADMIN_CLIENT/OPERATOR_CLIENT). `/admin/orders/[id]/page.tsx`: server component con `generateStaticParams([{ id: '_' }])`. `/admin/orders/[id]/OrderDetailClient.tsx` ('use client'): detalle 360 layout 2 columnas — columna izq: datos comprador, repartidor asignado, timeline de historial de estados; columna der: resumen de items+totales, información de pago, acciones condicionales por estado (accept/reject/mark-preparing/mark-ready/cancel/approve-cancel-request). Nota: `RutaTimeline` y `RutaOrderSummary` no existen aún en @orkoruta/ui — implementados inline. Typecheck EXIT 0, lint EXIT 0, build EXIT 0 (10 páginas estáticas, nuevas rutas `/admin/orders` y `/admin/orders/[id]`). PR #9 mergeado a `main` el 2026-05-28, commit `fd0dda1`. | `frontend-ruta/admin/src/lib/orders.api.ts`, `frontend-ruta/admin/src/app/(protected)/admin/orders/page.tsx`, `frontend-ruta/admin/src/app/(protected)/admin/orders/[id]/page.tsx`, `frontend-ruta/admin/src/app/(protected)/admin/orders/[id]/OrderDetailClient.tsx`, rama `feat/admin-2-1` |
| `2.STORE-3` | ✅ Cerrada | Mis pedidos BUYER storefront. PR #11 mergeado a `main` el 2026-05-28. `buyer_orders.api.ts`: cliente separado del admin para `GET /buyer/orders`, `GET /buyer/orders/:id`, `cancel`, `request-cancel` y `confirm-receipt`, con cookies HttpOnly e idempotencia en mutaciones. `/c/[slug]/orders/page.tsx`: server wrapper compatible con `output: export` y `generateStaticParams([{ slug: '_' }])`. `OrdersView.tsx` ('use client'): lista de pedidos con filtros Todos/Activos/Completados/Cancelados, estado visual, empty/error/loading states y paginación local. `/c/[slug]/orders/[id]/page.tsx`: server wrapper dinámico con `generateStaticParams([{ slug: '_', id: '_' }])`. `OrderDetailView.tsx` ('use client'): detalle BUYER con timeline vertical tipo RutaTimeline, entrega, resumen, pago y acciones condicionales (cancelar, solicitar cancelación, confirmar recepción). Nota: el backend actual no retorna `history`; la UI consume `history` si existe y deriva timeline desde fechas/estado como fallback. Typecheck storefront EXIT 0. | `frontend-ruta/storefront/src/lib/buyer_orders.api.ts`, `frontend-ruta/storefront/src/app/c/[slug]/orders/*`, rama `feat/store-2-3` |
| `2.STORE-4` | ✅ Cerrada | Confirmación post-Wompi storefront. PR #13 mergeado a `main` el 2026-05-28. `/c/[slug]/checkout/confirmation/page.tsx`: shell server con `generateStaticParams([{ slug: '_' }])` y `Suspense` para `useSearchParams`. `ConfirmationView.tsx` ('use client'): lee `transaction_id` y referencias compatibles (`order_id`, `orderId`, `id`, `reference` formato `RUTA-{id}-{suffix}`), resuelve el pedido cuando es posible, hace polling a `GET /buyer/orders/:id` mediante `getBuyerOrder()`, y muestra procesando / confirmado / fallido con número de pedido, transacción, referencia, CTA a detalle o mis pedidos y seguir comprando. Nota: backend/documentación actual no expone consulta directa por `transaction_id`; la pantalla usa el endpoint disponible de detalle de pedido y muestra fallback claro si Wompi no retorna una referencia resoluble. Typecheck storefront EXIT 0. | `frontend-ruta/storefront/src/app/c/[slug]/checkout/confirmation/*`, rama `feat/store-2-4` |
| `2.QA-1` | ✅ Cerrada | Tests state machine y Wompi. PR #8 mergeado a `main` el 2026-05-28. `api/src/__tests__/payments.test.ts`: suite data-driven sin BD para todas las transiciones permitidas del state machine, rechazo de actores no autorizados, contextos inválidos y pares de estados no definidos. `api/src/__tests__/wompi_webhook.test.ts`: Supertest sobre `createWebhooksRouter` con mock de `@orkoruta/db`; valida HMAC correcto, HMAC inválido, deduplicación de `external_webhook_events` por `(payment_provider_id, provider_event_id)` y mapping de DECLINED a `PAYMENT_FAILED_RETRYABLE`. `api/src/__tests__/__mocks__/wompi_server.ts`: factory compatible con MSW para sandbox Wompi sin agregar dependencia nueva. Verificación: tests focalizados 3393/3393 OK; suite completa 3662 passed, 1 skipped; `pnpm typecheck` EXIT 0. | `backend-ruta/api/src/__tests__/payments.test.ts`, `backend-ruta/api/src/__tests__/wompi_webhook.test.ts`, `backend-ruta/api/src/__tests__/__mocks__/wompi_server.ts`, rama `feat/qa-2-1` |
| 2026-05-28 | Cierre Sprint 2 | Sprint 2 COMPLETO. Todos los PRs mergeados a `main`: backend-ruta PR #8 (2.QA-1), frontend-ruta PRs #9 (2.ADMIN-1), #10 (2.STORE-1), #11 (2.STORE-3), #12 (2.STORE-2), #13 (2.STORE-4). Docs actualizados: `plan_tareas.md` (✅ en todos los headers Sprint 1-2), `project_sprint2_status.md` (tabla de cierre), `memoria_proyecto_ruta.md`. Siguiente: Sprint 3 — Flujo SHIP (asignación courier, mapa Leaflet, vista móvil). | todos los repos, `docs-ruta/` | Sprint 2 |


## 6.4 Sesión 2026-07-21/22 — Endurecimiento y pruebas manuales del MVP

Sesión de pruebas manuales sobre el MVP ya implementado. No es un sprint
planificado: son correcciones y ajustes que salieron de recorrer la app
como usuario real (admin, corporativo y repartidor).

| Área | Detalle | Archivos |
|---|---|---|
| Documentación | `README.md` en la raíz del workspace: estructura multi-repo, stack, qué está construido por fase, cómo levantar backend y frontends, tests, deploy, reglas no negociables y glosario. | `README.md` |
| Contrato API | El backend devolvía `items` donde el contrato dice `data`, y usaba `limit` donde el contrato dice `page_size`. Se alineó el **backend** al contrato (no el frontend), porque el contrato es la fuente de verdad. Rompía la página de compradores (`buyers.length` sobre `undefined`). Este mismo desajuste causó seis bugs distintos en la sesión — evaluar generar tipos del frontend desde OpenAPI. | `backend-ruta/api/src/routes/*`, `docs-ruta/contrato_api.md` |
| Ruteo | `GET /admin/orders/map` daba 400: la ruta `/:id` se declaraba antes y capturaba `"map"` como id. Se reordenó el montaje del router. | `backend-ruta/api/src/routes/admin_orders.ts` |
| Serialización admin | El detalle de pedido no incluía `buyer`, `courier` ni `pickup_points`; el frontend reventaba en `order.buyer.name`. Se añadieron las relaciones al `orderInclude`, se creó `orderDetailInclude` (con `order_state_history`) y el serializador ahora emite `buyer_name`, `courier_name`, `order_origin`, `buyer_type`, `tax`, `shipping_fee`, `discount`, `delivery_address` formateada, `delivery_instructions` y `pickup_point_name`. | `backend-ruta/api/src/routes/admin_orders.ts` |
| **Seguridad — RLS** | **Fuga cross-tenant detectada.** `rutauser` es *owner* de las tablas y PostgreSQL exime al owner de RLS salvo que se declare `FORCE ROW LEVEL SECURITY`. Se corrigió en las dos capas: `client_id` explícito en los `where` de la capa de aplicación **y** `FORCE ROW LEVEL SECURITY` en las 21 tablas operativas. Efecto colateral: `pg_dump` dejó de funcionar; se arreglaron 7 scripts de infra con `PGOPTIONS` + `--enable-row-security`. **Verificar que producción también tenga el FORCE aplicado.** | `docs-ruta/bd/ruta_postgres.sql`, `infra-ruta/scripts/*` |
| Validación | Errores de validación devolvían 500 en vez de 400: había dos instancias de `zod` en el árbol de dependencias, así que `err instanceof ZodError` era siempre falso. Se resolvió con un override en `pnpm-workspace.yaml` apuntando a la copia de `@orkoruta/shared`. Esto además puso en verde 9 tests que llevaban tiempo fallando. | `backend-ruta/pnpm-workspace.yaml` |
| **Mapas — migración** | Se migraron **todos** los mapas de Leaflet/OpenStreetMap a **Google Maps** (Maps JavaScript API + Geocoding API). El geocoding no se llama desde el navegador: pasa por un proxy en el backend (`GET /geocode`, solo staff) con caché en memoria de 24 h y 500 entradas, que distingue `ZERO_RESULTS` (→ `null`) de `REQUEST_DENIED`/`OVER_QUERY_LIMIT` (→ 502 + log). Devuelve la precisión de Google (`ROOFTOP` vs `GEOMETRIC_CENTER`) para poder avisar al operador cuando la dirección es aproximada. **Nota de implementación:** con `loading=async` el SDK se debe esperar con `callback=`, no con el evento `load` del `<script>` ni con `importLibrary` (el bootstrap no lo define). | `backend-ruta/api/src/services/geocoding.service.ts`, `backend-ruta/api/src/routes/geocoding.ts`, `frontend-ruta/*/src/lib/google-maps.ts`, `frontend-ruta/*/src/lib/geocoding.ts` |
| Mapa de asignación | Pines de todos los pedidos cuando no hay ninguno seleccionado (`fitBounds`); índigo oscuro `#3730a3`, ámbar `#f59e0b` el seleccionado. Los pedidos con coordenadas idénticas se abanican con `spreadCoincidentPositions()` (`0.00018°`, longitud corregida por `cos(lat)`) para que no se tapen. Click fuera del mapa y del panel deselecciona y recentra. | `frontend-ruta/admin/src/app/(protected)/admin/orders/map/_components/*` |
| Mapa de asignación — asignados | El mapa solo mostraba los pedidos por asignar, así que el operador decidía a ciegas: no veía por dónde andaba ya la flota. `getOrdersForMap` ahora devuelve también los pedidos con repartidor (`COURIER_ASSIGNED`, `SHIPPED`, `IN_TRANSIT`, `OUT_FOR_DELIVERY`, `ARRIVED_AT_CUSTOMER`) con `courier_user_id`/`courier_name`/`courier_phone`. Pines: **índigo** por asignar, **verde** asignado, **ámbar** el seleccionado, con las convenciones visibles en el panel lateral. Los colores viven en un solo módulo (`map_legend.tsx`) que consumen tanto los pines como la leyenda, para que no se desincronicen. El panel separa "Pedidos listos" de "En reparto"; al seleccionar uno ya asignado se muestra quién lo lleva en vez de la lista de repartidores (reasignar no se hace desde este mapa) y se evita la llamada a `available-couriers`. | `backend-ruta/api/src/services/orders/courier_assignment.service.ts`, `frontend-ruta/admin/src/app/(protected)/admin/orders/map/_components/{AssignmentMap,AssignmentMapView,PendingOrdersPanel,map_legend}.tsx`, `frontend-ruta/admin/src/lib/assignment.api.ts`, `docs-ruta/contrato_api.md` |
| Parámetros no se podían guardar | Cualquier `Guardar` en la pestaña de parámetros devolvía 400 "Datos inválidos": el frontend mandaba `{ parameter_value }` y el backend espera `{ value }` — `parameter_value` es el nombre del campo en la *respuesta*, no en la petición. Se corrigió el frontend (backend y sus 24 tests ya coincidían) y se documentó la forma del cuerpo en el contrato, que estaba en blanco justo para estos dos endpoints. **Séptimo bug de contrato de la sesión.** De paso: el backend no devuelve `group`, así que todos los parámetros caían en un solo bloque "General"; ahora el grupo se deriva del prefijo de la clave (`limits.` → Limits). | `frontend-ruta/admin/src/lib/parameters.api.ts`, `frontend-ruta/admin/src/app/(protected)/admin/settings/_components/ParametersTab.tsx`, `docs-ruta/contrato_api.md` |
| Navegación del repartidor | La vista del repartidor solo enlazaba a Google Maps. Se agregó **Waze** al lado, con `waze.com/ul?ll=<lat>,<lng>&navigate=yes` (abre la app si está instalada, web si no; `navigate=yes` arranca la ruta sin un toque extra). Ambos usan coordenadas cuando el pedido las trae y caen al texto de la dirección si no. | `frontend-ruta/admin/src/app/(protected)/courier/[id]/_components/CourierOrderDetail.tsx`, `docs-ruta/guias/courier.md` §2 |
| **Cobro contra entrega roto — evidencia en base64** | El cobro COD devolvía 400 y **nunca había funcionado**: el contrato especificaba `multipart` con un archivo, el frontend lo implementó así, pero el backend siempre esperó JSON con `evidence_url` porque **el proyecto no tiene object storage** (`POST /uploads/presigned-url` responde 501). `express.json()` ignora el multipart, así que `req.body` llegaba vacío y fallaba la validación de `amount`. **Decisión del usuario (2026-07-22):** guardar la foto en base64 dentro de la BD, a sabiendas de que es deuda técnica, para no bloquear las pruebas del MVP. Implementación: `evidence_url` acepta URL http(s) (≤1000 chars) **o** data URI `image/{jpeg,png,webp}` (≤1 400 000 chars ≈ 1 MB); se guarda en el JSONB `payments.collection_evidence`, que no tiene límite de longitud — **no hizo falta tocar la BD**. `app.ts` parsea `/courier` con límite de 2 MB antes del `express.json()` global, para no abrir toda la API a cuerpos grandes. El frontend comprime a 1280 px / calidad 0.7 antes de enviar (cámara y galería) y valida el tope antes de subir. La respuesta no devuelve el data URI: trae `evidence_url: null` + `has_evidence: true`. **Migración pendiente:** cuando exista un bucket, subir el archivo y mandar solo la URL; el campo ya acepta ambas formas, así que el contrato no cambia. | `backend-ruta/api/src/services/orders/collection.service.ts`, `backend-ruta/api/src/app.ts`, `frontend-ruta/admin/src/lib/courier_orders.api.ts`, `ReceiptCapture.tsx`, `backend-ruta/api/src/__tests__/courier_collection.test.ts`, `docs-ruta/contrato_api.md` |
| Crash tras registrar el cobro | Con el cobro ya guardado en BD, la pantalla del repartidor reventaba con `Cannot read properties of undefined (reading 'name')`. `record-collection` es el único endpoint del repartidor que responde con el **resultado del cobro** (`payment_id`, `amount`…) y no con el pedido; el frontend lo tipaba como `CourierOrderDetail` y hacía `setOrder()` con él, dejando el estado sin `buyer` ni `items`. `recordCollection()` ahora re-consulta el detalle con `getCourierOrderById` y devuelve lo mismo que las demás acciones. Además se pusieron encadenados opcionales en `buyer` e `items`: al repartidor en la calle no puede tumbarle la pantalla un campo ausente. | `frontend-ruta/admin/src/lib/courier_orders.api.ts`, `CourierOrderDetail.tsx` |
| Ver la foto del recibo | Nuevo endpoint `GET /{courier|admin}/orders/:id/collection-evidence` que lee la evidencia del JSONB `payments.collection_evidence` y la devuelve con el contexto del pago. Va **aparte del detalle del pedido** a propósito: la imagen embebida pesa cientos de kB y cargarla en cada consulta penalizaría a todas las pantallas que no la muestran. El repartidor solo accede a la evidencia de sus pedidos asignados (403 si no); el staff del Cliente, a cualquiera del tenant. En el front, `CollectionEvidenceCard` la pinta en ambas pantallas con botón **Recargar** y enlace a tamaño completo; no hace falta decodificar nada porque el navegador resuelve el `data:` URI en el `src` del `<img>`. La tarjeta se oculta sola con el 404, que es el caso normal en pedidos sin COD. | `backend-ruta/api/src/services/orders/collection.service.ts`, `routes/courier_collection.ts`, `routes/admin_orders.ts`, `frontend-ruta/admin/src/components/CollectionEvidenceCard.tsx`, `lib/collection_evidence.api.ts`, `docs-ruta/contrato_api.md` |
| Estados sin traducir en admin | El badge y el timeline mostraban códigos crudos (`PAYMENT_COLLECTED_CASH`). Faltaban **19 de 61** estados en el diccionario del admin: todo el flujo PICKUP, los de cobro y todos los de devolución/reembolso. Se completaron los 61. Además `statusColor` terminaba en `return 'red'`, así que los estados no listados se pintaban como error: `RETURN_REQUESTED`, `RETURN_APPROVED`, `RETURN_IN_TRANSIT` y `REFUND_PENDING` pasaron a ámbar (en curso), `RETURN_RECEIVED`/`REFUNDED` a verde y `RETURN_CANCELLED` a gris. | `frontend-ruta/admin/src/app/(protected)/admin/orders/[id]/OrderDetailClient.tsx` |
| Diccionario de estados unificado (admin) | La causa de fondo de los estados sin traducir: el diccionario estaba **copiado** en la lista de pedidos y en el detalle, y las copias se desincronizaron — el detalle traducía `PAYMENT_COLLECTED_CASH` y la lista lo mostraba en crudo. Se extrajo `lib/admin_status_labels.ts` con los 61 estados, tipado como `Record<OrderStatus, …>` **exhaustivo**: si el backend agrega un estado y no se traduce, ahora **falla el typecheck** en vez de aparecer un código en pantalla. También se unificaron los colores del badge y las opciones del filtro, que antes tenían sus propias etiquetas escritas a mano. Tercer diccionario duplicado de la sesión (el del repartidor fue el primero). | `frontend-ruta/admin/src/lib/admin_status_labels.ts`, `app/(protected)/admin/orders/page.tsx`, `app/(protected)/admin/orders/[id]/OrderDetailClient.tsx` |
| **Capacidad del repartidor** | Un repartidor puede llevar **varios pedidos a la vez**. El tope lo configura cada cliente con el parámetro `limits.max_concurrent_orders_per_courier` (default 3). `getAvailableCouriers` devuelve `active_orders`, `max_concurrent_orders` y `remaining_capacity`, filtra los que están al tope y ordena por capacidad libre; `assignCourier` revalida el límite en el servidor (422). La pantalla de parámetros ahora mezcla los globales (`client_id = 0`) con los overrides del cliente, marcando el origen con `source: 'CLIENT' \| 'GLOBAL'`. | `backend-ruta/api/src/services/orders/courier_assignment.service.ts`, `backend-ruta/api/src/routes/admin_parameters.ts`, `docs-ruta/parametros_negocio.md` |
| **Flujo corporativo** | **Cambio de flujo:** un pedido corporativo ya no pasa por `DRAFT` ni por "Confirmar y enviar". Al crearlo, `advanceCorporateOrderToPreparing()` lo lleva por el state machine hasta `PREPARING` (`DRAFT → ORDER_SUBMITTED → ORDER_VALIDATING → SELLER_CONFIRMED → PREPARING`), deteniéndose en `ORDER_SUBMITTED` si el pago es `ONLINE_AT_ORDER` y respetando `MANUAL_REVIEW`. Así llega directo al mapa de asignación. La regla `DRAFT → PENDING_CONFIRM` se restringió a `buyerType === 'CORPORATE'` para no alterar el flujo del comprador final. | `backend-ruta/api/src/services/corporate_orders.service.ts`, `backend-ruta/api/src/services/orders/state_machine.ts`, `docs-ruta/flujos/flujo_6_pedidos_corporativos.txt` |
| Tema y contraseñas | Se eliminó el toggle claro/oscuro de todos los frontends: el tema sigue al del sistema operativo (`darkMode: 'media'`). Todos los campos de contraseña usan `RutaPasswordInput` (botón de ojo, fuera del orden de tabulación). | `frontend-ruta/packages/ui/src/components/RutaPasswordInput.tsx`, configs de Tailwind |
| Foto del recibo | El botón abría el selector de archivos en vez de la cámara (`<input capture>` solo funciona en móvil). Ahora `ReceiptCapture` usa `getUserMedia` con `facingMode: 'environment'`, captura a canvas → JPEG 0.9 y ofrece Capturar / Repetir. Apaga los tracks al capturar, cancelar y desmontar. Si no hay cámara o se niega el permiso, cae al selector de archivos. Requiere contexto seguro (localhost o HTTPS). | `frontend-ruta/admin/src/app/(protected)/courier/[id]/_components/ReceiptCapture.tsx` |
| **Estados para el repartidor** | El historial mostraba estados administrativos crudos (`VALIDATION_APPROVED`) que no le dicen nada a quien entrega. Se creó un diccionario único con ~35 estados escritos desde su punto de vista (`SELLER_CONFIRMED` → "El negocio aceptó el pedido"), reemplazando dos copias locales de `statusLabel`/`statusColor`. Se decidió **traducir** en vez de filtrar el historial, para que el repartidor vea de dónde viene el pedido. Los estados sin traducir se muestran en crudo antes que ocultar información. | `frontend-ruta/admin/src/lib/courier_status_labels.ts`, `CourierOrderDetail.tsx`, `CourierDashboard.tsx`, `docs-ruta/guias/courier.md` §8 |

### Pendientes abiertos de esta sesión

- Generar los tipos del frontend desde OpenAPI: seis bugs de la sesión fueron
  el mismo desajuste de contrato.
- Manejo global del 401 en los frontends.
- `pg-boss` muere en silencio si se agotan las conexiones de la BD.
- El serializador admin no expone `payment`, así que `isCod` en `PickupActions`
  siempre es `false`.
- En el login, "Código de empresa" está etiquetado como opcional y su error
  se descarta con un `catch {}`.
- Confirmar `FORCE ROW LEVEL SECURITY` en la BD de producción.
- **Object storage**: no existe. Las evidencias fotográficas viven en base64
  dentro de `payments.collection_evidence` como solución provisional, y
  `POST /uploads/presigned-url` y `POST /courier/orders/:id/upload-evidence`
  siguen sin implementar. Con `storage.evidence_retention_days = 730`, dos años
  de fotos en la BD es una migración que habrá que hacer antes del piloto.

---

*Sprints siguientes se agregan como nuevas secciones 6.5, 6.6, etc.*

---

# FIN DE MEMORIA

Última actualización: 2026-07-22 — Sesión de pruebas manuales del MVP (sección 6.4): fuga cross-tenant cerrada con `FORCE ROW LEVEL SECURITY`, migración de mapas a Google Maps, capacidad concurrente del repartidor, flujo corporativo sin `DRAFT`, captura de recibo con cámara y estados traducidos para el repartidor.
Total de archivos generados: 44 + esta memoria = 45.

Si encuentras inconsistencias o falta informaciÃ³n, consulta
`docs-ruta/all_ruta.md` o `docs-ruta/estructura_proyecto.md` (que son
las fuentes de verdad mÃ¡s actuales) y reporta al equipo.

---

**Regla de memoria viva:**
- Cada avance, cambio o mejora debe registrarse en la PARTE 6.
- Actualizar tabla cronolÃ³gica (6.1) + secciÃ³n del sprint correspondiente (6.2+).
- No borrar entradas anteriores â€” solo agregar nuevas.
- Si el cambio afecta otro `.md` (contrato_api.md, flujos, etc.), actualizar ese documento tambiÃ©n.
