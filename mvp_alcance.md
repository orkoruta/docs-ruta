# RUTA — Definición de MVP y Roadmap

## Filosofía del MVP

Construir lo mínimo necesario para que un Cliente pueda **operar de
verdad** con RUTA (no demos), demostrando la propuesta de valor
diferencial: **multi-tenant + logística separada del cobro de dinero**.

El MVP se construye en 3 fases. Cada fase es desplegable a producción
y entrega valor independiente.

---

## Fase 1 — Cliente Full con frontend + checkout

**Tiempo estimado:** 8 a 10 semanas.

**Hipótesis a validar:** un negocio que no tiene plataforma de venta
propia puede contratar RUTA y empezar a vender en línea con logística
integrada en una semana.

### Qué entra en Fase 1

**Frontend de Compradores (Cliente Full, storefront):**

- Página del Cliente en `ruta.com/c/{slug}/`: catálogo público.
- Detalle de producto.
- Registro y login de BUYER (auto-registro público).
- Carrito.
- Checkout: selección de entrega (SHIP / PICKUP) y método de pago.
- Integración con Wompi para pagos online (sandbox y producción).
- Vista del Comprador: mis pedidos, detalle con timeline, mi
  perfil, direcciones guardadas.
- Diseño responsive (móvil + desktop).
- Modo claro y oscuro (galería de estilos).

**Frontend Admin (panel para Cliente y RUTA):**

- Login para ADMIN_CLIENT, OPERATOR_CLIENT, COURIER, ADMIN_RUTA.
- Dashboard del ADMIN_CLIENT con métricas básicas.
- Gestión de pedidos: lista, detalle, cambio de estado.
- **Mapa de asignación de Repartidores** con OpenStreetMap. Pedidos
  listos para despacho aparecen geolocalizados.
- Gestión de catálogo: crear, editar, importar masivo desde Excel.
- Gestión de Compradores.
- Gestión de Repartidores.
- Gestión de Puntos de Recogida (pickup points).
- Configuración del Cliente (info corporativa, logo, parámetros
  overridables).
- Configuración de proveedor de pago (Wompi).
- **Vista del Repartidor** (móvil-first, dentro del mismo admin):
  mis pedidos asignados, actualizar estado, registrar cobro contra
  entrega, subir evidencia.
- **Panel ADMIN_RUTA:** crear / activar / desactivar Clientes,
  Vista de Control con master password, log de auditoría.

**Backend:**

- Auth completa: JWT + refresh tokens en cookies HttpOnly,
  argon2 para passwords.
- Multi-tenant con RLS y particionamiento auto-creado al crear
  Cliente.
- **Flujo 1 completo:** creación, confirmación, pago online o contra
  entrega, validación, aceptación del vendedor, preparación.
- **Flujo 2 completo:** SHIP con flota propia (mapa) y courier externo,
  cobro contra entrega, devolución al origen, cancelaciones
  post-despacho.
- **Flujo 3 completo:** PICKUP con validación de identidad, cobro,
  expiración.
- Webhooks entrantes de Wompi para confirmar pagos.
- Webhooks salientes (suscripciones configurables) para eventos de
  pedidos.
- File storage en Supabase Storage para logos, imágenes de productos,
  evidencias.
- Jobs con pg-boss: expiración de pedidos, auto-confirmación
  DELIVERED → CONFIRMED_BY_SYSTEM, expiración de pickup, reintentos de
  webhooks, limpieza de sesiones e idempotency keys.
- Auditoría completa.
- Parámetros de negocio operativos (tabla `client_parameters`).
- API REST documentada (OpenAPI).

### Qué NO entra en Fase 1

**Funciones diferidas:**

- **Cliente API** (logística-as-a-service). Se hace en Fase 2.
- **Flujo 4 (reembolsos).** Se registra `REFUND_PENDING` cuando aplica
  pero no hay UI ni jobs de reembolso. ADMIN_CLIENT ejecuta
  manualmente fuera del sistema; en Fase 3 se hace UI completa.
- **Flujo 5 (recurrencia).**
- **Flujo 6 (pedidos corporativos).**
- **Flujo 7 (devoluciones post-cierre).**
- **Bloque 15 (disputas).**
- **Modalidad CUSTOM_LANDING_BY_RUTA.** Solo NATIVE_RUTA en Fase 1.
- Reportes avanzados (análisis de cohortes, predictivos, etc.).
- Notificaciones por SMS / push (solo email en Fase 1).

### Criterio de salida de Fase 1

- 1 Cliente Full piloto operando en producción.
- 50+ pedidos reales procesados end-to-end con pago online y contra
  entrega.
- 100+ BUYERs registrados.
- Wompi sandbox + producción funcionando, con conciliación manual.
- Cero pérdida de datos, cero estados huérfanos.
- Cero violaciones de aislamiento multi-tenant en pruebas.
- Tiempo de onboarding de un nuevo Cliente: <1 día.

---

## Fase 2 — Cliente API (logística-as-a-service)

**Tiempo estimado:** 4 a 6 semanas adicionales.

**Hipótesis a validar:** una empresa con su propia plataforma de
venta puede integrar RUTA en un día y empezar a usarlo como su backend
logístico.

### Qué entra en Fase 2

**Backend:**

- Endpoints expuestos vía API key para la plataforma del Cliente:
  - Crear pedido (entrando en PREPARING, AWAITING_COURIER_ASSIGNMENT,
    READY_TO_SHIP o READY_FOR_PICKUP).
  - Cancelar pedido.
  - Consultar estado de pedido.
  - Listar pedidos.
- Gestión de API keys (creación, revocación, scopes).
- Validación de que las funciones bloqueadas (refunds, returns,
  recurrencia, corporativos, disputas) son rechazadas con
  `422 LOGISTICS_ONLY_FEATURE_UNAVAILABLE` para pedidos de Cliente API.
- Webhooks salientes especialmente útiles para Cliente API.

**Frontend Admin:**

- UI para gestión de API keys.
- Vista diferenciada de pedidos de Cliente API (no muestran flujo de
  validación, no permiten reembolso, etc.).
- Configuración del tipo de Cliente al crearlo.

**Reutiliza de Fase 1:**

- Toda la lógica logística (flujos 2 y 3).
- Mapa de asignación de Repartidores.
- Vista del Repartidor.
- Auditoría, multi-tenant, particiones, etc.

### Criterio de salida de Fase 2

- 1 Cliente API piloto integrado y operando.
- 10+ pedidos reales procesados vía API.
- Tiempo de integración para nuevo Cliente API: <1 día.
- Webhook reliability >99%.

---

## Fase 3 — Funciones avanzadas

**Tiempo estimado:** 8 a 12 semanas, en sub-sprints independientes.

### Bloques (cada uno desplegable por separado)

**3.1 Reembolsos completos (Flujo 4).** UI completa para iniciar
reembolso, marcar ejecutado, subir comprobante. Integración con
Wompi para reembolso vía proveedor. Bifurcación por modalidad
(STORE_CREDIT / BANK_REFUND) y por método de pago original.

**3.2 Devoluciones post-cierre (Flujo 7).** Con los dos mecanismos
(BUYER_SHIPS_VIA_COURIER, CLIENT_PICKS_UP). Dispara reembolso en
paralelo.

**3.3 Disputas (Bloque 15).** UI para reclamos del Comprador y
resolución por el Admin del Cliente.

**3.4 Recurrencia (Flujo 5).** Plantillas, engine de generación
automática con `pg-boss`, repetir último pedido.

**3.5 Pedidos corporativos (Flujo 6).** Creación manual por el Admin
del Cliente para B2B.

**3.6 Landing custom.** Modalidad CUSTOM_LANDING_BY_RUTA: el equipo
RUTA desarrolla landings personalizadas por Cliente con su marca y
dominio, conectadas al mismo backend.

### Criterio de salida de Fase 3

- 5+ Clientes operando (mix de Full y API).
- Todas las funcionalidades del documento `all_ruta.md` activas.
- Producto listo para escalar comercialmente.

---

## Lo que cruza todas las fases

Estos elementos se construyen desde Fase 1 y se enriquecen en cada
fase:

- **Auditoría.** Activa desde día 1, todas las acciones se registran.
- **Multi-tenant + RLS.** Activos desde día 1 (ya en el SQL).
- **Vista de Control para ADMIN_RUTA.** Activa desde Fase 1.
- **Auto-creación de particiones por Cliente.** Activa desde día 1.
- **Idempotencia en mutaciones.** Activa desde Fase 1.
- **Jobs de mantenimiento.** Activos desde Fase 1.
- **Observabilidad** (logs estructurados, métricas). Activa desde
  Fase 1.

---

## Resumen visual

| Capability | Fase 1 | Fase 2 | Fase 3 |
|---|---|---|---|
| Cliente Full + storefront | ✅ | — | — |
| Pago online (Wompi) | ✅ | — | — |
| Pago contra entrega | ✅ | ✅ | — |
| Flujo 1 (creación + pago) | ✅ | (N/A para API) | — |
| Flujo 2 (SHIP) | ✅ | ✅ | — |
| Flujo 3 (PICKUP) | ✅ | ✅ | — |
| Mapa asignación repartidor | ✅ | ✅ | — |
| Vista del Repartidor | ✅ | ✅ | — |
| Auditoría / Vista de Control | ✅ | — | — |
| Cliente API + API keys | — | ✅ | — |
| Webhooks salientes | ✅ (básico) | ✅ (completo) | — |
| Reembolsos (Flujo 4) | — | — | ✅ |
| Devoluciones post-cierre (Flujo 7) | — | — | ✅ |
| Disputas (Bloque 15) | — | — | ✅ |
| Recurrencia (Flujo 5) | — | — | ✅ |
| Pedidos corporativos (Flujo 6) | — | — | ✅ |
| Landing custom (CUSTOM_LANDING_BY_RUTA) | — | — | ✅ |

---

## Sprints sugeridos dentro de Fase 1

| Sprint | Duración | Entregable |
|---|---|---|
| **0** | 1 semana | Setup de repos, BD inicial, CI/CD, infraestructura Render + Supabase, primer ADMIN_RUTA, primer Cliente piloto creado |
| **1** | 2 semanas | Auth completa, gestión de Clientes (ADMIN_RUTA), gestión de catálogo, registro de BUYER, Flujo 1 hasta `PENDING_CONFIRM` |
| **2** | 2 semanas | Checkout: carrito, integración Wompi sandbox, Flujo 1 hasta `PREPARING`, webhooks entrantes de Wompi |
| **3** | 2 semanas | Flujo 2 (SHIP): mapa de asignación, vista del Repartidor, registro de cobro contra entrega, cierre exitoso |
| **4** | 1 semana | Flujo 3 (PICKUP): pickup points, validación identidad, cierre |
| **5** | 1 semana | Vista de Control, auditoría UI, dashboards básicos, polish de UX |
| **6** | 1 semana | Hardening: tests E2E, observabilidad, deploy a producción, onboarding del Cliente piloto |

Total: 10 semanas para Fase 1 completa.

---

## Decisiones confirmadas y pendientes

- ¿Tamaño del equipo (humano + IA)? Esto afecta el cronograma.
- Cliente piloto para Fase 1: confirmado.
- Dominio principal: `ruta.com`.
- Credenciales Wompi sandbox / producción: pendientes; se integran cuando estén disponibles.
- ¿Hay landing pública de marketing antes del MVP, o entramos directo
  a la app?
