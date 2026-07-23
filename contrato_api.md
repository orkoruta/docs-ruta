# RUTA — Contrato de API REST (MVP Fase 1)

## Convenciones generales

- **Base URL:** `https://api.ruta.com` (sin prefijo de versión; el backend no usa `/v1`).
- **Formato:** JSON UTF-8. `Content-Type: application/json`.
- **Auth:** JWT en cookie HttpOnly `access_token` (modo navegador)
  o header `Authorization: Bearer <token>` (modo API).
- **Idempotencia:** header `X-Idempotency-Key: <uuid>` obligatorio en
  todos los POST / PUT / PATCH / DELETE.
- **Trazabilidad:** header `X-Request-Id: <uuid>` opcional; si no se
  envía, el servidor genera uno y lo retorna.
- **Multi-tenant:** el `client_id` se infiere del usuario autenticado
  (no se acepta en el body). ADMIN_RUTA puede operar sobre cualquier
  Cliente vía Vista de Control.
- **Códigos HTTP estándar:** 200 OK, 201 Created, 204 No Content,
  400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found,
  409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests,
  500 Internal Server Error.

## Códigos de error de aplicación

Todo error retorna `{ code, message, details? }`:

```json
{
  "code": "INVALID_STATE_TRANSITION",
  "message": "No se puede transicionar de DRAFT a DELIVERED",
  "details": { "from": "DRAFT", "to": "DELIVERED" }
}
```

Códigos relevantes:
`AUTHENTICATION_REQUIRED`, `FORBIDDEN`, `MISSING_OPERATOR_PERMISSION`,
`VALIDATION_ERROR`, `RESOURCE_NOT_FOUND`, `IDEMPOTENCY_CONFLICT`,
`INVALID_STATE_TRANSITION`, `LOGISTICS_ONLY_FEATURE_UNAVAILABLE` (Fase 2),
`OPTIMISTIC_LOCK_FAILED`, `PAYMENT_PROVIDER_ERROR`,
`WEBHOOK_SIGNATURE_INVALID`, `RATE_LIMITED`, `TENANT_ISOLATION_VIOLATION`.

---

## 1. Auth (público)

### `POST /auth/register`

Auto-registro de BUYER en una página de Cliente Full.

**Request:**
```json
{
  "client_slug": "restaurante-el-prado",
  "email": "comprador@example.com",
  "password": "MyP4ssw0rd!",
  "full_name": "Juan Pérez",
  "phone": "+57 300 1234567",
  "document_type": "CC",
  "document_number": "1010101010"
}
```

**Response 201:**
```json
{
  "user_id": 12345,
  "client_id": 7,
  "email": "comprador@example.com",
  "user_type": "BUYER"
}
```

Set-Cookie: `access_token` y `refresh_token` (HttpOnly,
Secure, SameSite=Strict).

### `POST /auth/guest`

Inicia una sesión de **invitado** (sin cuenta) para poder pedir. Crea un
comprador con `auth_mode = 'EXTERNAL_REFERENCE'` (sin contraseña) y
`external_buyer_id` con prefijo `guest-`, y emite las cookies normales, de modo
que todo el flujo de compra funciona igual que con una cuenta.

**Request:** `{ client_slug, full_name?, phone? }`. Solo para Cliente Full.
**Response 201:** igual a login + `is_guest: true`. Set-Cookie de sesión.

### `GET /buyer/me`

… incluye `is_guest` (true en sesiones de invitado). El storefront lo usa para
ocultarle "Mis pedidos" y la recurrencia.

### `PATCH /buyer/me`

Actualiza `{ full_name?, phone? }` del comprador. El invitado lo usa para
registrar su contacto al confirmar el pedido.

### `POST /auth/login`

**Request:**
```json
{
  "client_slug": "restaurante-el-prado",
  "email": "comprador@example.com",
  "password": "MyP4ssw0rd!"
}
```

Para ADMIN_CLIENT, OPERATOR_CLIENT, COURIER: igual pero el
`client_slug` se infiere del subdominio o del path. Para ADMIN_RUTA:
endpoint separado `/auth/ruta-admin/login` sin `client_slug`.

**Response 200:**
```json
{
  "user_id": 42,
  "client_id": 7,
  "user_type": "ADMIN_CLIENT",
  "expires_in_seconds": 900
}
```

### `POST /auth/refresh`

Renueva el access token usando el refresh cookie. Sin body.

**Response 200:**
```json
{ "expires_in_seconds": 900 }
```

### `POST /auth/logout`

Revoca la sesión actual.

### `POST /auth/ruta-admin/login`

Login del ADMIN_RUTA. Sin `client_slug`.

### `POST /auth/control-view/enter`

Entrar a Vista de Control. Requiere ADMIN_RUTA + master password.

**Request:**
```json
{
  "target_client_id": 7,
  "master_password": "...",
  "reason": "Soporte ticket #1234"
}
```

**Response 200:** nueva sesión con `acting_via_control_view = true`.

### `POST /auth/control-view/exit`

Salir de Vista de Control y volver a sesión normal de ADMIN_RUTA.

---

## 2. Catálogo público (storefront, sin auth)

### `GET /public/clients/:slug`

Información pública del Cliente Full (nombre, logo, descripción).

### `GET /public/clients/:slug/categories`

Categorías del catálogo.

### `GET /public/clients/:slug/products`

Productos del catálogo. Soporta query params:
`?category_id=&search=&page=1&limit=20&sort=name|price_asc|price_desc`.

**Response 200:**
```json
{
  "data": [
    {
      "id": 101,
      "name": "Pizza Hawaiana grande",
      "description": "...",
      "unit_price": 35000,
      "currency": "COP",
      "product_type": "VENTA_NORMAL",
      "image_url": "https://...",
      "stock_quantity": 12,
      "status": "ACTIVE",
      "category_id": 5
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 87 }
}
```

### `GET /public/clients/:slug/products/:id`

Detalle del producto.

### `GET /public/clients/:slug/pickup-points`

Puntos físicos activos del Cliente para PICKUP.

---

## 3. BUYER (storefront, auth requerida)

### `GET /buyer/me`

Perfil del Comprador autenticado. **Response 200:** `{ id, full_name, email, phone }`. Devuelve 401 si no hay sesión (el storefront lo usa para saber si saludar o mostrar "Iniciar sesión").

### `PATCH /buyer/me`

Actualizar datos básicos.

### `GET /buyer/addresses` / `POST` / `PATCH /:id` / `DELETE /:id`

Direcciones guardadas.

### `POST /buyer/orders`

Crear o reutilizar el pedido `DRAFT` activo del Comprador. En Fase 1,
el carrito vive en BD: un carrito es un pedido `DRAFT` con items. El
backend valida que el `client_id` del producto coincida con el
`client_id` del Comprador.

**Request:**
```json
{
  "items": [
    { "product_id": 101, "quantity": 2 },
    { "product_id": 107, "quantity": 1 }
  ]
}
```

**Response 201:**
```json
{
  "id": 5001,
  "order_status": "DRAFT",
  "subtotal": 105000,
  "total": 105000,
  "items": [ ... ]
}
```

### `PATCH /buyer/orders/:id/items/:item_id`

Actualizar cantidad de un item del carrito. Solo permitido si el
pedido está en `DRAFT`.

**Request:**
```json
{
  "quantity": 3
}
```

### `DELETE /buyer/orders/:id/items/:item_id`

Remover un item del carrito. Solo permitido si el pedido está en
`DRAFT`.

### `DELETE /buyer/orders/:id`

Vaciar / descartar el carrito. Solo permitido si el pedido está en
`DRAFT`; cambia el pedido a `CANCELLED_BY_CUSTOMER` o lo elimina
lógicamente según la implementación de estados.

### `PATCH /buyer/orders/:id/confirm`

Pasar a PENDING_CONFIRM con `delivery_type` y `payment_method`.

**Request:**
```json
{
  "delivery_type": "SHIP",
  "delivery_address": {
    "line": "Cra 7 # 100 - 50",
    "city": "Bogotá",
    "state": "Bogotá D.C.",
    "country": "CO",
    "postal_code": "110111",
    "latitude": 4.6951,
    "longitude": -74.0427,
    "instructions": "Apto 502, edificio azul"
  },
  "payment_method": "ONLINE_AT_ORDER",
  "payment_method_submethod": null
}
```

Para PICKUP: en lugar de `delivery_address`, enviar `pickup_point_id`.

### `POST /buyer/orders/:id/initiate-payment`

Para `payment_method = ONLINE_AT_ORDER`. Inicia la sesión con Wompi.

**Response 200:**
```json
{
  "order_id": 5001,
  "payment_status": "PENDING_ONLINE_PAYMENT",
  "wompi_checkout_url": "https://checkout.wompi.co/p/...",
  "wompi_reference": "RUTA-5001-AbC123"
}
```

### `GET /buyer/orders`

Lista de mis pedidos.

### `GET /buyer/orders/:id`

Detalle con timeline (estados históricos).

### `POST /buyer/orders/:id/cancel`

Solo permitido en DRAFT, PENDING_CONFIRM, PENDING_ONLINE_PAYMENT,
ORDER_SUBMITTED, ORDER_VALIDATING. Estados posteriores requieren
`POST /buyer/orders/:id/request-cancel` (queda como
`CUSTOMER_CANCEL_REQUEST` y el ADMIN_CLIENT decide).

### `POST /buyer/orders/:id/request-cancel`

Solicitar cancelación post-despacho.

### `POST /buyer/orders/:id/confirm-receipt`

`DELIVERED → CONFIRMED_BY_CUSTOMER`.

---

## 4. ADMIN_CLIENT / OPERATOR_CLIENT (admin panel)

### Clientes (propio)

- `GET /admin/client` — Info del Cliente propio.
- `PATCH /admin/client` — Editar info corporativa.
- `GET /admin/client/parameters` — Ver parámetros vigentes.
- `PATCH /admin/client/parameters/:key` — Override de parámetro.
  Body: `{ value }` — **no** `parameter_value`: ese es el nombre del campo en
  la *respuesta*, no en la petición. La respuesta devuelve el parámetro
  completo (`parameter_key`, `parameter_value`, `value_type`, `source`).

### Productos

- `GET /admin/products` — Lista con filtros.
- `POST /admin/products` — Crear.
- `GET /admin/products/:id` — Detalle.
- `PATCH /admin/products/:id` — Editar.
- `DELETE /admin/products/:id` — Eliminar (soft delete vía status).
- `POST /admin/products/bulk-import` — Subir Excel (multipart).
- `GET /admin/products/bulk-import/:job_id` — Estado de import.
- `GET /admin/categories` / `POST` / `PATCH /:id` / `DELETE /:id`.

### Compradores

- `GET /admin/buyers` — Lista de BUYERs del Cliente.
- `GET /admin/buyers/:id` — Detalle.
- `PATCH /admin/buyers/:id` — Editar (suspender, etc.).

### Repartidores

- `GET /admin/couriers` — Lista. Query: `?q=&status=&page=&page_size=`
  (**no** `search` ni `limit`: se ignoran en silencio y la búsqueda no filtra).
- `POST /admin/couriers` — Crear. Body:
  `{ email, password, full_name, phone, document_type?, document_number?, transport_mode? }`.
  El repartidor es un usuario que inicia sesión, así que `email`, `password`
  (mínimo 8) y `phone` son **obligatorios**. El medio de transporte se llama
  `transport_mode`, no `vehicle_type`.
- `GET /admin/couriers/:id` — Detalle. El transporte viene anidado:
  `profile: { transport_mode, vehicle_plate }`, no plano en la raíz.
- `PATCH /admin/couriers/:id` — Editar. Mismos campos que crear, todos
  opcionales, más `status`.
- `GET /admin/couriers/:id/orders` — Historial de pedidos del courier.
- `GET /admin/couriers/:id/metrics` — Métricas (entregas, tiempos).

### Puntos físicos

- `GET /admin/pickup-points` / `POST` / `PATCH /:id` / `DELETE /:id`.

### Pedidos (vista admin)

- `GET /admin/orders` — Lista con filtros amplios.
  `?status=&payment_status=&buyer_id=&courier_id=&date_from=&date_to=`.
- `GET /admin/orders/:id` — Detalle completo (incluye history,
  payments, evidence).
- `POST /admin/orders/:id/transition` — Transición manual de estado.
  Body: `{ to_state, reason, dimension: "order_status" }`.
- `POST /admin/orders/:id/accept` — VALIDATION_APPROVED →
  SELLER_CONFIRMED (Flujo 1).
- `POST /admin/orders/:id/reject` — Rechazar (vendedor).
- `POST /admin/orders/:id/mark-preparing` — → PREPARING.
- `POST /admin/orders/:id/mark-ready` — → READY_TO_SHIP o
  READY_FOR_PICKUP según delivery_type.
- `POST /admin/orders/:id/cancel` — Cancelación forzada por admin.
- `POST /admin/orders/:id/cancel-request/approve` — Aprobar
  CUSTOMER_CANCEL_REQUEST.
- `POST /admin/orders/:id/cancel-request/reject` — Rechazar.

### Mapa de asignación

- `GET /admin/orders/map` — Pedidos geolocalizados del mapa de asignación:
  los que esperan repartidor (`AWAITING_COURIER_ASSIGNMENT`) **y** los que ya
  lo tienen (`COURIER_ASSIGNED`, `SHIPPED`, `IN_TRANSIT`, `OUT_FOR_DELIVERY`,
  `ARRIVED_AT_CUSTOMER`). Solo devuelve pedidos con latitud y longitud.
  Cada elemento incluye `courier_user_id`, `courier_name` y `courier_phone`
  (`null` mientras el pedido está sin asignar), que es lo que distingue un
  grupo del otro en la UI.
- `GET /admin/orders/:id/collection-evidence` — Foto del recibo del cobro COD
  (misma respuesta que la versión del repartidor).
- `POST /admin/orders/:id/assign-courier` — Asignar.
  Body: `{ courier_user_id }`.
- `POST /admin/orders/:id/unassign-courier`.

### Pagos / cobros

- `GET /admin/payments` — Lista.
- `GET /admin/payments/:id` — Detalle (incluye evidencia).
- `PATCH /admin/payments/:id/reconcile` — Marcar como conciliado.

### Proveedores de pago

- `GET /admin/payment-providers/wompi` — Estado de la config Wompi del Cliente.
  Respuesta: `{ configured, enabled, public_key, has_private_key, has_webhook_secret, updated_at }`.
  **Los secretos nunca se devuelven** (private key, events secret): solo se informa si están cargados.
- `PUT /admin/payment-providers/wompi` — Autogestión de credenciales por el ADMIN_CLIENT.
  Body: `{ enabled, public_key, private_key?, events_secret? }`. Los secretos vacíos/omitidos **conservan**
  los guardados (para editar la public key sin re-escribirlos). Se guarda en `client_payment_providers`.

### Webhooks salientes

- `GET /admin/webhooks` — Lista de suscripciones.
- `POST /admin/webhooks` — Crear.
- `GET /admin/webhooks/:id/deliveries` — Historial de entregas.
- `POST /admin/webhooks/:id/deliveries/:delivery_id/retry`.

### Dashboard

- `GET /admin/dashboard/metrics?from=&to=` — KPIs principales.
- `GET /admin/dashboard/orders-by-status`.
- `GET /admin/dashboard/sales-trend`.

---

## 5. COURIER (vista móvil)

### `GET /courier/me`

Perfil del Repartidor autenticado.

### `GET /courier/orders/assigned`

Pedidos asignados a mí, agrupados por estado.

**Response 200:**
```json
{
  "active": [
    { "id": 5001, "order_status": "COURIER_ASSIGNED", ... },
    { "id": 5003, "order_status": "OUT_FOR_DELIVERY", ... }
  ],
  "completed_today": 12
}
```

### `GET /courier/orders/:id`

Detalle del pedido para el COURIER (incluye dirección, contacto del
Comprador, items, total a cobrar si aplica).

### `POST /courier/orders/:id/start-shipping`

`COURIER_ASSIGNED → SHIPPED → IN_TRANSIT`.

### `POST /courier/orders/:id/mark-out-for-delivery`

`IN_TRANSIT → OUT_FOR_DELIVERY`.

### `POST /courier/orders/:id/arrive`

`OUT_FOR_DELIVERY → ARRIVED_AT_CUSTOMER`.

### `POST /courier/orders/:id/record-collection`

Registrar cobro contra entrega.

**Request (application/json):** no es multipart. El backend no recibe archivos;
la foto viaja embebida en `evidence_url`.

```json
{
  "amount": 105000,
  "currency": "COP",
  "method": "CASH",
  "electronic_submethod": "DATAFONO",
  "external_txn_id": "...",
  "notes": "...",
  "evidence_url": "data:image/jpeg;base64,/9j/4AAQ..."
}
```

`evidence_url` (opcional) admite dos formas:

- una URL `http(s)` de máximo 1000 caracteres, o
- un data URI `data:image/{jpeg|png|webp};base64,…` de máximo 1 400 000
  caracteres (~1 MB de imagen).

**Provisional:** la foto embebida existe porque el proyecto todavía no tiene
object storage — `POST /uploads/presigned-url` responde 501. Se guarda dentro
del JSONB `payments.collection_evidence`. Cuando exista un bucket, el flujo pasa
a subir el archivo y mandar solo la URL; el campo ya acepta ambas formas, así
que la migración no rompe el contrato.

Las rutas bajo `/courier` se parsean con un límite de cuerpo de 2 MB; el resto
de la API mantiene el default de 100 kB.

**Response 200:** `{ payment_id, order_id, payment_status, order_status, amount,
currency, evidence_url, has_evidence, collected_at }`. Cuando la evidencia se
envió embebida, `evidence_url` vuelve en `null` y `has_evidence` en `true`: no
se repite un megabyte que el cliente ya tiene.

### `GET /courier/orders/:id/collection-evidence`

Foto del recibo del cobro contra entrega. El repartidor solo puede consultar
pedidos asignados a él (403 en caso contrario).

**Response 200:**
```json
{
  "payment_id": 1001,
  "order_id": 5001,
  "evidence_url": "data:image/jpeg;base64,/9j/4AAQ...",
  "notes": "Recibo firmado",
  "amount": 105000,
  "currency": "COP",
  "payment_method": "CASH_ON_DELIVERY",
  "payment_method_submethod": null,
  "collected_at": "2026-07-22T18:56:00.000Z"
}
```

404 `RESOURCE_NOT_FOUND` si el pedido no tiene evidencia de cobro — el caso
normal en pedidos sin COD, no un error.

Va en un endpoint aparte y no dentro del detalle del pedido: la imagen embebida
pesa cientos de kB y penalizaría a todas las pantallas que no la muestran.
El equivalente para el staff del Cliente es
`GET /admin/orders/:id/collection-evidence`, con la misma respuesta y sin la
restricción de pedido asignado.

### `POST /courier/orders/:id/mark-delivered`

`ARRIVED_AT_CUSTOMER → DELIVERED`. Si payment_method ON_DELIVERY,
exige `record-collection` previo.

### `POST /courier/orders/:id/attempt-failed`

`OUT_FOR_DELIVERY → DELIVERY_ATTEMPTED`. Body: `{ reason, notes }`.

### `POST /courier/orders/:id/return-to-origin`

Solicitar retorno al origen tras intentos agotados.

### `POST /courier/orders/:id/upload-evidence`

Subir foto/firma adicional. **No implementado** — depende del object storage
pendiente (ver la nota en `record-collection`).

---

## 6. ADMIN_RUTA

### Clientes (gestión global)

- `GET /ruta-admin/clients` — Lista de todos los Clientes.
- `POST /ruta-admin/clients` — Crear Cliente.
- `GET /ruta-admin/clients/:id` — Detalle.
- `PATCH /ruta-admin/clients/:id` — Editar.
- `POST /ruta-admin/clients/:id/activate` — `ACTIVE`.
- `POST /ruta-admin/clients/:id/deactivate` — `INACTIVE` + cascade
  cancellations.
- `DELETE /ruta-admin/clients/:id` — Purge (requiere doble
  confirmación y master password).

### Parámetros globales

- `GET /ruta-admin/parameters` — Parámetros globales.
- `PATCH /ruta-admin/parameters/:key` — Editar. Body: `{ value }`.

### Vista de Control

- `POST /ruta-admin/control-view/:client_id/enter` — Ya cubierto en
  Auth.
- `POST /ruta-admin/control-view/exit`.
- `GET /ruta-admin/control-view/active-session` — Mi sesión activa.

### Master password

- `POST /ruta-admin/master-password/set` — Establecer / cambiar.
- `POST /ruta-admin/master-password/verify` — Verificar (usado por el
  flujo de Vista de Control).

### Auditoría

- `GET /ruta-admin/audit?client_id=&actor=&action=&from=&to=` —
  Log global con filtros.

### Dashboard global

- `GET /ruta-admin/dashboard/metrics` — Métricas de toda la
  plataforma (cantidad de Clientes, total de pedidos, etc.).

---

## 7. Webhooks entrantes

### `POST /webhooks/wompi/:client_id/:provider_id`

Webhook de Wompi para confirmación de pagos. Path incluye
`client_id` y `provider_id` para enrutar correctamente al Cliente
adecuado (Wompi se configura con esta URL en cada `client_payment_providers`).

**Headers:**
- `X-Event-Checksum: <signature>` — firma HMAC-SHA256 con el
  `webhook_secret` del provider.

**Body:** payload nativo de Wompi.

**Response 200:** simple `{ "received": true }`. La lógica de
deduplicación usa `provider_event_id` contra `external_webhook_events`.

---

## 8. Storage (archivos)

### `POST /uploads/presigned-url`

Genera URL pre-firmada para subir archivos al servicio de file storage (por definir).

**Request:**
```json
{
  "purpose": "PRODUCT_IMAGE | DELIVERY_EVIDENCE | COLLECTION_EVIDENCE | RETURN_EVIDENCE | LOGO",
  "filename": "evidencia.jpg",
  "content_type": "image/jpeg",
  "related_entity_type": "order | product | client | refund | return",
  "related_entity_id": 5001
}
```

**Response 200:**
```json
{
  "upload_url": "https://<file-storage-provider>/upload/...?token=...",
  "public_url": "https://<file-storage-provider>/public/...",
  "expires_at": "2025-01-01T12:00:00Z"
}
```

El frontend sube directo al servicio de file storage con `PUT` a `upload_url`.
Una vez subido, llama a:

### `POST /uploads/confirm`

Registra el archivo subido y lo asocia a la entidad.

```json
{
  "upload_token": "...",
  "public_url": "..."
}
```

---

## Endpoints fuera de Fase 1 (anotados para referencia)

- `POST /buyer/orders/:id/dispute` — Bloque 15 (Fase 3).
- `POST /buyer/orders/:id/request-return` — Flujo 7 (Fase 3).
- `POST /admin/refunds/:id/execute` — Flujo 4 (Fase 3).
- `POST /admin/returns/:id/approve` — Flujo 7 (Fase 3).
- `POST /buyer/recurrence-templates` — Flujo 5 (Fase 3).
- `POST /admin/corporate-orders` — Flujo 6 (Fase 3).
- `POST /api/orders` (con API key) — Cliente API (Fase 2).
- `POST /admin/api-keys` — Cliente API (Fase 2).

---

## Headers de respuesta estándar

Toda respuesta incluye:

- `X-Request-Id: <uuid>` — para tracing.
- `X-Ratelimit-Limit`, `X-Ratelimit-Remaining`, `X-Ratelimit-Reset`.

Respuestas mutantes (201, 200 en mutaciones) incluyen:

- `Location:` el recurso creado / modificado.

---

## Geocodificación

### `GET /geocode?address=<texto>&region=<iso2>`

Resuelve una dirección a coordenadas. Proxy de la Geocoding API de Google: la
clave vive en el servidor porque Google solo acota las claves de web service por
IP. Requiere sesión (nunca abierto a internet): ADMIN_CLIENT, OPERATOR_CLIENT,
ADMIN_RUTA y **BUYER** (este último para ubicar su propia dirección de entrega
en el checkout). Cada consulta tiene costo; se acota con el debounce del
frontend y la caché de 24 h del servidor.

```json
{
  "data": {
    "latitude": 4.728,
    "longitude": -74.045,
    "formatted_address": "Cl. 160 #73-32, Bogotá, Colombia",
    "precision": "ROOFTOP",
    "is_precise": true
  }
}
```

`data` es `null` cuando la dirección no se reconoce. `is_precise` es `false`
cuando Google ubicó la vía o el sector en vez del predio.

---

## Paginación

Estándar `?page=1&page_size=20`. Respuesta:

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 152
  }
}
```

El envoltorio de la lista es siempre `data`, nunca `items`.

Listas grandes (ej. `/admin/orders` con muchos miles) pueden usar
cursor-based pagination con `?cursor=<opaque>&page_size=20`. Decidir
endpoint por endpoint en implementación.

---

## OpenAPI

Este documento describe el contrato a nivel funcional. La spec
OpenAPI 3.1 publicada por el backend se expone en:

`https://api.ruta.com/openapi.json` y
`https://api.ruta.com/docs`.

La spec publicada es operativa y debe mantenerse sincronizada con los
handlers de Express cuando cambie el contrato.
