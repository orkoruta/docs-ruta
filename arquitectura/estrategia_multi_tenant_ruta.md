# Estrategia Multi-Tenant para RUTA

## 1. Propósito del documento

Este documento define la estrategia **multi-tenant**, de autenticación, de autorización y de integración para **RUTA**.

El objetivo es que RUTA funcione como una sola plataforma compartida por múltiples **Clientes**, pero garantizando que cada Cliente opere como un entorno independiente, aislado y seguro. RUTA debe soportar dos tipos de Cliente: los que ya tienen su propia plataforma de comercio electrónico (Tipo A) y los que delegan el ciclo completo a RUTA (Tipo B).

Este documento toma como base la definición funcional en `All_RUTA.txt` y los 7 archivos de flujo en `docs/flujos/`.

> **Convención de naming**: todo el código, schema de base de datos y nombres de columna del proyecto se escriben en **inglés** (`clients`, `users`, `client_id`, `client_type`, etc.). La documentación se escribe en español.

---

## 2. Rol no financiero de RUTA

**Este es un principio fundacional de la arquitectura. Toda decisión técnica del sistema debe respetarlo.**

RUTA **no actúa** como recaudador financiero, custodio de fondos, intermediario de pagos ni entidad responsable por la liquidación del dinero entre Compradores y Clientes.

### Reglas obligatorias

```txt
- La responsabilidad del recaudo, conciliación, facturación, devolución
  efectiva del dinero y cierre financiero pertenece al Cliente o a los
  proveedores de pago que el Cliente utilice.

- RUTA únicamente registra, organiza y traza la información relacionada
  con pagos, pagos contra entrega, confirmaciones de pago, devoluciones
  y reembolsos, con el fin de apoyar la operación del pedido y permitir
  que el Cliente realice su propia conciliación financiera.

- Si el Comprador paga en línea, el pago debe ir directamente a la
  cuenta, pasarela o medio financiero definido por el Cliente.

- RUTA puede recibir una confirmación técnica del pago mediante
  webhook o API, pero no debe recibir ni custodiar el dinero.

- RUTA solo actualiza el estado del pedido y registra la trazabilidad
  del pago.

- Si el pago es contra entrega, el Repartidor registra en RUTA que
  recibió el dinero del Comprador.

- Ese dinero pertenece operativamente al Cliente, no a RUTA.

- RUTA solo deja evidencia del valor recaudado, el pedido asociado, el
  Repartidor responsable, la fecha, el estado y las novedades necesarias
  para que el Cliente realice su conciliación.
```

### Implicaciones técnicas inmediatas

- **No existe** una página de pago hospedada por RUTA (no hay `ruta.com/pay/...`).
- Cada Cliente configura sus propios proveedores de pago en `client_payment_providers`.
- Los pagos online redirigen al proveedor del Cliente; los webhooks de confirmación llegan a RUTA para actualizar `payment_status`.
- Los pagos contra entrega los registra el Repartidor en la app de RUTA con evidencia, pero el dinero pertenece al Cliente.
- Los reembolsos **nunca** los ejecuta RUTA. Los flujos de `docs/flujos/flujo_4_refund_completo.md` describen acciones del Cliente; RUTA solo registra el estado.

---

## 3. Decisiones arquitectónicas fundamentales

Resumen ejecutivo de las decisiones que rigen todo el resto del documento:

```txt
- RUTA es sistema de información operacional, no infraestructura financiera.
- Una sola aplicación, una sola base de datos, separación lógica por client_id.
- IDs internos: BIGINT. No se usa UUID para entidades operativas.
- Slug solo se usa para construir URLs amigables del Cliente.
- Dos tipos de Cliente: Tipo A (logística) y Tipo B (ciclo completo).
- Tabla única de usuarios (users) que identifica el tipo de actor.
- Compradores y Repartidores 100% independientes por Cliente.
- ADMIN_RUTA vive en users con client_id = 0.
- Vista de Control: ADMIN_RUTA opera como cualquier ADMIN_CLIENT con
  contraseña maestra.
- Autenticación de usuarios: JWT corto + refresh token revocable.
- Autenticación server-to-server: API keys con scopes.
- Un token = un login. Cambio manual de URL invalida la sesión.
- Webhooks bidireccionales: salientes (RUTA -> Cliente) e ingresantes
  (proveedor de pago -> RUTA).
- Auditoría obligatoria en tabla append-only.
- Particionamiento LIST por client_id, creación automática al registrar
  un Cliente nuevo.
- Row Level Security activado, con políticas especiales para ADMIN_RUTA.
- Naming en inglés en todo el código y schema.
```

---

## 4. Tipos de Cliente: Tipo A vs Tipo B

RUTA soporta dos tipos de Cliente con diferencias profundas en cómo se integran con la plataforma. La distinción se materializa en el campo `clients.client_type`.

### 4.1 Definición

```txt
API — Cliente solo logística

El Cliente ya tiene su propia plataforma de comercio electrónico
(catálogo, login del Comprador, carrito, checkout, pasarela de pagos).
Usa a RUTA exclusivamente como proveedor de logística (entrega o
recogida) y opcionalmente para cobro contra entrega.

FULL — Cliente con ciclo completo en RUTA

El Cliente delega a RUTA todo el ciclo del pedido: catálogo, gestión
de Compradores, checkout, integración con pasarela, despacho, entrega,
devoluciones, reembolsos. El frontend que ven los Compradores puede ser
la UI nativa de RUTA (`ruta.com/c/{slug}`) o una landing custom
desarrollada por el equipo de RUTA (en dominio del Cliente o en
ruta.com). En ambos casos el backend es RUTA.
```

### 4.2 Tabla comparativa

| Dimensión | Tipo A (Solo logística) | Tipo B (Ciclo completo) |
|---|---|---|
| Quién maneja el catálogo | Cliente | RUTA |
| Quién maneja la cuenta del Comprador | Cliente | RUTA |
| Dónde ve el Comprador la UI | Sistema del Cliente | UI de RUTA o landing custom hecha por RUTA |
| Quién hace el checkout | Cliente | RUTA |
| Quién integra con la pasarela | Cliente | Cliente (RUTA solo redirige y recibe confirmación) |
| Entrada del pedido en RUTA | API en `PREPARING`, `READY_TO_SHIP` o `READY_FOR_PICKUP` | Flujos 1, 5, 6 según corresponda |
| RUTA maneja pago online | No | No (redirige a pasarela del Cliente) |
| RUTA maneja cobro contra entrega | Sí (vía Repartidor) | Sí (vía Repartidor) |
| RUTA maneja reembolsos | No | Solo traza (Cliente ejecuta) |
| RUTA maneja devoluciones post-cierre | No | Sí (flujo 7) |
| RUTA maneja recurrencia | No | Sí (flujo 5) |
| RUTA maneja pedidos corporativos | No (irrelevante) | Sí (flujo 6) |
| RUTA maneja disputas | No | Sí (BLOQUE 15) |
| El Comprador accede a RUTA | No | Sí (login en `/c/{slug}/login` o landing custom) |
| API keys obligatorias | Sí | Opcional (para integraciones del Cliente con su ERP) |
| Webhooks RUTA -> Cliente | Sí (estado del pedido, eventos logísticos) | Sí (todos los eventos) |
| Webhooks pasarela -> RUTA | Solo si Tipo A usa cobro contra entrega electrónico | Sí (confirmaciones de pago online) |

### 4.3 Estados de los flujos por tipo

**Tipo A — Estados aplicables:**

```txt
Subset del flujo SHIP/PICKUP a partir del punto de entrada:
- PREPARING (si el Cliente quiere que RUTA muestre que está en preparación)
- AWAITING_COURIER_ASSIGNMENT / COURIER_ASSIGNED (si OWN_FLEET)
- READY_TO_SHIP, SHIPPED, IN_TRANSIT, OUT_FOR_DELIVERY,
  ARRIVED_AT_CUSTOMER, DELIVERED
- READY_FOR_PICKUP, AT_PICKUP_POINT,
  CUSTOMER_ARRIVED_AT_PICKUP_POINT, IDENTITY_VALIDATED, PICKED_UP
- Estados de excepción logística: ON_HOLD, DELIVERY_ATTEMPTED,
  DELIVERY_RESCHEDULED, LOST_IN_TRANSIT, RETURN_TO_ORIGIN,
  RETURN_TO_ORIGIN_RECEIVED, LOST_IN_RETURN
- Estados de cobro en entrega (si aplica): PAYMENT_COLLECTION_PENDING,
  PAYMENT_COLLECTED_*, CASH_COLLECTION_PENDING, CASH_PAYMENT_REJECTED
- Cierre: COMPLETED_SUCCESSFULLY, CLOSED, CANCELLED_BY_ADMIN,
  CANCELLED_NO_PAYMENT, DELIVERY_FAILED

Estados que NO aplican:
- DRAFT, PENDING_CONFIRM, ORDER_SUBMITTED, ORDER_VALIDATING,
  MANUAL_REVIEW, VALIDATION_APPROVED, VALIDATION_REJECTED,
  SELLER_CONFIRMED
- Toda la cadena de pago online previa al despacho.
```

**Tipo B — Todos los estados aplicables** según `docs/flujos/`.

### 4.4 Cuando un Cliente cambia de tipo

El `client_type` es modificable solo por `ADMIN_RUTA` y queda auditado. En la práctica un Cliente Tipo A puede evolucionar a Tipo B si decide adoptar la UI de RUTA, pero los pedidos ya creados conservan su tipo original — el campo `order_origin` registra de qué modo entró cada pedido independientemente del `client_type` actual.

---

## 5. Estrategia multi-tenant

RUTA implementa multi-tenancy bajo la siguiente estrategia:

```txt
Una sola aplicación.
Una sola base de datos compartida.
Separación lógica obligatoria por client_id.
Refuerzo físico mediante particionamiento LIST por client_id.
Refuerzo de seguridad mediante Row Level Security.
```

Toda entidad operativa debe pertenecer obligatoriamente a un Cliente mediante el campo `client_id BIGINT NOT NULL`.

---

## 6. Estrategia de identificadores

### 6.1 BIGINT como identificador interno

**Decisión:** los identificadores primarios internos de RUTA son `BIGINT GENERATED BY DEFAULT AS IDENTITY`.

**Justificación técnica:**

- **Performance**: BIGINT (8 bytes) ocupa la mitad que UUID (16 bytes) en cada índice. Con millones de registros por Cliente, esto reduce significativamente el tamaño de índices y mejora la velocidad de joins.
- **Eficiencia de almacenamiento**: cada FK BIGINT vs UUID ahorra 8 bytes por fila. En tablas con decenas de millones de filas, la diferencia es sustancial.
- **Simplicidad operativa**: BIGINT es más fácil de leer en logs, debugging y soporte.
- **Compatibilidad con particionamiento LIST**.

**Mitigación del riesgo de enumeración:**

```txt
- El BIGINT client_id NUNCA debe exponerse en URLs públicas.
- El BIGINT client_id NUNCA debe aparecer en payloads de API hacia el frontend.
- Para identificación pública del Cliente se usa exclusivamente el slug.
- El BIGINT client_id sí puede viajar dentro del JWT (firmado e ilegible).
```

### 6.2 Tres identificadores del Cliente

```txt
1. clients.id (BIGINT)
   Identificador técnico interno. Llave primaria y foránea operativa.
   Nunca se expone al frontend ni a URLs.

2. clients.business_code (TEXT, único)
   Código corporativo visible. Uso: facturación, contratos, reportes
   administrativos del ADMIN_RUTA. Ejemplo: "CLI-0001". No se usa en
   URLs ni queries operativas.

3. clients.slug (TEXT, único)
   Identificador para URLs amigables. Uso exclusivo: construir URLs
   (/c/{slug}/...). Ejemplo: "restaurante-el-prado". No se usa como
   llave foránea ni en queries operativas.
```

---

## 7. Tabla `clients`

```sql
CREATE TABLE clients (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  business_code TEXT UNIQUE NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  description TEXT,
  contact_person_name TEXT,
  contact_person_document TEXT,
  contact_person_phone TEXT,
  logo_url TEXT,

  -- Tipo de Cliente: define el modelo de integración
  client_type TEXT NOT NULL
    CHECK (client_type IN ('API', 'FULL')),

  -- Para Tipo B: define cómo se sirve el frontend al Comprador
  frontend_mode TEXT
    CHECK (frontend_mode IS NULL OR frontend_mode IN (
      'NATIVE_RUTA',           -- Comprador entra a ruta.com/c/{slug}
      'CUSTOM_LANDING_BY_RUTA' -- Landing desarrollada por equipo RUTA
    )),

  status TEXT NOT NULL DEFAULT 'ACTIVE'
    CHECK (status IN ('ACTIVE', 'INACTIVE')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Coherencia: frontend_mode solo aplica para FULL
  CONSTRAINT chk_frontend_mode_type CHECK (
    (client_type = 'API' AND frontend_mode IS NULL) OR
    (client_type = 'FULL' AND frontend_mode IS NOT NULL)
  )
);

-- Registro especial: client_id = 0 representa la plataforma RUTA.
-- Sirve como dueño lógico de los usuarios ADMIN_RUTA y satisface FKs.
INSERT INTO clients (id, business_code, slug, name, client_type, status)
VALUES (0, 'RUTA-PLATFORM', 'ruta-platform', 'RUTA Platform', 'FULL', 'ACTIVE');
```

**Notas:**

- La tabla `clients` es pequeña (máximo ~500 Clientes en horizonte 3-5 años) y no se particiona.
- Las llaves foráneas desde tablas particionadas apuntan a `clients(id)`.
- El registro `id = 0` se crea como bootstrap y nunca se elimina.

---

## 8. Tabla única `users`

Una sola tabla `users` contiene todos los usuarios de la plataforma, independientemente de su tipo.

```sql
CREATE TABLE users (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  user_type TEXT NOT NULL
    CHECK (user_type IN (
      'ADMIN_RUTA',
      'ADMIN_CLIENT',
      'OPERATOR_CLIENT',
      'BUYER',
      'COURIER'
    )),

  email TEXT NOT NULL,
  password_hash TEXT,  -- NULLABLE: ver auth_mode

  -- Modo de autenticación
  auth_mode TEXT NOT NULL DEFAULT 'PASSWORD'
    CHECK (auth_mode IN ('PASSWORD', 'EXTERNAL_REFERENCE')),

  -- Mirror de identidad en sistemas externos (Cliente Tipo A o Tipo B con landing custom)
  external_buyer_id TEXT,

  full_name TEXT,
  document_type TEXT,
  document_number TEXT,
  phone TEXT,

  status TEXT NOT NULL DEFAULT 'ACTIVE'
    CHECK (status IN ('ACTIVE', 'INACTIVE', 'SUSPENDED')),
  last_login_at TIMESTAMPTZ,
  password_changed_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (id, client_id),
  UNIQUE (client_id, email),
  UNIQUE (client_id, external_buyer_id),
  UNIQUE (client_id, document_type, document_number),

  CONSTRAINT fk_users_client FOREIGN KEY (client_id) REFERENCES clients(id),

  -- Consistencia: si auth_mode = PASSWORD, password_hash debe existir
  CONSTRAINT chk_password_consistency CHECK (
    (auth_mode = 'PASSWORD' AND password_hash IS NOT NULL) OR
    (auth_mode = 'EXTERNAL_REFERENCE' AND password_hash IS NULL)
  )
) PARTITION BY LIST (client_id);
```

### 8.1 Reglas críticas

```txt
- Un usuario pertenece SIEMPRE a un único client_id.
- ADMIN_RUTA tiene client_id = 0. Los demás user_type tienen client_id > 0.
- El email es único POR CLIENTE, no globalmente.
- Si la misma persona opera para dos Clientes (como BUYER o COURIER),
  se registran dos usuarios independientes con dos password_hash
  independientes (o ningún password_hash en EXTERNAL_REFERENCE).
- auth_mode = 'PASSWORD': el usuario se autentica con email + password en RUTA.
- auth_mode = 'EXTERNAL_REFERENCE': el usuario solo existe en RUTA
  como referencia (típicamente BUYERs en Clientes Tipo A); nunca se
  loguea en RUTA y su identidad se mantiene solo para trazabilidad
  de pedidos.
- La llave primaria es compuesta (id, client_id) por el particionamiento.
```

### 8.2 Patrón de uso del campo `auth_mode` por tipo de Cliente

| Tipo de Cliente | user_type | auth_mode | password_hash |
|---|---|---|---|
| Tipo A | ADMIN_CLIENT, OPERATOR_CLIENT | `PASSWORD` | obligatorio |
| Tipo A | BUYER | `EXTERNAL_REFERENCE` | NULL |
| Tipo A | COURIER | `PASSWORD` | obligatorio |
| Tipo B (`NATIVE_RUTA`) | todos | `PASSWORD` | obligatorio |
| Tipo B (`CUSTOM_LANDING_BY_RUTA`) | ADMIN_CLIENT, OPERATOR_CLIENT, COURIER | `PASSWORD` | obligatorio |
| Tipo B (`CUSTOM_LANDING_BY_RUTA`) | BUYER | `PASSWORD` | obligatorio (la landing custom de RUTA se autentica contra el backend de RUTA) |
| ADMIN_RUTA | siempre `PASSWORD` | obligatorio |

### 8.3 Datos operativos por tipo de usuario

Los BUYER y COURIER requieren datos operativos adicionales en tablas de extensión:

```sql
CREATE TABLE buyer_profiles (
  user_id BIGINT NOT NULL,
  client_id BIGINT NOT NULL,
  default_address TEXT,
  buyer_type TEXT NOT NULL DEFAULT 'INDIVIDUAL'
    CHECK (buyer_type IN ('INDIVIDUAL', 'CORPORATE')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (user_id, client_id),
  CONSTRAINT fk_bp_user FOREIGN KEY (user_id, client_id)
    REFERENCES users(id, client_id)
) PARTITION BY LIST (client_id);

CREATE TABLE courier_profiles (
  user_id BIGINT NOT NULL,
  client_id BIGINT NOT NULL,
  transport_method TEXT,
  vehicle_plate TEXT,
  additional_data JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (user_id, client_id),
  CONSTRAINT fk_cp_user FOREIGN KEY (user_id, client_id)
    REFERENCES users(id, client_id)
) PARTITION BY LIST (client_id);
```

---

## 9. ADMIN_RUTA y `client_id = 0`

El usuario `ADMIN_RUTA` es el único actor con alcance global de la plataforma.

```sql
INSERT INTO users (
  client_id, user_type, email, password_hash, auth_mode, full_name, status
) VALUES (
  0,
  'ADMIN_RUTA',
  'admin@ruta.com',
  '$argon2id$v=19$...',
  'PASSWORD',
  'Admin Principal',
  'ACTIVE'
);
```

### 9.1 Reglas para ADMIN_RUTA

```txt
- client_id = 0 es un valor sentinel que indica "alcance plataforma".
- La fila clients.id = 0 existe únicamente para sostener la FK.
- Ningún ADMIN_RUTA puede tener client_id > 0.
- Ningún otro user_type puede tener client_id = 0.
- ADMIN_RUTA es creado solo por otro ADMIN_RUTA o durante el bootstrap
  inicial del sistema.
- El primer ADMIN_RUTA se crea mediante un script de instalación.
```

### 9.2 Operación normal vs Vista de Control

```txt
Modo plataforma:
- JWT con client_id = 0, scope = GLOBAL.
- Endpoints administrativos bajo /api/platform/...
- Opera sobre la tabla clients (alta, baja, suspensión).
- Consulta métricas agregadas de todos los Clientes.
- Crea el primer ADMIN_CLIENT de cada Cliente nuevo.

Modo Vista de Control:
- JWT con client_id = X (el Cliente impersonado), scope = CLIENT.
- Claims impersonation = true, impersonator_user_id, control_view_session_id.
- Opera dentro del entorno del Cliente X como si fuera su Admin Client.
- Todas sus acciones se auditan con flag acting_via_control_view = true.
```

---

## 10. Vista de Control (modo soporte)

La Vista de Control permite que un `ADMIN_RUTA` ingrese al entorno operativo de cualquier Cliente para realizar tareas de soporte.

### 10.1 Flujo de activación

```txt
1. ADMIN_RUTA inicia sesión normal en /platform/login.
2. En su panel, selecciona "Vista de Control" y elige el Cliente destino.
3. Ingresa una contraseña maestra (DIFERENTE a su contraseña normal),
   almacenada en control_view_master_passwords.
4. El backend valida la contraseña maestra y el permiso explícito
   can_use_control_view del ADMIN_RUTA.
5. Se crea un registro en control_view_sessions.
6. Se emite un NUEVO JWT con:
     client_id = X (el Cliente impersonado)
     scope = CLIENT
     impersonation = true
     impersonator_user_id = <id del ADMIN_RUTA>
     control_view_session_id = <id del registro>
7. El frontend muestra un banner permanente de modo soporte.
8. Todas las acciones se auditan con acting_via_control_view = true.
```

### 10.2 Duración y notificaciones

```txt
- La sesión de Vista de Control NO tiene duración máxima.
- Dura hasta que el ADMIN_RUTA cierra sesión manualmente o cierra
  explícitamente la Vista de Control.
- El refresh token sigue las reglas normales.
- El Admin Cliente "real" NO recibe notificación cuando un ADMIN_RUTA
  entra por Vista de Control.
- La auditoría queda registrada y es consultable después.
```

### 10.3 Tablas asociadas

```sql
CREATE TABLE control_view_master_passwords (
  user_id BIGINT NOT NULL,
  client_id BIGINT NOT NULL DEFAULT 0,
  master_password_hash TEXT NOT NULL,
  can_use_control_view BOOLEAN NOT NULL DEFAULT FALSE,
  set_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMPTZ,
  PRIMARY KEY (user_id, client_id),
  CONSTRAINT fk_cvmp_user FOREIGN KEY (user_id, client_id)
    REFERENCES users(id, client_id),
  CONSTRAINT chk_cvmp_admin_ruta CHECK (client_id = 0)
);

CREATE TABLE control_view_sessions (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  admin_ruta_user_id BIGINT NOT NULL,
  impersonated_client_id BIGINT NOT NULL,
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  ended_at TIMESTAMPTZ,
  reason TEXT,
  ip_address INET,
  user_agent TEXT,
  CONSTRAINT fk_cvs_client FOREIGN KEY (impersonated_client_id)
    REFERENCES clients(id)
);
```

### 10.4 Reglas obligatorias

```txt
- Solo usuarios ADMIN_RUTA con can_use_control_view = TRUE pueden activar
  Vista de Control. Esto es un permiso explícito, no automático.
- La contraseña maestra es DIFERENTE de la contraseña normal del
  ADMIN_RUTA y vive en su propia tabla.
- Toda acción ejecutada durante Vista de Control queda registrada en
  audit_events con acting_via_control_view = TRUE.
- El frontend debe mostrar de forma permanente y visible que se está
  operando bajo Vista de Control.
- El ADMIN_RUTA puede salir en cualquier momento.
```

---

## 11. Endpoints del Cliente

### 11.1 Estructura de URLs

Por tipo de Cliente y rol:

```txt
Tipo A (Cliente solo logística):
  El Comprador NO accede a RUTA. La interfaz pública la maneja
  el Cliente en su dominio. RUTA solo expone:

  Login Admin/Operator del Cliente:
    https://ruta.com/c/{slug}/admin/login
  Login Repartidor:
    https://ruta.com/c/{slug}/courier/login
  API server-to-server:
    https://ruta.com/api/v1/orders (con Authorization: Bearer rk_live_...)

Tipo B (Cliente con ciclo completo):
  Frontend NATIVE_RUTA:
    https://ruta.com/c/{slug}              (landing por defecto)
    https://ruta.com/c/{slug}/login        (login Comprador)
    https://ruta.com/c/{slug}/admin/login  (Admin/Operator)
    https://ruta.com/c/{slug}/courier/login (Repartidor)

  Frontend CUSTOM_LANDING_BY_RUTA:
    https://{dominio-del-cliente}/...    (UI custom, llama a API de RUTA)
    https://ruta.com/c/{slug}/admin/login (Admin/Operator: SIEMPRE en ruta.com)
    https://ruta.com/c/{slug}/courier/login (Repartidor: SIEMPRE en ruta.com)

ADMIN_RUTA (independiente del tipo):
  https://ruta.com/platform/login
```

**Regla clave:** la app del Repartidor, la app del Admin/Operator del Cliente y el dashboard del Cliente **siempre** viven en `ruta.com`, sin importar el tipo de Cliente. Solo la experiencia del Comprador puede variar.

### 11.2 Resolución del slug

```sql
SELECT id, status, client_type, frontend_mode
FROM clients
WHERE slug = :slug;

-- Si status != 'ACTIVE': rechazar con 404 o 410.
-- Si el slug no existe: 404.
-- El resultado se guarda en el contexto de la request como
-- "requested_client_id" para validación posterior contra el JWT.
```

### 11.3 Validación URL ↔ JWT

**Regla crítica:** un token solo es válido para el Cliente cuyo slug aparece en la URL. Cambio manual de URL = sesión invalidada.

```txt
Flujo de validación en cada request a /c/{slug}/...:

1. Backend resuelve slug -> requested_client_id.
2. Backend valida JWT y extrae token_client_id.
3. Si requested_client_id != token_client_id:
     - Revocar la sesión (sessions.revoked_at = NOW()).
     - Limpiar la cookie de refresh token.
     - Retornar 401 + redirigir al login.
4. Si coinciden, continuar.

Excepción única: ADMIN_RUTA en modo plataforma (scope = GLOBAL,
client_id = 0) opera bajo /platform/... no bajo /c/{slug}/...

Durante Vista de Control: token_client_id = Cliente impersonado.
Si el ADMIN_RUTA intenta acceder a /c/{otro-slug}/..., también se
invalida la sesión.
```

---

## 12. Roles y alcance

RUTA maneja cinco roles formales:

```txt
ADMIN_RUTA
ADMIN_CLIENT
OPERATOR_CLIENT
BUYER
COURIER
```

### 12.1 ADMIN_RUTA

```txt
- client_id: 0
- scope: GLOBAL (modo plataforma) o CLIENT (Vista de Control)
- Funciones:
  * Alta, baja, suspensión y reactivación de Clientes.
  * Cambio de client_type y frontend_mode de un Cliente.
  * Creación del primer ADMIN_CLIENT de cada Cliente nuevo.
  * Configuración inicial de proveedores de pago del Cliente
    (opcional; el ADMIN_CLIENT también puede hacerlo después).
  * Consulta de métricas agregadas globales.
  * Operación dentro de cualquier Cliente vía Vista de Control.
  * Eliminación definitiva de datos de Clientes desactivados.
```

### 12.2 ADMIN_CLIENT

```txt
- client_id: > 0
- scope: CLIENT
- Funciones:
  * Gestiona usuarios internos de su Cliente.
  * Gestiona catálogo (en Tipo B; opcional en Tipo A).
  * Gestiona proveedores de pago de su Cliente.
  * Configura suscripciones de webhooks salientes.
  * Genera y rota API keys del Cliente.
  * Consulta y administra pedidos, pagos, entregas, devoluciones,
    reembolsos (según aplique al tipo).
  * Accede al dashboard.
- Ingreso: /c/{slug}/admin/login (independiente del frontend_mode).
- Creación: el primer ADMIN_CLIENT lo crea ADMIN_RUTA.
```

### 12.3 OPERATOR_CLIENT

```txt
- client_id: > 0
- scope: CLIENT
- Funciones: subset de ADMIN_CLIENT, según permisos asignados.
- Ingreso: mismo endpoint que ADMIN_CLIENT; el rol se determina al
  consultar users.
```

### 12.4 BUYER

```txt
- client_id: > 0
- scope: CLIENT
- actor_type: BUYER
- actor_id: igual a users.id

Diferencias por tipo de Cliente:

Tipo A:
  - auth_mode = EXTERNAL_REFERENCE.
  - Sin password_hash. Nunca se loguea en RUTA.
  - Solo existe como entidad para trazabilidad de pedidos.

Tipo B:
  - auth_mode = PASSWORD.
  - Se loguea normalmente y opera la UI (nativa o custom).
  - Puede ver sus pedidos, solicitar devoluciones, etc.
```

### 12.5 COURIER

```txt
- client_id: > 0
- scope: CLIENT
- actor_type: COURIER
- auth_mode = PASSWORD (independiente del tipo de Cliente).
- Funciones:
  * Consultar pedidos asignados a él dentro del Cliente.
  * Actualizar estados de entrega permitidos por el flujo.
  * Registrar novedades.
  * Reportar pagos contra entrega (con evidencia).
- Ingreso: /c/{slug}/courier/login (siempre en ruta.com).
- Misma persona trabajando para varios Clientes = varios users
  independientes.
```

---

## 13. Estrategia de autenticación

Para sesiones humanas: **JWT corto + refresh token revocable en servidor**.
Para integraciones server-to-server: **API keys con scopes** (sección 14).

```txt
No se usan tokens opacos validados contra Redis en cada request en la
primera versión. Esa opción queda reservada para fases posteriores si
se detectan módulos que requieran revocación inmediata absoluta.
```

### 13.1 Access token JWT

El access token es un JWT firmado, de corta duración.

**Duraciones por rol:**

```txt
ADMIN_RUTA           : 5 minutos
ADMIN_CLIENT         : 10 minutos
OPERATOR_CLIENT      : 10 minutos
BUYER                : 15 minutos
COURIER              : 15 minutos
```

**Claims:**

```json
{
  "user_id": 1001,
  "role": "COURIER",
  "client_id": 1,
  "actor_id": 1001,
  "actor_type": "COURIER",
  "scope": "CLIENT",
  "session_id": 7890,
  "impersonation": false,
  "impersonator_user_id": null,
  "control_view_session_id": null,
  "iat": 1710000000,
  "exp": 1710000900
}
```

### 13.2 Refresh token

```txt
- Formato: 256 bits aleatorios codificados en base64url (~43 caracteres).
  No es un JWT, es opaco.
- Almacenamiento en cliente: cookie HttpOnly, Secure, SameSite=Strict,
  dominio ruta.com.
- Almacenamiento en servidor: hash del refresh token en sessions.
- Rotación: cada uso genera un nuevo refresh token; el anterior se revoca.
- Duración: 7 días para ADMIN_RUTA, 30 días para los demás.
- Inactividad máxima: 7 días sin uso = revocación automática.
```

### 13.3 Tabla `sessions`

```sql
CREATE TABLE sessions (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  client_id BIGINT NOT NULL,
  user_id BIGINT NOT NULL,
  refresh_token_hash TEXT NOT NULL,
  control_view_session_id BIGINT,
  ip_address INET,
  user_agent TEXT,
  issued_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_used_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL,
  revoked_at TIMESTAMPTZ,
  revocation_reason TEXT,

  CONSTRAINT fk_sessions_user FOREIGN KEY (user_id, client_id)
    REFERENCES users(id, client_id),
  CONSTRAINT fk_sessions_cvs FOREIGN KEY (control_view_session_id)
    REFERENCES control_view_sessions(id)
);

CREATE INDEX idx_sessions_user_active
  ON sessions(client_id, user_id) WHERE revoked_at IS NULL;
CREATE INDEX idx_sessions_refresh_hash
  ON sessions(refresh_token_hash) WHERE revoked_at IS NULL;
```

### 13.4 Ciclo de vida del token

```txt
Login:
1. Usuario envía credenciales.
2. Backend valida usuario, contraseña y status = ACTIVE.
3. Backend genera access_token y refresh_token.
4. Backend guarda hash del refresh_token en sessions.
5. Backend retorna access_token en body, refresh_token en cookie.

Uso normal:
1. Cliente envía access_token en Authorization: Bearer.
2. Backend valida firma, expiración y claims.
3. Backend verifica que sessions.id = token.session_id exista y no
   esté revocada.
4. Backend valida que client_id del token corresponda a la URL.
5. Backend ejecuta la operación.

Renovación:
1. Cliente llama /auth/refresh con la cookie del refresh_token.
2. Backend valida que sessions.refresh_token_hash coincide y no esté
   revocada.
3. Backend genera nuevo access y nuevo refresh.
4. Backend actualiza sessions (rotación obligatoria).
5. Backend retorna nuevos tokens.
6. Reuso del refresh anterior = ataque -> revocar toda la cadena.

Logout:
1. Cliente llama /auth/logout.
2. Backend marca sessions.revoked_at = NOW(), reason = 'LOGOUT'.
3. Backend limpia la cookie del refresh_token.
4. El access_token sigue vivo hasta expiración natural (máx. 15 min).
```

### 13.5 Casos de revocación inmediata del refresh token

```txt
- Logout manual del usuario.
- Desactivación o suspensión del usuario.
- Cambio de rol o permisos.
- Cambio de contraseña.
- Desactivación del Cliente al que pertenece.
- Sospecha de compromiso (acción manual del ADMIN_RUTA).
- Reuso de un refresh token ya rotado.
- Cambio manual de URL del Cliente (slug != client_id del token).
```

### 13.6 Un token = un login

Cada sesión es independiente y el access token está vinculado a un único registro de `sessions`. Si la sesión se revoca, el siguiente intento de refresh fallará y el usuario quedará sin acceso aunque su access_token siga vigente unos minutos más.

---

## 14. API keys server-to-server

Para integraciones server-to-server, no se usan JWT (que son para sesiones humanas). Se usan **API keys** con scopes.

```sql
CREATE TABLE client_api_keys (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  key_id TEXT UNIQUE NOT NULL,         -- prefijo público "rk_live_xxxxx"
  secret_hash TEXT NOT NULL,            -- hash Argon2id del secreto
  scopes TEXT[] NOT NULL,
  name TEXT,                            -- "Production server", "Staging"
  last_used_at TIMESTAMPTZ,
  last_used_ip INET,
  expires_at TIMESTAMPTZ,
  revoked_at TIMESTAMPTZ,
  revocation_reason TEXT,
  created_by_user_id BIGINT,
  created_at TIMESTAMPTZ DEFAULT NOW(),

  PRIMARY KEY (id, client_id),
  CONSTRAINT fk_cak_client FOREIGN KEY (client_id) REFERENCES clients(id)
) PARTITION BY LIST (client_id);

CREATE INDEX idx_cak_key_id_active
  ON client_api_keys(key_id) WHERE revoked_at IS NULL;
```

### 14.1 Uso

Header en cada request:

```txt
Authorization: Bearer rk_live_xxxxxxxxxxxxxxxxxxxxxx
```

- `key_id` (`rk_live_...`) es el prefijo público; se busca en la tabla.
- El secreto completo se compara con `secret_hash` usando Argon2id.
- El `client_id` se deriva de la API key — nunca se acepta de headers o body.

### 14.2 Scopes

Lista controlada de scopes:

```txt
- orders:create
- orders:read
- orders:update
- catalog:read
- catalog:write
- buyers:read
- buyers:write
- payments:read
- webhooks:configure
- reports:read
```

Cada API key tiene un subset. Una key con solo `orders:create` no puede llamar a endpoints de catálogo.

### 14.3 Obligatoriedad por tipo de Cliente

```txt
Tipo A: API keys OBLIGATORIAS (la integración con RUTA es 100% por API).
Tipo B: API keys OPCIONALES (útiles si el Cliente quiere integrar con
        su ERP, BI, sistema contable, etc.).
```

### 14.4 Reglas de seguridad

```txt
- El secreto completo se muestra UNA SOLA VEZ al crear la key, al
  ADMIN_CLIENT que la creó. Después solo se almacena el hash.
- Las API keys deben rotarse periódicamente (recomendado: cada 90 días).
- Las API keys pueden tener fecha de expiración explícita.
- El ADMIN_CLIENT puede revocar una API key en cualquier momento.
- Toda llamada con API key se audita en audit_events con
  actor_type = 'API_KEY' y actor_id = api_key_id.
- Rate limiting por API key (no por usuario), configurable por Cliente.
```

---

## 15. Autorización por rol

Toda request operativa valida:

```txt
1. Token válido y no expirado (o API key válida).
2. Sesión asociada activa (no revocada) [solo para JWT].
3. Rol o scope con permiso para la operación.
4. client_id del token/key corresponde al recurso solicitado.
5. Para BUYER y COURIER: actor_id corresponde a la entidad operada.
6. La operación no cruza fronteras de Cliente.
```

### 15.1 Ejemplos por rol

**ADMIN_CLIENT** (ve todos los pedidos del Cliente):

```sql
SELECT * FROM orders
WHERE client_id = :client_id_from_token;
```

**BUYER** (solo sus propios pedidos):

```sql
SELECT * FROM orders
WHERE client_id = :client_id_from_token
  AND buyer_id = :actor_id_from_token;
```

**COURIER** (solo pedidos asignados):

```sql
SELECT * FROM orders
WHERE client_id = :client_id_from_token
  AND courier_user_id = :actor_id_from_token;
```

**API key con scope `orders:read`** (visibilidad equivalente a ADMIN_CLIENT pero limitada por scopes):

```sql
SELECT * FROM orders
WHERE client_id = :client_id_from_api_key;
```

**ADMIN_RUTA en modo plataforma**:

```sql
SELECT id, business_code, name, status, client_type FROM clients;
```

---

## 16. Modelo de `orders`

Alineado con la separación de dominios de los flujos.

```sql
CREATE TABLE orders (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  buyer_id BIGINT NOT NULL,
  courier_user_id BIGINT,

  -- Origen del pedido: define cómo y por dónde entró
  order_origin TEXT NOT NULL
    CHECK (order_origin IN (
      'BUYER_UI',               -- Tipo B: Comprador en UI nativa o custom de RUTA
      'CORPORATE_MANUAL',       -- Tipo B: Admin/Operator Cliente (flujo 6)
      'RECURRENCE',             -- Tipo B: Generado por recurrencia (flujo 5)
      'FULL_LANDING_API',      -- Tipo B: API desde landing externa, ciclo completo
      'API_LOGISTICS'  -- Tipo A: API desde landing del Cliente, solo logística
    )),

  -- Estados separados por dominio
  order_status TEXT NOT NULL,            -- CHECK con todos los estados de los flujos
  payment_status TEXT NOT NULL,
  refund_status TEXT NOT NULL DEFAULT 'REFUND_NOT_REQUIRED',
  return_status TEXT,
  dispute_status TEXT,

  -- Atributos de bifurcación (BLOQUE 0 de los flujos)
  delivery_type TEXT NOT NULL CHECK (delivery_type IN ('SHIP', 'PICKUP')),
  delivery_carrier_type TEXT
    CHECK (delivery_carrier_type IS NULL OR delivery_carrier_type IN (
      'OWN_FLEET', 'EXTERNAL_COURIER'
    )),
  payment_method TEXT NOT NULL
    CHECK (payment_method IN (
      'ONLINE_AT_ORDER',
      'ELECTRONIC_ON_DELIVERY',
      'CASH_ON_DELIVERY',
      'EXTERNAL_PREPAID'           -- Solo para API_LOGISTICS
    )),
  payment_method_submethod TEXT,
  refund_modality TEXT,
  return_mechanism TEXT,
  buyer_type TEXT NOT NULL DEFAULT 'INDIVIDUAL',
  closure_reason TEXT,

  -- Referencias a proveedor de pago configurado por el Cliente
  payment_provider_id BIGINT,            -- FK a client_payment_providers
  external_payment_reference TEXT,        -- ID de la transacción en el sistema externo

  -- Recurrencia
  recurrence_template_id BIGINT,

  -- Financiero (datos informativos, RUTA no custodia)
  subtotal NUMERIC(12,2) NOT NULL DEFAULT 0,
  total NUMERIC(12,2) NOT NULL DEFAULT 0,
  currency TEXT NOT NULL DEFAULT 'COP',

  -- Concurrencia
  version BIGINT NOT NULL DEFAULT 1,

  -- Timestamps
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  closed_at TIMESTAMPTZ,
  archived BOOLEAN NOT NULL DEFAULT FALSE,
  archived_at TIMESTAMPTZ,

  PRIMARY KEY (id, client_id),
  CONSTRAINT fk_orders_client FOREIGN KEY (client_id) REFERENCES clients(id),
  CONSTRAINT fk_orders_buyer FOREIGN KEY (buyer_id, client_id)
    REFERENCES users(id, client_id),
  CONSTRAINT fk_orders_courier FOREIGN KEY (courier_user_id, client_id)
    REFERENCES users(id, client_id),
  CONSTRAINT fk_orders_payment_provider FOREIGN KEY (payment_provider_id, client_id)
    REFERENCES client_payment_providers(id, client_id),

  -- Coherencia: EXTERNAL_PREPAID solo aplica para API_LOGISTICS
  CONSTRAINT chk_prepaid_origin CHECK (
    payment_method != 'EXTERNAL_PREPAID' OR
    order_origin = 'API_LOGISTICS'
  )
) PARTITION BY LIST (client_id);
```

### 16.1 Entrada por `order_origin`

| `order_origin` | Estado de entrada permitido |
|---|---|
| `BUYER_UI` | `DRAFT` |
| `CORPORATE_MANUAL` | `DRAFT` con `buyer_type = CORPORATE` |
| `RECURRENCE` | `ORDER_SUBMITTED` |
| `FULL_LANDING_API` | `ORDER_SUBMITTED` |
| `API_LOGISTICS` | `PREPARING`, `READY_TO_SHIP` o `READY_FOR_PICKUP` (lo decide el Cliente en la llamada API) |

El backend valida en la creación que el estado inicial sea compatible con el origen.

### 16.2 Historial de transiciones

```sql
CREATE TABLE order_state_history (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  order_id BIGINT NOT NULL,
  state_dimension TEXT NOT NULL
    CHECK (state_dimension IN (
      'order_status', 'payment_status', 'refund_status',
      'return_status', 'dispute_status'
    )),
  previous_value TEXT,
  new_value TEXT NOT NULL,
  changed_by_user_id BIGINT,
  changed_by_actor_type TEXT,
  reason TEXT,
  metadata JSONB,
  occurred_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (id, client_id),
  CONSTRAINT fk_osh_order FOREIGN KEY (order_id, client_id)
    REFERENCES orders(id, client_id)
) PARTITION BY LIST (client_id);

REVOKE UPDATE, DELETE ON order_state_history FROM application_role;
```

---

## 17. Validación de consistencia entre entidades

Antes de cualquier operación que cruce entidades, el backend debe validar que todas pertenecen al mismo Cliente.

```sql
-- Antes de crear un pedido para un buyer:
SELECT id FROM users
WHERE id = :buyer_id
  AND client_id = :client_id_from_token
  AND user_type = 'BUYER'
  AND status = 'ACTIVE';

-- Antes de asignar un courier:
SELECT id FROM users
WHERE id = :courier_id
  AND client_id = :client_id_from_token
  AND user_type = 'COURIER'
  AND status = 'ACTIVE';
```

**Cruces prohibidos:**

```txt
- Pedido del Cliente 1 con Buyer del Cliente 2.
- Pedido del Cliente 1 con Courier del Cliente 2.
- Payment del Cliente 1 asociado a Pedido del Cliente 2.
- payment_provider_id de un Cliente usado en pedido de otro Cliente.
- Cualquier referencia cruzada entre Clientes.
```

Refuerzo:

```txt
1. Schema: FK compuestas (id, client_id).
2. Aplicación: validaciones explícitas.
3. Base de datos: RLS (sección 22).
```

---

## 18. Pago: rol de RUTA y proveedores configurables

Esta sección desarrolla las implicaciones técnicas del **principio de la sección 2** (rol no financiero de RUTA).

### 18.1 Modelo general

```txt
- Cada Cliente configura sus propios proveedores de pago en
  client_payment_providers (Wompi, datáfonos, transferencia,
  link de pago, QR, etc.).

- Para pago ONLINE_AT_ORDER:
  El Comprador es redirigido a la pasarela/URL del Cliente.
  El dinero va directo a la cuenta del Cliente.
  RUTA recibe un webhook de confirmación técnica (entrada).

- Para pago contra entrega:
  El Repartidor registra el cobro en RUTA con evidencia.
  El dinero pertenece al Cliente; el Repartidor lo entrega físicamente
  o por transferencia según el acuerdo Cliente-Repartidor.
  RUTA solo deja trazabilidad para que el Cliente concilie.

- Para pago EXTERNAL_PREPAID (Tipo A):
  El Cliente ya cobró en su propia plataforma.
  RUTA recibe el pedido con payment_status = PAID.
  RUTA no procesa nada relacionado con pago.
  Opcionalmente se guarda external_payment_reference para conciliación.

- Para reembolsos:
  RUTA NUNCA ejecuta reembolsos.
  El flujo 4 (refund) describe acciones del Cliente.
  RUTA solo actualiza refund_status según las confirmaciones del Cliente.
```

### 18.2 Tabla `client_payment_providers`

```sql
CREATE TABLE client_payment_providers (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  provider_type TEXT NOT NULL
    CHECK (provider_type IN (
      'PAYMENT_GATEWAY',   -- Wompi, PayU, ePayco
      'DATAFONO',           -- Datáfono físico (Gold, Redeban, etc.)
      'BANK_TRANSFER',      -- Cuenta bancaria del Cliente
      'PAYMENT_LINK',       -- Link de pago configurable
      'QR'                  -- QR del banco del Cliente
    )),
  provider_name TEXT NOT NULL,           -- "Wompi", "Gold", "Bancolombia QR"
  display_name TEXT,                      -- visible al Comprador
  config JSONB NOT NULL,                  -- credenciales y URLs del Cliente
  webhook_ingress_path TEXT,              -- path único en RUTA para webhook entrante
  webhook_secret TEXT,                    -- HMAC secret para validar webhooks entrantes
  status TEXT NOT NULL DEFAULT 'ACTIVE'
    CHECK (status IN ('ACTIVE', 'INACTIVE')),
  is_default BOOLEAN DEFAULT FALSE,
  applicable_payment_methods TEXT[] NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (id, client_id),
  CONSTRAINT fk_cpp_client FOREIGN KEY (client_id) REFERENCES clients(id)
) PARTITION BY LIST (client_id);
```

**Notas:**

```txt
- config JSONB contiene credenciales del Cliente para su pasarela
  (no de RUTA). Se almacena cifrado at-rest (extensión pgcrypto o
  cifrado a nivel de columna).
- webhook_ingress_path: path único expuesto por RUTA para recibir
  webhooks del proveedor (ej. /webhooks/incoming/cli-0001/wompi-prod).
- webhook_secret: secret compartido entre el proveedor y RUTA para
  validar firmas HMAC de webhooks entrantes.
- applicable_payment_methods: define qué payment_method del pedido
  puede usar este proveedor.
```

### 18.3 Flujo de pago ONLINE_AT_ORDER

```txt
1. Comprador en la UI (RUTA o landing custom) confirma el pedido.
2. Backend crea el order con payment_status = PENDING_ONLINE_PAYMENT.
3. Backend identifica el client_payment_providers a usar (default o
   seleccionado por el Comprador).
4. Backend retorna al frontend la URL de redirect del proveedor
   (construida con la config del provider y los datos del pedido).
5. El Comprador es redirigido a la pasarela del Cliente.
6. El Comprador paga; el dinero va a la cuenta del Cliente.
7. El proveedor envía un webhook a webhook_ingress_path de RUTA con
   el resultado de la transacción.
8. RUTA valida la firma HMAC con webhook_secret.
9. RUTA actualiza payment_status -> PAID (o PAYMENT_REJECTED_FINAL si
   falló) y dispara el siguiente paso del flujo.
10. El proveedor también redirige al Comprador de vuelta a la URL de
    retorno (configurada por el Cliente) con un parámetro de resultado.
```

### 18.4 Flujo de pago contra entrega (ELECTRONIC_ON_DELIVERY o CASH_ON_DELIVERY)

```txt
1. El pedido entra a PAYMENT_COLLECTION_PENDING o CASH_COLLECTION_PENDING
   en el handoff (flujo 2 o 3, BLOQUE 9).
2. El Repartidor en su app de RUTA registra el cobro:
   - Monto cobrado.
   - Método (datáfono, efectivo, transferencia, etc.).
   - Evidencia: foto del recibo, ID de transacción del datáfono, etc.
   - Novedades si aplica.
3. Para ELECTRONIC: si el cobro es por datáfono y el proveedor envía
   webhook, RUTA correlaciona la confirmación técnica con el registro
   del Repartidor.
4. RUTA actualiza payment_status -> PAYMENT_COLLECTED y avanza el flujo.
5. RUTA deja en payments un registro completo para conciliación:
   - Quién cobró (courier_user_id).
   - Cuándo y cómo.
   - Monto.
   - Estado de la entrega física del dinero al Cliente
     (delivered_to_client_at, opcional, lo registra el Cliente).
```

**Importante**: el dinero recolectado por el Repartidor pertenece al Cliente. RUTA no es responsable de su custodia ni de su entrega al Cliente. RUTA solo deja evidencia.

### 18.5 Flujo de reembolso

RUTA **nunca** transfiere dinero. El flujo 4 (`docs/flujos/flujo_4_refund_completo.md`) describe acciones del Cliente; RUTA solo actualiza estados.

```txt
1. Disparo: cancelación / devolución aprobada -> refund_status = REFUND_PENDING.
2. Bifurcación por refund_modality (STORE_CREDIT o BANK_REFUND).
3. En todos los casos, el ACTOR que ejecuta es el Cliente:
   - STORE_CREDIT: el Cliente acredita un crédito interno.
   - BANK_REFUND interno: el Cliente transfiere desde su banco.
   - BANK_REFUND vía proveedor: el Cliente solicita reembolso a su
     pasarela/datáfono.
4. El Cliente (ADMIN_CLIENT u OPERATOR_CLIENT) marca en RUTA el
   resultado: refund_status = REFUNDED / PARTIALLY_REFUNDED / REFUND_FAILED.
5. RUTA registra evidencia (comprobante, referencia de transferencia,
   ID de operación en pasarela) en metadata.
6. RUTA notifica al Comprador (solo Tipo B con BUYER en RUTA) o emite
   webhook al Cliente (Tipo A) que se encargará de notificar.
```

---

## 19. Webhooks bidireccionales

RUTA expone dos clases de webhooks:

```txt
SALIENTES (RUTA -> Cliente):
  Notifican al Cliente sobre eventos del pedido y de la operación.
  El Cliente configura URLs y eventos a los que se suscribe.

ENTRANTES (proveedor de pago -> RUTA):
  Permiten que la pasarela del Cliente confirme técnicamente un pago
  o un reembolso. Configurados en client_payment_providers.
```

### 19.1 Webhooks salientes (RUTA -> Cliente)

```sql
CREATE TABLE webhook_subscriptions (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  url TEXT NOT NULL,
  events TEXT[] NOT NULL,
  signing_secret TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'ACTIVE'
    CHECK (status IN ('ACTIVE', 'INACTIVE')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),

  PRIMARY KEY (id, client_id),
  CONSTRAINT fk_ws_client FOREIGN KEY (client_id) REFERENCES clients(id)
);

CREATE TABLE webhook_deliveries (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  subscription_id BIGINT NOT NULL,
  event_type TEXT NOT NULL,
  payload JSONB NOT NULL,
  attempt_number INT NOT NULL DEFAULT 1,
  response_status INT,
  response_body TEXT,
  delivered_at TIMESTAMPTZ,
  next_retry_at TIMESTAMPTZ,
  failed_permanently BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (id, client_id)
) PARTITION BY LIST (client_id);
```

**Catálogo inicial de eventos:**

```txt
order.created
order.status_changed
order.delivered
order.closed
payment.collected             (cobro contra entrega registrado)
payment.confirmed             (confirmación técnica recibida del proveedor)
payment.failed
refund.status_changed
return.requested
return.completed
courier.assigned
courier.out_for_delivery
```

**Reglas:**

```txt
- Firma HMAC-SHA256 del payload con signing_secret en header
  X-Ruta-Signature.
- Reintentos exponenciales: 1m, 5m, 15m, 1h, 6h, 24h.
- Tras 6 fallos consecutivos: failed_permanently = TRUE y se emite
  alerta al ADMIN_CLIENT.
- El timeout de cada intento es 10 segundos.
- El Cliente debe responder con HTTP 2xx para confirmar recepción.
```

### 19.2 Webhooks entrantes (proveedor -> RUTA)

Cada `client_payment_providers` tiene un `webhook_ingress_path` único y un `webhook_secret`. El proveedor del Cliente envía POST a esa URL con la confirmación de transacción.

**Ejemplo de path:**

```txt
POST https://ruta.com/api/webhooks/incoming/{client_slug}/{provider_id}
Headers:
  X-Provider-Signature: <HMAC firmado con webhook_secret>
Body: <payload específico del proveedor>
```

**Procesamiento:**

```txt
1. RUTA recibe el POST.
2. Resuelve client_slug -> client_id, busca client_payment_providers
   por (client_id, id).
3. Valida la firma con webhook_secret.
4. Deduplica por (provider, external_event_id) usando
   external_webhook_events.
5. Mapea el payload del proveedor a la estructura interna de RUTA.
6. Actualiza payments.status y orders.payment_status según el resultado.
7. Dispara webhooks salientes correspondientes (payment.confirmed,
   payment.failed).
```

```sql
CREATE TABLE external_webhook_events (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  client_id BIGINT NOT NULL,
  payment_provider_id BIGINT NOT NULL,
  provider_event_id TEXT NOT NULL,
  payload JSONB NOT NULL,
  processed_at TIMESTAMPTZ,
  processing_result TEXT,
  received_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (payment_provider_id, provider_event_id)
);
```

---

## 20. Funcionalidades bloqueadas en Tipo A

Como Tipo A solo usa RUTA para logística, ciertos flujos no aplican y deben rechazarse si alguien intenta dispararlos sobre un pedido con `order_origin = 'API_LOGISTICS'`.

### 20.1 Flujos no aplicables

```txt
- Reembolso (BLOQUE 12 / flujo 4): el Cliente maneja reembolsos en su
  sistema. RUTA no traza refund_status para Tipo A.
- Devolución post-cierre (BLOQUE 14 / flujo 7): el Cliente maneja
  devoluciones.
- Recurrencia (BLOQUE 17 / flujo 5): el Cliente tiene su propio engine.
- Disputas (BLOQUE 15): el Cliente maneja la relación con su Comprador.
- Pedidos corporativos (BLOQUE 18 / flujo 6): irrelevante; los Clientes
  Tipo A manejan su propio modelo de Comprador.
```

### 20.2 Materialización

Un middleware de aplicación valida, antes de cualquier mutación en los flujos bloqueados, que el pedido objetivo no tenga `order_origin = 'API_LOGISTICS'`. Si tiene, retorna HTTP 422 con código `LOGISTICS_ONLY_FEATURE_UNAVAILABLE`.

RUTA **sí** sigue tracking de excepciones logísticas en Tipo A (entrega fallida, paquete perdido, devolución al origen por logística). Estos son parte de la logística, no de la queja del Comprador.

---

## 21. Catálogo de productos por tipo

### 21.1 Tipo B

```txt
- El catálogo es OBLIGATORIO en RUTA.
- Los order_items referencian products por FK.
- El catálogo se gestiona desde la UI del ADMIN_CLIENT o vía API.
- Carga masiva mediante Excel disponible para ADMIN_CLIENT.
```

### 21.2 Tipo A

```txt
- El catálogo en RUTA es OPCIONAL.
- Los order_items guardan datos planos: product_name, sku, quantity,
  unit_price (sin FK a products).
- Si el Cliente quiere reporting por producto en su dashboard de RUTA,
  puede sincronizar su catálogo vía API. Pero no es requerido para
  crear pedidos.
```

---

## 22. Concurrencia e idempotencia

### 22.1 Optimistic locking

Toda tabla con mutaciones críticas (`orders`, `users`, etc.) tiene `version BIGINT NOT NULL DEFAULT 1`. Cada UPDATE incluye la versión esperada.

```sql
UPDATE orders
SET order_status = 'COURIER_ASSIGNED',
    courier_user_id = :courier_id,
    version = version + 1,
    updated_at = NOW()
WHERE id = :order_id
  AND client_id = :client_id
  AND version = :expected_version;
-- 0 filas afectadas -> conflicto; recargar y reintentar o devolver 409.
```

### 22.2 Asignación atómica del repartidor (BLOQUE 6B)

```sql
UPDATE orders
SET order_status = 'COURIER_ASSIGNED',
    courier_user_id = :courier_id,
    version = version + 1,
    updated_at = NOW()
WHERE id = :order_id
  AND client_id = :client_id
  AND order_status = 'AWAITING_COURIER_ASSIGNMENT'
  AND courier_user_id IS NULL;
-- 0 filas -> ya fue asignado por otro operador.
```

### 22.3 Idempotency keys

Todo endpoint que crea o modifica recursos acepta `X-Idempotency-Key`.

```sql
CREATE TABLE idempotency_keys (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,
  user_or_api_key_id BIGINT NOT NULL,
  actor_type TEXT NOT NULL,  -- 'USER' | 'API_KEY'
  idempotency_key TEXT NOT NULL,
  request_hash TEXT NOT NULL,
  response_status INT NOT NULL,
  response_body JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '24 hours'),

  PRIMARY KEY (id, client_id),
  UNIQUE (client_id, actor_type, user_or_api_key_id, idempotency_key)
) PARTITION BY LIST (client_id);
```

```txt
- Endpoints de mutación requieren X-Idempotency-Key.
- Misma key + mismo request_hash -> devolver respuesta cacheada.
- Misma key + request_hash DIFERENTE -> HTTP 409.
- Expiración: 24 horas.
```

### 22.4 Idempotencia hacia proveedores externos

Toda llamada de RUTA a proveedores externos (pasarelas, datáfonos) envía una clave de idempotencia generada por RUTA, para evitar doble cobro o doble reembolso ante reintentos del Cliente.

---

## 23. Auditoría: tabla `audit_events`

Toda acción significativa se registra append-only.

```sql
CREATE TABLE audit_events (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY,
  client_id BIGINT NOT NULL,

  actor_user_id BIGINT,
  actor_api_key_id BIGINT,
  actor_type TEXT,
  actor_role TEXT,
  acting_via_control_view BOOLEAN NOT NULL DEFAULT FALSE,
  impersonator_user_id BIGINT,
  control_view_session_id BIGINT,

  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id BIGINT,

  ip_address INET,
  user_agent TEXT,
  metadata JSONB,
  result TEXT NOT NULL CHECK (result IN ('SUCCESS', 'FAILURE', 'DENIED')),

  occurred_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  PRIMARY KEY (id, client_id)
) PARTITION BY LIST (client_id);

REVOKE UPDATE, DELETE ON audit_events FROM application_role;
```

**Acciones auditadas obligatoriamente:**

```txt
- Login (exitoso y fallido) de usuarios humanos.
- Uso de API keys (request, scope verificado, resultado).
- Logout, cambio de contraseña, cambio de rol/permisos.
- Creación, suspensión, desactivación, reactivación de Clientes.
- Cambio de client_type o frontend_mode.
- Activación y cierre de Vista de Control.
- Todas las acciones durante Vista de Control.
- Cambios de estado en orders.
- Asignación, desasignación, rechazo de repartidor.
- Registros de cobro por Repartidor con evidencia.
- Recepción y procesamiento de webhooks entrantes.
- Disparos y resultados de webhooks salientes.
- Cambios en client_payment_providers.
- Creación, expiración y revocación de API keys.
- Eliminación definitiva de datos.
- Operaciones DENIED por autorización.
```

---

## 24. Política de desactivación de Clientes

### 24.1 Desactivación (soft)

Cuando un ADMIN_RUTA desactiva un Cliente:

```txt
1. clients.status = 'INACTIVE', clients.updated_at = NOW().
2. Pedidos en curso se cancelan en cascada:
   - Orders con order_status no terminal -> CANCELLED_BY_ADMIN -> CLOSED,
     closure_reason = CLIENT_DEACTIVATED.
   - Si payment_status = PAID y order_origin != API_LOGISTICS,
     se dispara refund_status = REFUND_PENDING (el Cliente lo ejecutará
     externamente cuando reactive operación).
3. Usuarios BUYER: status = 'INACTIVE'. Sesiones revocadas.
4. Usuarios COURIER: SE CONSERVAN con status = 'ACTIVE' porque pueden
   trabajar para otros Clientes. Sus sesiones de este Cliente se revocan.
5. Usuarios ADMIN_CLIENT y OPERATOR_CLIENT: status = 'INACTIVE'.
6. API keys del Cliente: revoked_at = NOW().
7. Webhook_subscriptions: status = 'INACTIVE' (no se dispararán salientes).
8. Catálogo, pedidos históricos, payments: SE CONSERVAN.
9. Acción auditada como CLIENT_DEACTIVATION.
```

### 24.2 Reactivación

```txt
1. clients.status = 'ACTIVE'.
2. Los usuarios INACTIVE permanecen así hasta reactivación manual.
3. Los pedidos cancelados NO se reabren.
4. Las API keys revocadas NO se restauran; deben crearse nuevas.
5. Acción auditada como CLIENT_REACTIVATION.
```

### 24.3 Eliminación definitiva (hard delete)

Solo ADMIN_RUTA, solo sobre Clientes INACTIVE:

```txt
1. Flujo manual desde panel /platform/...
2. ADMIN_RUTA selecciona Cliente inactivo.
3. Sistema muestra resumen: cantidad de usuarios, pedidos, payments, etc.
4. ADMIN_RUTA debe escribir el slug exacto del Cliente para confirmar.
5. Re-confirmación con la contraseña maestra de Vista de Control.
6. Job asíncrono:
   a) DROP TABLE de todas las particiones del Cliente.
   b) DELETE FROM clients WHERE id = <client_id>.
   c) audit_events del Cliente eliminado SE CONSERVAN indefinidamente
      como evidencia legal.
7. Acción auditada como CLIENT_PERMANENT_DELETION con detalle del impacto.
```

---

## 25. Row Level Security (RLS)

### 25.1 Activación

```sql
ALTER TABLE users                ENABLE ROW LEVEL SECURITY;
ALTER TABLE buyer_profiles       ENABLE ROW LEVEL SECURITY;
ALTER TABLE courier_profiles     ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders               ENABLE ROW LEVEL SECURITY;
ALTER TABLE order_state_history  ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_events         ENABLE ROW LEVEL SECURITY;
ALTER TABLE client_payment_providers ENABLE ROW LEVEL SECURITY;
ALTER TABLE client_api_keys      ENABLE ROW LEVEL SECURITY;
-- (todas las tablas operativas)
```

### 25.2 Contexto

Antes de cada query autenticada, el backend establece:

```sql
SET LOCAL app.current_client_id = '1';
SET LOCAL app.current_user_role = 'COURIER';
```

Si el contexto no se establece, las políticas RLS deniegan acceso por defecto.

### 25.3 Política general por Cliente

```sql
CREATE POLICY tenant_isolation
ON orders FOR ALL
USING (
  client_id = current_setting('app.current_client_id')::BIGINT
);
```

### 25.4 Política para ADMIN_RUTA

```sql
CREATE POLICY admin_ruta_global_access
ON orders FOR ALL
USING (
  current_setting('app.current_user_role') = 'ADMIN_RUTA'
);
```

Durante Vista de Control, el contexto se establece como `current_client_id = <cliente_impersonado>` y `current_user_role = 'ADMIN_CLIENT'`, de modo que aplica la política normal de tenant.

### 25.5 Reglas obligatorias

```txt
- RLS NUNCA debe bypass-earse en sesiones de usuario normales.
- El bypass de RLS (BYPASSRLS) se reserva exclusivamente para:
  * Jobs internos del backend (limpieza, reportes globales).
  * Tareas administrativas controladas y auditadas.
  * Service keys que nunca se exponen al frontend.
- application_role NO tiene BYPASSRLS.
- background_jobs_role tiene BYPASSRLS pero solo se usa desde procesos
  del servidor, nunca desde requests HTTP.
```

---

## 26. Particionamiento por `client_id`

### 26.1 Estrategia

```txt
LIST partitioning por client_id.
Una partición por Cliente activo.
Creación automática al insertar un Cliente nuevo.
Eliminación al hard-delete del Cliente.
Horizonte: máximo ~500 Clientes.
```

### 26.2 Tablas particionadas

```txt
- users
- buyer_profiles
- courier_profiles
- orders
- order_state_history
- payments
- deliveries
- returns
- refunds
- products
- order_items
- idempotency_keys
- audit_events
- client_payment_providers
- client_api_keys
- webhook_deliveries
```

### 26.3 Tablas NO particionadas

```txt
- clients
- sessions (queries por refresh_token_hash sin contexto)
- control_view_sessions
- control_view_master_passwords
- external_webhook_events
- webhook_subscriptions (volumen bajo)
- Tablas de configuración global.
```

### 26.4 Creación automática

```sql
CREATE OR REPLACE FUNCTION create_client_partitions(p_client_id BIGINT)
RETURNS VOID AS $$
DECLARE
  partitioned_tables TEXT[] := ARRAY[
    'users', 'buyer_profiles', 'courier_profiles',
    'orders', 'order_state_history', 'payments',
    'deliveries', 'returns', 'refunds', 'products',
    'order_items', 'idempotency_keys', 'audit_events',
    'client_payment_providers', 'client_api_keys',
    'webhook_deliveries'
  ];
  t TEXT;
BEGIN
  FOREACH t IN ARRAY partitioned_tables LOOP
    EXECUTE format(
      'CREATE TABLE IF NOT EXISTS %I_p%s PARTITION OF %I FOR VALUES IN (%L)',
      t, p_client_id, t, p_client_id
    );
  END LOOP;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION trg_create_client_partitions()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM create_client_partitions(NEW.id);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER auto_create_partitions
AFTER INSERT ON clients
FOR EACH ROW
EXECUTE FUNCTION trg_create_client_partitions();

SELECT create_client_partitions(0);
```

---

## 27. Índices obligatorios

```sql
-- orders
CREATE INDEX idx_orders_client_status         ON orders (client_id, order_status);
CREATE INDEX idx_orders_client_buyer_status   ON orders (client_id, buyer_id, order_status);
CREATE INDEX idx_orders_client_courier        ON orders (client_id, courier_user_id);
CREATE INDEX idx_orders_client_created_desc   ON orders (client_id, created_at DESC);
CREATE INDEX idx_orders_client_payment_status ON orders (client_id, payment_status);
CREATE INDEX idx_orders_client_origin         ON orders (client_id, order_origin);
CREATE INDEX idx_orders_client_refund_status  ON orders (client_id, refund_status)
  WHERE refund_status != 'REFUND_NOT_REQUIRED';

-- users
CREATE INDEX idx_users_client_email           ON users (client_id, email);
CREATE INDEX idx_users_client_type_status     ON users (client_id, user_type, status);
CREATE INDEX idx_users_client_external_buyer  ON users (client_id, external_buyer_id)
  WHERE external_buyer_id IS NOT NULL;

-- audit_events
CREATE INDEX idx_audit_client_occurred_desc   ON audit_events (client_id, occurred_at DESC);
CREATE INDEX idx_audit_client_actor           ON audit_events (client_id, actor_user_id, occurred_at DESC);
CREATE INDEX idx_audit_client_entity          ON audit_events (client_id, entity_type, entity_id);

-- client_api_keys
CREATE INDEX idx_cak_key_id_active            ON client_api_keys (key_id) WHERE revoked_at IS NULL;
```

---

## 28. Seguridad en frontend

```txt
- El slug del Cliente se usa solo para construir URLs y mostrar branding.
- El frontend NUNCA confía en el client_id de la URL como fuente de verdad.
- El frontend NUNCA envía client_id en body o query params confiables;
  el backend lo ignora si llega.
- Los tokens NUNCA se guardan en localStorage. Access token en memoria,
  refresh token en cookie HttpOnly Secure SameSite=Strict.
- Cambio manual de URL a otro slug -> backend detecta desalineamiento
  (sección 11.3) y revoca la sesión.
- El frontend NUNCA permite cambiar de Cliente sin pasar por logout completo.
```

---

## 29. Seguridad en base de datos

```txt
- Toda tabla operativa tiene client_id BIGINT NOT NULL.
- FK compuestas (id, client_id) en lugar de FK simples.
- Particionamiento LIST por client_id.
- Índices prefijados por client_id.
- RLS activo con política tenant_isolation.
- Política admin_ruta_global_access solo cuando el contexto lo justifica.
- Tablas append-only con UPDATE y DELETE revocados:
  audit_events, order_state_history, external_webhook_events,
  webhook_deliveries.
- BYPASSRLS solo en roles internos para jobs.
- Contraseñas hasheadas con Argon2id (o bcrypt cost >= 12 como mínimo).
- Credenciales de proveedores de pago en client_payment_providers.config
  cifradas at-rest (pgcrypto o equivalente).
- Webhook secrets nunca expuestos en logs ni en respuestas API.
```

---

## 30. Resumen de decisiones obligatorias

```txt
PRINCIPIO FUNDACIONAL
- RUTA es sistema de información operacional, no infraestructura
  financiera. RUTA no custodia dinero.

ARQUITECTURA
- Multi-tenant con una sola base de datos.
- Separación lógica por client_id BIGINT NOT NULL.
- Identificadores internos BIGINT; slug solo para URLs.

TIPOS DE CLIENTE
- API: solo logística. Cliente con landing propia. Entrada por API
  en PREPARING / READY_TO_SHIP / READY_FOR_PICKUP.
- FULL: ciclo completo en RUTA. Frontend puede ser nativo de RUTA o
  landing custom hecha por equipo RUTA.

USUARIOS
- Tabla única users con user_type discriminador.
- ADMIN_RUTA tiene client_id = 0.
- BUYER y COURIER 100% independientes por Cliente.
- auth_mode: PASSWORD (login en RUTA) o EXTERNAL_REFERENCE (solo identidad).

ADMIN_RUTA
- Vista de Control para soporte con contraseña maestra.
- Sin duración máxima, sin notificación al Admin Cliente.
- Permiso explícito can_use_control_view.

AUTENTICACIÓN
- JWT corto (5-15 min según rol) + refresh token revocable.
- Refresh token en cookie HttpOnly Secure SameSite=Strict en ruta.com.
- Rotación obligatoria de refresh tokens; reuso = ataque.
- Un token = un login. Cambio de URL invalida la sesión.
- API keys server-to-server con scopes para integraciones.

PEDIDOS
- 5 columnas de estado separadas alineadas con docs/flujos/.
- order_origin define cómo entró el pedido y qué flujos aplican.
- Optimistic locking con columna version.
- Idempotency keys obligatorias en endpoints de mutación.

PAGOS
- Cada Cliente configura sus pasarelas en client_payment_providers.
- Pago online: redirect a pasarela del Cliente; webhook entrante a RUTA.
- Pago contra entrega: Repartidor registra con evidencia.
- EXTERNAL_PREPAID solo para API: orden entra con payment_status = PAID.
- Reembolsos: RUTA solo registra; el Cliente ejecuta.

WEBHOOKS
- Salientes (RUTA -> Cliente) firmados HMAC-SHA256, reintentos exponenciales.
- Entrantes (proveedor -> RUTA) validados con webhook_secret del provider.

AUDITORÍA
- audit_events append-only.
- Auditoría obligatoria de Vista de Control, uso de API keys, todas
  las acciones críticas.

DESACTIVACIÓN
- Cancela pedidos en curso, desactiva Buyers y Admin/Operator,
  conserva Couriers (pueden trabajar para otros).
- Hard delete por ADMIN_RUTA con doble confirmación via DROP de
  particiones.

SEGURIDAD
- RLS activo en todas las tablas operativas.
- Particionamiento LIST por client_id con creación automática (~500 Clientes).
- Naming en inglés. Slug solo para URLs.
```

---

## 31. Principio final

RUTA debe comportarse como una plataforma única compartida por múltiples Clientes, pero cada Cliente debe operar como si tuviera su propio entorno privado, aislado y seguro.

RUTA es **sistema de información operacional**, no infraestructura financiera. Toda decisión técnica debe respetar este límite.

La regla central de la arquitectura es:

```txt
Todo dato operativo pertenece a un Cliente.
Todo acceso operativo se valida por client_id.
client_id siempre es BIGINT.
Un token = un login = un Cliente.
RUTA traza, no custodia dinero.

La separación se refuerza en cuatro capas:
  1. Aplicación (filtros explícitos por client_id, scopes de API keys).
  2. Base de datos (FK compuestas, particionamiento).
  3. Row Level Security (políticas por contexto).
  4. Auditoría (todo intento de cruce queda registrado).
```
