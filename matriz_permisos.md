# RUTA — Matriz de Permisos por Rol

## Roles

- **ADMIN_RUTA** — Equipo interno de RUTA. Transversal entre Clientes.
- **ADMIN_CLIENT** — Administrador principal del Cliente. Full sobre
  su Cliente.
- **OPERATOR_CLIENT** — Ejecuta operaciones diarias. Permisos
  configurables por el ADMIN_CLIENT.
- **BUYER** — Comprador. Solo Cliente Full.
- **COURIER** — Repartidor. Solo ve sus pedidos asignados.

## Convenciones de la matriz

- ✅ = Permiso completo.
- 🟡 = Permiso parcial / condicional (ver nota debajo de cada tabla).
- ❌ = Sin permiso. Endpoint retorna `403 FORBIDDEN`.
- 🔵 = Vía Vista de Control (impersonación), no acceso directo.
- 🔧 = Configurable por ADMIN_CLIENT (puede otorgar/quitar al
  OPERATOR_CLIENT).

---

## 1. Gestión de Clientes (tenants)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear Cliente | ✅ | ❌ | ❌ | ❌ | ❌ |
| Listar todos los Clientes | ✅ | ❌ | ❌ | ❌ | ❌ |
| Ver detalle de cualquier Cliente | ✅ | ❌ | ❌ | ❌ | ❌ |
| Ver detalle del Cliente propio | ✅ | ✅ | 🟡 (solo lectura) | ❌ | ❌ |
| Editar info corporativa del Cliente | ✅ | ✅ | ❌ | ❌ | ❌ |
| Activar / desactivar Cliente | ✅ | ❌ | ❌ | ❌ | ❌ |
| Eliminar Cliente (purge) | ✅ | ❌ | ❌ | ❌ | ❌ |
| Cambiar tipo de Cliente (API ↔ Full) | ✅ | ❌ | ❌ | ❌ | ❌ |

**Nota:** Solo ADMIN_RUTA puede modificar el `client_type` o el `frontend_mode`.

---

## 2. Gestión de Usuarios

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear ADMIN_CLIENT en cualquier Cliente | ✅ | ❌ | ❌ | ❌ | ❌ |
| Crear ADMIN_CLIENT adicional en mi Cliente | ❌ | ✅ | ❌ | ❌ | ❌ |
| Crear OPERATOR_CLIENT en mi Cliente | ❌ | ✅ | ❌ | ❌ | ❌ |
| Crear COURIER en mi Cliente | ❌ | ✅ | 🔧 | ❌ | ❌ |
| Crear BUYER (Cliente Full) | ❌ | ✅ | 🔧 | ❌ | ❌ |
| Auto-registrarse como BUYER (Cliente Full) | ❌ | ❌ | ❌ | ✅ (público) | ❌ |
| Ver lista de usuarios del propio Cliente | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Editar mi propio perfil | ✅ | ✅ | ✅ | ✅ | ✅ |
| Cambiar mi propia contraseña | ✅ | ✅ | ✅ | ✅ | ✅ |
| Resetear contraseña de otro usuario | ✅ | ✅ (en mi Cliente) | 🔧 | ❌ | ❌ |
| Suspender / activar otro usuario | ✅ | ✅ (en mi Cliente) | 🔧 | ❌ | ❌ |
| Eliminar usuario | ✅ | ✅ (en mi Cliente) | ❌ | ❌ | ❌ |
| Crear otro ADMIN_RUTA | ✅ | ❌ | ❌ | ❌ | ❌ |

**Nota:** Los BUYER y COURIER se registran como usuarios del Cliente
(no de la plataforma). Un BUYER puede autorregistrarse en la página
pública del Cliente Full; un COURIER siempre lo registra el Cliente.

---

## 3. Catálogo de productos (Cliente Full mayormente)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear producto | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Editar producto | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Activar / inactivar producto | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Eliminar producto | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Carga masiva por Excel | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Ver catálogo (público) | ✅ | ✅ | ✅ | ✅ | ❌ |
| Gestionar categorías | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Gestionar inventario | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |

**Nota:** Para Cliente API, el catálogo es opcional y solo
referencial. Los productos del pedido vienen como flat data en el
payload de la API.

---

## 4. Pedidos

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear pedido en UI (Cliente Full) | ❌ | ❌ | ❌ | ✅ | ❌ |
| Crear pedido vía API (Cliente API) | N/A | N/A | N/A | N/A | N/A |
| Crear pedido corporativo manual | ✅ 🔵 | ✅ (Full) | 🔧 (Full) | ❌ | ❌ |
| Ver lista de todos los pedidos del Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver mis pedidos | ✅ | ✅ | ✅ | ✅ (los míos) | ✅ (asignados) |
| Ver pedido específico (no mío, sí del Cliente) | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Aceptar / rechazar pedido (vendedor) | ✅ 🔵 | ✅ (Full) | 🔧 (Full) | ❌ | ❌ |
| Pasar pedido a PREPARING | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Pasar pedido a READY_TO_SHIP / READY_FOR_PICKUP | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Asignar Repartidor (en mapa) | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Desasignar Repartidor | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Actualizar estado de entrega (SHIPPED, IN_TRANSIT, etc.) | ❌ | ❌ | ❌ | ❌ | ✅ (asignados) |
| Cancelar pedido (BUYER, antes de despacho) | ❌ | ❌ | ❌ | 🟡 | ❌ |
| Cancelar pedido (ADMIN forzado, cualquier estado) | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Solicitar cancelación post-despacho (Cliente Full) | ❌ | ❌ | ❌ | ✅ | ❌ |
| Aprobar / rechazar solicitud de cancelación | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Cancelar pedido vía API (Cliente API) | API key | API key | API key | ❌ | ❌ |
| Reprogramar entrega | ✅ 🔵 | ✅ | ✅ | ❌ | ✅ (asignados) |
| Marcar pedido como entregado (DELIVERED) | ❌ | ❌ | ❌ | ❌ | ✅ (asignados) |
| Subir evidencia de entrega | ❌ | ❌ | ❌ | ❌ | ✅ (asignados) |
| Confirmar recepción (CONFIRMED_BY_CUSTOMER) | ❌ | ❌ | ❌ | ✅ (los míos, Cliente Full) | ❌ |
| Marcar como confirmado por sistema (auto) | (job) | (job) | (job) | (job) | (job) |

**Notas:**
- 🟡 BUYER puede cancelar solo en estados `DRAFT`, `PENDING_CONFIRM`,
  `PENDING_ONLINE_PAYMENT`, `ORDER_SUBMITTED` (antes de PREPARING).
  Después solo puede solicitar cancelación, no forzarla.
- "API key" significa que la llamada vía la plataforma del Cliente
  con su API key tiene permiso, independiente del usuario humano.
- Para Cliente API, los flujos de aceptación del vendedor (BLOQUE 6),
  validación humana (BLOQUE 5) y disputas (BLOQUE 15) no aplican —
  son rechazados con `422 LOGISTICS_ONLY_FEATURE_UNAVAILABLE`.

---

## 5. Pagos

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Configurar proveedores de pago (Wompi, etc.) | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Marcar proveedor como default | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Procesar pago online (vía pasarela) | (Comprador en pasarela) | | | | |
| Registrar cobro contra entrega (efectivo o electrónico) | ❌ | ❌ | ❌ | ❌ | ✅ (asignados) |
| Subir evidencia de cobro | ❌ | ❌ | ❌ | ❌ | ✅ (asignados) |
| Ver lista de pagos del Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver mis pagos | ✅ | ✅ | ✅ | ✅ (los míos) | ✅ (cobrados por mí) |
| Marcar pago como reconciliado con el Cliente | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Reportes de conciliación | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |

---

## 6. Reembolsos (Cliente Full)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Iniciar reembolso manual | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Marcar reembolso como ejecutado | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Subir comprobante de reembolso | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Solicitar reembolso al proveedor (Wompi, datáfono, etc.) | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Ver lista de reembolsos | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver mis reembolsos | ✅ | ✅ | ✅ | ✅ (los míos) | ❌ |
| Reembolso parcial | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |

---

## 7. Devoluciones (Cliente Full)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Solicitar devolución (post-cierre) | ❌ | ❌ | ❌ | ✅ (los míos) | ❌ |
| Aprobar / rechazar devolución | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Programar recogida de devolución (CLIENT_PICKS_UP) | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Asignar Repartidor para recogida | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Ejecutar recogida (PICKUP_OUT_FOR_COLLECTION → PICKUP_COLLECTED) | ❌ | ❌ | ❌ | ❌ | ✅ (asignados) |
| Marcar producto como recibido en bodega | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Configurar mecanismo de devolución (CLIENT_PICKS_UP / BUYER_SHIPS) | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver lista de devoluciones del Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver mis devoluciones | ✅ | ✅ | ✅ | ✅ (las mías) | ❌ |

---

## 8. Disputas (Cliente Full)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Abrir disputa (post-entrega) | ❌ | ❌ | ❌ | ✅ (los míos) | ❌ |
| Resolver disputa (sin acción) | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Resolver disputa con devolución | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Resolver disputa con reembolso (sin devolución) | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver disputas del Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver mis disputas | ✅ | ✅ | ✅ | ✅ (las mías) | ❌ |

---

## 9. Recurrencia (Cliente Full)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Marcar pedido propio como recurrente | ❌ | ❌ | ❌ | ✅ | ❌ |
| Crear plantilla recurrente para comprador corporativo | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Pausar / reanudar / cancelar mi recurrencia | ❌ | ❌ | ❌ | ✅ (mías) | ❌ |
| Editar mi plantilla de recurrencia | ❌ | ❌ | ❌ | ✅ (mías) | ❌ |
| Pausar / cancelar cualquier recurrencia del Cliente | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Ver lista de recurrencias activas del Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |

---

## 10. Repartidores

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear COURIER en mi Cliente | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Editar COURIER de mi Cliente | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Activar / inactivar COURIER | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Ver mi lista de repartidores | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver mi perfil de COURIER | ✅ | ✅ | ✅ | ❌ | ✅ |
| Ver rendimiento / métricas por COURIER | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |

---

## 11. Compradores (BUYERs)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Ver lista de BUYERs del Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Ver detalle de BUYER específico | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Editar datos de un BUYER | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Suspender / activar BUYER | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |

---

## 12. Pickup Points

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear pickup point | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Editar pickup point | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Activar / inactivar pickup point | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Ver lista (público para el flujo de checkout) | ✅ | ✅ | ✅ | ✅ | ❌ |

---

## 13. API Keys (principalmente Cliente API)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Generar API key para mi Cliente | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver lista de API keys de mi Cliente | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver secreto de API key (solo al crearla) | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Revocar API key | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Configurar scopes de API key | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |

**Nota:** El secreto solo se muestra una vez (al crearla). Después
solo se ve el `key_id` y un hash. OPERATOR_CLIENT nunca debe poder
generar API keys (es una operación de seguridad sensible).

---

## 14. Webhooks salientes (notificaciones a Cliente)

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Crear suscripción de webhook | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Editar URL / eventos del webhook | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver historial de entregas de webhooks | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Reintentar manualmente un webhook fallido | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Regenerar signing secret | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |

---

## 15. Dashboards y reportes

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Dashboard global de plataforma (todos los Clientes) | ✅ | ❌ | ❌ | ❌ | ❌ |
| Dashboard del mi Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Reporte de pedidos por rango de fecha | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Reporte de pagos por rango de fecha | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Reporte de reembolsos | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Reporte de devoluciones | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Reporte de repartidores | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |
| Reporte de productos (top, rotación) | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Reporte de compradores | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |
| Exportar reportes a Excel / CSV | ✅ 🔵 | ✅ | 🔧 | ❌ | ❌ |

---

## 16. Vista de Control y auditoría

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Entrar a Vista de Control sobre cualquier Cliente | ✅ (con master password + permiso) | ❌ | ❌ | ❌ | ❌ |
| Ver log de auditoría de mi Cliente | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver log de auditoría global | ✅ | ❌ | ❌ | ❌ | ❌ |
| Filtrar auditoría por usuario / acción / fecha | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver auditoría de mis propias acciones | ✅ | ✅ | ✅ | ✅ | ✅ |

**Nota:** El permiso `can_use_control_view` es un flag adicional en
la tabla `control_view_master_passwords`. No todos los ADMIN_RUTA lo
tienen automáticamente — debe otorgarse explícitamente.

---

## 17. Parámetros de negocio

| Acción | ADMIN_RUTA | ADMIN_CLIENT | OPERATOR_CLIENT | BUYER | COURIER |
|---|---|---|---|---|---|
| Editar parámetros globales de plataforma | ✅ | ❌ | ❌ | ❌ | ❌ |
| Editar parámetros específicos de mi Cliente | ✅ 🔵 | ✅ | ❌ | ❌ | ❌ |
| Ver parámetros vigentes para mi Cliente | ✅ 🔵 | ✅ | ✅ | ❌ | ❌ |

**Nota:** Los parámetros vienen de la tabla `client_parameters`
(ver `parametros_negocio.md`). Cada parámetro tiene un valor global
default (de la plataforma) que puede sobrescribirse por Cliente.

---

## Permisos configurables (🔧) — modelo

OPERATOR_CLIENT es un rol con permisos **granulares configurables**
por el ADMIN_CLIENT. La tabla `operator_permissions` registra qué
acciones puede ejecutar cada OPERATOR_CLIENT.

```sql
CREATE TABLE ruta.operator_permissions (
  operator_user_id BIGINT NOT NULL,
  client_id BIGINT NOT NULL,
  permission_key TEXT NOT NULL,
  granted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  granted_by_user_id BIGINT NOT NULL,
  PRIMARY KEY (operator_user_id, client_id, permission_key),
  CONSTRAINT fk_op_user FOREIGN KEY (operator_user_id, client_id)
    REFERENCES ruta.users(id, client_id)
);
```

### Permission keys propuestos

```
PRODUCTS_CREATE
PRODUCTS_EDIT
PRODUCTS_DELETE
PRODUCTS_BULK_IMPORT
CATEGORIES_MANAGE
INVENTORY_MANAGE

ORDERS_CREATE_CORPORATE
ORDERS_ACCEPT_SELLER
ORDERS_CANCEL_FORCED
ORDERS_APPROVE_CANCEL_REQUEST

COURIERS_MANAGE
COURIERS_ASSIGN
COURIERS_METRICS

BUYERS_VIEW
BUYERS_EDIT
BUYERS_SUSPEND

PICKUP_POINTS_MANAGE

REFUNDS_INITIATE
REFUNDS_MARK_EXECUTED
REFUNDS_PROVIDER_REQUEST

RETURNS_REVIEW
RETURNS_PICKUP_SCHEDULE

DISPUTES_RESOLVE_NO_ACTION
DISPUTES_RESOLVE_RETURN

RECURRENCE_MANAGE_CLIENT_WIDE

WEBHOOKS_VIEW_HISTORY
WEBHOOKS_RETRY_FAILED

PAYMENTS_RECONCILE
REPORTS_PRODUCTS
REPORTS_BUYERS
EXPORTS_GENERATE
```

ADMIN_CLIENT tiene todos por default. OPERATOR_CLIENT no tiene
ninguno por default — se otorgan uno a uno.

---

## Implementación a nivel de código

### Middleware

```ts
// authorize.ts
export function authorize(...allowedRoles: UserType[]) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.userType)) {
      return res.status(403).json({ code: 'FORBIDDEN' });
    }
    next();
  };
}

export function requireOperatorPermission(key: string) {
  return async (req, res, next) => {
    if (req.user.userType === 'ADMIN_CLIENT' ||
        req.user.userType === 'ADMIN_RUTA') {
      return next();
    }
    if (req.user.userType !== 'OPERATOR_CLIENT') {
      return res.status(403).json({ code: 'FORBIDDEN' });
    }
    const has = await checkOperatorPermission(
      req.user.userId, req.user.clientId, key
    );
    if (!has) {
      return res.status(403).json({
        code: 'MISSING_OPERATOR_PERMISSION',
        permission: key
      });
    }
    next();
  };
}
```

### Aplicación

```ts
router.post(
  '/products',
  authenticate,
  tenantContext,
  authorize('ADMIN_CLIENT', 'OPERATOR_CLIENT', 'ADMIN_RUTA'),
  requireOperatorPermission('PRODUCTS_CREATE'),
  productsController.create
);
```

---

## Lo que pediste ajustar

> ADMIN_CLIENT tiene full permisos sobre la operación y data del Cliente.

Reflejado en toda la matriz: ADMIN_CLIENT tiene ✅ en todas las
acciones de su Cliente, excepto:

- No puede crear / eliminar Clientes (solo ADMIN_RUTA).
- No puede cambiar el tipo del Cliente (API ↔ Full).
- No puede otorgar Vista de Control a otros usuarios.
- No puede ver datos de otros Clientes.

¿Esta es la lectura correcta de "full sobre operación y data del
Cliente"? Si quieres que el ADMIN_CLIENT pueda hacer algo de lo
restringido, dime y lo ajusto.
