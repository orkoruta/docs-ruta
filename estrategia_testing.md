# RUTA — Estrategia de Testing

## Filosofía

RUTA es un sistema con dinero, identidad y estados de pedido que no
admiten errores. Por eso el testing tiene tres reglas no negociables:

1. **Toda transición de estado debe estar testeada.** El state machine
   del pedido es el corazón del negocio.
2. **Toda regla de aislamiento multi-tenant debe estar testeada.** Si
   un Cliente puede ver datos de otro, es una falla crítica.
3. **Todo flujo financiero debe tener pruebas que validen el principio
   de RUTA no custodia dinero.** Ninguna API debe permitir transferir
   o crear créditos en nombre de RUTA.

---

## Pirámide de testing

```
                       /\
                      /  \    E2E      (pocos, críticos)
                     /----\
                    /      \   Integration
                   /        \  (API + DB real)
                  /----------\
                 /            \  Unit
                /              \  (servicios, state machine, validators)
               /________________\
```

### Distribución objetivo

- **Unit:** 70% del esfuerzo, ejecución <10s.
- **Integration:** 25% del esfuerzo, ejecución <2min.
- **E2E:** 5% del esfuerzo, ejecución <10min.

---

## Stack recomendado

| Capa | Herramienta | Razón |
|---|---|---|
| Test runner backend | **Vitest** | Más rápido que Jest, ESM nativo, mismas APIs |
| Test runner frontend | **Vitest** + **React Testing Library** | Coherencia de tooling |
| HTTP integration tests | **Supertest** | Estándar Express |
| BD de prueba | **PostgreSQL en Docker** vía `testcontainers` o `pg-isolation` | DB real, no mocks |
| E2E web | **Playwright** | Mejor que Cypress para nuestro caso (multi-browser, móvil) |
| Mocks de pasarela | **MSW (Mock Service Worker)** | Mocks de Wompi reusables entre backend y frontend |
| Coverage | **Vitest coverage v8** | Built-in |

---

## Tipos de test

### 1. Tests unitarios

**Qué se testea:**

- Servicios puros (sin DB): cálculos, validaciones, transformaciones.
- State machine: cada transición permitida y rechazada.
- Validators (Zod schemas).
- Helpers: parsers, formatters, generadores de IDs.

**Ejemplo: state machine de orden**

```ts
// apps/api/src/services/orders/state_machine.test.ts
import { canTransition } from './state_machine';

describe('Order state machine', () => {
  describe('order_status', () => {
    it('permite DRAFT → PENDING_CONFIRM', () => {
      expect(canTransition('order_status', 'DRAFT', 'PENDING_CONFIRM'))
        .toBe(true);
    });

    it('rechaza DRAFT → DELIVERED (no permitido)', () => {
      expect(canTransition('order_status', 'DRAFT', 'DELIVERED'))
        .toBe(false);
    });

    it('permite DELIVERED → DELIVERY_DISPUTED solo para Cliente Full', () => {
      expect(canTransition('order_status', 'DELIVERED', 'DELIVERY_DISPUTED', {
        clientType: 'FULL'
      })).toBe(true);
      expect(canTransition('order_status', 'DELIVERED', 'DELIVERY_DISPUTED', {
        clientType: 'API'
      })).toBe(false);
    });
  });
});
```

**Cobertura objetivo unit:** 90% de líneas en `services/` y `lib/`.

---

### 2. Tests de integración

**Qué se testea:**

- Endpoints HTTP completos: request → handler → service → DB → response.
- RLS y multi-tenant: que un Cliente NO pueda leer pedidos de otro.
- Transacciones: que rollbacks dejen el estado consistente.
- Triggers de BD: que `order_state_history` se llene correctamente.
- Auto-creación de particiones al crear un Cliente nuevo.
- Idempotencia: misma `X-Idempotency-Key` retorna la misma respuesta.

**Setup:**

- Cada test corre contra un PostgreSQL real (testcontainers o BD
  dedicada en CI).
- Antes de cada test: `BEGIN; ... ROLLBACK;` (rollback automático).
- Para tests que requieren commit (triggers, etc.): cleanup explícito.

**Ejemplo: aislamiento entre Clientes**

```ts
// apps/api/src/tests/integration/tenant_isolation.test.ts
describe('Tenant isolation', () => {
  let clientA: number;
  let clientB: number;
  let adminA_token: string;

  beforeAll(async () => {
    clientA = await createClient({ name: 'Cliente A', type: 'FULL' });
    clientB = await createClient({ name: 'Cliente B', type: 'FULL' });
    adminA_token = await loginAsAdmin(clientA);
    await createOrder(clientB, { ... });
  });

  it('Admin del Cliente A no ve pedidos del Cliente B', async () => {
    const res = await request(app)
      .get('/api/orders')
      .set('Authorization', `Bearer ${adminA_token}`)
      .expect(200);

    expect(res.body.orders.every(o => o.client_id === clientA)).toBe(true);
  });

  it('Admin del Cliente A recibe 404 al pedir pedido específico del Cliente B', async () => {
    const orderB = await getFirstOrder(clientB);
    await request(app)
      .get(`/api/orders/${orderB.id}`)
      .set('Authorization', `Bearer ${adminA_token}`)
      .expect(404);
  });
});
```

**Ejemplo: idempotencia**

```ts
it('Mismas peticiones con misma idempotency-key retornan misma respuesta', async () => {
  const key = randomUUID();
  const payload = { items: [...], delivery_type: 'SHIP' };

  const res1 = await request(app)
    .post('/api/orders')
    .set('X-Idempotency-Key', key)
    .send(payload)
    .expect(201);

  const res2 = await request(app)
    .post('/api/orders')
    .set('X-Idempotency-Key', key)
    .send(payload)
    .expect(201);

  expect(res1.body.id).toBe(res2.body.id);
});
```

**Cobertura objetivo integration:** 100% de los endpoints públicos
tienen al menos un test de happy-path y uno de error.

---

### 3. Tests E2E

**Qué se testea:** los flujos críticos de extremo a extremo, en
navegador real, con backend real y DB real.

**Flujos E2E mínimos (MVP Fase 1 — Cliente API):**

1. **Onboarding de Cliente API.**
   - ADMIN_RUTA crea Cliente API.
   - ADMIN_CLIENT activa cuenta.
   - ADMIN_CLIENT genera API key.
   - Plataforma del Cliente (mock) envía primer pedido vía API.
   - Pedido aparece en panel del ADMIN_CLIENT.

2. **Flujo SHIP completo con flota propia.**
   - Pedido entra vía API en READY_TO_SHIP.
   - ADMIN_CLIENT asigna repartidor en mapa.
   - COURIER ve el pedido en su app móvil.
   - COURIER marca SHIPPED → IN_TRANSIT → OUT_FOR_DELIVERY → ARRIVED_AT_CUSTOMER → DELIVERED.
   - COURIER registra cobro en efectivo con evidencia.
   - Webhook saliente notifica a la plataforma del Cliente.
   - Pedido cierra como COMPLETED_SUCCESSFULLY.

3. **Flujo PICKUP completo.**
   - Pedido entra vía API en READY_FOR_PICKUP.
   - Pedido aparece en el panel del Cliente.
   - Comprador llega al punto físico (simulado).
   - OPERATOR_CLIENT valida identidad.
   - OPERATOR_CLIENT registra cobro electrónico.
   - Pedido cierra.

**Flujos E2E adicionales (MVP Fase 2 — Cliente Full):**

4. **Compra completa con pago online.**
   - BUYER se registra.
   - BUYER navega catálogo, agrega al carrito.
   - BUYER confirma pedido y elige pagar online.
   - BUYER es redirigido a Wompi (sandbox).
   - Pago se confirma vía webhook entrante.
   - Pedido pasa a SELLER_CONFIRMED, PREPARING, etc.
   - Flujo continúa hasta DELIVERED.

5. **Cancelación con reembolso.**
   - BUYER hace pedido con pago online y queda PAID.
   - ADMIN_CLIENT cancela el pedido.
   - Se dispara REFUND_PENDING.
   - ADMIN_CLIENT ejecuta reembolso vía Wompi (sandbox).
   - refund_status → REFUNDED.

**Estructura Playwright:**

```
apps/web/tests/e2e/
├── fixtures/                    # Datos de prueba
│   ├── clients.fixture.ts
│   ├── users.fixture.ts
│   └── orders.fixture.ts
├── flows/                       # Page Objects
│   ├── login.page.ts
│   ├── admin_dashboard.page.ts
│   ├── courier_app.page.ts
│   └── checkout.page.ts
├── api_client_onboarding.spec.ts
├── ship_full_flow.spec.ts
├── pickup_full_flow.spec.ts
├── buyer_checkout_online_payment.spec.ts
└── cancellation_with_refund.spec.ts
```

---

## Tests específicos de RUTA

### State machine completo

Para cada estado del catálogo (`state_catalog`), generar
automáticamente:

- Test de "estado existe en el catálogo de la BD".
- Tests de transiciones permitidas (lista manual del flow doc).
- Tests negativos para transiciones aleatorias (deben fallar).

Esto previene drift entre la documentación de flujos y el código.

### Aislamiento multi-tenant (CRÍTICO)

Suite específica que prueba que **cada endpoint** rechaza acceso
cross-tenant. Se ejecuta como gate en CI: si falla, no se mergea.

```ts
// apps/api/src/tests/integration/cross_tenant_security.test.ts
const ALL_ENDPOINTS = [
  { method: 'GET', path: '/api/orders' },
  { method: 'GET', path: '/api/orders/:id' },
  { method: 'POST', path: '/api/orders' },
  // ... todos los endpoints
];

describe.each(ALL_ENDPOINTS)(
  'Cross-tenant security: $method $path',
  ({ method, path }) => {
    it('rechaza acceso del Cliente A a recursos del Cliente B', async () => {
      // ... lógica genérica
    });
  }
);
```

### Validación del principio financiero

Tests que validan que **ningún endpoint mueve dinero**:

```ts
describe('RUTA non-financial role', () => {
  it('no expone endpoint para transferir dinero', () => {
    expect(routes).not.toContain('/api/transfers');
    expect(routes).not.toContain('/api/payouts');
  });

  it('registrar cobro no modifica saldos en RUTA', async () => {
    // ... crea pedido, courier cobra, verifica que solo se actualizó
    // payments.collected_at y orders.payment_status. No hay tabla de balances.
  });

  it('reembolso solo registra estado, no transfiere', async () => {
    // ... crea refund, verifica que no llama a ninguna API externa de
    // transferencia, solo registra el cambio de status.
  });
});
```

### Webhooks (outbound)

- Test de firma HMAC-SHA256 correcta.
- Test de retries con backoff exponencial.
- Test de 6 fallos consecutivos → `failed_permanently`.
- Test de orden de eventos (FIFO por pedido).

### Webhooks (inbound de Wompi)

- Test de verificación de firma de Wompi.
- Test de deduplicación por `provider_event_id`.
- Test de eventos fuera de orden (idempotencia).

### Particionamiento

- Test de que crear un Cliente nuevo crea TODAS las particiones
  esperadas (19+ tablas).
- Test de que eliminar un Cliente elimina sus particiones (excepto
  audit_events).
- Test de que queries cross-cliente funcionan (para ADMIN_RUTA) sin
  perder performance.

### Vista de Control

- Test de que entrar a Vista de Control requiere master password.
- Test de que las acciones quedan auditadas con
  `acting_via_control_view = TRUE`.
- Test de que el ADMIN_CLIENT NO recibe notificación.
- Test de que las acciones dentro de Vista de Control respetan los
  permisos del rol impersonado.

---

## Test data y fixtures

### Estrategia

- **Factory functions** para crear entidades de prueba (no fixtures
  estáticos en JSON).
- Usar **`@faker-js/faker`** para datos realistas.
- Cada test crea sus propios datos (no comparten estado entre tests).
- BD se resetea entre tests con `TRUNCATE ... CASCADE` o `ROLLBACK`.

```ts
// apps/api/src/tests/fixtures/order.factory.ts
export async function createOrder(overrides: Partial<Order> = {}) {
  const client = overrides.clientId ?? await createClient();
  const buyer = overrides.buyerId ?? await createBuyer(client.id);
  return db.orders.create({
    data: {
      client_id: client.id,
      buyer_id: buyer.id,
      order_origin: 'BUYER_UI',
      order_status: 'DRAFT',
      payment_status: 'PAYMENT_NOT_STARTED',
      delivery_type: 'SHIP',
      payment_method: 'ONLINE_AT_ORDER',
      ...overrides
    }
  });
}
```

---

## CI / CD: gates de testing

En cada PR:

1. **Lint + typecheck** (rápido, <30s).
2. **Unit tests** (medio, <2min).
3. **Integration tests** (lento, <5min, contra BD efímera).
4. **Cross-tenant security suite** (crítico, no negociable).
5. **Coverage threshold:** unit >85%, integration cobertura de
   endpoints 100%.

En cada merge a `main`:

6. **E2E tests** (Playwright, contra entorno staging).
7. **Smoke tests post-deploy** (handful de tests críticos contra
   producción tras despliegue).

---

## Datos de prueba en entornos

### Desarrollo local

- BD vacía + seed script (`pnpm db:seed`).
- Crea: 1 ADMIN_RUTA, 1 Cliente API piloto, 1 Cliente Full piloto,
  3 BUYERs por Cliente, 5 COURIERs por Cliente, 20 productos en
  Cliente Full, 10 pedidos en distintos estados.

### Staging

- Datos sintéticos generados con faker, ~50 Clientes, 500
  COURIERs, 5000 pedidos. Refresh semanal.

### Producción

- Sin datos de prueba. Todo es real.

---

## Mocks de Wompi

Wompi sandbox es lento e impredecible para tests automatizados. Usamos
**MSW** para interceptar llamadas a Wompi:

```ts
// apps/api/src/tests/fixtures/wompi.mocks.ts
export const wompiHandlers = [
  rest.post('https://sandbox.wompi.co/v1/transactions', (req, res, ctx) => {
    return res(ctx.json({
      data: { id: 'mock_txn_123', status: 'APPROVED', ... }
    }));
  }),
  rest.post('https://sandbox.wompi.co/v1/refunds', (req, res, ctx) => {
    return res(ctx.json({ data: { id: 'mock_refund_456', ... } }));
  })
];
```

Tests de integración usan estos mocks. Tests E2E contra Wompi sandbox
real son menos frecuentes (uno por flow crítico, ejecutado solo en
staging).

---

## Reporte de coverage

Se publica en cada PR como comentario:

```
Coverage report
───────────────────────────────────────
Statements   : 87.3% ( 2341/2683 )
Branches     : 82.1% ( 412/502 )
Functions    : 89.7% ( 287/320 )
Lines        : 87.1% ( 2298/2638 )

Critical paths (must be 100%):
  ✅ State machine               100%
  ✅ Tenant isolation            100%
  ✅ Auth middleware             100%
  ⚠️  Webhook signature           94%  (-)
  ✅ Idempotency                  100%
```
