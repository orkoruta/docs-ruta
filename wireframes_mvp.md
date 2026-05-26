# RUTA — Wireframes del MVP Fase 1

Documento de **descripción funcional** de pantallas. No son mockups
gráficos; describen layout, componentes (referencia a `galeria_estilos_ruta.md`)
y comportamiento.

---

## Convenciones

- **Componentes** entre comillas refieren al inventario de la galería
  de estilos (`RutaCard`, `RutaButton`, etc.).
- **Estados** indica las variantes a soportar: loading, empty, error,
  success.
- **Acciones** son los botones / interacciones principales con su
  endpoint asociado.
- Todos los textos visibles en **español**.
- Todas las pantallas tienen `RutaThemeToggle` (modo claro/oscuro).
- Mobile-first donde se indica explícitamente; resto desktop-first
  pero responsive.

---

# PARTE A — STOREFRONT (Compradores)

Todas las pantallas viven bajo `/c/{slug}/...`. El `{slug}` identifica
al Cliente Full. El layout base incluye:

- **Header:** logo del Cliente (cargado dinámicamente), nombre,
  buscador, ícono de carrito (con badge de items), ícono de cuenta
  (login si no autenticado, menú si lo está).
- **Footer:** info del Cliente, links secundarios, "Powered by RUTA"
  discreto.

---

## A1. Catálogo `/c/{slug}/`

**Propósito:** punto de entrada del Comprador. Muestra los productos
del Cliente.

**Layout (desktop):**
```
+---------------------------------------------------------+
| HEADER (logo cliente, buscador, carrito, cuenta)        |
+---------------------------------------------------------+
| HERO opcional (imagen del Cliente)                      |
+---------------------------------------------------------+
| FILTROS (sidebar izq)  |  GRID DE PRODUCTOS (3-4 col)   |
| - Categorías           |  +-------+ +-------+ +-------+|
| - Rango de precio      |  | prod  | | prod  | | prod  ||
| - Solo en stock        |  +-------+ +-------+ +-------+|
| - Solo promociones     |                              |
+---------------------------------------------------------+
| FOOTER                                                  |
+---------------------------------------------------------+
```

**Layout (móvil):** sin sidebar; filtros en sheet inferior.

**Componentes:** `RutaCard` (cada producto), `RutaPill` para tags
"Promoción", `RutaButton` para "Agregar al carrito" en cada card.

**Estados:**
- Loading: skeletons grises tipo card.
- Empty (sin productos): mensaje "Este comercio aún no tiene
  productos publicados".
- Error: mensaje "No pudimos cargar el catálogo. Reintentar".

**Acciones:**
- Click en producto → `/c/{slug}/product/{id}`.
- Click en "Agregar al carrito" → persiste el carrito en BD mediante
  un pedido `DRAFT` del Comprador.
  - **Decisión:** el carrito vive en BD, no en `localStorage`.

**Endpoints:**
- `GET /public/clients/:slug` — info del Cliente para el header.
- `GET /public/clients/:slug/categories`.
- `GET /public/clients/:slug/products?...`.

---

## A2. Detalle de producto `/c/{slug}/product/{id}`

**Propósito:** página de un producto.

**Layout:**
```
+---------------------------------------------------------+
| HEADER                                                  |
+---------------------------------------------------------+
| IMAGEN (galería)  |  NOMBRE DEL PRODUCTO               |
|                   |  Precio: $XX.XXX                    |
|                   |  RutaPill "Promoción" si aplica     |
|                   |  Descripción                        |
|                   |  Cantidad: [- 1 +]                  |
|                   |  RutaButton AZUL "Agregar al carrito"|
|                   |  Stock disponible: 12               |
+---------------------------------------------------------+
| FOOTER                                                  |
+---------------------------------------------------------+
```

**Acciones:**
- Cambio de cantidad.
- Agregar al carrito → toast "Agregado", carrito en header
  incrementa.

---

## A3. Carrito `/c/{slug}/cart`

**Propósito:** ver y editar items antes del checkout.

**Layout:**
```
+---------------------------------------------------------+
| HEADER                                                  |
+---------------------------------------------------------+
| MI CARRITO (h1)                                         |
|                                                         |
| +-----------------------------------------------------+|
| | Producto A  $XX  [- qty +]  Subtotal: $XX     [x] ||
| | Producto B  $XX  [- qty +]  Subtotal: $XX     [x] ||
| +-----------------------------------------------------+|
|                                                         |
| Subtotal:     $XX.XXX                                  |
| RutaButton AZUL "Ir a checkout"                        |
+---------------------------------------------------------+
```

**Estados:**
- Empty: "Tu carrito está vacío" + RutaButton "Ver catálogo".
- Item sin stock: badge rojo + bloqueo de checkout.

**Acciones:**
- Cambiar cantidad / eliminar item.
- "Ir a checkout" → si no autenticado, redirige a login con
  `?return=/c/{slug}/checkout`.

---

## A4. Checkout `/c/{slug}/checkout`

**Propósito:** seleccionar entrega, dirección, método de pago y
confirmar.

Solo accesible si está autenticado y hay items en el carrito.

**Layout:**
```
+---------------------------------------------------------+
| HEADER                                                  |
+---------------------------------------------------------+
| Paso 1: TIPO DE ENTREGA                                 |
|   ( ) SHIP — Entrega a domicilio                       |
|   ( ) PICKUP — Recoger en punto físico                 |
|                                                         |
| Paso 2 (si SHIP):                                       |
|   DIRECCIÓN DE ENTREGA                                  |
|   [Mis direcciones guardadas] dropdown                  |
|   [Mapa OSM para confirmar ubicación]                   |
|   Instrucciones adicionales: [textarea]                 |
|                                                         |
| Paso 2 (si PICKUP):                                     |
|   ELEGIR PUNTO FÍSICO                                   |
|   [Lista con cards de pickup_points + Mapa OSM]         |
|                                                         |
| Paso 3: MÉTODO DE PAGO                                  |
|   ( ) Pago online (Wompi)                              |
|   ( ) Pago electrónico contra entrega                   |
|       (submétodo: Datáfono / QR / Transferencia)        |
|   ( ) Pago en efectivo contra entrega                   |
|                                                         |
| RESUMEN DEL PEDIDO (lateral o inferior)                 |
|   - Items (RutaOrderSummary)                            |
|   - Subtotal                                            |
|   - Envío                                               |
|   - Total                                               |
|                                                         |
| RutaButton VERDE "Confirmar pedido"                     |
+---------------------------------------------------------+
```

**Estados:**
- Validación: si falta info, deshabilitar botón con tooltip.
- Procesando: spinner en botón al hacer submit.

**Acciones:**
- "Confirmar pedido":
  - Si payment_method = ONLINE_AT_ORDER:
    `POST /buyer/orders` (DRAFT) → `PATCH /buyer/orders/:id/confirm`
    → `POST /buyer/orders/:id/initiate-payment` → redirect a
    `wompi_checkout_url`.
  - Si ON_DELIVERY:
    `POST /buyer/orders` (DRAFT) → `PATCH /buyer/orders/:id/confirm`
    → redirect a `/c/{slug}/orders/{id}` (ya está enviado al sistema).

**Componentes:** `RutaCard` por sección, `RutaSectionHeader`,
mapa con Leaflet + OSM.

---

## A5. Confirmación de pago `/c/{slug}/checkout/confirmation`

**Propósito:** página de retorno desde Wompi.

**Layout:**
```
+---------------------------------------------------------+
| HEADER                                                  |
+---------------------------------------------------------+
| ICONO + MENSAJE GRANDE                                  |
| (verde: éxito / ámbar: procesando / rojo: rechazado)    |
|                                                         |
| Estado actual del pedido: [estado actual]               |
| Número de pedido: #5001                                 |
|                                                         |
| RutaButton "Ver mi pedido"                              |
| RutaButton secundario "Seguir comprando"                |
+---------------------------------------------------------+
```

**Flujo:** Wompi redirige aquí con query params indicando resultado.
El frontend hace polling a `GET /buyer/orders/{id}` hasta ver el
estado definitivo (puede demorar segundos por el webhook).

---

## A6. Mis pedidos `/c/{slug}/orders`

**Propósito:** historial del Comprador.

**Layout:**
```
+---------------------------------------------------------+
| HEADER                                                  |
+---------------------------------------------------------+
| MIS PEDIDOS                                             |
| Filtros: [Todos] [Activos] [Completados] [Cancelados]   |
|                                                         |
| +-----------------------------------------------------+|
| | #5001 — 05/Ene/2025                                ||
| | 3 items — $105.000                                 ||
| | RutaStatusButton AZUL "En tránsito"                ||
| | RutaButton secundario "Ver detalle"               ||
| +-----------------------------------------------------+|
| +-----------------------------------------------------+|
| | #4999 — 02/Ene/2025                                ||
| | RutaStatusButton VERDE "Entregado"                 ||
| +-----------------------------------------------------+|
+---------------------------------------------------------+
```

**Estados:** Empty: "Aún no tienes pedidos". Pagination.

---

## A7. Detalle de pedido (vista Comprador) `/c/{slug}/orders/{id}`

**Propósito:** ver estado completo y timeline.

**Layout:**
```
+---------------------------------------------------------+
| HEADER                                                  |
+---------------------------------------------------------+
| Pedido #5001                                            |
| RutaStatusButton AZUL "En tránsito"                     |
|                                                         |
| TIMELINE (RutaTimeline vertical)                        |
|   ✓ Pedido creado — 05/Ene 10:30                       |
|   ✓ Pago confirmado — 05/Ene 10:31                     |
|   ✓ Preparando — 05/Ene 10:45                          |
|   ● En tránsito — 05/Ene 11:20  (estado actual)        |
|   ○ Entregado                                           |
|                                                         |
| DETALLE DE ENTREGA                                      |
|   Tipo: SHIP                                            |
|   Dirección: ...                                        |
|   Repartidor: Juan Pérez (oculto si no asignado)        |
|                                                         |
| RESUMEN DEL PEDIDO (RutaOrderSummary)                   |
|   3 items, $105.000                                     |
|                                                         |
| PAGO                                                    |
|   Método: Online (Wompi)                                |
|   Estado: Pagado                                        |
|                                                         |
| ACCIONES (condicionales)                                |
|   - RutaButton ROJO "Cancelar pedido" (solo si estado lo permite) |
|   - RutaButton AMBAR "Solicitar cancelación" (post-despacho) |
|   - RutaButton VERDE "Confirmar recepción" (en DELIVERED) |
+---------------------------------------------------------+
```

**Acciones:**
- Cancelar / solicitar cancelación según estado.
- Confirmar recepción.
- En Fase 3: "Solicitar devolución", "Abrir disputa".

---

## A8. Mi cuenta `/c/{slug}/account`

**Propósito:** perfil del Comprador.

Secciones:
- Información personal (editable).
- Direcciones guardadas (CRUD).
- Cambiar contraseña.
- Cerrar sesión.

---

## A9. Login / Registro `/c/{slug}/(auth)/login` y `/register`

**Login:** email + contraseña + RutaButton "Iniciar sesión" + link a
"Crear cuenta" + link a "Olvidé mi contraseña".

**Registro:** email + contraseña + nombre + documento + teléfono +
RutaButton "Crear cuenta" + link a "Ya tengo cuenta".

Diseño limpio, centrado, max-width 480px. RutaCard contenedor.

---

# PARTE B — ADMIN (Staff del Cliente y RUTA)

Vive bajo `app.ruta.com` o `/admin/...`. Layout base:

- **Sidebar izquierda colapsable** (RutaSidebar): navegación según rol.
- **Header** (RutaHeader): nombre del Cliente activo, indicador de
  Vista de Control si aplica, búsqueda global, menú de usuario.

---

## B1. Login admin `/login`

Login unificado para ADMIN_RUTA, ADMIN_CLIENT, OPERATOR_CLIENT,
COURIER. El sistema detecta el rol del usuario y redirige al landing
correspondiente:

- ADMIN_RUTA → `/ruta-admin/clients`
- ADMIN_CLIENT, OPERATOR_CLIENT → `/admin/dashboard`
- COURIER → `/courier`

---

## B2. Dashboard del Cliente `/admin/dashboard`

**Propósito:** vista panorámica para ADMIN_CLIENT y OPERATOR_CLIENT.

**Layout:**
```
+---------------------------------------------------------+
| SIDEBAR | HEADER (RutaHeader)                          |
+---------+-----------------------------------------------+
|         | RutaSectionHeader "Resumen de hoy"           |
| Nav     |                                               |
|         | 4 RutaMetricCard en grid:                    |
| Dash    |  +-----+ +-----+ +-----+ +-----+             |
| Pedidos |  |Pedi-| |En   | |Entre| |$$ en|             |
| Mapa    |  |dos  | |trans| |gados| |trans|             |
| Cat.    |  |hoy  | |sito | |hoy  | |ito  |             |
| Compr.  |  +-----+ +-----+ +-----+ +-----+             |
| Repart. |                                               |
| Puntos  | RutaSectionHeader "Acciones rápidas"          |
| Webhook |  - Ver pedidos pendientes                     |
| Settings|  - Abrir mapa de asignación                   |
|         |  - Crear producto                             |
|         |                                               |
|         | RutaSectionHeader "Últimos pedidos"           |
|         |  Tabla compacta de los 10 más recientes       |
+---------+-----------------------------------------------+
```

**Datos:**
- `GET /admin/dashboard/metrics?from=today&to=today`.
- `GET /admin/orders?limit=10&sort=created_at_desc`.

---

## B3. Lista de pedidos `/admin/orders`

**Propósito:** vista operativa para gestionar pedidos.

**Layout:**
```
+---------+-----------------------------------------------+
| SIDEBAR | HEADER                                        |
+---------+-----------------------------------------------+
|         | FILTROS (RutaCard horizontal):                |
|         |  [Estado ▼] [Tipo entrega ▼] [Pago ▼]        |
|         |  [Rango fecha] [Repartidor ▼] [Buscar]        |
|         |  RutaButton "Exportar" (gris)                 |
|         |                                               |
|         | TABLA:                                        |
|         |  | # | Comprador | Items | $ | Estado | ...| |
|         |  |---|-----------|-------|---|--------|----|  |
|         |  |5001| Juan P.  | 3     |$X | EN_TRA | →  |  |
|         |  |...                                       |  |
|         |                                               |
|         | Paginación                                    |
+---------+-----------------------------------------------+
```

**Estados:**
- Cada fila tiene RutaStatusButton del estado.
- Bulk select: checkbox en cada fila → barra inferior con acciones
  masivas (asignar, exportar). [Diferir a polish.]

**Acciones:**
- Click en fila → `/admin/orders/{id}`.

---

## B4. Detalle de pedido (admin) `/admin/orders/{id}`

**Propósito:** vista 360 del pedido, acción según rol y estado.

**Layout (2 columnas):**
```
Col izq (timeline + datos):
  - Pedido #5001
  - RutaStatusButton estado actual
  - Timeline completa (RutaTimeline)
  - Detalle Comprador (nombre, contacto, dirección)
  - Detalle entrega (tipo, repartidor, etc.)

Col der (resumen + acciones):
  - RutaOrderSummary (items + totales)
  - Pago: estado, método, evidencia (link)
  - Acciones según estado (RutaButton):
    * En VALIDATION_APPROVED: "Aceptar pedido" / "Rechazar"
    * En SELLER_CONFIRMED: "Marcar preparando"
    * En PREPARING: "Marcar listo para despacho/pickup"
    * En AWAITING_COURIER_ASSIGNMENT: "Ir al mapa"
    * En cualquier estado intermedio: "Cancelar"
    * En CUSTOMER_CANCEL_REQUEST: "Aprobar" / "Rechazar"
  - Historial completo (collapsible)
  - Auditoría (collapsible)
```

---

## B5. Mapa de asignación `/admin/orders/map`

**Propósito:** asignar Repartidores a pedidos listos.

**Layout:**
```
+---------+-----------------------------------------------+
| SIDEBAR | HEADER                                        |
+---------+-----------------------------------------------+
|         | LEFT (mapa OSM full)         RIGHT (panel)   |
|         |  +------------------+  +-------------------+ |
|         |  |                  |  | PEDIDOS LISTOS    | |
|         |  |  ●  Pin pedido   |  | (AWAITING_COUR..) | |
|         |  |  ●  Pin pedido   |  | - #5001 (10am)    | |
|         |  |  ●  Pin pedido   |  | - #5003 (10:30am) | |
|         |  |                  |  | - #5005           | |
|         |  |                  |  |                   | |
|         |  |  ▲ Pin repartid. |  | REPARTIDORES      | |
|         |  |                  |  | DISPONIBLES       | |
|         |  +------------------+  | ▲ Juan (3 act.)   | |
|         |                        | ▲ María (1 act.)  | |
|         |                        +-------------------+ |
+---------+-----------------------------------------------+
```

**Interacción:**
1. Click en un pedido → se resalta en el mapa, abre panel inferior
   con detalles.
2. Click en un Repartidor → se resalta su ubicación.
3. Botón "Asignar a [Repartidor]" → confirma → llama
   `POST /admin/orders/{id}/assign-courier`.
4. Auto-refresh cada 30s.

**Componentes:** Leaflet + OSM tiles. Pines custom según estado.

---

## B6. Gestión de productos `/admin/products`

**Lista:** tabla con buscador, filtros por categoría y estado,
RutaButton "Crear producto", RutaButton "Importar Excel" (abre
modal).

**Crear / editar producto:** formulario con campos del modelo
(`nombre`, `descripción`, `precio`, `categoría`, `tipo`, `imagen`,
`stock`, `status`). Upload de imagen vía presigned URL.

**Importar Excel:** modal con drop zone, parseo del archivo, preview
de las primeras filas, validación, botón "Importar". Reporte de
resultados (success / errors).

---

## B7. Gestión de Repartidores `/admin/couriers`

Lista + CRUD básico. Detalle muestra historial de pedidos y métricas
(entregas completadas, tasa de éxito, tiempo promedio).

---

## B8. Gestión de Compradores `/admin/buyers`

Lista con buscador. Detalle muestra perfil, direcciones, historial
de pedidos. Acciones: suspender / activar.

---

## B9. Puntos físicos `/admin/pickup-points`

Lista + CRUD. Cada pickup point tiene: nombre, dirección, mapa para
ubicar (drag de pin), horario, teléfono.

---

## B10. Configuración del Cliente `/admin/settings`

Sub-tabs:
- **Información corporativa:** nombre, descripción, logo, contacto.
- **Pasarelas de pago:** lista de proveedores configurados, botón
  "Agregar Wompi", formulario con `public_key`, `private_key`,
  `events_secret`, `webhook_url` (read-only, generada por RUTA).
- **Webhooks salientes:** suscripciones, URL, eventos seleccionados,
  signing secret.
- **Parámetros operativos:** lista de parámetros overridables, con
  valor global y opción de override.

---

# PARTE C — COURIER (móvil-first)

URL base: `/courier/...` o subdominio `courier.ruta.com`.

Layout mobile-first, sin sidebar. Header simple con nombre del
Repartidor + ícono de cerrar sesión.

---

## C1. Mis pedidos `/courier`

**Propósito:** lista de pedidos asignados al Repartidor.

**Layout:**
```
+----------------------------------+
| ☰ RUTA — Hola, Juan              |
+----------------------------------+
| Tabs: [Activos] [Completados hoy]|
+----------------------------------+
| RutaCard #5001                   |
|  RutaStatusButton "En tránsito"  |
|  Dirección: Cra 7 # 100-50      |
|  $35.000 — Pago contra entrega   |
|  [Abrir →]                       |
+----------------------------------+
| RutaCard #5002                   |
|  RutaStatusButton "Asignado"     |
|  ...                             |
+----------------------------------+
```

Pull-to-refresh. Filtro por estado en tabs.

---

## C2. Detalle de pedido (Courier) `/courier/orders/{id}`

**Layout:**
```
+----------------------------------+
| ← Volver                         |
+----------------------------------+
| Pedido #5001                     |
| RutaStatusButton estado          |
|                                  |
| COMPRADOR                        |
|  Juan Pérez                      |
|  📞 +57 300 ...   [Llamar]      |
|                                  |
| DIRECCIÓN                        |
|  Cra 7 # 100-50, Apto 502        |
|  [Ver en mapa]                   |
|                                  |
| ITEMS                            |
|  - 2x Pizza Hawaiana             |
|  - 1x Coca-Cola                  |
|  Total: $35.000                  |
|                                  |
| PAGO                             |
|  Método: Efectivo contra entrega |
|  Monto a cobrar: $35.000         |
|                                  |
| ACCIONES (según estado):         |
|  [Iniciar despacho] o            |
|  [Llegué al cliente] o           |
|  [Registrar cobro] o             |
|  [Marcar entregado] o            |
|  [Intento fallido] o             |
|  [Solicitar retorno]             |
+----------------------------------+
```

Botones grandes (44px alto mínimo) para uso móvil.

---

## C3. Registrar cobro `/courier/orders/{id}/collect`

**Solo aparece cuando** payment_method es contra entrega y el COURIER
ya está en ARRIVED_AT_CUSTOMER.

**Layout:**
```
+----------------------------------+
| Registrar cobro                  |
|                                  |
| Monto a cobrar: $35.000          |
| Monto recibido: [_______]        |
|                                  |
| Método:                          |
|   ( ) Efectivo                   |
|   ( ) Electrónico                |
|       Submétodo: [Datáfono ▼]    |
|       ID transacción: [____]     |
|                                  |
| Subir evidencia:                 |
|  [📷 Tomar foto del recibo]      |
|  [imagen subida]                 |
|                                  |
| Notas (opcional): [textarea]     |
|                                  |
| RutaButton VERDE                 |
|  "Confirmar cobro"               |
+----------------------------------+
```

Validación: monto >= total, evidencia obligatoria (por parámetro
`collection.evidence_required`).

---

# PARTE D — ADMIN_RUTA

URL: `app.ruta.com/ruta-admin/...`. Layout idéntico al admin pero
con sidebar diferente.

---

## D1. Lista de Clientes `/ruta-admin/clients`

Tabla con buscador, filtros por tipo (API / FULL), estado, fecha de
creación. RutaButton "Crear Cliente". Cada fila: nombre, slug, tipo,
estado, fecha, acciones (Ver, Activar/Desactivar, Vista de Control).

---

## D2. Detalle de Cliente `/ruta-admin/clients/{id}`

Vista 360: info corporativa, métricas globales, último login del
ADMIN_CLIENT, estado de configuración (pasarela configurada o no,
puntos físicos, etc.). Acciones: Editar, Activar/Desactivar,
Entrar a Vista de Control, Eliminar (con doble confirmación).

---

## D3. Crear Cliente `/ruta-admin/clients/new`

Formulario:
- Identificador corporativo único (auto-generable).
- Slug (con preview de URL).
- Nombre, descripción.
- Responsable: nombre, documento, teléfono, email.
- Logo (upload).
- **Tipo de Cliente:** API / FULL (radio).
- Si FULL: **Modalidad frontend:** NATIVE_RUTA / CUSTOM_LANDING (con
  nota de que CUSTOM se desarrolla aparte).
- RutaButton "Crear Cliente".

Al guardar, se crean automáticamente las particiones (vía trigger
de BD) y se le envía email al responsable con instrucciones para
configurar contraseña del primer ADMIN_CLIENT.

---

## D4. Vista de Control `/ruta-admin/control-view/{id}`

**Flujo:**
1. Pantalla intermedia: "Vas a entrar al espacio de [Nombre del
   Cliente]. Por favor ingresa tu **contraseña maestra** y la razón
   de soporte."
2. Campos: `master_password`, `reason` (textarea).
3. RutaButton AZUL "Entrar a Vista de Control".
4. Al confirmar: el navegador es redirigido a `/admin/dashboard`
   con la sesión ahora actuando sobre el Cliente target. Aparece
   un **banner ámbar fijo en la parte superior**:
   ```
   ⚠ Estás en Vista de Control del Cliente [Nombre]. Todas tus
   acciones quedan auditadas. [Salir de Vista de Control]
   ```

Al hacer click en "Salir", la sesión vuelve a ser ADMIN_RUTA normal.

---

## D5. Auditoría `/ruta-admin/audit`

Tabla filtrada por:
- Cliente.
- Actor (usuario o api_key).
- Acción.
- Rango de fechas.
- Acting via control view (sí/no).

Cada fila muestra: fecha, actor, acción, entidad, resultado, IP.
Click en fila → modal con detalle completo (metadata JSON).

---

## D6. Dashboard global `/ruta-admin/dashboard`

KPIs globales: cantidad de Clientes activos, pedidos totales hoy,
ingresos transaccionados hoy (suma de `total` de pedidos), pedidos
por estado en barra apilada, etc. Solo para ADMIN_RUTA, no acceso
a datos comerciales específicos por Cliente.

---

# Resumen de pantallas del MVP Fase 1

| # | Pantalla | App | Rol | Prioridad |
|---|---|---|---|---|
| A1 | Catálogo | storefront | público | Alta |
| A2 | Detalle producto | storefront | público | Alta |
| A3 | Carrito | storefront | BUYER | Alta |
| A4 | Checkout | storefront | BUYER | Alta |
| A5 | Confirmación pago | storefront | BUYER | Alta |
| A6 | Mis pedidos | storefront | BUYER | Alta |
| A7 | Detalle pedido | storefront | BUYER | Alta |
| A8 | Mi cuenta | storefront | BUYER | Media |
| A9 | Login / Registro | storefront | público | Alta |
| B1 | Login admin | admin | todos los staff | Alta |
| B2 | Dashboard | admin | ADMIN/OPERATOR_CLIENT | Alta |
| B3 | Lista pedidos | admin | ADMIN/OPERATOR_CLIENT | Alta |
| B4 | Detalle pedido (admin) | admin | ADMIN/OPERATOR_CLIENT | Alta |
| B5 | Mapa asignación | admin | ADMIN/OPERATOR_CLIENT | Alta |
| B6 | Productos | admin | ADMIN_CLIENT | Alta |
| B7 | Repartidores | admin | ADMIN_CLIENT | Alta |
| B8 | Compradores | admin | ADMIN/OPERATOR_CLIENT | Media |
| B9 | Pickup points | admin | ADMIN_CLIENT | Media |
| B10 | Configuración | admin | ADMIN_CLIENT | Media |
| C1 | Mis pedidos courier | admin (mobile) | COURIER | Alta |
| C2 | Detalle pedido courier | admin (mobile) | COURIER | Alta |
| C3 | Registrar cobro | admin (mobile) | COURIER | Alta |
| D1 | Lista Clientes | admin | ADMIN_RUTA | Alta |
| D2 | Detalle Cliente | admin | ADMIN_RUTA | Alta |
| D3 | Crear Cliente | admin | ADMIN_RUTA | Alta |
| D4 | Vista de Control | admin | ADMIN_RUTA | Alta |
| D5 | Auditoría | admin | ADMIN_RUTA | Media |
| D6 | Dashboard global | admin | ADMIN_RUTA | Baja |

**Total:** 28 pantallas, todas para Fase 1.

---

## Cómo generar mockups visuales

Cada descripción de pantalla aquí + `galeria_estilos_ruta.md` da
suficiente contexto a una IA (Claude, v0, etc.) para generar el
mockup en HTML/Tailwind o Figma. El prompt sugerido está en la
galería de estilos sección 11.
