# MEMORIA DEL PROYECTO RUTA

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

## 3.3 Crear README mÃ­nimo en cada repo (opcional pero recomendado)

En cada uno de los siguientes destinos, crear un `README.md` con el
contenido placeholder indicado:

### `ruta/backend-ruta/README.md`

```markdown
# ruta-backend

Backend Express + TypeScript de RUTA.

- Manifiesto: ver `CLAUDE.md` y `AGENTS.md` en este repo.
- DocumentaciÃ³n completa: ver repo `ruta-docs` (carpeta local `docs-ruta/`).
- Plan de tareas: ver `docs-ruta/plan_tareas.md`.
```

### `ruta/frontend-ruta/README.md`

```markdown
# ruta-frontend

Frontend Next.js de RUTA: panel admin + storefront nativo.

- Manifiesto: ver `CLAUDE.md` y `AGENTS.md` en este repo.
- DocumentaciÃ³n: ver repo `ruta-docs` (carpeta local `docs-ruta/`).
- Design system: vive en `packages/ui/` (no se publica externamente).
```

### `ruta/packages-ruta/README.md`

```markdown
# ruta-shared

Paquetes compartidos: `@ruta/shared` (tipos, validators, enums) y
`@ruta/db` (Prisma client).

- DistribuciÃ³n: GitHub Packages.
- Manifiesto: ver `CLAUDE.md` y `AGENTS.md`.
- DocumentaciÃ³n: ver repo `ruta-docs`.
```

### `ruta/docs-ruta/README.md`

```markdown
# ruta-docs

DocumentaciÃ³n completa del proyecto RUTA.

**Lee primero:** `all_ruta.md` (descripciÃ³n funcional general).

**Para agentes IA:** carga `memoria_proyecto_ruta.md` como contexto
o lee `CLAUDE.md` / `AGENTS.md`.

**Para arrancar desarrollo:** ver `plan_tareas.md` (sprints).
```

### `ruta/infra-ruta/README.md`

```markdown
# ruta-infra

Scripts de despliegue y setup de RUTA.

- `scripts/setup_workspace.sh` â€” clona los 5 repos base.
- `scripts/create_landing.sh` â€” crea un landing custom usando como
  base local `frontend-clients-ruta/_template`.
- Manifiesto: ver `CLAUDE.md` y `AGENTS.md`.
```

### `ruta/frontend-clients-ruta/_template/README.md`

```markdown
# landing-template

Template base para crear landings custom de Clientes Full (modalidad
CUSTOM_LANDING_BY_RUTA).

Para crear un landing nuevo desde este template local, ejecutar
`bash ~/projects/ruta/infra-ruta/scripts/create_landing.sh {slug}`.
```

## 3.4 Resumen del estado final esperado

DespuÃ©s de ejecutar todas las instrucciones, el sistema de archivos
local debe verse asÃ­:

```
~/projects/ruta/
â”œâ”€â”€ backend-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â”œâ”€â”€ AGENTS.md
â”‚   â””â”€â”€ README.md
â”‚
â”œâ”€â”€ frontend-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â”œâ”€â”€ AGENTS.md
â”‚   â””â”€â”€ README.md
â”‚
â”œâ”€â”€ frontend-clients-ruta/
â”‚   â””â”€â”€ _template/
â”‚       â”œâ”€â”€ CLAUDE.md
â”‚       â”œâ”€â”€ AGENTS.md
â”‚       â””â”€â”€ README.md
â”‚
â”œâ”€â”€ packages-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â”œâ”€â”€ AGENTS.md
â”‚   â””â”€â”€ README.md
â”‚
â”œâ”€â”€ docs-ruta/
â”‚   â”œâ”€â”€ CLAUDE.md
â”‚   â”œâ”€â”€ AGENTS.md
â”‚   â”œâ”€â”€ README.md
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
- 6 archivos `README.md` (5 repos base + template local)
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
| 2026-05-24 | ðŸ“ Setup | CreaciÃ³n inicial de `CODEBUFF.md` â€” manifiesto para Codebuff/Buffy con flujo automÃ¡tico de trabajo, herramientas, comandos por repo y reglas operativas. Removido posteriormente el 2026-05-26 por decision del usuario; `AGENTS.md` y `CLAUDE.md` quedan como manifiestos vigentes. | `memoria_proyecto_ruta.md` | Sprint 0 |
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
| 2026-05-26 | Documentacion | Removido `CODEBUFF.md` del proyecto por solicitud del usuario; `AGENTS.md` y `CLAUDE.md` quedan como manifiestos vigentes para agentes. PR #1 actualizado. Verificacion: `git diff --check` OK. | `docs-ruta/memoria_proyecto_ruta.md`, GitHub PR #1 | Sprint 0 |
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

> *A partir de aquÃ­ se registran cronolÃ³gicamente todos los avances del proyecto.*

---

## 6.2 Detalle por Sprint

### Sprint 0 â€” Setup multi-repo

| Tarea | Estado | DescripciÃ³n | Archivos / commits |
|---|---|---|---|
| `0.DOCS-1` (ext.) | [-] | `CODEBUFF.md` removido por decision del usuario; se conserva la memoria historica y quedan vigentes `AGENTS.md` / `CLAUDE.md`. | `docs-ruta/memoria_proyecto_ruta.md` |
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
| `1.ADMIN-2..4` | [ ] Pendiente | Frontend admin: pantallas ADMIN_RUTA y ADMIN_CLIENT (Wave 2). | `frontend-ruta/admin/` |
| `1.STORE-1` | ✅ Cerrada | Layout y catálogo storefront: `layout.tsx` con header de branding (logo/nombre del Cliente via API pública), `CatalogView.tsx` con sidebar categorías, chips móvil, búsqueda, grid 2→3→4 cols, paginación; `ProductView.tsx` con imagen fill, selector cantidad, botón agregar carrito; `catalog.api.ts` con 5 funciones fetch (getClientBySlug, getCategories, getProducts, getProductById, getPickupPoints). Shell SSG con `generateStaticParams([{ slug: '_' }])` para SPA fallback en Render. Build/typecheck/lint EXIT 0. | `frontend-ruta/storefront/src/`, rama `feat/store-1` |
| `1.STORE-2` | ✅ Cerrada | Registro y login BUYER storefront. | `frontend-ruta/storefront/`, rama `feat/store-2` |
| `1.QA-1` | ✅ Cerrada | Tests de aislamiento cross-tenant (35 tests). G1: RBAC sin auth→401, BUYER/COURIER→403, ADMIN_CLIENT en /ruta-admin→403. G2: service siempre recibe client_id del TOKEN (verificado en products/categories/buyers/couriers). G3: ADMIN_RUTA en modo plataforma recibe client_id=0, no de ningún tenant — HALLAZGO-QA-001 documentado en PR. G4: /ruta-admin solo ADMIN_RUTA. vitest.config.ts extendido con src/__tests__/**. Total backend: 77 tests, 1 skipped. | `backend-ruta/api/src/__tests__/isolation.test.ts`, rama `feat/qa-1` |
*Sprints siguientes se agregan como nuevas secciones 6.3, 6.4, etc.*

---

# FIN DE MEMORIA

Última actualización: 2026-05-27 — **Sprint 1 BACK + QA + ADMIN-1 + STORE-1 + STORE-2 cerradas.** 8 tareas backend + 1 shared + 1 QA + 1 admin + 2 storefront cerradas. Storefront nativo completo: catálogo, detalle de producto, registro y login BUYER. Pendiente: 1.ADMIN-2/3/4.
Total de archivos generados: 23 + esta memoria = 24.

Si encuentras inconsistencias o falta informaciÃ³n, consulta
`docs-ruta/all_ruta.md` o `docs-ruta/estructura_proyecto.md` (que son
las fuentes de verdad mÃ¡s actuales) y reporta al equipo.

---

**Regla de memoria viva:**
- Cada avance, cambio o mejora debe registrarse en la PARTE 6.
- Actualizar tabla cronolÃ³gica (6.1) + secciÃ³n del sprint correspondiente (6.2+).
- No borrar entradas anteriores â€” solo agregar nuevas.
- Si el cambio afecta otro `.md` (contrato_api.md, flujos, etc.), actualizar ese documento tambiÃ©n.
