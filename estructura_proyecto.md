# RUTA — Estructura del Proyecto (Repositorios Separados)

## Decisión confirmada

El código de RUTA se organiza en **repositorios Git separados**.
Localmente se clonan bajo una carpeta `ruta/` común.

**6 repos base** + **N repos** (uno por cada Cliente Full con landing
custom en modalidad `CUSTOM_LANDING_BY_RUTA`).

```
ruta/                                    ← carpeta local (NO es un repo)
│
├── backend-ruta/                             ← repo: ruta-backend
│   └── api/                             ← Express API
│
├── frontend-ruta/                            ← repo: ruta-frontend
│   ├── admin/                           ← Next.js panel administrativo
│   └── storefront/                      ← Next.js storefront nativo
│
├── frontend-clients-ruta/                    ← carpeta local (NO es un repo)
│   ├── _template/                       ← repo: landing-template (GitHub template)
│   ├── cliente-1/                       ← repo independiente: landing-{slug-cliente-1}
│   ├── cliente-2/                       ← repo independiente: landing-{slug-cliente-2}
│   └── cliente-n/                       ← repo independiente: landing-{slug-cliente-n}
│
├── packages-ruta/                            ← repo: ruta-shared
│   ├── shared/                          ← @ruta/shared (npm)
│   └── db/                              ← @ruta/db (npm)
│
├── docs-ruta/                                ← repo: ruta-docs
└── infra-ruta/                               ← repo: ruta-infra
```

---

## Mapeo final de repositorios

### 6 repos base

| # | Carpeta local | Repositorio Git | Tecnología |
|---|---|---|---|
| 1 | `ruta/backend-ruta/api/` | `ruta-backend` | Express + TypeScript |
| 2 | `ruta/frontend-ruta/{admin,storefront}/` | `ruta-frontend` | Next.js (pnpm workspace) |
| 3 | `ruta/packages-ruta/{shared,db}/` | `ruta-shared` | Paquetes npm publicables |
| 4 | `ruta/docs-ruta/` | `ruta-docs` | Markdown + SQL |
| 5 | `ruta/infra-ruta/` | `ruta-infra` | Scripts, YAMLs |
| 6 | `ruta/frontend-clients-ruta/_template/` | `landing-template` | Next.js template (GitHub template repo) |

### Repos por Cliente Full (uno por landing custom)

**Naming convention:** `landing-{slug}` donde `{slug}` es el slug del
Cliente Full en RUTA.

| Cliente Full | Carpeta local | Repositorio Git |
|---|---|---|
| Restaurante El Prado | `ruta/frontend-clients-ruta/cliente-1/` | `landing-restaurante-el-prado` |
| Panadería La Lomita | `ruta/frontend-clients-ruta/cliente-2/` | `landing-panaderia-la-lomita` |
| ... | ... | ... |

Cada landing nuevo se crea desde el template `landing-template` con un
click (botón "Use this template" de GitHub).

---

## Decisiones confirmadas

| # | Decisión | Valor |
|---|---|---|
| 1 | Carpeta local para landings | `frontend-clients` (en inglés, alineado al resto) |
| 2 | Naming de repos de landing | `landing-{slug}` (sin prefijo `ruta-`) |
| 3 | Crear `landing-template` desde Sprint 0 | Sí |
| 4 | Dominios de landings | Dominio propio del Cliente (ej. `elprado.com.co`), no subdominio de RUTA |
| 5 | Design system en landings custom | NO heredan. Cada landing trae su propio branding completo |

---

## Cómo se comparten tipos y código entre repos

`@ruta/shared` y `@ruta/db` viven en `ruta-shared` y son consumidos
vía **GitHub Packages** por:

- `ruta-backend`
- `ruta-frontend` (admin y storefront)
- Cada `landing-{slug}`

Flujo de actualización:

1. PR a `ruta-shared` con cambio.
2. CI bumpea versión y publica `@ruta/shared@X.Y.Z` a GitHub Packages.
3. Repos consumidores adoptan con `pnpm update @ruta/shared @ruta/db`.

Cada repo consumidor tiene en su `.npmrc`:

```
@ruta:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NPM_TOKEN}
```

---

## Estructura interna de cada repositorio

### `ruta-backend`

```
ruta-backend/
├── README.md
├── CLAUDE.md
├── AGENTS.md
├── .gitignore
├── .nvmrc
├── .npmrc                       # Auth a GitHub Packages
├── package.json                 # name: @ruta/backend
├── tsconfig.json
├── .env.example
├── render.yaml
└── api/
    ├── src/
    │   ├── index.ts
    │   ├── app.ts
    │   ├── config/
    │   ├── middleware/
    │   ├── routes/
    │   ├── services/
    │   ├── jobs/
    │   ├── lib/
    │   └── tests/
    ├── package.json
    └── tsconfig.json
```

Dependencias: `@ruta/shared`, `@ruta/db` desde GitHub Packages.

### `ruta-frontend`

```
ruta-frontend/
├── README.md
├── CLAUDE.md
├── AGENTS.md
├── .npmrc
├── package.json                 # name: @ruta/frontend-root
├── pnpm-workspace.yaml          # workspace interno admin + storefront + ui
├── tsconfig.base.json
├── admin/
│   ├── package.json             # @ruta/admin
│   ├── next.config.js
│   ├── tailwind.config.ts
│   └── src/
├── storefront/
│   ├── package.json             # @ruta/storefront
│   ├── next.config.js
│   ├── tailwind.config.ts
│   └── src/
└── packages-ruta/
    └── ui/                      # @ruta/ui (interno al workspace, NO publicado)
        ├── package.json
        └── src/
            ├── RutaCard.tsx
            ├── RutaButton.tsx
            ├── RutaStatusButton.tsx
            ├── RutaPill.tsx
            ├── RutaSectionHeader.tsx
            ├── RutaMetricCard.tsx
            ├── RutaTimeline.tsx
            ├── RutaOrderSummary.tsx
            ├── RutaThemeToggle.tsx
            ├── RutaSidebar.tsx
            └── RutaHeader.tsx
```

`@ruta/ui` vive dentro del workspace de `ruta-frontend` y es
consumido por admin y storefront. **No se publica a GitHub Packages**
porque las landings custom no lo necesitan (decisión 5).

Dependencias externas: `@ruta/shared` desde GitHub Packages.

### `landing-template`

Marcado como **GitHub Template Repository** para que cualquier miembro
del equipo cree un nuevo `landing-{slug}` con un click.

```
landing-template/
├── README.md                    # Instrucciones para customizar
├── CLAUDE.md                    # Manifiesto para landings custom
├── AGENTS.md
├── .gitignore
├── .nvmrc
├── .npmrc
├── package.json                 # name: @ruta/landing-template
├── next.config.js
├── tailwind.config.ts           # Tokens BASE — se sobrescriben por landing
├── tsconfig.json
├── .env.example                 # API_URL=https://api.ruta.com, CLIENT_SLUG=...
├── render.yaml.example
├── public/
│   └── PLACEHOLDER_logo.svg     # Reemplazar con logo del Cliente
└── src/
    ├── app/
    │   ├── layout.tsx
    │   ├── page.tsx             # Skeleton del catálogo
    │   ├── product/[id]/
    │   ├── cart/
    │   ├── checkout/
    │   ├── orders/
    │   └── (auth)/
    │       ├── login/
    │       └── register/
    ├── components/
    │   ├── brand/               # Placeholder para componentes custom
    │   └── shared/              # Wrappers sobre @ruta/shared (sin Ruta*)
    ├── lib/
    │   └── api_client.ts        # Cliente HTTP listo, autentica con CLIENT_SLUG
    └── styles/
        └── globals.css          # Tokens base que el branding sobrescribe
```

Dependencias: solo `@ruta/shared` (tipos y validators).
**NO depende** de `@ruta/ui` ni del design system de `ruta-frontend`.

### `landing-{slug}` (cada Cliente Full con landing custom)

Idéntico estructuralmente al template, pero customizado por el equipo
RUTA para el Cliente específico:

```
landing-restaurante-el-prado/
├── README.md                    # Específico del Cliente
├── CLAUDE.md
├── AGENTS.md
├── .npmrc
├── package.json                 # name: @ruta/landing-restaurante-el-prado
├── next.config.js
├── tailwind.config.ts           # Tokens custom (colores, fuentes del Cliente)
├── tsconfig.json
├── .env.example
├── render.yaml
├── public/
│   ├── logo.svg                 # Logo real del Cliente
│   ├── favicon.ico
│   └── hero.jpg
└── src/
    ├── app/                     # Páginas customizadas
    ├── components/
    │   ├── brand/               # Componentes con identidad del Cliente
    │   └── shared/
    ├── lib/
    │   └── api_client.ts        # API_URL=https://api.ruta.com, CLIENT_SLUG=restaurante-el-prado
    └── styles/
        └── globals.css          # Branding completo del Cliente
```

Cada landing apunta al MISMO backend (`api.ruta.com`) y autentica
BUYERs con el `client_slug` correspondiente. La diferencia entre
landings es 100% visual y de branding; la lógica de negocio sigue en
el backend único.

Despliegue: cada landing tiene su propio servicio Render con dominio
propio del Cliente (decisión 4).

### `ruta-shared`

```
ruta-shared/
├── README.md
├── CLAUDE.md
├── AGENTS.md
├── package.json                 # name: @ruta/shared-root
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── shared/                      # @ruta/shared (publicable)
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── types/
│       ├── enums/
│       ├── validators/
│       └── constants/
├── db/                          # @ruta/db (publicable)
│   ├── package.json
│   ├── tsconfig.json
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── migrations/
│   └── src/
│       ├── client.ts
│       └── tenant.ts
└── .github/workflows/
    └── publish.yml
```

### `ruta-docs`

```
ruta-docs/
├── README.md
├── CLAUDE.md                    # Master manifest (referencia)
├── AGENTS.md                    # Master manifest (referencia)
├── all_ruta.md
├── mvp_alcance.md
├── estructura_proyecto.md       # ESTE archivo
├── plan_tareas.md
├── matriz_permisos.md
├── parametros_negocio.md
├── contrato_api.md
├── wireframes_mvp.md
├── estrategia_testing.md
├── arquitectura/
│   └── estrategia_multi_tenant_ruta.md
├── seguridad/
│   └── ciclo_vida_token.txt
├── flujos/
│   ├── reglas_para_diagramar_flujos.txt
│   └── flujo_1..flujo_7.txt
├── bd/
│   ├── ruta_postgres.sql
│   ├── ruta_oracle.sql
│   └── migrations/
└── diseno/
    └── galeria_estilos_ruta.md
```

### `ruta-infra`

```
ruta-infra/
├── README.md
├── CLAUDE.md
├── AGENTS.md
├── render.yaml.example
├── supabase/
│   ├── config.toml
│   └── storage_buckets.sql
├── workspace.config.json        # Lista de repos a clonar
└── scripts/
    ├── setup_workspace.sh
    ├── clone_landing.sh         # Clona un landing existente en frontend-clients/
    ├── create_landing.sh        # Crea un nuevo landing desde landing-template
    ├── apply_migrations.sh
    ├── seed_dev_data.sh
    └── create_first_admin_ruta.sh
```

---

## Despliegue en Render

| Servicio Render | Repo origen | Sub-path | URL |
|---|---|---|---|
| `ruta-api` | `ruta-backend` | `api/` | `api.ruta.com` |
| `ruta-api-worker` | `ruta-backend` | `api/` | (background worker) |
| `ruta-admin` | `ruta-frontend` | `admin/` | `app.ruta.com` |
| `ruta-storefront` | `ruta-frontend` | `storefront/` | `tienda.ruta.com` |
| `landing-restaurante-el-prado` | `landing-restaurante-el-prado` | (raíz) | `elprado.com.co` |
| `landing-{slug}` | `landing-{slug}` | (raíz) | dominio propio del Cliente |

Supabase es externo y compartido.

---

## Setup inicial en una máquina nueva

```bash
# 1. Crear carpeta de trabajo
mkdir -p ~/projects/ruta
cd ~/projects/ruta

# 2. Configurar GitHub Packages auth (una sola vez)
echo "@ruta:registry=https://npm.pkg.github.com" >> ~/.npmrc
echo "//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}" >> ~/.npmrc

# 3. Clonar los 5 repos base
git clone git@github.com:org/ruta-backend.git backend-ruta
git clone git@github.com:org/ruta-frontend.git frontend-ruta
git clone git@github.com:org/ruta-shared.git packages-ruta
git clone git@github.com:org/ruta-docs.git docs-ruta
git clone git@github.com:org/ruta-infra.git infra-ruta

# 4. Crear carpeta organizativa de landings
mkdir -p frontend-clients-ruta
cd frontend-clients-ruta
git clone git@github.com:org/landing-template.git _template

# 5. (Cuando existan) Clonar landings de Clientes Full activos
git clone git@github.com:org/landing-restaurante-el-prado.git cliente-1
git clone git@github.com:org/landing-panaderia-la-lomita.git cliente-2
# ...

# Alternativa: usar el script
cd ~/projects/ruta
bash infra/scripts/setup_workspace.sh
```

El script `setup_workspace.sh` lee `infra/workspace.config.json` y
clona todo automáticamente.

---

## Crear un nuevo landing custom (Fase 3+)

```bash
# 1. En GitHub: usar "Use this template" sobre landing-template
#    → crear repo "landing-{slug-del-nuevo-cliente}"

# 2. Localmente, clonar dentro de frontend-clients
cd ~/projects/ruta/frontend-clients-ruta
git clone git@github.com:org/landing-{slug}.git cliente-N

# 3. Customizar:
#    - Reemplazar logo.svg y assets
#    - Editar tailwind.config.ts con tokens del Cliente
#    - Editar globals.css con branding
#    - Configurar .env: CLIENT_SLUG, dominio
#    - Customizar páginas según diseño aprobado por el Cliente

# 4. Desplegar a Render con dominio propio del Cliente

# Alternativa con script:
bash infra/scripts/create_landing.sh "{slug}"
```

---

## Convenciones de naming

- **Código:** inglés. **UI y docs:** español.
- **Services y routes:** `snake_case` para alinear con BD.
- **Variables/funciones:** `camelCase`.
- **Tipos/clases:** `PascalCase`.
- **Constantes:** `SCREAMING_SNAKE_CASE`.
- **Repos:** prefijo `ruta-` para los 5 base; `landing-{slug}` para
  landings (sin prefijo `ruta-`).

---

## CI/CD por repositorio

Cada repo tiene su propio `.github/workflows/`:

- **`ruta-backend`:** lint, typecheck, tests unit + integration,
  build, deploy a Render.
- **`ruta-frontend`:** lint, typecheck, tests + E2E Playwright,
  build separado de admin y storefront, deploy independiente.
- **`ruta-shared`:** auto-publish a GitHub Packages en merge a `main`.
- **`landing-template`:** lint, build de smoke (verifica que el
  template sigue funcionando).
- **`landing-{slug}`:** lint, build, deploy a Render del Cliente.
- **`ruta-docs`:** lint Markdown.
- **`ruta-infra`:** sin CI complejo.

---

## Implicaciones de tener 6+N repos

### Ventajas

- Aislamiento total entre landings.
- Branding y customizaciones por Cliente sin riesgo cruzado.
- Permisos granulares: si un Cliente quiere auditar su landing, se le
  da acceso solo a su repo.
- Deploy independiente por servicio.

### Overhead

- Onboarding: clonar 5 repos base + N landings activas.
- Cambios en `@ruta/shared`: bump + publish + update en N+2 repos.
- CLAUDE.md / AGENTS.md por repo (mantenidos con un script de sync).

### Mitigaciones

- `infra/scripts/setup_workspace.sh` automatiza clone inicial.
- `landing-template` reduce costo de crear landings nuevos.
- Master CLAUDE.md / AGENTS.md viven en `ruta-docs`; cada repo tiene
  versión específica que extiende el master.
