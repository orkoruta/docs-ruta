# Levantamiento de información del Cliente (onboarding)

Qué hay que pedirle a un Cliente nuevo para poder cargarlo completo en la
base de datos y dejarlo operando.

Cada bloque está mapeado contra las tablas y columnas reales de
`bd/ruta_postgres.sql` y contra lo que los endpoints validan hoy
(esquemas Zod de `@orkoruta/shared`). Donde la BD tiene una columna que
todavía **no se puede llenar por la aplicación**, está marcado
explícitamente: esos datos se piden igual, pero se insertan por SQL.

Fuentes: `bd/ruta_postgres.sql`, `packages-ruta/shared/src/validators/`,
`backend-ruta/api/src/services/`, `matriz_permisos.md`.

---

## 0. Orden de carga

El orden importa: hay claves foráneas y el alta del Cliente dispara la
creación automática de sus particiones.

1. `clients` — el tenant. Sin esto no existe nada más.
2. `users` (ADMIN_CLIENT) — quien va a administrar.
3. `client_payment_providers` — Wompi / Nequi.
4. `pickup_points` — solo si hay PICKUP.
5. `product_categories` → `products`.
6. `users` (COURIER) + `courier_profiles` — solo si hay flota propia.
7. `client_parameters` — solo los que se aparten del default.
8. `client_api_keys` + `webhook_subscriptions` — solo Cliente API.
9. `users` (BUYER corporativos) — solo si aplica.

---

## Bloque 1 — Identidad del negocio

Tabla `clients`. **Todo esto es obligatorio en la primera reunión.**

| Campo BD | Qué preguntar | Obl. | Formato / validación |
|---|---|:--:|---|
| `business_code` | Código interno con el que lo vamos a identificar | ✅ | Texto ≤ 20, único en toda la plataforma |
| `slug` | Nombre para la URL pública (`/tienda/<slug>`) | ✅ | ≤ 100, solo `a-z`, `0-9` y guiones. Único. No se puede cambiar sin romper enlaces |
| `name` | Razón social o nombre comercial | ✅ | ≤ 200 |
| `description` | Una línea de qué vende | — | Texto libre |
| `client_type` | ¿Tiene plataforma propia o le damos todo? | ✅ | `API` (solo logística) o `FULL` (ciclo completo) |
| `frontend_mode` | Si es FULL: ¿storefront genérico o landing propio? | ✅ si FULL | `NATIVE_RUTA` o `CUSTOM_LANDING_BY_RUTA`. **Debe ser NULL si es API** |
| `logo_url` | Logo | — | URL |
| `status` | — | ✅ | `ACTIVE` al crear |

### Datos de contacto — se piden, se insertan por SQL

Estas columnas existen en `clients` pero **ningún endpoint ni pantalla
las escribe hoy** (`createClientSchema` no las incluye). Se piden igual y
se cargan con un `UPDATE` directo:

| Campo BD | Qué preguntar |
|---|---|
| `contact_person_name` | Nombre del responsable del negocio |
| `contact_person_document_type` | Tipo de documento (CC / NIT / CE) |
| `contact_person_document_number` | Número de documento |
| `contact_person_phone` | Celular |
| `contact_person_email` | Correo |

---

## Bloque 2 — Modelo operativo

No son columnas: son las respuestas que determinan qué bloques de abajo
hay que llenar y cuáles se saltan.

| Pregunta | Opciones | Qué desbloquea |
|---|---|---|
| ¿Entrega a domicilio, recogida en punto, o ambas? | `SHIP` / `PICKUP` / ambas | PICKUP → Bloque 5 obligatorio |
| Si hay SHIP: ¿quién entrega? | `OWN_FLEET` (flota propia) / `EXTERNAL_COURIER` (mensajería) | OWN_FLEET → Bloque 4 obligatorio |
| ¿Cómo cobra? | `ONLINE_AT_ORDER`, `ELECTRONIC_ON_DELIVERY`, `CASH_ON_DELIVERY` | ONLINE → Bloque 7 obligatorio |
| ¿Moneda? | Por defecto `COP` | `catalog.default_currency` |
| ¿Maneja pedidos recurrentes? | Sí / No | Solo Cliente FULL |
| ¿Maneja devoluciones y reembolsos? | Sí / No | Solo Cliente FULL |

**Si el Cliente es API**, no aplican y no hay que preguntar por: creación
de pedidos desde UI, reembolsos, recurrencia, pedidos corporativos,
devoluciones post-cierre ni disputas. La API rechaza esos endpoints con
422 `LOGISTICS_ONLY_FEATURE_UNAVAILABLE`. Su método de pago será
`EXTERNAL_PREPAID` (el pago ya ocurrió en su plataforma).

---

## Bloque 3 — Usuarios del staff

Tabla `users`.

### ADMIN_CLIENT (obligatorio, mínimo uno)

| Campo BD | Qué preguntar | Obl. |
|---|---|:--:|
| `email` | Correo de acceso | ✅ |
| `password_hash` | Contraseña inicial (mínimo 10 caracteres, con complejidad) | ✅ |
| `full_name` | Nombre completo | ✅ |
| `phone` | Celular | — |
| `document_type` / `document_number` | Documento | — |

> **No hay endpoint para crear ADMIN_CLIENT.** La API solo crea usuarios
> `BUYER` y `COURIER`. El administrador se inserta por SQL, con el hash
> argon2 generado aparte — mismo procedimiento que
> `infra-ruta/scripts/seed_dev_data.sh:135`.

### OPERATOR_CLIENT (opcional)

Mismos campos, más la lista de permisos que se le otorgan. Los permisos
viven en `operator_permissions` (`permission_key` uno por fila).

Claves disponibles (de `matriz_permisos.md`):

```
PRODUCTS_CREATE          ORDERS_CREATE_CORPORATE      REFUNDS_INITIATE
PRODUCTS_EDIT            ORDERS_ACCEPT_SELLER         REFUNDS_MARK_EXECUTED
PRODUCTS_DELETE          ORDERS_CANCEL_FORCED         REFUNDS_PROVIDER_REQUEST
PRODUCTS_BULK_IMPORT     ORDERS_APPROVE_CANCEL_REQUEST
CATEGORIES_MANAGE                                     RETURNS_REVIEW
INVENTORY_MANAGE         COURIERS_MANAGE              RETURNS_PICKUP_SCHEDULE
                         COURIERS_ASSIGN
BUYERS_VIEW              COURIERS_METRICS             DISPUTES_RESOLVE_NO_ACTION
BUYERS_EDIT                                           DISPUTES_RESOLVE_RETURN
BUYERS_SUSPEND           PICKUP_POINTS_MANAGE
                                                      WEBHOOKS_VIEW_HISTORY
PAYMENTS_RECONCILE       RECURRENCE_MANAGE_CLIENT_WIDE WEBHOOKS_RETRY_FAILED
REPORTS_PRODUCTS         REPORTS_BUYERS               EXPORTS_GENERATE
```

ADMIN_CLIENT los tiene todos implícitamente. OPERATOR_CLIENT no tiene
ninguno por defecto: se otorgan uno a uno.

> La tabla existe y tiene RLS, pero **la asignación de permisos tampoco
> está expuesta** por API ni por pantalla. Igual: se pide la lista y se
> insertan por SQL.

---

## Bloque 4 — Repartidores (solo si `OWN_FLEET`)

Tablas `users` (`user_type = 'COURIER'`) + `courier_profiles`.
Se crean por `POST /admin/couriers`. Máximo 200 por Cliente
(`limits.max_couriers_per_client`).

Por cada repartidor:

| Campo | Qué preguntar | Obl. |
|---|---|:--:|
| `email` | Correo de acceso a la app del repartidor | ✅ |
| `password` | Contraseña inicial (8–128 caracteres) | ✅ |
| `full_name` | Nombre completo | ✅ |
| `phone` | Celular — es el canal real de contacto | ✅ |
| `document_type` / `document_number` | Documento | — |
| `transport_mode` | Moto, carro, bicicleta, a pie… | — |
| `vehicle_plate` | Placa | — |

Pídelo como tabla de Excel: es lo más rápido para cargar diez o veinte de
un tirón.

> Ojo operativo: el token de sesión del repartidor **no expira**
> (`auth.jwt_lifetime_courier_minutes = 0`). Si un repartidor sale de la
> empresa hay que desactivarlo (`status = 'INACTIVE'`), no basta con
> pedirle que cierre sesión.

---

## Bloque 5 — Puntos de recogida (solo si `PICKUP`)

Tabla `pickup_points`. `POST /admin/pickup-points`. Máximo 50
(`limits.max_pickup_points_per_client`).

Por cada punto:

| Campo | Qué preguntar | Obl. | Formato |
|---|---|:--:|---|
| `name` | Nombre del punto ("Sede Chapinero") | ✅ | ≤ 200 |
| `address_line` | Dirección completa | ✅ | ≤ 500 |
| `city` | Ciudad | — | ≤ 100 |
| `state` | Departamento | — | ≤ 100 |
| `country` | País | — | **≤ 10 caracteres** — usar código ISO (`CO`) |
| `postal_code` | Código postal | — | ≤ 20 |
| `latitude` / `longitude` | Coordenadas exactas | — | Se pueden geocodificar desde la dirección, pero si el Cliente las tiene, mejor |
| `opening_hours` | Horario de atención | — | Una franja por línea: `lunes-domingo: 12:00-22:00` |
| `contact_phone` | Teléfono del punto | — | ≤ 30 |

---

## Bloque 6 — Catálogo

### Categorías (`product_categories`)

| Campo | Qué preguntar |
|---|---|
| `name` | Nombre de la categoría |
| `parent_category_id` | Si cuelga de otra (soporta jerarquía) |
| `display_order` | Orden en que quiere que aparezcan |

### Productos (`products`)

| Campo | Qué preguntar | Obl. | Formato |
|---|---|:--:|---|
| `name` | Nombre del producto | ✅ | ≤ 300 |
| `unit_price` | Precio | ✅ | **Entero, sin decimales ni separadores.** `25000`, no `25.000` ni `25000.00` |
| `product_type` | ¿Venta normal o promoción? | ✅ | `VENTA_NORMAL` / `PROMOCION` |
| `sku` | Código interno | — | Único por Cliente |
| `description` | Descripción | — | |
| `category_id` | Categoría a la que pertenece | — | |
| `stock_quantity` | Inventario inicial | — | Entero ≥ 0 |
| `image_url` | Imagen | — | URL válida. Máx. 5 imágenes por producto |
| `currency` | | — | `COP` por defecto |

**Pídelo en Excel.** El importador masivo
(`POST /admin/products/bulk-import`) lee la primera hoja y espera
exactamente estas columnas:

```
name | unit_price | product_type | description | category_id | stock_quantity | image_url
```

Las tres primeras son obligatorias por fila. Las categorías tienen que
existir antes (el Excel referencia `category_id` numérico, no el nombre).

> **Límite real: 1000 filas por archivo.** El parámetro
> `catalog.bulk_import_max_rows` dice 5000, pero el código rechaza en
> 1000 (`bulk_import.service.ts:40`). Si el catálogo es más grande, hay
> que partir el archivo. Está anotado abajo como inconsistencia a
> resolver.

---

## Bloque 7 — Cobros

Tabla `client_payment_providers`. Configurable desde el panel
(Configuración → Pagos).

**Recordatorio para la conversación con el Cliente: RUTA no custodia,
no transfiere ni acredita dinero.** El dinero va directo a las cuentas
del Cliente; nosotros solo registramos estados para que él concilie.

### Si cobra online con Wompi

| Campo (`config` JSONB) | Qué pedir |
|---|---|
| `public_key` | Llave pública de Wompi (no es secreta, viaja al navegador) |
| `private_key` | Llave privada |
| `events_secret` | Secreto de eventos, para validar la firma del webhook |

Además hay que **darle a él** la URL de webhook para que la registre en
su panel de Wompi.

Pregunta también si las llaves son de sandbox o de producción: arrancar
con sandbox y cambiar antes de salir en vivo es lo sensato.

### Si cobra con link de Nequi

| Campo | Qué pedir |
|---|---|
| `payment_link` | URL del link de pago de Nequi Negocios (`http://` o `https://`) |

> El link de Nequi **no tiene webhook de vuelta**: el Cliente confirma
> cada pago a mano desde el detalle del pedido. Hay que decírselo
> explícitamente, es carga operativa diaria para él.

### Si cobra contra entrega

Nada que configurar en BD, pero sí que definir:

- ¿Electrónico (datáfono) o efectivo, o ambos?
- ¿Exige foto del recibo como evidencia? (`collection.evidence_required`)
- ¿Cuál es la devuelta máxima razonable en efectivo?
  (`collection.cash_change_max_amount`, default 100.000 COP)

---

## Bloque 8 — Parámetros de negocio

Tabla `client_parameters`. Todo tiene default global (`client_id = 0`),
así que **esto no bloquea el arranque**: se pregunta solo si el Cliente
tiene una política propia. Configurable desde Configuración → Parámetros.

Estos son los que el Cliente puede sobrescribir:

### Plazos del pedido

| Parámetro | Default | Pregunta |
|---|---|---|
| `order.draft_expiration_minutes` | 1440 (24 h) | ¿Cuánto vive un carrito abandonado? |
| `order.pending_confirm_timeout_minutes` | 60 | ¿Cuánto espera un pedido sin confirmar? |
| `order.pending_online_payment_timeout_minutes` | 15 | ¿Cuánto tiene el comprador para pagar? |
| `order.validation_max_duration_minutes` | 5 | |
| `order.manual_review_sla_hours` | 24 | ¿En cuánto se compromete a revisar un pedido marcado? |
| `order.seller_confirmation_timeout_hours` | 4 | ¿En cuánto acepta o rechaza un pedido? |
| `order.pickup_expiration_hours` | 24 | |

### Despacho a domicilio

| Parámetro | Default | Pregunta |
|---|---|---|
| `ship.courier_assignment_timeout_minutes` | 30 | ¿Cuánto puede quedar un pedido sin repartidor? |
| `ship.preparation_max_hours` | 24 | ¿Cuánto tarda en alistar? |
| `ship.delivery_attempt_max_retries` | 3 | ¿Cuántos intentos de entrega antes de devolver? |
| `ship.delivery_attempt_retry_hours` | 24 | ¿Cuánto espera entre intentos? |
| `ship.in_transit_max_hours` | 72 | |
| `ship.cancel_request_review_hours` | 2 | ¿En cuánto responde una solicitud de cancelación? |

### Recogida en punto

| Parámetro | Default | Pregunta |
|---|---|---|
| `pickup.expiration_hours` | 48 | ¿Cuánto guarda un pedido en el punto? |
| `pickup.notification_reminder_hours` | 24 | ¿Con cuánta anticipación recuerda al comprador? |

### Cierre del pedido

| Parámetro | Default | Pregunta |
|---|---|---|
| `closure.auto_confirm_delivered_hours` | 72 | ¿Cuánto espera la confirmación del comprador antes de cerrar solo? |
| `closure.dispute_window_hours` | 72 | ¿Cuánto tiempo tiene el comprador para reclamar? |

### Reembolsos y devoluciones (solo Cliente FULL)

| Parámetro | Default | Pregunta |
|---|---|---|
| `refund.processing_sla_business_days` | 5 | ¿En cuántos días hábiles devuelve el dinero? |
| `refund.provider_response_timeout_hours` | 72 | |
| `return.review_sla_hours` | 72 | ¿En cuánto aprueba o rechaza una devolución? |
| `return.review_auto_action` | `APPROVE` | Si no responde a tiempo, ¿se aprueba o se rechaza? |
| `return.buyer_ships_deadline_days` | 15 | ¿Cuánto tiene el comprador para despachar la devolución? |
| `return.pickup_schedule_max_attempts` | 2 | |

### Recurrencia (solo Cliente FULL)

| Parámetro | Default |
|---|---|
| `recurrence.generation_lookahead_hours` | 24 |
| `recurrence.max_skipped_cycles` | 3 |

### Cobro contra entrega

| Parámetro | Default |
|---|---|
| `collection.electronic_retry_max_attempts` | 3 |
| `collection.cash_change_max_amount` | 100000 |
| `collection.evidence_required` | `true` |

### Catálogo, mapa, notificaciones y límites

| Parámetro | Default | Pregunta |
|---|---|---|
| `catalog.default_currency` | `COP` | |
| `catalog.bulk_import_max_rows` | 5000 | (ver límite real de 1000 en Bloque 6) |
| `map.default_center_lat` / `map.default_center_lng` | 4.7110 / −74.0721 (Bogotá) | ¿En qué ciudad opera? Ajustar el centro del mapa |
| `notifications.email_enabled` | `true` | |
| `notifications.sms_enabled` | `false` | ¿Quiere SMS? |
| `notifications.push_enabled` | `false` | |
| `storage.evidence_retention_days` | 14 | ¿Cuántos días conservamos la foto del recibo? |
| `storage.product_image_max_count` | 5 | |
| `limits.max_pickup_points_per_client` | 50 | |
| `limits.max_couriers_per_client` | 200 | |
| `limits.max_api_keys_per_client` | 5 | |
| `limits.max_active_recurrences_per_buyer` | 10 | |
| `limits.max_buyer_addresses` | 10 | |
| `cleanup.guest_orphan_minutes` | 10 | |

Los parámetros de auth, sesiones, webhooks, idempotencia y rate limiting
**no son negociables** (`is_overrideable_by_client = FALSE`). No los
pongas sobre la mesa.

---

## Bloque 9 — Solo Cliente API

### API keys (`client_api_keys`)

| Campo | Qué preguntar |
|---|---|
| `name` | Para qué es la llave ("integración ERP", "pruebas") |
| `scopes` | `orders:read`, `orders:write`, o ambos |
| `expires_at` | ¿Quiere que caduque? (opcional) |

Máximo 5 activas. **El secreto se muestra una sola vez** al crearla: hay
que acordar antes por qué canal se le entrega.

### Webhooks salientes (`webhook_subscriptions`)

| Campo | Qué preguntar |
|---|---|
| `url` | ¿A qué endpoint suyo le notificamos? Debe ser HTTPS |
| `events` | ¿Qué eventos quiere recibir? (lista abajo) |
| `signing_secret` | Lo generamos nosotros; él lo usa para validar la firma HMAC-SHA256 |
| `description` | Para qué sirve el endpoint |

Eventos que emite la plataforma hoy:

```
ORDER_CREATED           ORDER_OUT_FOR_DELIVERY    ORDER_RETURN_TO_ORIGIN
ORDER_CANCELLED         ORDER_DELIVERED           PAYMENT_COLLECTED
COURIER_ASSIGNED        ORDER_DELIVERY_FAILED
ORDER_SHIPPED           ORDER_PICKED_UP
```

Reintentos automáticos: 1 m, 5 m, 15 m, 60 m, 6 h, 24 h. Timeout de 10 s
por intento — vale la pena avisarle que su endpoint tiene que responder
rápido y ser idempotente.

> **`webhook_subscriptions` no tiene CRUD.** Hay pantalla para ver el
> historial de entregas y reintentar, pero el alta de la suscripción se
> hace con un `INSERT` directo.

### Datos de sus pedidos

Si va a mandar pedidos por API, pregúntale además:

- ¿Los pagos ya vienen resueltos de su lado? (entonces `EXTERNAL_PREPAID`)
- ¿Cómo identifica a sus compradores? Ese identificador va en
  `users.external_buyer_id` con `auth_mode = 'EXTERNAL_REFERENCE'` (sin
  contraseña, único por Cliente).

---

## Bloque 10 — Solo Cliente FULL con landing propio

Si `frontend_mode = 'CUSTOM_LANDING_BY_RUTA'`, además de todo lo
anterior hay que pedirle el paquete de marca. Esto no vive en la BD sino
en el repo `landing-{slug}`:

- Logo en vectorial (SVG o AI) y en PNG con fondo transparente.
- Paleta: color primario, secundario, y los de acento que use.
- Tipografías (archivos o licencia de Google Fonts).
- Fotos de producto en alta resolución.
- Textos: quiénes son, términos y condiciones, política de datos.
- Dominio donde va a vivir y quién controla el DNS.
- Redes sociales y datos de contacto para el pie de página.

Si es `NATIVE_RUTA`, solo el logo y los colores.

---

## Bloque 11 — Compradores corporativos (opcional)

Solo si el Cliente le vende a empresas con cuenta abierta. Tablas
`users` (`BUYER`) + `buyer_profiles`.

| Campo | Qué preguntar | Obl. |
|---|---|:--:|
| `email` | Correo de la empresa compradora | ✅ |
| `password` | Contraseña inicial (≥ 8) | ✅ |
| `full_name` | Nombre del contacto | ✅ |
| `phone` | Celular | — |
| `document_type` / `document_number` | Documento del contacto | — |
| `buyer_type` | `CORPORATE` | — |
| `corporate_name` | Razón social de la empresa compradora | — |
| `corporate_tax_id` | NIT | — |
| `default_address_*` | Dirección de entrega habitual (línea, ciudad, departamento, país, código postal, lat/lng) | — |

> `POST /admin/buyers` solo escribe correo, contraseña, nombre, teléfono
> y documento. `buyer_type`, los campos corporativos y la dirección por
> defecto **se leen pero nunca se escriben** desde la API: van por SQL.

---

## Mínimo para arrancar

Lo estrictamente indispensable para que el Cliente entre a la plataforma
y empiece a operar:

- [ ] Nombre, código de negocio y slug.
- [ ] Tipo de Cliente (`API` / `FULL`) y, si es FULL, modalidad de frontend.
- [ ] Un ADMIN_CLIENT: nombre, correo y contraseña inicial.
- [ ] Decisión SHIP / PICKUP y quién entrega.
- [ ] Al menos un punto de recogida (si hay PICKUP).
- [ ] Al menos un repartidor (si hay flota propia).
- [ ] Catálogo inicial en Excel, con las categorías definidas antes.
- [ ] Llaves de Wompi o link de Nequi (si cobra online).

Todo lo demás —parámetros, permisos granulares, compradores
corporativos, webhooks— se puede ajustar después sin frenar el arranque.

---

## Huecos conocidos

Cosas que hay que resolver, y que mientras tanto se cargan por SQL. No
son bloqueantes, pero conviene tenerlas presentes al planear la sesión
de onboarding:

1. **Datos de contacto del Cliente** (`clients.contact_person_*`): cinco
   columnas en la BD sin ningún endpoint ni pantalla que las escriba.
2. **Alta de ADMIN_CLIENT y OPERATOR_CLIENT**: la API solo crea `BUYER`
   y `COURIER`. Los usuarios de staff se insertan a mano.
3. **Permisos de operador**: `operator_permissions` existe con RLS, pero
   no hay forma de otorgarlos desde la aplicación.
4. **Suscripciones de webhook**: hay pantalla de historial y reintento,
   no de alta.
5. **Campos corporativos y dirección por defecto del comprador**: se
   leen y se muestran, no se escriben.
6. **Límite de importación masiva**: el parámetro dice 5000 filas, el
   código rechaza en 1000. Uno de los dos está mal.
