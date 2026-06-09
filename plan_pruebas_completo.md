# PLAN DE PRUEBAS — RUTA MVP COMPLETO

**Fecha:** 2026-06-04
**Versión:** 2.0
**Autor:** QA Senior / Analista Funcional RUTA
**Proyecto:** RUTA — SaaS multi-tenant gestión de pedidos/logística
**URLs Producción:**
- Admin: https://ruta-admin.onrender.com
- API: https://ruta-orko-api.onrender.com
- Storefront: https://ruta-by-orko.onrender.com

---

## 0. Resumen Ejecutivo

### Objetivo del plan

Verificar de manera exhaustiva que el MVP Fase 1 de RUTA funciona correctamente en todos sus módulos funcionales antes del deploy a producción y del onboarding del cliente piloto. El plan cubre pruebas funcionales, de integración, de seguridad, de permisos por rol y de experiencia de usuario.

### Alcance — qué se prueba

- Autenticación y gestión de sesiones para los 4 roles staff (ADMIN_RUTA, ADMIN_CLIENT, OPERATOR_CLIENT, COURIER).
- Gestión de Clientes/Tenants por ADMIN_RUTA.
- Catálogo de productos (creación individual, edición, estados, importación masiva Excel).
- Gestión de Compradores (BUYERS): listado, detalle, filtros.
- Gestión de Repartidores (COURIERS): CRUD, activación.
- Pickup Points: CRUD, activación.
- Flujo completo de pedidos SHIP: desde DRAFT hasta CLOSED (incluyendo cancelaciones, COD, Wompi).
- Flujo completo de pedidos PICKUP: desde DRAFT hasta CLOSED.
- Vista del Repartidor (móvil-first): gestión de entregas, cobro COD, evidencias.
- Configuración del Cliente: 4 tabs (info, Wompi, webhooks salientes, parámetros).
- Auditoría y métricas (dashboards).
- Vista de Control (impersonación ADMIN_RUTA → ADMIN_CLIENT).
- Seguridad: aislamiento multi-tenant, control de acceso por rol, manipulación de tokens.
- UI/UX: responsive, estados vacíos, carga, modo claro/oscuro.

### Alcance — qué NO se prueba en este plan

- Flujo de compradores en storefront (front público de compras) — fuera del admin.
- Flujos de devoluciones post-cierre (Flujo 7) — alcance Fase 2.
- Disputas (Flujo 7 / Bloque 15) — alcance Fase 2.
- Recurrencia automática de pedidos (Flujo 5) — alcance Fase 2.
- Pruebas de carga / rendimiento (requieren herramientas específicas).
- Pruebas de penetración profesionales (pen test externo).
- Integraciones con ERP externos vía webhooks salientes (solo se prueba la configuración y el envío de prueba).

### Roles que ejecutan las pruebas

| Rol tester | Responsabilidad |
|---|---|
| QA funcional | Ejecución de casos PT-001 a PT-300 |
| QA seguridad | Ejecución FASE M (PT-281 a PT-300) |
| QA UX/Mobile | Ejecución FASE N (PT-301 a PT-320), especialmente vista COURIER |
| Dev responsable | Soporte en debugging de defectos bloqueantes |

### Entorno de pruebas requerido

- Instancia de staging dedicada con datos de prueba (no producción).
- BD PostgreSQL con schema completo, particiones y seeds de parámetros.
- Servicio backend corriendo (Express + pg-boss activo).
- Frontend admin Next.js corriendo apuntando al backend de staging.
- Acceso a Wompi sandbox con credenciales de prueba.
- Dispositivos o emuladores: desktop (Chrome/Firefox), tablet (768px), móvil (375px).
- Postman o Bruno para pruebas directas de API.
- DevTools del navegador para inspección de cookies/tokens.

### Criterios de inicio (Entry Criteria)

- [ ] Entorno de staging desplegado y accesible.
- [ ] Seeds de datos ejecutados (`seed_dev_data.sh`).
- [ ] Al menos un usuario por cada rol creado en BD.
- [ ] Credenciales de Wompi sandbox disponibles y configuradas.
- [ ] Plan de pruebas revisado y aprobado por el equipo.
- [ ] Documento de defectos vacío y listo para recibir hallazgos.

---

## 1. Módulos Detectados

| # | Módulo | Descripción |
|---|---|---|
| 1 | **Autenticación y Sesiones** | Login, logout, refresh de tokens, acceso por rol, sesión expirada. Roles: ADMIN_RUTA, ADMIN_CLIENT, OPERATOR_CLIENT, COURIER. |
| 2 | **Dashboard ADMIN_CLIENT** | Métricas del día, acciones rápidas, últimos pedidos. |
| 3 | **Dashboard ADMIN_RUTA** | Métricas globales de plataforma: clientes activos, pedidos totales, ingresos del día. |
| 4 | **Gestión de Clientes / Tenants** | CRUD de clientes (solo ADMIN_RUTA): crear tipo FULL/API, editar, activar/desactivar, listar. |
| 5 | **Vista de Control** | Impersonación auditada de ADMIN_RUTA sobre ADMIN_CLIENT con master password. |
| 6 | **Catálogo — Categorías** | Crear, editar, activar/desactivar categorías del cliente. |
| 7 | **Catálogo — Productos** | Crear, editar, activar/desactivar productos. Upload de imagen. Importación masiva Excel. |
| 8 | **Gestión de Compradores (BUYERS)** | Listar, buscar, ver detalle, activar/desactivar. |
| 9 | **Gestión de Repartidores (COURIERS)** | CRUD de couriers: crear, editar, activar/desactivar, ver métricas básicas. |
| 10 | **Pickup Points** | CRUD de puntos de retiro: crear con mapa, editar, activar/desactivar. |
| 11 | **Lista de Pedidos** | Tabla de pedidos con filtros por estado, payment_status, fecha, búsqueda. Paginación. |
| 12 | **Detalle de Pedido (Admin)** | Vista 360 del pedido: timeline, datos del comprador, resumen, acciones según estado. |
| 13 | **Mapa de Asignación** | Mapa OSM con pedidos geolocalizados, lista de couriers, asignación manual. |
| 14 | **Flujo SHIP** | Ciclo completo: DRAFT → PREPARING → AWAITING_COURIER_ASSIGNMENT → SHIPPED → IN_TRANSIT → OUT_FOR_DELIVERY → ARRIVED_AT_CUSTOMER → DELIVERED → CLOSED. |
| 15 | **Flujo PICKUP** | Ciclo completo: DRAFT → PREPARING → READY_FOR_PICKUP → AT_PICKUP_POINT → CUSTOMER_ARRIVED → IDENTITY_VALIDATED → PICKED_UP → DELIVERED → CLOSED. |
| 16 | **Cobro COD (Contra Entrega)** | Registro de cobro efectivo/electrónico por COURIER. Evidencia fotográfica. |
| 17 | **Pagos Wompi (online)** | Configuración de credenciales, webhook HMAC, actualización de payment_status. |
| 18 | **Vista del Repartidor (COURIER)** | Interfaz mobile-first: mis pedidos, detalle, acciones de estado, cobro, evidencias. |
| 19 | **Configuración del Cliente** | 4 tabs: Información corporativa, Wompi, Webhooks salientes, Parámetros. |
| 20 | **Auditoría** | Log de eventos por cliente: filtros por usuario, acción, fecha. |
| 21 | **Seguridad y Permisos** | Aislamiento multi-tenant, acceso 403 por rol, manipulación de JWT. |
| 22 | **UI/UX y Responsive** | Layout desktop/tablet/móvil, modo claro/oscuro, estados vacíos y de carga. |

---

## 2. Configuración del Entorno de Pruebas

### URL de la aplicación

| Componente | URL de Staging |
|---|---|
| Frontend Admin | `https://admin.staging.ruta.com` |
| Backend API | `https://api.staging.ruta.com` |
| Vista Courier | `https://admin.staging.ruta.com/courier` |
| Vista RUTA Admin | `https://admin.staging.ruta.com/ruta-admin` |

### Credenciales de prueba por rol

Crear los siguientes usuarios en la BD de staging antes de iniciar:

| Rol | Email | Contraseña | Cliente |
|---|---|---|---|
| ADMIN_RUTA | `admin.ruta@test.ruta.com` | `TestRuta2026!` | N/A (plataforma) |
| ADMIN_CLIENT | `admin@cliente-piloto.com` | `TestAdmin2026!` | cliente-piloto |
| OPERATOR_CLIENT | `operador@cliente-piloto.com` | `TestOper2026!` | cliente-piloto |
| COURIER | `repartidor@cliente-piloto.com` | `TestCour2026!` | cliente-piloto |
| ADMIN_CLIENT (cliente2) | `admin@cliente-dos.com` | `TestAdmin2026!` | cliente-dos |

**Nota sobre master password Vista de Control:** configurar `master_password` en tabla `control_view_master_passwords` para el usuario ADMIN_RUTA de prueba. Valor de prueba: `MasterTest2026!`.

### Datos de prueba necesarios

Ejecutar antes de iniciar las pruebas:

| Dato | Cantidad mínima | Descripción |
|---|---|---|
| Clientes (tenants) | 2 | `cliente-piloto` (FULL, NATIVE_RUTA) y `cliente-dos` (FULL) |
| Categorías | 3 | "Bebidas", "Alimentos", "Otros" — en cliente-piloto |
| Productos activos | 10 | Precio entre $10.000 y $100.000 COP, con stock > 0 |
| Productos inactivos | 2 | Para pruebas de filtros |
| Couriers activos | 3 | Para pruebas de asignación en mapa |
| Couriers inactivos | 1 | Para prueba de filtro |
| Compradores (BUYERS) | 5 | Con direcciones de entrega en Bogotá |
| Pickup Points activos | 2 | Con coordenadas válidas en Bogotá |
| Pickup Points inactivos | 1 | Para prueba de filtros |
| Pedidos en distintos estados | 15+ | Distribuidos en estados clave para pruebas |

### Credenciales Wompi Sandbox

| Campo | Valor |
|---|---|
| Public key sandbox | `pub_stagtest_...` (obtener de cuenta Wompi sandbox) |
| Private key sandbox | `prv_stagtest_...` |
| Events secret | `stagtest_events_...` |
| URL webhook entrante | Generada automáticamente por RUTA en la tab Wompi |

### Herramientas requeridas

| Herramienta | Uso |
|---|---|
| Google Chrome (última versión) | Pruebas funcionales principales |
| Firefox | Pruebas de compatibilidad |
| Chrome DevTools | Inspección de cookies, network, consola |
| Postman o Bruno | Pruebas directas de API (seguridad, edge cases) |
| Simulador iOS / Android o Chrome DevTools Mobile | Pruebas responsive y vista COURIER |
| ngrok o equivalente | Para recibir webhooks de Wompi en staging local |

---

## 3. Plan de Pruebas Detallado

---

### FASE A — Autenticación y Sesiones (PT-001 a PT-020)

---

| Campo | Valor |
|---|---|
| **ID** | PT-001 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login exitoso como ADMIN_RUTA |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Usuario ADMIN_RUTA creado en BD. Entorno de staging activo. |
| **Datos de prueba** | Email: `alexander.marquez@orko.com.co` / Contraseña: (la del usuario ADMIN_RUTA creado) |
| **Pasos** | 1. Ir a `http://localhost:3002/login`. 2. Ingresar email. 3. Ingresar contraseña. 4. Hacer clic en "Iniciar sesión". |
| **Resultado esperado** | El sistema redirige a `/ruta-admin/dashboard`. El sidebar muestra opciones de ADMIN_RUTA. |
| **Resultado obtenido** | Login funcional. Redirige a `/ruta-admin/dashboard` (corregido de `/ruta-admin/clients`). |
| **Estado** | ✅ Aprobada |
| **Evidencia requerida** | Captura de pantalla post-login mostrando la ruta `/ruta-admin/dashboard` y el menú lateral. |
| **Observaciones** | **2 bugs corregidos durante la prueba:** (1) `/` no redirigía a `/login` — corregido con client-side redirect. (2) ADMIN_RUTA aterrizaba en `/ruta-admin/clients` en lugar del dashboard — corregido en login REDIRECT map. |
| **Responsable** | Alexander Márquez |
| **Fecha ejecución** | 2026-05-31 |

---

| Campo | Valor |
|---|---|
| **ID** | PT-002 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login exitoso como ADMIN_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Usuario ADMIN_CLIENT creado en BD asociado a cliente-piloto. |
| **Datos de prueba** | Email: `admin@cliente-piloto.com` / Contraseña: `TestAdmin2026!` |
| **Pasos** | 1. Ir a `/login`. 2. Ingresar email. 3. Ingresar contraseña. 4. Clic en "Iniciar sesión". |
| **Resultado esperado** | Redirige a `/admin/dashboard`. Sidebar muestra módulos de ADMIN_CLIENT. No hay acceso a `/ruta-admin`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de pantalla del dashboard. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-003 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login exitoso como OPERATOR_CLIENT |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Usuario OPERATOR_CLIENT creado en BD. |
| **Datos de prueba** | Email: `operador@cliente-piloto.com` / Contraseña: `TestOper2026!` |
| **Pasos** | 1. Ir a `/login`. 2. Ingresar email y contraseña. 3. Clic en "Iniciar sesión". |
| **Resultado esperado** | Redirige a `/admin/dashboard`. El sidebar muestra solo los módulos permitidos (sin Configuración si no tiene permiso). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de pantalla del dashboard y del sidebar. |
| **Observaciones** | Comparar sidebar contra ADMIN_CLIENT para verificar restricciones. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-004 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login exitoso como COURIER |
| **Rol** | COURIER |
| **Precondiciones** | Usuario COURIER creado en BD asociado a cliente-piloto. |
| **Datos de prueba** | Email: `repartidor@cliente-piloto.com` / Contraseña: `TestCour2026!` |
| **Pasos** | 1. Abrir el navegador en modo móvil (375px). 2. Ir a `/login`. 3. Ingresar email y contraseña. 4. Clic en "Iniciar sesión". |
| **Resultado esperado** | Redirige a `/courier`. Interfaz mobile-first. No hay sidebar de admin. Solo visible la lista de pedidos asignados al courier. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de pantalla de la vista `/courier` en mobile. |
| **Observaciones** | El JWT del COURIER tiene 30 min de duración (vs 15 min de admin). |
| **Responsable** | QA funcional / QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-005 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login fallido — credenciales incorrectas |
| **Rol** | Cualquiera |
| **Precondiciones** | Ninguna especial. |
| **Datos de prueba** | Email: `admin@cliente-piloto.com` / Contraseña: `ClaveIncorrecta123!` |
| **Pasos** | 1. Ir a `/login`. 2. Ingresar email válido y contraseña incorrecta. 3. Clic en "Iniciar sesión". |
| **Resultado esperado** | El sistema muestra un mensaje de error genérico (sin revelar si el email existe). No se crea sesión. No se establece ninguna cookie de token. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error en pantalla. |
| **Observaciones** | El mensaje no debe decir "contraseña incorrecta" ni "usuario no existe" — debe ser ambiguo por seguridad. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-006 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login fallido — usuario inactivo/suspendido |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Tener un usuario ADMIN_CLIENT con estado `inactive` en BD. |
| **Datos de prueba** | Email del usuario inactivo / Contraseña correcta. |
| **Pasos** | 1. Ir a `/login`. 2. Ingresar credenciales del usuario inactivo. 3. Clic en "Iniciar sesión". |
| **Resultado esperado** | El sistema retorna error. No se establece sesión. El mensaje informa que la cuenta no está activa o similar. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error. Verificar en DevTools que no hay cookies. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-007 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Bloqueo por múltiples intentos fallidos |
| **Rol** | Cualquiera |
| **Precondiciones** | Parámetro `auth.max_failed_login_attempts = 5`. |
| **Datos de prueba** | Email válido / Contraseñas incorrectas (repetir 5+ veces). |
| **Pasos** | 1. Ir a `/login`. 2. Ingresar email válido y contraseña incorrecta. 3. Repetir el paso 2 exactamente 5 veces. 4. En el 6to intento, ingresar la contraseña correcta. |
| **Resultado esperado** | Después del 5to intento fallido, el sistema bloquea la cuenta durante 30 minutos. El 6to intento (con contraseña correcta) retorna error de cuenta bloqueada. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de bloqueo en el 6to intento. |
| **Observaciones** | Parámetro: `auth.lockout_duration_minutes = 30`. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-008 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Logout exitoso |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Hacer clic en el menú de usuario en la esquina superior derecha. 3. Hacer clic en "Cerrar sesión". |
| **Resultado esperado** | Redirige a `/login`. Las cookies `access_token` y `refresh_token` son eliminadas del navegador. Al intentar navegar a `/admin/dashboard` redirige a `/login`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de DevTools → Cookies mostrando que los tokens fueron eliminados. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-009 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Sesión expirada — refresh automático |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. Configurar `auth.jwt_lifetime_admin_client_minutes = 15`. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Esperar 15 minutos sin actividad (o manipular el token de acceso para que expire). 3. Intentar realizar cualquier acción en la UI (ej. navegar a `/admin/orders`). |
| **Resultado esperado** | El sistema renueva el access token automáticamente usando el refresh token (sin que el usuario lo note) y la navegación continúa sin interrupciones. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de DevTools → Network mostrando el request a `/auth/refresh` y la respuesta 200. |
| **Observaciones** | El refresh token tiene 30 días de duración para roles admin. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-010 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Acceso directo a URL protegida sin sesión |
| **Rol** | Ninguno (sin sesión) |
| **Precondiciones** | Navegar en modo incógnito o limpiar cookies. |
| **Datos de prueba** | URL: `/admin/orders` |
| **Pasos** | 1. Abrir navegador en modo incógnito. 2. Navegar directamente a `https://admin.staging.ruta.com/admin/orders`. |
| **Resultado esperado** | El sistema redirige automáticamente a `/login`. No se muestra ningún dato del sistema. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la redirección a `/login`. |
| **Observaciones** | Probar también con rutas: `/admin/dashboard`, `/ruta-admin/clients`, `/courier`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-011 |
| **Módulo** | Autenticación / Control de Acceso |
| **Funcionalidad** | COURIER intenta acceder a módulo de admin (403) |
| **Rol** | COURIER |
| **Precondiciones** | Sesión activa como COURIER. |
| **Datos de prueba** | URL objetivo: `/admin/orders` |
| **Pasos** | 1. Iniciar sesión como COURIER. 2. En la barra de direcciones, navegar manualmente a `https://admin.staging.ruta.com/admin/orders`. |
| **Resultado esperado** | El sistema muestra un mensaje de "Acceso restringido" o redirige a la vista del courier. No se muestran pedidos del admin. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de pantalla del mensaje de error o redirección. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-012 |
| **Módulo** | Autenticación / Control de Acceso |
| **Funcionalidad** | ADMIN_CLIENT intenta acceder a módulo ADMIN_RUTA (403) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | URL objetivo: `/ruta-admin/clients` |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Navegar manualmente a `https://admin.staging.ruta.com/ruta-admin/clients`. |
| **Resultado esperado** | El sistema muestra mensaje de acceso denegado o redirige a `/admin/dashboard`. No se muestran datos de clientes de la plataforma. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta de acceso denegado. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-013 |
| **Módulo** | Autenticación / Control de Acceso |
| **Funcionalidad** | OPERATOR_CLIENT intenta acceder a Configuración sin permiso (403) |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Sesión activa como OPERATOR_CLIENT sin permisos de configuración. |
| **Datos de prueba** | URL objetivo: `/admin/settings` |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT. 2. Navegar a `https://admin.staging.ruta.com/admin/settings`. |
| **Resultado esperado** | El sistema muestra "Acceso restringido" y no renderiza las tabs de configuración. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de pantalla del mensaje de restricción. |
| **Observaciones** | La página de settings valida `session.user_type === 'ADMIN_CLIENT' || session.user_type === 'ADMIN_RUTA'`. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-014 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Refresh token expirado — forzar re-login |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Modificar el refresh token en la BD para que esté expirado o revocar la sesión. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Revocar el refresh token en la BD (`UPDATE sessions SET revoked_at = NOW()`). 3. Esperar a que expire el access token (15 min) o limpiar el access_token. 4. Intentar navegar a cualquier página protegida. |
| **Resultado esperado** | El sistema no puede renovar el token, elimina las cookies y redirige a `/login` con un mensaje apropiado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la redirección a login. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-015 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Rate limiting en endpoint de login |
| **Rol** | Ninguno |
| **Precondiciones** | Postman configurado para enviar múltiples requests. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Usando Postman, enviar 6 requests de POST `/auth/login` en menos de 1 minuto desde la misma IP. |
| **Resultado esperado** | A partir del 6to request, el servidor retorna HTTP 429 Too Many Requests con código `RATE_LIMITED`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 429 en Postman. |
| **Observaciones** | Parámetro: `ratelimit.auth_attempts_per_minute_per_ip = 5`. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-016 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Tokens NO almacenados en localStorage |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Abrir DevTools → Application → Local Storage. 3. Revisar si existen tokens almacenados. 4. Revisar DevTools → Application → Cookies. |
| **Resultado esperado** | No hay ningún token en localStorage ni sessionStorage. Los tokens (`access_token`, `refresh_token`) están en Cookies con flags HttpOnly y Secure. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de DevTools mostrando Cookies (con HttpOnly) y localStorage vacío de tokens. |
| **Observaciones** | Regla del proyecto: "No tokens en localStorage". |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-017 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login con email en formato inválido |
| **Rol** | Cualquiera |
| **Precondiciones** | Ninguna. |
| **Datos de prueba** | Email: `noesunmail` / Contraseña: `TestAdmin2026!` |
| **Pasos** | 1. Ir a `/login`. 2. Ingresar `noesunmail` en el campo email. 3. Ingresar una contraseña. 4. Clic en "Iniciar sesión". |
| **Resultado esperado** | El formulario valida el formato del email y muestra un mensaje de error de validación antes de enviar el request al servidor. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación en el formulario. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-018 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Login con campos vacíos |
| **Rol** | Cualquiera |
| **Precondiciones** | Ninguna. |
| **Datos de prueba** | Dejar ambos campos vacíos. |
| **Pasos** | 1. Ir a `/login`. 2. No ingresar nada en ningún campo. 3. Clic en "Iniciar sesión". |
| **Resultado esperado** | El formulario muestra mensajes de validación para campos obligatorios. No se envía ningún request al servidor. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los mensajes de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-019 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Sesión persistente — refresh token en re-apertura del navegador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Cerrar completamente el navegador (sin logout). 3. Reabrir el navegador y navegar a `https://admin.staging.ruta.com/admin/dashboard`. |
| **Resultado esperado** | El sistema detecta el refresh token en la cookie persistente, renueva el access token y el usuario sigue autenticado sin necesidad de volver a ingresar credenciales. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard activo tras reapertura. |
| **Observaciones** | Aplica solo si las cookies son persistentes (no solo de sesión). |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-020 |
| **Módulo** | Autenticación |
| **Funcionalidad** | Cierre de sesión en múltiples pestañas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa en dos pestañas del mismo navegador. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Abrir una segunda pestaña del mismo navegador y navegar a `/admin/orders`. 3. En la primera pestaña, hacer logout. 4. Volver a la segunda pestaña e intentar navegar a otra página. |
| **Resultado esperado** | Al navegar en la segunda pestaña después del logout, el sistema detecta que la sesión ya no es válida y redirige a `/login`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la redirección en la segunda pestaña. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE B — Gestión de Clientes/Tenants — ADMIN_RUTA (PT-021 a PT-040)

---

| Campo | Valor |
|---|---|
| **ID** | PT-021 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Crear cliente tipo FULL |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Sesión activa como ADMIN_RUTA. Estar en `/ruta-admin/clients`. |
| **Datos de prueba** | Nombre: "Restaurante La Prueba", Slug: "restaurante-la-prueba", Tipo: FULL, Modalidad: NATIVE_RUTA, Responsable: "Carlos García", Email: `carlos@restauranteprueba.com`, Teléfono: +57 300 0000001. |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Hacer clic en "Crear Cliente". 3. Completar el formulario con los datos de prueba. 4. Seleccionar tipo "FULL" y modalidad "NATIVE_RUTA". 5. Hacer clic en "Crear Cliente". |
| **Resultado esperado** | El cliente se crea exitosamente. El sistema redirige al detalle del nuevo cliente o a la lista. El cliente aparece en la tabla con estado "Activo" y tipo "FULL". Las particiones de BD se crean automáticamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del cliente creado en la lista. |
| **Observaciones** | Verificar en BD que se crearon las particiones para el nuevo client_id. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-022 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Crear cliente tipo API |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Sesión activa como ADMIN_RUTA. |
| **Datos de prueba** | Nombre: "LogiTech API", Slug: "logitech-api", Tipo: API. |
| **Pasos** | 1. Ir a `/ruta-admin/clients/new`. 2. Completar el formulario. 3. Seleccionar tipo "API". 4. Hacer clic en "Crear Cliente". |
| **Resultado esperado** | El cliente tipo API se crea exitosamente. No hay campo de "modalidad frontend" visible (solo aplica a FULL). El cliente aparece en la lista con tipo "API". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del formulario con tipo API y captura del cliente en la lista. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-023 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Crear cliente con slug duplicado (validación) |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existe un cliente con slug "cliente-piloto". |
| **Datos de prueba** | Slug: "cliente-piloto" (duplicado). |
| **Pasos** | 1. Ir a `/ruta-admin/clients/new`. 2. Completar el formulario con slug "cliente-piloto". 3. Hacer clic en "Crear Cliente". |
| **Resultado esperado** | El sistema retorna un error de validación indicando que el slug ya está en uso. No se crea el cliente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-024 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Editar información de un cliente |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existe el cliente "cliente-piloto". Sesión activa como ADMIN_RUTA. |
| **Datos de prueba** | Nuevo nombre: "Cliente Piloto Editado". |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Hacer clic en el cliente "cliente-piloto". 3. Hacer clic en "Editar". 4. Cambiar el nombre a "Cliente Piloto Editado". 5. Guardar. |
| **Resultado esperado** | Los datos se actualizan. El nombre nuevo aparece en el detalle y en la lista. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura antes y después de la edición. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-025 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Desactivar un cliente |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existe el cliente "cliente-dos" activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Hacer clic en "cliente-dos". 3. Hacer clic en "Desactivar". 4. Confirmar la acción en el diálogo de confirmación. |
| **Resultado esperado** | El cliente cambia a estado "Inactivo". Los usuarios de ese cliente no pueden iniciar sesión. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del cliente en estado inactivo. |
| **Observaciones** | Verificar que al intentar login como `admin@cliente-dos.com` el sistema rechaza la sesión. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-026 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Reactivar un cliente |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | El cliente "cliente-dos" está inactivo (PT-025 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Hacer clic en "cliente-dos". 3. Hacer clic en "Activar". 4. Confirmar. |
| **Resultado esperado** | El cliente vuelve a estado "Activo". Sus usuarios pueden iniciar sesión nuevamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del cliente en estado activo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-027 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Listar clientes con filtro por tipo |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existen clientes tipo FULL y tipo API en la BD. |
| **Datos de prueba** | Filtro: Tipo = "API". |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Aplicar filtro por tipo "API". |
| **Resultado esperado** | La lista muestra solo los clientes de tipo API. Los clientes FULL no aparecen. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-028 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Listar clientes con filtro por estado |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existen clientes activos e inactivos. |
| **Datos de prueba** | Filtro: Estado = "Inactivo". |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Aplicar filtro por estado "Inactivo". |
| **Resultado esperado** | Solo aparecen clientes inactivos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-029 |
| **Módulo** | Gestión de Clientes |
| **Funcionalidad** | Buscar cliente por nombre |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existen múltiples clientes. |
| **Datos de prueba** | Búsqueda: "piloto". |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. En el buscador, escribir "piloto". |
| **Resultado esperado** | Solo aparecen clientes cuyo nombre contiene "piloto". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado de búsqueda. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-030 |
| **Módulo** | Dashboard ADMIN_RUTA |
| **Funcionalidad** | Ver métricas globales de plataforma |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Sesión activa como ADMIN_RUTA. Existen pedidos en la plataforma. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_RUTA. 2. Ir a `/ruta-admin/dashboard`. |
| **Resultado esperado** | El dashboard muestra: cantidad de clientes activos, pedidos totales del día, ingresos transaccionados del día, pedidos por estado. Los datos son coherentes con la BD. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard con métricas visibles. |
| **Observaciones** | Verificar que ADMIN_CLIENT no puede acceder a esta pantalla. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-031 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Crear OPERATOR_CLIENT (ADMIN_CLIENT crea usuario operador con permisos básicos) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT de cliente-piloto. Módulo de Usuarios disponible. |
| **Datos de prueba** | Nombre: "Luis Operador", Email: `luis.operador@cliente-piloto.com`, Permisos iniciales: ORDERS_VIEW, PRODUCTS_VIEW. |
| **Pasos** | 1. Ir al menú lateral → Usuarios. 2. Hacer clic en "Crear usuario". 3. Seleccionar rol "Operador". 4. Completar nombre y email. 5. Asignar permisos ORDERS_VIEW y PRODUCTS_VIEW. 6. Guardar. |
| **Resultado esperado** | El OPERATOR_CLIENT se crea exitosamente. Aparece en la lista de usuarios del cliente. Recibe email de bienvenida con instrucciones para establecer contraseña. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del usuario creado en la lista con permisos asignados. |
| **Observaciones** | Los permisos disponibles están definidos en `operator_permissions`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-032 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Crear segundo ADMIN_CLIENT para el mismo tenant |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | Nombre: "María Admin", Email: `maria.admin@cliente-piloto.com`. |
| **Pasos** | 1. Ir a Usuarios. 2. Hacer clic en "Crear usuario". 3. Seleccionar rol "Admin". 4. Completar nombre y email. 5. Guardar. |
| **Resultado esperado** | El segundo ADMIN_CLIENT se crea. Tiene acceso completo al panel de administración de cliente-piloto. Puede gestionar configuración, pedidos, productos y usuarios. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del segundo ADMIN_CLIENT en la lista de usuarios. |
| **Observaciones** | Un tenant puede tener múltiples ADMIN_CLIENT. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-033 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Asignar permiso ORDERS_ACCEPT_SELLER a OPERATOR_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe el OPERATOR_CLIENT creado en PT-031. |
| **Datos de prueba** | Permiso a asignar: ORDERS_ACCEPT_SELLER. |
| **Pasos** | 1. Ir a Usuarios. 2. Hacer clic en "Luis Operador". 3. En la sección de permisos, activar el permiso ORDERS_ACCEPT_SELLER. 4. Guardar. |
| **Resultado esperado** | El permiso ORDERS_ACCEPT_SELLER queda activo para el operador. Al iniciar sesión como ese operador, ahora puede aceptar/rechazar pedidos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del panel de permisos con ORDERS_ACCEPT_SELLER activo. |
| **Observaciones** | La asignación de permisos es granular por operador. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-034 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Revocar permiso PRODUCTS_CREATE de OPERATOR_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | El OPERATOR_CLIENT "Luis Operador" tiene el permiso PRODUCTS_CREATE. |
| **Datos de prueba** | Permiso a revocar: PRODUCTS_CREATE. |
| **Pasos** | 1. Ir a Usuarios → hacer clic en "Luis Operador". 2. Desactivar el permiso PRODUCTS_CREATE. 3. Guardar. |
| **Resultado esperado** | El permiso se revoca inmediatamente. Si el operador tiene sesión activa, en la próxima acción de crear producto recibirá HTTP 403. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del panel de permisos sin PRODUCTS_CREATE. Captura del 403 al intentar crear producto. |
| **Observaciones** | El cambio de permisos surte efecto en el siguiente request después de que el token del operador sea renovado. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-035 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Editar perfil propio — cambiar nombre (ADMIN_CLIENT) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | Nuevo nombre: "Admin Cliente Actualizado". |
| **Pasos** | 1. Ir al menú de perfil (esquina superior derecha). 2. Seleccionar "Mi perfil" o equivalente. 3. Cambiar el nombre. 4. Guardar. |
| **Resultado esperado** | El nombre se actualiza. El nuevo nombre aparece en el header del admin y en la lista de usuarios. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del nombre actualizado en el header. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-036 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Cambiar contraseña propia |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | Contraseña actual: `TestAdmin2026!`. Nueva contraseña: `NuevaClaveAdmin2026!`. |
| **Pasos** | 1. Ir al perfil de usuario. 2. Hacer clic en "Cambiar contraseña". 3. Ingresar contraseña actual. 4. Ingresar nueva contraseña. 5. Confirmar nueva contraseña. 6. Guardar. |
| **Resultado esperado** | La contraseña se actualiza. Al cerrar sesión y volver a ingresar, solo funciona la nueva contraseña. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de éxito al cambiar contraseña. Captura del login exitoso con la nueva contraseña. |
| **Observaciones** | La contraseña se hashea con argon2. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-037 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Desactivar OPERATOR_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe "Luis Operador" activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Usuarios. 2. Hacer clic en "Luis Operador". 3. Hacer clic en "Desactivar usuario". 4. Confirmar. |
| **Resultado esperado** | El operador pasa a estado "Inactivo". No puede iniciar sesión ni realizar acciones. Si tenía sesión activa, su próxima acción retorna 401. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del usuario en estado inactivo. Captura del 401 al intentar actuar con sesión expirada del operador. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-038 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Reactivar OPERATOR_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | "Luis Operador" está inactivo (PT-037 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Usuarios. 2. Hacer clic en "Luis Operador". 3. Hacer clic en "Activar usuario". 4. Confirmar. |
| **Resultado esperado** | El operador vuelve a estado "Activo". Puede iniciar sesión y operar con normalidad. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del usuario en estado activo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-039 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | Listar usuarios del cliente (ADMIN_CLIENT ve su propio equipo) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen múltiples usuarios en cliente-piloto. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Usuarios en el menú lateral. |
| **Resultado esperado** | Se muestra la lista de todos los usuarios del cliente-piloto (ADMIN_CLIENT, OPERATOR_CLIENT, COURIER). No aparecen usuarios de otros tenants. Columnas: nombre, email, rol, estado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista de usuarios. |
| **Observaciones** | Aislamiento multi-tenant: solo usuarios del propio cliente. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-040 |
| **Módulo** | Gestión de Usuarios |
| **Funcionalidad** | ADMIN_RUTA crea ADMIN_CLIENT para cualquier tenant |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Sesión activa como ADMIN_RUTA. Existe el tenant "cliente-piloto". |
| **Datos de prueba** | Nombre: "Admin Nuevo", Email: `admin.nuevo@cliente-piloto.com`, Cliente: cliente-piloto. |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Hacer clic en "cliente-piloto". 3. En la sección de usuarios del cliente, hacer clic en "Crear usuario admin". 4. Completar nombre y email. 5. Guardar. |
| **Resultado esperado** | El ADMIN_CLIENT se crea para el tenant especificado. El nuevo usuario recibe instrucciones para establecer contraseña. Solo tiene acceso al tenant "cliente-piloto". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del nuevo ADMIN_CLIENT en la lista de usuarios de cliente-piloto. |
| **Observaciones** | El ADMIN_RUTA puede crear usuarios admin para cualquier tenant desde la Vista de Control. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE C — Catálogo (PT-041 a PT-070)

---

| Campo | Valor |
|---|---|
| **ID** | PT-041 |
| **Módulo** | Catálogo — Categorías |
| **Funcionalidad** | Crear categoría |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT de cliente-piloto. |
| **Datos de prueba** | Nombre de categoría: "Postres Premium". |
| **Pasos** | 1. En el menú lateral, ir a Catálogo → Categorías. 2. Hacer clic en "Crear categoría". 3. Ingresar nombre "Postres Premium". 4. Guardar. |
| **Resultado esperado** | La categoría "Postres Premium" aparece en la lista de categorías con estado "Activo". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la categoría creada en la lista. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-042 |
| **Módulo** | Catálogo — Categorías |
| **Funcionalidad** | Editar categoría existente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe la categoría "Postres Premium". |
| **Datos de prueba** | Nuevo nombre: "Postres y Dulces". |
| **Pasos** | 1. Ir a Catálogo → Categorías. 2. Hacer clic en "Postres Premium". 3. Cambiar el nombre a "Postres y Dulces". 4. Guardar. |
| **Resultado esperado** | El nombre de la categoría se actualiza. La lista muestra "Postres y Dulces". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la categoría editada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-043 |
| **Módulo** | Catálogo — Categorías |
| **Funcionalidad** | Desactivar categoría |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe la categoría "Postres y Dulces" activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Catálogo → Categorías. 2. Usar el interruptor de la fila de "Postres y Dulces" para desactivarla. |
| **Resultado esperado** | La categoría cambia a estado "Inactivo". Los productos de esa categoría siguen existiendo pero la categoría no aparece disponible para asignar nuevos productos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la categoría en estado inactivo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-044 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Crear producto con todos los campos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. Existe al menos una categoría activa. |
| **Datos de prueba** | Nombre: "Torta de Chocolate", Descripción: "Deliciosa torta artesanal", Precio: $35000, Categoría: "Alimentos", Tipo: "PHYSICAL", Imagen: archivo JPG válido, Stock: 50, Estado: Activo. |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Clic en "Crear producto". 3. Completar todos los campos con los datos de prueba. 4. Subir imagen JPG (< 10MB). 5. Clic en "Guardar". |
| **Resultado esperado** | El producto se crea. Aparece en la lista de productos con nombre, precio, categoría y estado "Activo". La imagen se muestra en el detalle del producto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del producto en la lista y en la vista de detalle. |
| **Observaciones** | `storage.max_file_size_mb = 10`, formatos permitidos: jpg, jpeg, png, webp. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-045 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Crear producto con nombre vacío (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: vacío, Precio: $10000. |
| **Pasos** | 1. Ir a Catálogo → Productos → Crear producto. 2. Dejar el campo nombre vacío. 3. Ingresar precio $10000. 4. Clic en "Guardar". |
| **Resultado esperado** | El formulario muestra error de validación "El nombre es obligatorio". No se envía el request al servidor. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-046 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Crear producto con precio negativo (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: "Producto Inválido", Precio: -500. |
| **Pasos** | 1. Ir a Catálogo → Productos → Crear producto. 2. Ingresar nombre "Producto Inválido". 3. Ingresar precio -500. 4. Clic en "Guardar". |
| **Resultado esperado** | El sistema rechaza el precio negativo con un mensaje de validación. No se crea el producto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-047 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Crear producto con imagen en formato inválido |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: "Producto Con PDF", Imagen: archivo .pdf. |
| **Pasos** | 1. Ir a Catálogo → Productos → Crear producto. 2. Ingresar nombre. 3. En el campo de imagen, subir un archivo PDF. 4. Clic en "Guardar". |
| **Resultado esperado** | El sistema rechaza el archivo PDF. Muestra error indicando los formatos permitidos: jpg, jpeg, png, webp. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de formato. |
| **Observaciones** | Parámetro: `storage.allowed_image_formats = ["jpg","jpeg","png","webp"]`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-048 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Editar producto existente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe el producto "Torta de Chocolate". |
| **Datos de prueba** | Nuevo precio: $40000. Nueva descripción: "Torta artesanal de chocolate belga". |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Hacer clic en "Torta de Chocolate". 3. Editar precio a $40000 y descripción. 4. Guardar. |
| **Resultado esperado** | El producto se actualiza. El precio nuevo es $40000. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del producto con el precio actualizado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-049 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Activar / Inactivar producto desde lista |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un producto activo en la lista. |
| **Datos de prueba** | Producto: "Torta de Chocolate". |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. En la fila de "Torta de Chocolate", usar el interruptor de estado para desactivarlo. 3. Verificar el estado en la lista. 4. Volver a activarlo con el mismo interruptor. |
| **Resultado esperado** | El producto cambia de Activo a Inactivo y viceversa sin necesidad de abrir el formulario de edición. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del producto en estado activo e inactivo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-050 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Listar productos con filtro por categoría |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen productos de distintas categorías. |
| **Datos de prueba** | Filtro: Categoría = "Bebidas". |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Usar el filtro de categoría y seleccionar "Bebidas". |
| **Resultado esperado** | Solo se muestran productos de la categoría "Bebidas". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-051 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Listar productos con filtro por estado |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen productos activos e inactivos. |
| **Datos de prueba** | Filtro: Estado = "Inactivo". |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Filtrar por estado "Inactivo". |
| **Resultado esperado** | Solo se muestran productos inactivos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado filtrado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-052 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Búsqueda de producto por nombre |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen múltiples productos. |
| **Datos de prueba** | Búsqueda: "Torta". |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. En el buscador, escribir "Torta". |
| **Resultado esperado** | Solo aparecen productos cuyo nombre contiene "Torta". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado de búsqueda. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-053 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Estado vacío — sin productos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Cliente sin productos (usar cliente de prueba vacío o limpiar productos). |
| **Datos de prueba** | Filtro que retorna 0 resultados, ej. buscar "xyzNoExiste". |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Buscar "xyzNoExiste". |
| **Resultado esperado** | La lista muestra un mensaje de estado vacío ("No hay productos para los filtros seleccionados" o similar). No hay errores en consola. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-054 |
| **Módulo** | Catálogo — Importación masiva |
| **Funcionalidad** | Importar Excel con formato válido |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Archivo Excel de prueba preparado con columnas correctas: nombre, descripcion, precio, categoria_id, stock. |
| **Datos de prueba** | Archivo Excel con 20 filas válidas. |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Clic en "Importar Excel". 3. En el modal, arrastrar el archivo Excel válido. 4. Verificar la vista previa de las primeras filas. 5. Clic en "Importar". |
| **Resultado esperado** | Se muestra vista previa correctamente. Al importar, el sistema procesa los 20 productos y muestra reporte: "20 importados correctamente, 0 errores". Los productos aparecen en la lista. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del reporte de importación y de los productos en la lista. |
| **Observaciones** | `catalog.bulk_import_max_rows = 5000`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-055 |
| **Módulo** | Catálogo — Importación masiva |
| **Funcionalidad** | Importar Excel con filas con errores |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Archivo Excel con mezcla de filas válidas e inválidas. |
| **Datos de prueba** | Archivo Excel con 10 filas válidas y 3 filas con precio vacío o categoría inválida. |
| **Pasos** | 1. Ir a Catálogo → Productos → Importar Excel. 2. Subir el archivo con errores. 3. Verificar la vista previa. 4. Clic en "Importar". |
| **Resultado esperado** | El sistema importa las 10 filas válidas y reporta los 3 errores con descripción de cada uno. Los productos válidos quedan en la BD. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del reporte indicando errores por fila. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-056 |
| **Módulo** | Catálogo — Importación masiva |
| **Funcionalidad** | Importar archivo no-Excel (formato inválido) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Ninguna. |
| **Datos de prueba** | Archivo CSV o TXT con datos. |
| **Pasos** | 1. Ir a Catálogo → Productos → Importar Excel. 2. Intentar subir un archivo CSV. |
| **Resultado esperado** | El sistema rechaza el archivo y muestra error indicando que solo se aceptan archivos Excel (.xlsx o .xls). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de formato. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-057 |
| **Módulo** | Catálogo — Importación masiva |
| **Funcionalidad** | Importar Excel que excede el límite de filas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Archivo Excel con más de 5000 filas preparado. |
| **Datos de prueba** | Archivo con 5001 filas. |
| **Pasos** | 1. Ir a Catálogo → Productos → Importar Excel. 2. Subir el archivo con 5001 filas. |
| **Resultado esperado** | El sistema rechaza el archivo antes de procesar, indicando que el límite es de 5000 filas. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de límite excedido. |
| **Observaciones** | Parámetro: `catalog.bulk_import_max_rows = 5000`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-058 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | OPERATOR_CLIENT con permiso PRODUCTS_CREATE puede crear producto |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | El OPERATOR_CLIENT tiene permiso `PRODUCTS_CREATE` en la tabla `operator_permissions`. |
| **Datos de prueba** | Nombre: "Producto Operador", Precio: $20000. |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT con permiso PRODUCTS_CREATE. 2. Ir a Catálogo → Productos. 3. Clic en "Crear producto". 4. Completar formulario. 5. Guardar. |
| **Resultado esperado** | El producto se crea exitosamente. El OPERATOR_CLIENT tiene acceso al botón "Crear producto". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del producto creado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-059 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | OPERATOR_CLIENT sin permiso PRODUCTS_CREATE no puede crear producto |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | El OPERATOR_CLIENT NO tiene permiso `PRODUCTS_CREATE`. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT sin permiso PRODUCTS_CREATE. 2. Ir a Catálogo → Productos. 3. Verificar si aparece el botón "Crear producto". 4. Si aparece, intentar hacer clic en él. |
| **Resultado esperado** | El botón "Crear producto" no aparece, o al intentar crear el sistema muestra un mensaje de acceso denegado (HTTP 403). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la pantalla sin botón de creación o del mensaje de error. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-060 |
| **Módulo** | Catálogo — Productos |
| **Funcionalidad** | Paginación de lista de productos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Más de 20 productos activos en la BD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Verificar que hay paginación. 3. Hacer clic en "Siguiente" para ir a la página 2. 4. Hacer clic en "Anterior" para volver a la página 1. |
| **Resultado esperado** | La paginación funciona correctamente. La página 2 muestra los siguientes productos. El número de página y total se actualizan. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de página 1 y página 2. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE D — Gestión de Compradores / BUYERS (PT-061 a PT-075)

---

| Campo | Valor |
|---|---|
| **ID** | PT-061 |
| **Módulo** | Compradores |
| **Funcionalidad** | Listar compradores del cliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen al menos 5 BUYERS en cliente-piloto. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Ir al menú lateral → Compradores. |
| **Resultado esperado** | La tabla muestra todos los compradores del cliente. Columnas: nombre, email, fecha de registro, estado. No aparecen compradores de otros clientes. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista de compradores. |
| **Observaciones** | Verificar aislamiento multi-tenant: no deben aparecer compradores de cliente-dos. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-062 |
| **Módulo** | Compradores |
| **Funcionalidad** | Buscar comprador por nombre o email |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen compradores en la BD. |
| **Datos de prueba** | Búsqueda: email parcial "juan". |
| **Pasos** | 1. Ir a Compradores. 2. En el buscador, escribir "juan". |
| **Resultado esperado** | Solo aparecen compradores cuyo nombre o email contiene "juan". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado de búsqueda. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-063 |
| **Módulo** | Compradores |
| **Funcionalidad** | Ver detalle de un comprador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un comprador con historial de pedidos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Hacer clic en un comprador de la lista. |
| **Resultado esperado** | Se muestra el detalle del comprador: nombre, email, teléfono, documento, direcciones guardadas, historial de pedidos. Las acciones "Activar/Desactivar" están disponibles. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle del comprador. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-064 |
| **Módulo** | Compradores |
| **Funcionalidad** | Desactivar un comprador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un comprador activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Hacer clic en un comprador. 3. Hacer clic en "Desactivar". 4. Confirmar. |
| **Resultado esperado** | El comprador pasa a estado "Inactivo". El comprador no puede iniciar sesión ni crear pedidos mientras esté inactivo. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del comprador en estado inactivo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-065 |
| **Módulo** | Compradores |
| **Funcionalidad** | OPERATOR_CLIENT ve la lista de compradores |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | OPERATOR_CLIENT sin restricciones especiales (tiene permiso de ver buyers). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT. 2. Ir a Compradores. |
| **Resultado esperado** | La lista de compradores es visible. El acceso está permitido según la matriz de permisos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista. |
| **Observaciones** | Ver lista de BUYERS del Cliente: OPERATOR_CLIENT tiene ✅. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-066 |
| **Módulo** | Compradores |
| **Funcionalidad** | Filtrar compradores por estado |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen compradores activos e inactivos. |
| **Datos de prueba** | Filtro: Estado = "Inactivo". |
| **Pasos** | 1. Ir a Compradores. 2. Aplicar filtro por estado "Inactivo". |
| **Resultado esperado** | Solo aparecen compradores con estado inactivo. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-067 |
| **Módulo** | Compradores |
| **Funcionalidad** | Reactivar un comprador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un comprador inactivo (PT-064 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Filtrar por inactivos. 3. Hacer clic en el comprador inactivo. 4. Hacer clic en "Activar". 5. Confirmar. |
| **Resultado esperado** | El comprador vuelve a estado "Activo". Puede iniciar sesión y crear pedidos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del comprador en estado activo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-068 |
| **Módulo** | Compradores |
| **Funcionalidad** | Ver historial de pedidos de un comprador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un comprador con al menos 3 pedidos en distintos estados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Hacer clic en un comprador con historial. 3. En la vista de detalle, revisar la sección "Historial de pedidos". |
| **Resultado esperado** | Se muestra la lista de pedidos del comprador con: número de pedido, fecha, estado, total. Los pedidos están ordenados por fecha descendente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial de pedidos del comprador. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-069 |
| **Módulo** | Compradores |
| **Funcionalidad** | Ver direcciones guardadas de un comprador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un comprador con múltiples direcciones de entrega. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Hacer clic en un comprador con direcciones. 3. Revisar la sección "Direcciones". |
| **Resultado esperado** | Se muestran todas las direcciones del comprador con: alias, dirección completa, ciudad. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de las direcciones del comprador. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-070 |
| **Módulo** | Compradores |
| **Funcionalidad** | Paginación de la lista de compradores |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Más de 20 compradores en la BD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Verificar que hay paginación. 3. Hacer clic en "Siguiente" para ir a la página 2. 4. Hacer clic en "Anterior" para volver. |
| **Resultado esperado** | La paginación funciona. La página 2 muestra los siguientes compradores. El número de página se actualiza. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de página 1 y página 2. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-071 |
| **Módulo** | Compradores |
| **Funcionalidad** | Estado vacío — sin compradores |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Usar filtro que retorne 0 resultados. |
| **Datos de prueba** | Búsqueda: "xyzNoExisteComprador". |
| **Pasos** | 1. Ir a Compradores. 2. En el buscador, escribir "xyzNoExisteComprador". |
| **Resultado esperado** | La lista muestra estado vacío con mensaje informativo. No hay errores. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-072 |
| **Módulo** | Compradores |
| **Funcionalidad** | ADMIN_CLIENT no puede crear compradores manualmente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Compradores. 2. Verificar si existe un botón "Crear comprador". |
| **Resultado esperado** | No existe botón de creación manual de compradores. Los compradores se crean solo vía storefront o API del cliente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista sin botón de creación. |
| **Observaciones** | Según matriz de permisos: creación de BUYER = ❌ para rol admin (solo vía storefront). |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-073 |
| **Módulo** | Compradores |
| **Funcionalidad** | Aislamiento de compradores entre tenants |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen compradores en cliente-piloto y cliente-dos. |
| **Datos de prueba** | ID de un comprador de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. En Postman, enviar GET `/admin/buyers/{id_comprador_de_cliente_dos}`. |
| **Resultado esperado** | HTTP 403 o 404. El ADMIN_CLIENT de cliente-piloto no puede acceder a compradores de otro tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 403/404. |
| **Observaciones** | Aislamiento multi-tenant crítico. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-074 |
| **Módulo** | Compradores |
| **Funcionalidad** | Buscar comprador por número de documento |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen compradores con documento registrado. |
| **Datos de prueba** | Búsqueda: número de cédula de uno de los compradores. |
| **Pasos** | 1. Ir a Compradores. 2. En el buscador, ingresar el número de cédula. |
| **Resultado esperado** | Aparece el comprador que tiene ese número de documento. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado de búsqueda. |
| **Observaciones** | La búsqueda debe funcionar por nombre, email y documento. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-075 |
| **Módulo** | Compradores |
| **Funcionalidad** | Ver métricas del comprador (total compras, última compra) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Comprador con historial de pedidos cerrados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de un comprador con historial. 2. Revisar las métricas del comprador. |
| **Resultado esperado** | Se muestran: total de pedidos realizados, total gastado (COP), fecha del último pedido. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de las métricas del comprador. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE E — Gestión de Couriers / Repartidores (PT-076 a PT-095)

---

| Campo | Valor |
|---|---|
| **ID** | PT-076 |
| **Módulo** | Couriers |
| **Funcionalidad** | Crear courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | Nombre: "Pedro Ramírez", Email: `pedro.ramirez@test.com`, Teléfono: +57 311 0000001. |
| **Pasos** | 1. Ir al menú lateral → Repartidores. 2. Clic en "Crear repartidor". 3. Completar el formulario con los datos de prueba. 4. Guardar. |
| **Resultado esperado** | El courier se crea con estado "Activo". Aparece en la lista de repartidores. El sistema envía instrucciones al email del courier para establecer contraseña (verificar el proceso descrito en guía). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del courier en la lista. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-077 |
| **Módulo** | Couriers |
| **Funcionalidad** | Crear courier con email duplicado |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Ya existe un courier con el email `pedro.ramirez@test.com`. |
| **Datos de prueba** | Email duplicado: `pedro.ramirez@test.com`. |
| **Pasos** | 1. Ir a Repartidores → Crear repartidor. 2. Ingresar el email ya existente. 3. Guardar. |
| **Resultado esperado** | El sistema muestra error de validación indicando que el email ya está en uso. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-078 |
| **Módulo** | Couriers |
| **Funcionalidad** | Editar datos de un courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe el courier "Pedro Ramírez". |
| **Datos de prueba** | Nuevo teléfono: +57 311 0000099. |
| **Pasos** | 1. Ir a Repartidores. 2. Hacer clic en "Pedro Ramírez". 3. Editar el teléfono. 4. Guardar. |
| **Resultado esperado** | Los datos del courier se actualizan. El nuevo teléfono aparece en el detalle. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del courier editado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-079 |
| **Módulo** | Couriers |
| **Funcionalidad** | Desactivar courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe courier activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Repartidores. 2. Hacer clic en el courier. 3. Usar el interruptor para desactivarlo. |
| **Resultado esperado** | El courier pasa a "Inactivo". No aparece como opción disponible en el mapa de asignación. El courier no puede iniciar sesión. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del courier inactivo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-080 |
| **Módulo** | Couriers |
| **Funcionalidad** | Listar couriers con filtros |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen couriers activos e inactivos. |
| **Datos de prueba** | Filtro: Estado = Inactivo. |
| **Pasos** | 1. Ir a Repartidores. 2. Aplicar filtro por estado "Inactivo". |
| **Resultado esperado** | Solo aparecen los couriers inactivos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-081 |
| **Módulo** | Couriers |
| **Funcionalidad** | Ver detalle del courier con métricas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Courier con historial de entregas. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Repartidores. 2. Hacer clic en un courier con historial de entregas. |
| **Resultado esperado** | Se muestra perfil del courier, historial de pedidos asignados/completados, métricas básicas (entregas completadas, tasa de éxito). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle con métricas. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-082 |
| **Módulo** | Couriers |
| **Funcionalidad** | Reactivar courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Courier "Pedro Ramírez" en estado inactivo (PT-079 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Repartidores. 2. Hacer clic en "Pedro Ramírez". 3. Usar el interruptor para activarlo. |
| **Resultado esperado** | El courier vuelve a estado "Activo". Aparece como opción disponible en el mapa de asignación. Puede iniciar sesión. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del courier activo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-083 |
| **Módulo** | Couriers |
| **Funcionalidad** | Crear courier con teléfono inválido (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: "Courier Inválido", Teléfono: "abc123". |
| **Pasos** | 1. Ir a Repartidores → Crear repartidor. 2. Ingresar un teléfono inválido "abc123". 3. Guardar. |
| **Resultado esperado** | El formulario valida el formato del teléfono y muestra error de validación. No se crea el courier. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | Teléfono debe ser formato colombiano o internacional válido. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-084 |
| **Módulo** | Couriers |
| **Funcionalidad** | Búsqueda de courier por nombre |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen múltiples couriers. |
| **Datos de prueba** | Búsqueda: "Pedro". |
| **Pasos** | 1. Ir a Repartidores. 2. En el buscador, escribir "Pedro". |
| **Resultado esperado** | Solo aparecen couriers cuyo nombre contiene "Pedro". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado de búsqueda. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-085 |
| **Módulo** | Couriers |
| **Funcionalidad** | Estado vacío — sin couriers activos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Filtrar por estado activo con todos desactivados. |
| **Datos de prueba** | Búsqueda: "xyzNoExisteCourier". |
| **Pasos** | 1. Ir a Repartidores. 2. Buscar "xyzNoExisteCourier". |
| **Resultado esperado** | La lista muestra estado vacío con mensaje informativo. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-086 |
| **Módulo** | Couriers |
| **Funcionalidad** | Aislamiento de couriers entre tenants |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen couriers en cliente-piloto y cliente-dos. |
| **Datos de prueba** | ID de un courier de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. En Postman, enviar GET `/admin/couriers/{id_courier_de_cliente_dos}`. |
| **Resultado esperado** | HTTP 403 o 404. El ADMIN_CLIENT no puede ver couriers de otro tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 403/404. |
| **Observaciones** | Aislamiento multi-tenant. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-087 |
| **Módulo** | Couriers |
| **Funcionalidad** | OPERATOR_CLIENT puede ver la lista de couriers |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Sesión activa como OPERATOR_CLIENT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT. 2. Ir al menú lateral → Repartidores. |
| **Resultado esperado** | La lista de repartidores es visible. Solo lectura (sin botón de crear ni editar si el operador no tiene el permiso). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista visible para el operador. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-088 |
| **Módulo** | Couriers |
| **Funcionalidad** | Courier que inicia sesión ve solo su vista mobile |
| **Rol** | COURIER |
| **Precondiciones** | Sesión activa como COURIER. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como COURIER. 2. Intentar navegar a `/admin/couriers`. |
| **Resultado esperado** | El COURIER es redirigido a `/courier`. No puede acceder al panel de gestión de repartidores del admin. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del acceso denegado o redirección. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-089 |
| **Módulo** | Couriers |
| **Funcionalidad** | Paginación de la lista de couriers |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Más de 20 couriers en la BD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Repartidores. 2. Verificar paginación. 3. Clic en "Siguiente". |
| **Resultado esperado** | La paginación funciona correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de páginas. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-090 |
| **Módulo** | Couriers |
| **Funcionalidad** | Ver tasa de entrega del courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Courier con pedidos entregados y fallidos en historial. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de un courier con historial. 2. Ver la tasa de entrega. |
| **Resultado esperado** | Se muestra la tasa de entrega como porcentaje (entregas exitosas / total asignadas). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la métrica de tasa de entrega. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-091 |
| **Módulo** | Couriers |
| **Funcionalidad** | Ver promedio de tiempo de entrega del courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Courier con pedidos entregados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle del courier. 2. Ver el promedio de tiempo de entrega. |
| **Resultado esperado** | Se muestra el tiempo promedio desde SHIPPED hasta DELIVERED en horas/minutos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la métrica de tiempo promedio. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-092 |
| **Módulo** | Couriers |
| **Funcionalidad** | Editar email de courier a uno ya existente (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen al menos 2 couriers con emails distintos. |
| **Datos de prueba** | Email ya en uso: `pedro.ramirez@test.com`. |
| **Pasos** | 1. Ir a Repartidores. 2. Editar un courier diferente. 3. Cambiar el email al del otro courier. 4. Guardar. |
| **Resultado esperado** | El sistema rechaza el cambio con error de email duplicado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-093 |
| **Módulo** | Couriers |
| **Funcionalidad** | Courier desactivado no aparece en mapa de asignación |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Courier desactivado. Mapa de asignación con pedidos pendientes. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Desactivar un courier. 2. Ir al Mapa de asignación. 3. Revisar la lista de couriers disponibles. |
| **Resultado esperado** | El courier desactivado NO aparece en la lista de couriers disponibles para asignar. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mapa sin el courier desactivado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-094 |
| **Módulo** | Couriers |
| **Funcionalidad** | Ver pedidos activos del courier desde su perfil admin |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Courier con pedidos activos asignados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle del courier. 2. Ver la sección de pedidos activos. |
| **Resultado esperado** | Se muestran los pedidos actualmente asignados al courier con sus estados. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los pedidos activos del courier. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-095 |
| **Módulo** | Couriers |
| **Funcionalidad** | Eliminar courier sin pedidos activos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un courier sin pedidos activos ni historial. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Crear un courier de prueba sin asignaciones. 2. Ir a su detalle. 3. Intentar eliminarlo (si existe la opción). |
| **Resultado esperado** | El courier se puede desactivar permanentemente. Si existe opción de eliminar, se requiere confirmación. El courier no puede ser eliminado si tiene historial de pedidos (soft delete o error). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la acción de eliminación. |
| **Observaciones** | La política del proyecto es soft delete (desactivar) antes que eliminar. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE F — Pickup Points (PT-096 a PT-115)

---

| Campo | Valor |
|---|---|
| **ID** | PT-096 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Crear pickup point |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | Nombre: "Bodega Norte", Dirección: "Cra 7 # 100-50, Bogotá", Horario: "Lunes a Viernes 8am-6pm", Teléfono: +57 601 0000001. Coordenadas: lat 4.7110, lng -74.0721. |
| **Pasos** | 1. Ir al menú lateral → Puntos de retiro. 2. Clic en "Crear punto de retiro". 3. Completar el formulario. 4. En el mapa, arrastrar el pin a la ubicación exacta. 5. Guardar. |
| **Resultado esperado** | El punto de retiro "Bodega Norte" se crea con estado "Activo". Aparece en la lista. El pin en el mapa refleja las coordenadas ingresadas. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del formulario con el mapa y captura del pickup point en la lista. |
| **Observaciones** | El mapa usa OSM + Leaflet. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-097 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Editar pickup point |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe el pickup point "Bodega Norte". |
| **Datos de prueba** | Nuevo horario: "Lunes a Sábado 8am-8pm". |
| **Pasos** | 1. Ir a Puntos de retiro. 2. Clic en "Bodega Norte". 3. Editar el horario. 4. Guardar. |
| **Resultado esperado** | El horario se actualiza correctamente en el detalle del punto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del punto editado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-098 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Desactivar pickup point |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un pickup point activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Puntos de retiro. 2. Usar el interruptor de "Bodega Norte" para desactivarlo. |
| **Resultado esperado** | El punto pasa a estado "Inactivo". No aparece como opción para los compradores en el checkout (pantalla de storefront). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del punto en estado inactivo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-099 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Listar pickup points con filtros |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen puntos activos e inactivos. |
| **Datos de prueba** | Filtro: Estado = Activo. |
| **Pasos** | 1. Ir a Puntos de retiro. 2. Aplicar filtro por estado "Activo". |
| **Resultado esperado** | Solo se muestran puntos activos. El punto desactivado en PT-098 no aparece. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-100 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Crear pickup point sin dirección (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Ninguna. |
| **Datos de prueba** | Nombre: "Punto Sin Dirección". Dirección: vacía. |
| **Pasos** | 1. Ir a Puntos de retiro → Crear punto. 2. Ingresar nombre pero dejar dirección vacía. 3. Guardar. |
| **Resultado esperado** | El formulario muestra error de validación indicando que la dirección es obligatoria. No se crea el punto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-101 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Reactivar pickup point |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pickup point inactivo (PT-098 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Puntos de retiro. 2. Filtrar por inactivos. 3. Usar el interruptor para activar "Bodega Norte". |
| **Resultado esperado** | El punto vuelve a estado "Activo". Aparece como opción disponible en el checkout de los compradores. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del punto activo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-102 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Crear pickup point con coordenadas inválidas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Latitud: 999.99, Longitud: -999.99. |
| **Pasos** | 1. Ir a Puntos de retiro → Crear punto. 2. Ingresar coordenadas inválidas (fuera del rango). 3. Guardar. |
| **Resultado esperado** | El sistema rechaza las coordenadas inválidas con error de validación. Las coordenadas válidas son lat [-90,90] y lng [-180,180]. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-103 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Ver pickup point en el mapa con pin correcto |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe el pickup point "Bodega Norte" con coordenadas válidas. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle del pickup point "Bodega Norte". 2. Ver el mapa. |
| **Resultado esperado** | El mapa OSM muestra el pin exactamente en las coordenadas ingresadas (lat 4.7110, lng -74.0721). El pin tiene el nombre del punto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mapa con el pin en la ubicación correcta. |
| **Observaciones** | Mapa usa OSM + Leaflet. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-104 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Búsqueda de pickup points por nombre |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen múltiples pickup points. |
| **Datos de prueba** | Búsqueda: "Bodega". |
| **Pasos** | 1. Ir a Puntos de retiro. 2. En el buscador, escribir "Bodega". |
| **Resultado esperado** | Solo aparecen puntos cuyo nombre contiene "Bodega". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado de búsqueda. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-105 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Paginación de la lista de pickup points |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Más de 20 pickup points en la BD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Puntos de retiro. 2. Verificar paginación. 3. Clic en "Siguiente". |
| **Resultado esperado** | La paginación funciona correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de páginas. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-106 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Aislamiento de pickup points entre tenants |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pickup points en cliente-piloto y cliente-dos. |
| **Datos de prueba** | ID de un pickup point de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. En Postman, GET `/admin/pickup-points/{id_de_cliente_dos}`. |
| **Resultado esperado** | HTTP 403 o 404. Aislamiento multi-tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta de error. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-107 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Ver pedidos activos en un pickup point |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pedidos PICKUP asignados a "Bodega Norte". |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de "Bodega Norte". 2. Ver la sección de pedidos activos. |
| **Resultado esperado** | Se listan los pedidos en estados AT_PICKUP_POINT o READY_FOR_PICKUP asignados a este punto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los pedidos del pickup point. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-108 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Editar coordenadas del pickup point desde el mapa |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe "Bodega Norte" con coordenadas. |
| **Datos de prueba** | Nuevas coordenadas: lat 4.7200, lng -74.0800. |
| **Pasos** | 1. Ir al detalle de "Bodega Norte". 2. Hacer clic en "Editar". 3. En el mapa, arrastrar el pin a una nueva ubicación. 4. Guardar. |
| **Resultado esperado** | Las coordenadas se actualizan. El pin en el mapa refleja la nueva posición. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pin en la nueva ubicación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-109 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Estado vacío — sin pickup points |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Filtro que retorne 0 resultados. |
| **Datos de prueba** | Búsqueda: "xyzNoExistePunto". |
| **Pasos** | 1. Ir a Puntos de retiro. 2. Buscar "xyzNoExistePunto". |
| **Resultado esperado** | La lista muestra estado vacío con mensaje informativo. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-110 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Crear pickup point con nombre duplicado (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe "Bodega Norte". |
| **Datos de prueba** | Nombre: "Bodega Norte" (duplicado). |
| **Pasos** | 1. Ir a Puntos de retiro → Crear punto. 2. Ingresar "Bodega Norte" como nombre. 3. Guardar. |
| **Resultado esperado** | El sistema acepta el nombre duplicado O muestra advertencia (los nombres no son necesariamente únicos pero es buena práctica). Verificar comportamiento real del sistema. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado. |
| **Observaciones** | Documentar si el sistema permite o no nombres duplicados. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-111 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | OPERATOR_CLIENT puede ver la lista de pickup points |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Sesión activa como OPERATOR_CLIENT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT. 2. Ir a Puntos de retiro. |
| **Resultado esperado** | La lista de pickup points es visible. Solo lectura si el operador no tiene permiso de edición. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista. |
| **Observaciones** | Verificar con la matriz de permisos. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-112 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Horario del pickup point se muestra correctamente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe "Bodega Norte" con horario configurado. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de "Bodega Norte". 2. Verificar el campo de horario. |
| **Resultado esperado** | El horario "Lunes a Sábado 8am-8pm" (editado en PT-097) se muestra claramente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle con horario visible. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-113 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Historial de pedidos recogidos en el pickup point |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pickup point con pedidos cerrados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de "Bodega Norte". 2. Ver el historial de pedidos recogidos. |
| **Resultado esperado** | Se muestran los pedidos PICKUP cerrados (COMPLETED_SUCCESSFULLY) que pasaron por este punto. Con fecha y total. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-114 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Número de teléfono del pickup point es obligatorio |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: "Punto Sin Tel", Dirección: "Cr 1 #2-3". Teléfono: vacío. |
| **Pasos** | 1. Ir a Puntos de retiro → Crear punto. 2. Dejar teléfono vacío. 3. Guardar. |
| **Resultado esperado** | El formulario indica si el teléfono es obligatorio o si se puede guardar sin él. Documentar el comportamiento real. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado. |
| **Observaciones** | Verificar requerimientos del campo según el formulario. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-115 |
| **Módulo** | Pickup Points |
| **Funcionalidad** | Pickup point inactivo no aparece en el listado de asignación de pedidos PICKUP |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pickup point "Bodega Norte" en estado inactivo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Desactivar "Bodega Norte". 2. Crear un pedido nuevo con delivery_type = PICKUP. 3. Intentar seleccionar el pickup point. |
| **Resultado esperado** | "Bodega Norte" no aparece como opción para nuevos pedidos PICKUP. Solo los puntos activos están disponibles. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del selector de pickup point sin "Bodega Norte". |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE G — Flujo de Pedidos SHIP — Creación y Ciclo Completo (PT-116 a PT-155)

---

| Campo | Valor |
|---|---|
| **ID** | PT-116 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Ver lista de pedidos como ADMIN_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pedidos en distintos estados para cliente-piloto. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Ir al menú lateral → Pedidos. |
| **Resultado esperado** | La tabla muestra todos los pedidos del cliente con columnas: #, Comprador, Items, Total, Estado, Repartidor, Fecha. Solo aparecen pedidos de cliente-piloto (no de otros clientes). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista de pedidos. |
| **Observaciones** | Verificar aislamiento multi-tenant. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-117 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Filtrar pedidos por estado |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos en distintos estados. |
| **Datos de prueba** | Filtro: Estado = "En tránsito" (IN_TRANSIT). |
| **Pasos** | 1. Ir a Pedidos. 2. Seleccionar "En tránsito" en el filtro de estado. |
| **Resultado esperado** | Solo aparecen pedidos con estado IN_TRANSIT. El contador de registros se actualiza. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del filtro aplicado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-118 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Filtrar pedidos por payment_status |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos con distintos payment_status. |
| **Datos de prueba** | Filtro: Payment Status = "Pagado" (PAID). |
| **Pasos** | 1. Ir a Pedidos. 2. Seleccionar "Pagado" en el filtro de pago. |
| **Resultado esperado** | Solo aparecen pedidos con payment_status = PAID. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del filtro aplicado. |
| **Observaciones** | Los 11 estados de payment_status deben estar disponibles en el selector. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-119 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Filtrar pedidos por rango de fechas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos creados en distintas fechas. |
| **Datos de prueba** | Fecha desde: hoy, Fecha hasta: hoy. |
| **Pasos** | 1. Ir a Pedidos. 2. Ingresar fecha desde y fecha hasta como la fecha de hoy. 3. Hacer clic en "Filtrar". |
| **Resultado esperado** | Solo aparecen pedidos creados hoy. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-120 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Buscar pedido por número |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un pedido con ID conocido. |
| **Datos de prueba** | ID del pedido de prueba (ej. #5001). |
| **Pasos** | 1. Ir a Pedidos. 2. En el buscador, escribir "#5001" o "5001". 3. Hacer clic en "Filtrar". |
| **Resultado esperado** | Aparece solo el pedido #5001. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-121 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Limpiar filtros |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Filtros aplicados en la lista de pedidos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Pedidos con algún filtro aplicado. 2. Hacer clic en "Limpiar". |
| **Resultado esperado** | Todos los filtros se resetean. La lista muestra todos los pedidos del cliente (sin filtros). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista sin filtros. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-122 |
| **Módulo** | Pedidos — Detalle |
| **Funcionalidad** | Ver detalle 360 de un pedido |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un pedido con varios eventos en el historial. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Pedidos. 2. Hacer clic en cualquier pedido de la lista. |
| **Resultado esperado** | Se muestra la vista 360: estado actual con badge, timeline de eventos, datos del comprador (nombre, contacto, dirección), detalle de entrega, resumen de items y totales, estado del pago, acciones disponibles según estado actual. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura completa del detalle del pedido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-123 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Aceptar pedido (VALIDATION_APPROVED → SELLER_CONFIRMED) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un pedido con estado VALIDATION_APPROVED. |
| **Datos de prueba** | Pedido en estado VALIDATION_APPROVED. |
| **Pasos** | 1. Ir a Pedidos. 2. Abrir el pedido en estado "Validación aprobada". 3. En el panel de acciones, hacer clic en "Aceptar pedido". 4. Confirmar. |
| **Resultado esperado** | El pedido cambia al estado SELLER_CONFIRMED. La timeline muestra el nuevo evento. El botón de acción siguiente ("Marcar preparando") aparece disponible. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado SELLER_CONFIRMED en el detalle. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-124 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Rechazar pedido (VALIDATION_APPROVED → CANCELLED_BY_SELLER) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe un pedido con estado VALIDATION_APPROVED. |
| **Datos de prueba** | Pedido en estado VALIDATION_APPROVED. |
| **Pasos** | 1. Abrir el pedido en estado VALIDATION_APPROVED. 2. Hacer clic en "Rechazar". 3. Confirmar. |
| **Resultado esperado** | El pedido cambia a CANCELLED_BY_SELLER → CLOSED. Si había pago online (PAID), el sistema inicia reembolso (refund_status = REFUND_PENDING). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado CANCELLED_BY_SELLER. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-125 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Marcar pedido como en preparación (SELLER_CONFIRMED → PREPARING) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en estado SELLER_CONFIRMED. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido en estado SELLER_CONFIRMED. 2. Hacer clic en "Marcar preparando". |
| **Resultado esperado** | El pedido cambia a PREPARING. La timeline se actualiza con la marca de tiempo del evento. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado PREPARING. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-126 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Marcar listo para despacho (PREPARING → AWAITING_COURIER_ASSIGNMENT) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en estado PREPARING con delivery_type = SHIP y delivery_carrier_type = OWN_FLEET. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido en estado PREPARING. 2. Hacer clic en "Marcar listo para despacho". |
| **Resultado esperado** | El pedido pasa a AWAITING_COURIER_ASSIGNMENT. El pedido aparece en el mapa de asignación como pin pendiente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado AWAITING_COURIER_ASSIGNMENT y captura del mapa mostrando el pin del pedido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-127 |
| **Módulo** | Pedidos — Mapa de Asignación |
| **Funcionalidad** | Ver mapa con pedidos pendientes de asignación |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Al menos un pedido en estado AWAITING_COURIER_ASSIGNMENT con dirección geolocalizable. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al menú lateral → Mapa. 2. El mapa carga con OSM tiles. |
| **Resultado esperado** | El mapa muestra pines para cada pedido en AWAITING_COURIER_ASSIGNMENT. El panel derecho muestra la lista de pedidos listos y la lista de repartidores disponibles. El mapa se actualiza cada 30 segundos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mapa con pines y el panel derecho. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-128 |
| **Módulo** | Pedidos — Mapa de Asignación |
| **Funcionalidad** | Asignar courier a pedido desde el mapa |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Mapa abierto. Al menos un pedido en AWAITING_COURIER_ASSIGNMENT y al menos un courier activo disponible. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al Mapa. 2. Hacer clic en un pin de pedido en el mapa. 3. En el panel derecho, seleccionar un repartidor disponible. 4. Hacer clic en "Asignar". 5. Confirmar la asignación en el diálogo. |
| **Resultado esperado** | El pedido cambia a estado COURIER_ASSIGNED. El pin del pedido cambia de color/ícono. El courier asignado aparece en el detalle del pedido. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del modal de asignación y captura del detalle del pedido con el courier asignado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-129 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pedido SHIPPED → IN_TRANSIT → OUT_FOR_DELIVERY (por COURIER) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en estado COURIER_ASSIGNED asignado al courier de prueba. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como COURIER. 2. Abrir el pedido asignado. 3. Tocar "Iniciar despacho". 4. Tocar "Marcar en camino" (IN_TRANSIT). 5. Tocar "Salir a reparto" (OUT_FOR_DELIVERY). |
| **Resultado esperado** | Cada acción actualiza el estado del pedido. La timeline en el admin (vista ADMIN_CLIENT) refleja los cambios en tiempo real. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de cada estado en la vista COURIER y verificación en la vista admin. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-130 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pedido llegó al comprador (ARRIVED_AT_CUSTOMER) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en estado OUT_FOR_DELIVERY. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En la vista COURIER, abrir el pedido. 2. Tocar "Llegué al domicilio". |
| **Resultado esperado** | El pedido pasa a ARRIVED_AT_CUSTOMER. Las acciones siguientes dependen del payment_method: si es online ya pagado, aparece "Marcar como entregado"; si es COD, aparece "Registrar cobro". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la vista COURIER en estado ARRIVED_AT_CUSTOMER. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-131 |
| **Módulo** | Pedidos — SHIP Flujo / Pago Online |
| **Funcionalidad** | Marcar pedido DELIVERED (pago online ya realizado) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en ARRIVED_AT_CUSTOMER con payment_status = PAID (pago online confirmado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En la vista COURIER, pedido en ARRIVED_AT_CUSTOMER. 2. Tocar "Marcar como entregado". |
| **Resultado esperado** | El pedido pasa a DELIVERED. El comprador puede confirmar recepción. La timeline en admin se actualiza. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado DELIVERED. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-132 |
| **Módulo** | Pedidos — COD (Cobro contra entrega) |
| **Funcionalidad** | Registrar cobro en efectivo y marcar entregado |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en ARRIVED_AT_CUSTOMER con payment_method = CASH_ON_DELIVERY. |
| **Datos de prueba** | Monto a cobrar: $35.000. Monto recibido: $50.000 (con devuelta). Evidencia: imagen JPG del recibo. |
| **Pasos** | 1. En la vista COURIER, pedido en ARRIVED_AT_CUSTOMER con COD. 2. Tocar "Registrar cobro". 3. Ingresar monto recibido: $50.000. 4. Seleccionar método "Efectivo". 5. Subir foto del recibo. 6. Tocar "Confirmar cobro". 7. Tocar "Marcar como entregado". |
| **Resultado esperado** | El cobro queda registrado con payment_status = PAYMENT_COLLECTED. El pedido pasa a DELIVERED. La evidencia fotográfica queda asociada al pedido. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la pantalla de cobro completado y del estado DELIVERED. |
| **Observaciones** | `collection.evidence_required = true`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-133 |
| **Módulo** | Pedidos — COD |
| **Funcionalidad** | Registrar cobro electrónico (datáfono) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en ARRIVED_AT_CUSTOMER con payment_method = ELECTRONIC_ON_DELIVERY. |
| **Datos de prueba** | Monto: $45.000. Submétodo: Datáfono. ID transacción: "TXN-20260531-001". Evidencia: imagen del comprobante. |
| **Pasos** | 1. En la vista COURIER, tocar "Registrar cobro". 2. Seleccionar método "Electrónico". 3. Seleccionar submétodo "Datáfono". 4. Ingresar ID de transacción. 5. Subir evidencia. 6. Tocar "Confirmar cobro". |
| **Resultado esperado** | El cobro electrónico se registra. payment_status cambia a PAYMENT_COLLECTED. El ID de transacción queda registrado para conciliación. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la pantalla de cobro electrónico completado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-134 |
| **Módulo** | Pedidos — COD |
| **Funcionalidad** | Registrar cobro sin subir evidencia (cuando es requerida) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido COD en ARRIVED_AT_CUSTOMER. `collection.evidence_required = true`. |
| **Datos de prueba** | No subir ninguna imagen. |
| **Pasos** | 1. En la pantalla de cobro. 2. Ingresar monto. 3. Seleccionar método. 4. NO subir evidencia. 5. Tocar "Confirmar cobro". |
| **Resultado esperado** | El sistema muestra error de validación indicando que la evidencia es obligatoria. No se confirma el cobro. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | Parámetro: `collection.evidence_required = true`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-135 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Auto-confirmación por sistema (DELIVERED → CONFIRMED_BY_SYSTEM) |
| **Rol** | Sistema (job pg-boss) |
| **Precondiciones** | Pedido en estado DELIVERED hace más de 72 horas (o ajustar parámetro a 1 minuto para prueba). |
| **Datos de prueba** | Pedido en DELIVERED. Parámetro `closure.auto_confirm_delivered_hours = 72` (ajustar a 1 para prueba). |
| **Pasos** | 1. Crear un pedido y llevarlo a estado DELIVERED. 2. Modificar el parámetro `closure.auto_confirm_delivered_hours` a 0.017 (1 minuto) en la BD. 3. Esperar 2 minutos. 4. Verificar el estado del pedido. |
| **Resultado esperado** | El job de pg-boss detecta el pedido en DELIVERED por más de 1 minuto y lo transiciona automáticamente a CONFIRMED_BY_SYSTEM → COMPLETED_SUCCESSFULLY → CLOSED. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en estado CONFIRMED_BY_SYSTEM y CLOSED. Verificar en logs de pino. |
| **Observaciones** | Restaurar el parámetro a 72 horas después de la prueba. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-136 |
| **Módulo** | Pedidos — Cancelaciones |
| **Funcionalidad** | Cancelar pedido en estado PREPARING (by Admin) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en estado PREPARING. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido en PREPARING. 2. Hacer clic en el botón "Cancelar". 3. Confirmar. |
| **Resultado esperado** | El pedido pasa a CANCELLED_BY_ADMIN → CLOSED. Si tenía pago online, se registra refund_status = REFUND_PENDING. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido cancelado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-137 |
| **Módulo** | Pedidos — Cancelaciones |
| **Funcionalidad** | Solicitud de cancelación post-despacho (CUSTOMER_CANCEL_REQUEST) |
| **Rol** | ADMIN_CLIENT (aprobando solicitud del comprador) |
| **Precondiciones** | Pedido en estado IN_TRANSIT con una solicitud de cancelación del comprador activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido en estado CUSTOMER_CANCEL_REQUEST. 2. Hacer clic en "Aprobar cancelación". 3. Confirmar. |
| **Resultado esperado** | El pedido pasa a CANCEL_REQUEST_APPROVED → RETURN_TO_ORIGIN. Si tenía pago, se registra reembolso pendiente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en RETURN_TO_ORIGIN. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-138 |
| **Módulo** | Pedidos — Cancelaciones |
| **Funcionalidad** | Rechazar solicitud de cancelación post-despacho |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en estado CUSTOMER_CANCEL_REQUEST. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido en CUSTOMER_CANCEL_REQUEST. 2. Hacer clic en "Rechazar cancelación". 3. Confirmar. |
| **Resultado esperado** | El pedido vuelve al estado operativo previo (IN_TRANSIT u OUT_FOR_DELIVERY). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en el estado previo. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-139 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Intento de entrega fallido (DELIVERY_ATTEMPTED) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en OUT_FOR_DELIVERY asignado al courier. |
| **Datos de prueba** | Motivo: "Comprador no estaba en casa". |
| **Pasos** | 1. En la vista COURIER, pedido en OUT_FOR_DELIVERY. 2. Tocar "Intento fallido". 3. Ingresar motivo. 4. Confirmar. |
| **Resultado esperado** | El pedido pasa a DELIVERY_ATTEMPTED. La nota de motivo queda registrada. El admin puede ver el intento fallido en la timeline. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado DELIVERY_ATTEMPTED y la nota de motivo. |
| **Observaciones** | `ship.delivery_attempt_max_retries = 3`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-140 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pedido con pago Wompi online — flujo completo de pago |
| **Rol** | N/A (sistema) |
| **Precondiciones** | Credenciales Wompi sandbox configuradas. Pedido con payment_method = ONLINE_AT_ORDER en estado PENDING_ONLINE_PAYMENT. |
| **Datos de prueba** | Tarjeta de prueba Wompi sandbox. |
| **Pasos** | 1. Crear un pedido con pago online (desde storefront o manualmente). 2. Completar el pago en la pasarela Wompi sandbox con tarjeta de prueba aprobada. 3. Wompi envía webhook a RUTA. 4. Verificar el estado del pedido en el admin. |
| **Resultado esperado** | Tras la confirmación del webhook de Wompi, el payment_status cambia a PAID. El pedido avanza automáticamente a ORDER_VALIDATING. El log de auditoría registra el evento del webhook. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido con payment_status = PAID. Captura del log de auditoría con el evento de webhook. |
| **Observaciones** | Verificar que la firma HMAC del webhook es validada correctamente. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-141 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pago Wompi rechazado (PAYMENT_REJECTED_FINAL) |
| **Rol** | N/A (sistema) |
| **Precondiciones** | Credenciales Wompi sandbox. Pedido en PENDING_ONLINE_PAYMENT. |
| **Datos de prueba** | Tarjeta de prueba Wompi sandbox para pago rechazado. |
| **Pasos** | 1. Intentar pagar con tarjeta de prueba para rechazo (número de tarjeta de Wompi para simular rechazo). 2. Wompi envía webhook de rechazo. 3. Verificar estado del pedido. |
| **Resultado esperado** | payment_status cambia a PAYMENT_REJECTED_FINAL. El pedido se cierra (CLOSED) con closure_reason = PAYMENT_REJECTED. No se cobra al comprador. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido cerrado con causa PAYMENT_REJECTED. |
| **Observaciones** | |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-142 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Transición de estado inválida (debe ser rechazada) |
| **Rol** | N/A (API directa) |
| **Precondiciones** | Pedido en estado DRAFT. Postman configurado. |
| **Datos de prueba** | Intentar PATCH a DELIVERED desde DRAFT directamente vía API. |
| **Pasos** | 1. Obtener el token de autenticación. 2. En Postman, enviar PATCH /admin/orders/{id}/deliver con el pedido en DRAFT. |
| **Resultado esperado** | El servidor retorna HTTP 422 con código `INVALID_STATE_TRANSITION` y detalle de las transiciones permitidas. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 422 en Postman. |
| **Observaciones** | El state machine solo permite transiciones válidas. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-143 |
| **Módulo** | Pedidos — Flujo SHIP End-to-End |
| **Funcionalidad** | Ciclo SHIP completo con pago COD — end-to-end |
| **Rol** | ADMIN_CLIENT + COURIER |
| **Precondiciones** | Entorno completo con courier activo y pickup point. |
| **Datos de prueba** | Pedido con delivery_type = SHIP, payment_method = CASH_ON_DELIVERY. |
| **Pasos** | 1. Crear pedido manualmente (como ADMIN_CLIENT). 2. Pasar por: VALIDATION_APPROVED → SELLER_CONFIRMED → PREPARING → AWAITING_COURIER_ASSIGNMENT. 3. Asignar courier desde el mapa. 4. Como COURIER: SHIPPED → IN_TRANSIT → OUT_FOR_DELIVERY → ARRIVED_AT_CUSTOMER. 5. Registrar cobro en efectivo con evidencia. 6. Marcar DELIVERED. 7. Verificar auto-confirmación (o confirmar manualmente). |
| **Resultado esperado** | El pedido completa el ciclo exitosamente terminando en COMPLETED_SUCCESSFULLY → CLOSED. Todos los estados aparecen en la timeline. payment_status = PAYMENT_COLLECTED. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la timeline completa del pedido cerrado. |
| **Observaciones** | Este es el caso de regresión más crítico del sistema. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-144 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Ciclo SHIP completo con pago Wompi online — end-to-end |
| **Rol** | ADMIN_CLIENT + Sistema |
| **Precondiciones** | Credenciales Wompi sandbox configuradas. Courier activo. |
| **Datos de prueba** | delivery_type: SHIP, payment_method: ONLINE_AT_ORDER. Tarjeta Wompi sandbox aprobada. |
| **Pasos** | 1. Crear pedido con pago online (o usar uno creado por el storefront). 2. Completar pago en Wompi sandbox. 3. Verificar cambio de payment_status a PAID. 4. Avanzar: SELLER_CONFIRMED → PREPARING → AWAITING_COURIER_ASSIGNMENT. 5. Asignar courier. 6. COURIER: SHIPPED → IN_TRANSIT → OUT_FOR_DELIVERY → ARRIVED_AT_CUSTOMER → DELIVERED. 7. Verificar auto-confirmación. |
| **Resultado esperado** | El ciclo completa exitosamente. Estado final: COMPLETED_SUCCESSFULLY → CLOSED. payment_status = PAID (no se solicita cobro). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la timeline completa con payment_status = PAID y estado CLOSED. |
| **Observaciones** | Caso de regresión crítico junto con PT-143. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-145 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Cancelar pedido por cliente durante IN_TRANSIT (solicitud) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido SHIP en estado IN_TRANSIT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido en IN_TRANSIT. 2. Crear manualmente la solicitud de cancelación del comprador (o usar el endpoint API). 3. El pedido pasa a CUSTOMER_CANCEL_REQUEST. 4. Como ADMIN_CLIENT, rechazar la solicitud. |
| **Resultado esperado** | El pedido vuelve a IN_TRANSIT. La solicitud denegada queda registrada en la timeline. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en IN_TRANSIT con la solicitud rechazada en el historial. |
| **Observaciones** | Ver PT-138 para el flujo de aprobación. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-146 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pedido SHIP con courier externo (EXTERNAL_COURIER) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido con delivery_carrier_type = EXTERNAL_COURIER. |
| **Datos de prueba** | delivery_type: SHIP, delivery_carrier_type: EXTERNAL_COURIER. Código de guía: "GUIA-12345". |
| **Pasos** | 1. Abrir un pedido SHIP con carrier externo en PREPARING. 2. Marcar listo para despacho. 3. Registrar número de guía del transportista externo. 4. Marcar como despachado. |
| **Resultado esperado** | El pedido avanza a SHIPPED con el código de guía externo registrado. No hay asignación de courier interno. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido con código de guía externo. |
| **Observaciones** | EXTERNAL_COURIER no aparece en el mapa de asignación interna. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-147 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Verificar que el pedido DRAFT expira automáticamente |
| **Rol** | Sistema (pg-boss) |
| **Precondiciones** | Pedido en DRAFT. Parámetro ajustado a 1 minuto para prueba. |
| **Datos de prueba** | `order.draft_expiration_minutes = 1` (para prueba). |
| **Pasos** | 1. Crear un pedido y dejarlo en DRAFT. 2. Ajustar parámetro a 1 minuto. 3. Esperar 2 minutos. 4. Verificar estado. |
| **Resultado esperado** | El pedido pasa a EXPIRED → CLOSED con closure_reason = DRAFT_EXPIRED. El log de pg-boss registra la ejecución. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en CLOSED con reason DRAFT_EXPIRED. |
| **Observaciones** | Restaurar parámetro a 1440 min después de la prueba. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-148 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Reasignar courier en pedido ya asignado |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en COURIER_ASSIGNED con un courier asignado. |
| **Datos de prueba** | Nuevo courier: diferente al asignado actualmente. |
| **Pasos** | 1. Abrir el pedido en COURIER_ASSIGNED. 2. Ir al Mapa. 3. Seleccionar el pedido y asignar un courier diferente. 4. Confirmar. |
| **Resultado esperado** | El courier se reasigna. El historial registra el cambio de asignación. El nuevo courier ve el pedido en su lista. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido con el nuevo courier asignado. |
| **Observaciones** | Verificar que el courier anterior ya no ve el pedido. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-149 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pedido retorna al origen y queda cerrado |
| **Rol** | COURIER + ADMIN_CLIENT |
| **Precondiciones** | Pedido en RETURN_TO_ORIGIN asignado al courier. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Pedido en RETURN_TO_ORIGIN. 2. El COURIER registra la devolución (lleva el paquete al origen). 3. El ADMIN_CLIENT confirma la devolución recibida. |
| **Resultado esperado** | El pedido pasa a RETURNED_TO_ORIGIN → CLOSED con closure_reason = RETURNED. Si había pago, refund_status queda como REFUND_PENDING. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en CLOSED con reason RETURNED. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-150 |
| **Módulo** | Pedidos — SHIP Flujo |
| **Funcionalidad** | Pedido SHIP con múltiples intentos de entrega (hasta el máximo) |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en OUT_FOR_DELIVERY. Parámetro `ship.delivery_attempt_max_retries = 3`. |
| **Datos de prueba** | Motivo de fallo: "No hay nadie en casa". |
| **Pasos** | 1. En la vista COURIER, registrar 3 intentos fallidos consecutivos. 2. Verificar qué sucede en el 4to intento o después del 3ro. |
| **Resultado esperado** | Después del 3er intento fallido, el sistema inicia automáticamente el proceso de retorno (RETURN_TO_ORIGIN) o el admin recibe una alerta para decidir. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado tras el 3er intento fallido. |
| **Observaciones** | Parámetro: `ship.delivery_attempt_max_retries = 3`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-151 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Paginación de la lista de pedidos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Más de 20 pedidos en la BD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Pedidos. 2. Verificar paginación. 3. Clic en "Siguiente". 4. Clic en "Anterior". |
| **Resultado esperado** | La paginación funciona. Cada página muestra 20 pedidos. El total de registros es correcto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de página 1 y 2. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-152 |
| **Módulo** | Pedidos — Lista |
| **Funcionalidad** | Ordenar pedidos por fecha (descendente/ascendente) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen múltiples pedidos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Pedidos. 2. Hacer clic en el encabezado "Fecha" para ordenar ascendente. 3. Clic nuevamente para ordenar descendente. |
| **Resultado esperado** | La lista se ordena correctamente en ambas direcciones. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del ordenamiento ascendente y descendente. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-153 |
| **Módulo** | Pedidos — Detalle |
| **Funcionalidad** | Ver timeline completa con todos los eventos del pedido |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido con historial completo de estados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de un pedido con múltiples transiciones de estado. 2. Revisar la sección de timeline. |
| **Resultado esperado** | Se muestran todos los eventos del pedido en orden cronológico con: estado, fecha/hora, actor (usuario o sistema). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la timeline completa. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-154 |
| **Módulo** | Pedidos — Detalle |
| **Funcionalidad** | Acciones de pedido desactivadas en estados terminales |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en estado CLOSED. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle de un pedido en estado CLOSED. 2. Revisar el panel de acciones. |
| **Resultado esperado** | No hay botones de acción disponibles. El pedido está en estado terminal y no puede ser modificado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle sin acciones disponibles. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-155 |
| **Módulo** | Pedidos — Detalle |
| **Funcionalidad** | Ver evidencias de cobro COD desde el detalle admin |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido COD con cobro registrado y evidencia fotográfica (PT-132 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al detalle del pedido con cobro COD registrado. 2. Buscar la sección de evidencias de cobro. |
| **Resultado esperado** | Se muestra la imagen de evidencia del cobro. El monto cobrado, método y (si aplica) ID de transacción están visibles. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la evidencia fotográfica visible en el detalle del pedido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE H — Flujo PICKUP (PT-156 a PT-175)

---

| Campo | Valor |
|---|---|
| **ID** | PT-156 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Crear pedido manual con delivery_type = PICKUP |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pickup points activos. Comprador activo. Productos con stock. |
| **Datos de prueba** | delivery_type: PICKUP, Pickup Point: "Bodega Norte". |
| **Pasos** | 1. Ir a Pedidos → Crear pedido (si existe botón). 2. Seleccionar comprador. 3. Agregar productos. 4. Seleccionar tipo de entrega: PICKUP. 5. Seleccionar el pickup point "Bodega Norte". 6. Confirmar. |
| **Resultado esperado** | El pedido se crea con delivery_type = PICKUP y el pickup_point_id asignado. El pedido inicia en el flujo de validación estándar. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido creado con tipo PICKUP. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-157 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pasar pedido PICKUP a READY_FOR_PICKUP |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido PICKUP en estado PREPARING. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido PICKUP en estado PREPARING. 2. Hacer clic en "Marcar listo para retiro". |
| **Resultado esperado** | El pedido pasa a READY_FOR_PICKUP. No hay asignación de courier (no aplica). El comprador es notificado que su pedido está listo. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado READY_FOR_PICKUP. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-158 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pedido disponible en punto físico (AT_PICKUP_POINT) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en READY_FOR_PICKUP. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido en READY_FOR_PICKUP. 2. Hacer clic en "Marcar disponible en punto" (o la acción que corresponda según la UI). |
| **Resultado esperado** | El pedido pasa a AT_PICKUP_POINT. El comprador puede ver que su pedido está listo para recoger en el punto físico. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado AT_PICKUP_POINT. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-159 |
| **Módulo** | Pedidos — PICKUP Flujo / Acciones Pickup |
| **Funcionalidad** | Registrar llegada del comprador y validar identidad |
| **Rol** | ADMIN_CLIENT / OPERATOR_CLIENT |
| **Precondiciones** | Pedido en AT_PICKUP_POINT. Componente PickupActions disponible en el detalle. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido en AT_PICKUP_POINT. 2. Usar las acciones de PICKUP: registrar llegada del comprador (CUSTOMER_ARRIVED_AT_PICKUP_POINT). 3. Validar identidad del comprador (IDENTITY_VALIDATED). |
| **Resultado esperado** | El estado avanza de AT_PICKUP_POINT → CUSTOMER_ARRIVED_AT_PICKUP_POINT → IDENTITY_VALIDATED. Las acciones de cobro (si aplica) o entrega aparecen disponibles. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los estados CUSTOMER_ARRIVED y IDENTITY_VALIDATED. |
| **Observaciones** | Componente: `PickupActions.tsx` en el frontend. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-160 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Registrar cobro en pickup point y marcar entregado |
| **Rol** | ADMIN_CLIENT / OPERATOR_CLIENT |
| **Precondiciones** | Pedido PICKUP en IDENTITY_VALIDATED con payment_method = CASH_ON_DELIVERY. |
| **Datos de prueba** | Monto cobrado: $25.000. Evidencia: foto del recibo. |
| **Pasos** | 1. Con identidad validada, proceder al cobro. 2. Registrar cobro en efectivo con evidencia. 3. Confirmar cobro. 4. Marcar pedido como entregado (PICKED_UP → DELIVERED). |
| **Resultado esperado** | El cobro se registra. El pedido pasa a PICKED_UP → DELIVERED. payment_status = PAYMENT_COLLECTED. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del cobro registrado y del estado DELIVERED. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-161 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pedido PICKUP expirado sin retiro (PICKUP_EXPIRED) |
| **Rol** | Sistema (job pg-boss) |
| **Precondiciones** | Pedido en AT_PICKUP_POINT hace más de 48 horas (ajustar parámetro a 1 min para prueba). |
| **Datos de prueba** | `pickup.expiration_hours = 48` (ajustar a 0.017 para prueba). |
| **Pasos** | 1. Crear pedido PICKUP y llevarlo a AT_PICKUP_POINT. 2. Ajustar parámetro a 1 minuto en BD. 3. Esperar 2 minutos. 4. Verificar estado del pedido. |
| **Resultado esperado** | El job de pg-boss cierra el pedido con PICKUP_EXPIRED → CLOSED. closure_reason = PICKUP_EXPIRED. Si había pago, refund_status = REFUND_PENDING. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en PICKUP_EXPIRED y CLOSED. |
| **Observaciones** | Restaurar parámetro a 48h después. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-162 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Identidad no válida en pickup (PICKUP_AUTH_FAILED) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en CUSTOMER_ARRIVED_AT_PICKUP_POINT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido en CUSTOMER_ARRIVED_AT_PICKUP_POINT. 2. Usar la acción "Identidad no válida" (o equivalente). |
| **Resultado esperado** | El pedido vuelve a AT_PICKUP_POINT. El intento fallido queda registrado. El comprador puede intentar de nuevo. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado AT_PICKUP_POINT después del fallo de identidad. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-163 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Ciclo PICKUP completo end-to-end con pago online |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pickup point activo. Producto con stock. |
| **Datos de prueba** | delivery_type: PICKUP, payment_method: ONLINE_AT_ORDER (pago ya confirmado). |
| **Pasos** | 1. Crear pedido PICKUP con pago online ya confirmado (payment_status = PAID). 2. Avanzar: SELLER_CONFIRMED → PREPARING → READY_FOR_PICKUP → AT_PICKUP_POINT. 3. Registrar llegada: CUSTOMER_ARRIVED_AT_PICKUP_POINT → IDENTITY_VALIDATED. 4. Marcar entregado: PICKED_UP → DELIVERED. 5. Confirmar recepción o esperar auto-confirmación. |
| **Resultado esperado** | El ciclo completa exitosamente. Estado final: COMPLETED_SUCCESSFULLY → CLOSED. No se solicita cobro (ya estaba pagado). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la timeline completa del pedido PICKUP cerrado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |


---

| Campo | Valor |
|---|---|
| **ID** | PT-164 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Cancelar pedido PICKUP antes de READY_FOR_PICKUP |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido PICKUP en estado PREPARING. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido PICKUP en PREPARING. 2. Hacer clic en "Cancelar". 3. Confirmar. |
| **Resultado esperado** | El pedido pasa a CANCELLED_BY_ADMIN → CLOSED. Si tenía pago online, refund_status = REFUND_PENDING. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido PICKUP cancelado. |
| **Observaciones** | Las cancelaciones en PICKUP siguen las mismas reglas que SHIP antes del despacho. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-165 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pedido PICKUP con pago COD — cobro en punto de retiro |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Pedido PICKUP en IDENTITY_VALIDATED con payment_method = CASH_ON_DELIVERY. |
| **Datos de prueba** | Monto cobrado: $30.000. Evidencia: imagen del recibo. |
| **Pasos** | 1. Como OPERATOR_CLIENT, abrir el pedido en IDENTITY_VALIDATED. 2. Registrar cobro en efectivo. 3. Subir evidencia. 4. Confirmar cobro. 5. Marcar entregado. |
| **Resultado esperado** | El cobro se registra con payment_status = PAYMENT_COLLECTED. El pedido pasa a PICKED_UP → DELIVERED. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del cobro y del estado DELIVERED. |
| **Observaciones** | El cobro en PICKUP lo realiza el operador del punto, no el courier. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-166 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Búsqueda de pedido PICKUP por número en la lista |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pedidos PICKUP en la lista. |
| **Datos de prueba** | Filtro: Tipo = PICKUP. |
| **Pasos** | 1. Ir a Pedidos. 2. Filtrar por tipo "PICKUP". |
| **Resultado esperado** | Solo aparecen pedidos con delivery_type = PICKUP. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista filtrada por PICKUP. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-167 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pedido PICKUP — cliente llega sin hacer cita, se registra llegada espontánea |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Pedido en AT_PICKUP_POINT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Como OPERATOR_CLIENT, abrir el detalle del pedido en AT_PICKUP_POINT. 2. Registrar CUSTOMER_ARRIVED_AT_PICKUP_POINT. 3. Validar identidad. 4. Entregar. |
| **Resultado esperado** | El flujo funciona sin necesidad de notificación previa del comprador. El operador puede registrar la llegada del comprador directamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los estados avanzando. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-168 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Múltiples intentos de validación de identidad |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Pedido en CUSTOMER_ARRIVED_AT_PICKUP_POINT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Registrar fallo de identidad (PT-162). 2. El pedido vuelve a AT_PICKUP_POINT. 3. Registrar llegada nuevamente. 4. Validar identidad exitosamente. |
| **Resultado esperado** | El segundo intento de validación es exitoso. El flujo continúa normalmente hasta la entrega. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los dos intentos en el historial y la entrega final. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-169 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pedido PICKUP en lista del admin muestra pickup point asignado |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pedidos PICKUP. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Pedidos. 2. Filtrar por tipo PICKUP. 3. Ver la columna de pickup point en la lista. |
| **Resultado esperado** | Cada pedido PICKUP muestra el nombre del pickup point asignado en la columna correspondiente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista con la columna de pickup point. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-170 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Confirmar recepción manual por comprador (PICKED_UP → CONFIRMED_BY_BUYER) |
| **Rol** | Sistema / Comprador |
| **Precondiciones** | Pedido en DELIVERED (PICKUP). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Pedido PICKUP en DELIVERED. 2. El comprador confirma recepción (desde storefront o API). |
| **Resultado esperado** | El pedido pasa a CONFIRMED_BY_BUYER → COMPLETED_SUCCESSFULLY → CLOSED. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del pedido en CLOSED con reason CONFIRMED_BY_BUYER. |
| **Observaciones** | Alternativa: auto-confirmación por el sistema (PT-135 equivalente para PICKUP). |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-171 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Estado vacío — no hay pedidos PICKUP activos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Filtrar pedidos PICKUP en estado AT_PICKUP_POINT sin resultados. |
| **Datos de prueba** | Filtro: Tipo = PICKUP, Estado = AT_PICKUP_POINT (si no hay ninguno). |
| **Pasos** | 1. Ir a Pedidos. 2. Filtrar por PICKUP y estado AT_PICKUP_POINT. 3. Si no hay resultados, verificar el mensaje. |
| **Resultado esperado** | La lista muestra estado vacío con mensaje informativo. No hay errores. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-172 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Detalle del pedido PICKUP muestra información del punto de retiro |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido PICKUP con pickup point asignado. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle de un pedido PICKUP. 2. Revisar la sección de entrega. |
| **Resultado esperado** | Se muestra el nombre del pickup point, su dirección, horario y teléfono de contacto. No hay campo de repartidor (no aplica para PICKUP). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle con información del pickup point. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-173 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Pedido PICKUP expirado — el comprador no recogió en tiempo |
| **Rol** | Sistema |
| **Precondiciones** | Ejecutado PT-161 (expiration). Verificar que el pedido queda con closure_reason correcto. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Verificar el pedido expirado de PT-161. 2. Revisar el detalle para ver closure_reason = PICKUP_EXPIRED. 3. Verificar refund_status si aplica. |
| **Resultado esperado** | El pedido cerrado tiene closure_reason = PICKUP_EXPIRED. Si había pago online, refund_status = REFUND_PENDING. Si era COD, no se registró cobro. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle del pedido expirado con closure_reason visible. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-174 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Aislamiento PICKUP — acceso a pedido PICKUP de otro tenant |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pedidos PICKUP en cliente-piloto y cliente-dos. |
| **Datos de prueba** | ID de pedido PICKUP de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. En Postman, GET `/admin/orders/{id_pickup_de_cliente_dos}`. |
| **Resultado esperado** | HTTP 403 o 404. Aislamiento multi-tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta de error. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-175 |
| **Módulo** | Pedidos — PICKUP Flujo |
| **Funcionalidad** | Crear pedido PICKUP sin pickup point activo disponible |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Todos los pickup points del cliente están inactivos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Desactivar todos los pickup points. 2. Intentar crear un pedido con delivery_type = PICKUP. 3. Verificar el selector de pickup point. |
| **Resultado esperado** | El sistema informa que no hay puntos de retiro activos disponibles. No se puede completar la creación del pedido PICKUP. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error o selector vacío. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE I — Vista del Repartidor / COURIER (PT-176 a PT-200)

---

| Campo | Valor |
|---|---|
| **ID** | PT-176 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Ver lista de pedidos asignados — tabs Activos/Completados |
| **Rol** | COURIER |
| **Precondiciones** | Courier con pedidos asignados en distintos estados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como COURIER. 2. Ver la pantalla `/courier`. 3. Revisar el tab "Activos". 4. Cambiar al tab "Completados". |
| **Resultado esperado** | Tab "Activos" muestra pedidos en estados activos (asignado, en tránsito, etc.). Tab "Completados" muestra los pedidos entregados del día. Solo aparecen pedidos asignados a este courier. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de ambos tabs en vista mobile. |
| **Observaciones** | |
| **Responsable** | QA funcional / QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-177 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Ver detalle de pedido asignado |
| **Rol** | COURIER |
| **Precondiciones** | Pedido asignado al courier. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En `/courier`, hacer clic en una tarjeta de pedido. |
| **Resultado esperado** | Se muestra: nombre del comprador con opción "Llamar", dirección con opción "Ver en mapa", items del pedido, total, método de pago, monto a cobrar (si COD). Botones de acción grandes (min 44px) según el estado actual. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del detalle del pedido en vista mobile. |
| **Observaciones** | Botones deben ser usables en móvil: min 44px de altura. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-178 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | COURIER no puede ver pedidos de otros couriers |
| **Rol** | COURIER |
| **Precondiciones** | Existen pedidos asignados a otros couriers. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como COURIER (courier A). 2. En `/courier`, ver la lista de pedidos. 3. Intentar acceder directamente a la URL `/courier/{id_pedido_de_otro_courier}`. |
| **Resultado esperado** | Solo aparecen los pedidos asignados al courier A. Al intentar acceder a un pedido de otro courier, el sistema retorna 403 o "no encontrado". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del intento de acceso a pedido de otro courier. |
| **Observaciones** | Aislamiento de datos crítico. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-179 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Secuencia completa de acciones de entrega |
| **Rol** | COURIER |
| **Precondiciones** | Pedido asignado en estado COURIER_ASSIGNED con pago online ya confirmado. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el pedido. 2. Tocar "Iniciar despacho" (SHIPPED). 3. Tocar "Marcar en camino" (IN_TRANSIT). 4. Tocar "Salir a reparto" (OUT_FOR_DELIVERY). 5. Tocar "Llegué al domicilio" (ARRIVED_AT_CUSTOMER). 6. Tocar "Marcar como entregado" (DELIVERED). |
| **Resultado esperado** | Cada botón está disponible en el momento correcto del flujo. Los estados se actualizan correctamente. Las acciones son grandes y fáciles de tocar en móvil. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de cada estado en la app del courier. |
| **Observaciones** | |
| **Responsable** | QA funcional / QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-180 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Volver a la lista desde detalle del pedido |
| **Rol** | COURIER |
| **Precondiciones** | Dentro del detalle de un pedido. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Desde el detalle de un pedido, tocar el botón "← Volver". |
| **Resultado esperado** | El sistema navega de vuelta a la lista de pedidos sin perder el estado de los tabs. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista al volver. |
| **Observaciones** | |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-181 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Devolver pedido al origen |
| **Rol** | COURIER |
| **Precondiciones** | Pedido donde el comprador no estaba (DELIVERY_ATTEMPTED) después de 3 intentos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En el pedido en DELIVERY_ATTEMPTED (o donde el admin indica retorno). 2. Tocar "Devolver al origen". 3. Confirmar. |
| **Resultado esperado** | El pedido pasa a RETURN_TO_ORIGIN. La admin recibe la notificación. El courier debe llevar el paquete al punto de origen. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado RETURN_TO_ORIGIN. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-182 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Responsive y usabilidad en móvil 375px |
| **Rol** | COURIER |
| **Precondiciones** | Dispositivo o emulador a 375px de ancho. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir la app en dispositivo o emulador de 375px. 2. Navegar por la lista de pedidos. 3. Abrir detalle de un pedido. 4. Verificar que todos los botones son visibles y tocables. 5. Verificar que el texto no está cortado. 6. Verificar scroll vertical. |
| **Resultado esperado** | Toda la interfaz es legible y funcional a 375px. Los botones son grandes (mín 44px). No hay scroll horizontal. El texto no se desborda. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas a 375px de lista y detalle. |
| **Observaciones** | Interfaz mobile-first según los wireframes. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-183 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Datos del comprador — botón "Llamar" abre marcador |
| **Rol** | COURIER |
| **Precondiciones** | Pedido asignado con teléfono del comprador. Dispositivo móvil. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido en la vista COURIER. 2. Tocar el botón "Llamar" junto al nombre del comprador. |
| **Resultado esperado** | El dispositivo abre la app de llamadas con el número del comprador prellenado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del marcador abierto con el número. |
| **Observaciones** | Usar `tel:` URI scheme. Probar en dispositivo real o emulador. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-184 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Datos del comprador — botón "Ver en mapa" abre navegación |
| **Rol** | COURIER |
| **Precondiciones** | Pedido asignado con dirección de entrega geocodificada. Dispositivo móvil. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido. 2. Tocar el botón "Ver en mapa" o "Cómo llegar". |
| **Resultado esperado** | El dispositivo abre la app de mapas (Google Maps, Waze u OSM) con la dirección de entrega como destino. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la app de mapas abierta con el destino. |
| **Observaciones** | Usar `geo:` URI o `https://maps.google.com?q=...`. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-185 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | La vista COURIER no carga pedidos de otras jornadas |
| **Rol** | COURIER |
| **Precondiciones** | Courier con pedidos de días anteriores ya cerrados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como COURIER. 2. Revisar el tab "Activos". 3. Revisar el tab "Completados". |
| **Resultado esperado** | El tab "Activos" solo muestra pedidos activos (no cerrados). El tab "Completados" muestra solo los del día actual (hoy). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del tab completados mostrando solo pedidos de hoy. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-186 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Mostrar monto a cobrar (COD) prominentemente en el detalle |
| **Rol** | COURIER |
| **Precondiciones** | Pedido COD asignado al courier. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle de un pedido COD. 2. Verificar que el monto a cobrar es visible. |
| **Resultado esperado** | El monto a cobrar se muestra de forma prominente y clara (ej. "Cobrar: $35.000"). El método de pago (efectivo/electrónico) también se indica. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del monto destacado en el detalle. |
| **Observaciones** | El courier debe saber exactamente cuánto cobrar antes de llegar. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-187 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Subir evidencia fotográfica — validación de tamaño |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en pantalla de cobro. |
| **Datos de prueba** | Imagen de más de 10MB. |
| **Pasos** | 1. En la pantalla de registro de cobro. 2. Intentar subir una imagen > 10MB. |
| **Resultado esperado** | El sistema rechaza la imagen con mensaje indicando el límite de tamaño. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de tamaño. |
| **Observaciones** | `storage.max_file_size_mb = 10`. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-188 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Vista COURIER en landscape (modo horizontal) |
| **Rol** | COURIER |
| **Precondiciones** | Dispositivo o emulador con rotación activada. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir la vista COURIER. 2. Rotar el dispositivo a modo landscape. |
| **Resultado esperado** | La interfaz se adapta correctamente en landscape. Los botones siguen siendo accesibles. No hay desbordamiento de contenido. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura en modo landscape. |
| **Observaciones** | |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-189 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Estado vacío — courier sin pedidos asignados |
| **Rol** | COURIER |
| **Precondiciones** | Courier sin pedidos asignados. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como un COURIER sin pedidos asignados. 2. Ver la pantalla principal `/courier`. |
| **Resultado esperado** | La pantalla muestra un mensaje de estado vacío amigable (ej. "No tienes pedidos asignados por ahora"). No hay error. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío en la vista COURIER. |
| **Observaciones** | |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-190 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Resumen del día en la vista COURIER |
| **Rol** | COURIER |
| **Precondiciones** | Courier con pedidos completados el día de hoy. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al tab "Completados" de la vista COURIER. 2. Ver el resumen del día. |
| **Resultado esperado** | Se muestra un resumen con: pedidos entregados hoy, total recaudado en efectivo (COD), total recaudado electrónico. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resumen del día. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-191 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Botones de acción desactivados en estados donde no aplican |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en IN_TRANSIT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir el detalle del pedido en IN_TRANSIT. 2. Verificar los botones disponibles. 3. Intentar acceder a acciones futuras no disponibles aún. |
| **Resultado esperado** | Solo el botón correspondiente al siguiente estado está activo. Los demás botones están desactivados o no visibles. Esto evita errores de transición inválida. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los botones activos/inactivos en cada estado. |
| **Observaciones** | |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-192 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Confirmación antes de ejecutar acción crítica |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en ARRIVED_AT_CUSTOMER con pago online. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Tocar "Marcar como entregado". 2. Verificar si aparece un diálogo de confirmación. |
| **Resultado esperado** | El sistema muestra un diálogo de confirmación antes de marcar el pedido como entregado (acción irreversible). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del diálogo de confirmación. |
| **Observaciones** | Las acciones irreversibles deben requerir confirmación. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-193 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | La vista COURIER carga rápido en 3G |
| **Rol** | COURIER |
| **Precondiciones** | Throttle de red a "Slow 3G" en DevTools. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Activar throttle "Slow 3G". 2. Navegar a `/courier`. 3. Medir el tiempo de carga. |
| **Resultado esperado** | La lista de pedidos carga en menos de 5 segundos en condiciones de red 3G. Se muestra un indicador de carga mientras espera. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del DevTools Network con tiempos de carga. |
| **Observaciones** | La performance es crítica para la experiencia del repartidor en campo. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-194 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Vista COURIER en iOS Safari |
| **Rol** | COURIER |
| **Precondiciones** | Dispositivo iOS o simulador con Safari. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir Safari en iOS. 2. Navegar a la URL de la app. 3. Iniciar sesión como COURIER. 4. Navegar por la lista y el detalle. |
| **Resultado esperado** | La interfaz funciona correctamente en Safari iOS. Los botones son tocables. No hay problemas de viewport o overflow. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la vista en iOS Safari. |
| **Observaciones** | iOS Safari tiene particularidades con las cookies y el viewport. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-195 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Vista COURIER en Android Chrome |
| **Rol** | COURIER |
| **Precondiciones** | Dispositivo Android o emulador con Chrome. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir Chrome en Android. 2. Navegar a la app. 3. Iniciar sesión como COURIER. 4. Navegar y ejecutar acciones. |
| **Resultado esperado** | La interfaz funciona correctamente en Android Chrome. Todos los botones y formularios son funcionales. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la vista en Android Chrome. |
| **Observaciones** | |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-196 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Agregar como PWA en la pantalla de inicio |
| **Rol** | COURIER |
| **Precondiciones** | Navegador compatible con PWA instalation (Chrome Android / Safari iOS). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En Chrome Android, ir a la URL de la app. 2. Usar "Agregar a la pantalla de inicio". 3. Abrir desde el icono de la pantalla de inicio. |
| **Resultado esperado** | La app se puede añadir a la pantalla de inicio. Al abrir desde el icono, se carga correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del icono en la pantalla de inicio y de la app abierta. |
| **Observaciones** | Si la app tiene manifest.json configurado. Verificar si aplica. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-197 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Logout del courier |
| **Rol** | COURIER |
| **Precondiciones** | Sesión activa como COURIER. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En la vista COURIER, buscar la opción de logout. 2. Hacer logout. |
| **Resultado esperado** | La sesión del courier se cierra. Las cookies se eliminan. Al intentar volver a `/courier`, se redirige a `/login`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la redirección a `/login`. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-198 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Token COURIER expira a los 30 minutos (refresh automático) |
| **Rol** | COURIER |
| **Precondiciones** | Sesión activa como COURIER. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como COURIER. 2. Esperar 30 minutos sin cerrar la app (o ajustar el token). 3. Intentar realizar una acción. |
| **Resultado esperado** | El sistema renueva automáticamente el access token del courier usando el refresh token. La operación continúa sin interrupción. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de DevTools mostrando el request de refresh y la respuesta 200. |
| **Observaciones** | JWT COURIER tiene 30 min de duración (vs 15 min de admin). |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-199 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Foto de evidencia tomada con cámara del dispositivo |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en pantalla de cobro. Dispositivo con cámara. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En la pantalla de cobro, tocar el botón de evidencia. 2. Seleccionar "Tomar foto". 3. Tomar una foto con la cámara. 4. Confirmar. |
| **Resultado esperado** | La foto tomada con la cámara queda adjunta como evidencia. El sistema la sube correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la foto adjunta en el registro de cobro. |
| **Observaciones** | Usar `<input type="file" capture="camera">` o API de cámara. |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-200 |
| **Módulo** | Vista COURIER |
| **Funcionalidad** | Nota de intento fallido es obligatoria |
| **Rol** | COURIER |
| **Precondiciones** | Pedido en OUT_FOR_DELIVERY. |
| **Datos de prueba** | Sin ingresar nota. |
| **Pasos** | 1. Tocar "Intento fallido" en el pedido. 2. No ingresar nota de motivo. 3. Intentar confirmar. |
| **Resultado esperado** | El sistema valida que se ingrese un motivo antes de registrar el intento fallido. Muestra error de validación. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación de la nota. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE J — Configuración del Cliente (PT-201 a PT-230)

---

| Campo | Valor |
|---|---|
| **ID** | PT-201 |
| **Módulo** | Configuración — Info corporativa |
| **Funcionalidad** | Editar información corporativa del negocio |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | Nuevo nombre: "Cliente Piloto Actualizado", Nueva descripción: "El mejor restaurante de Bogotá". |
| **Pasos** | 1. Ir a Configuración (menú lateral). 2. Verificar que está en el tab "Información". 3. Editar el nombre y descripción. 4. Hacer clic en "Guardar". |
| **Resultado esperado** | Los datos se actualizan. El nombre nuevo aparece en el header del admin. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los datos guardados. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-202 |
| **Módulo** | Configuración — Info corporativa |
| **Funcionalidad** | Subir logo del negocio |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Archivo PNG de logo, < 10MB. |
| **Pasos** | 1. Ir a Configuración → tab Información. 2. Hacer clic en el área de logo. 3. Seleccionar un archivo PNG válido. 4. Guardar. |
| **Resultado esperado** | El logo se sube y aparece en el header del admin. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del logo visible en el header. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-203 |
| **Módulo** | Configuración — Wompi |
| **Funcionalidad** | Configurar credenciales Wompi sandbox |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Credenciales Wompi sandbox disponibles. |
| **Datos de prueba** | Public key: `pub_stagtest_...`, Private key: `prv_stagtest_...`, Events secret: `stagtest_events_...`. |
| **Pasos** | 1. Ir a Configuración → tab "Wompi". 2. Hacer clic en "Agregar Wompi". 3. Ingresar Public key, Private key y Events secret. 4. Guardar. |
| **Resultado esperado** | Las credenciales se guardan. La URL del webhook generada automáticamente se muestra en modo solo lectura. El estado de Wompi cambia a "Configurado". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la configuración guardada y la URL del webhook visible. |
| **Observaciones** | La URL del webhook es generada por RUTA, no ingresada por el usuario. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-204 |
| **Módulo** | Configuración — Wompi |
| **Funcionalidad** | Intentar guardar Wompi con public key vacía |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Public key: vacía. |
| **Pasos** | 1. Ir a Configuración → tab Wompi. 2. Hacer clic en "Agregar Wompi". 3. Dejar Public key vacía. 4. Ingresar Private key y Events secret. 5. Guardar. |
| **Resultado esperado** | El formulario muestra error de validación indicando que la Public key es obligatoria. No se guarda. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-205 |
| **Módulo** | Configuración — Webhooks salientes |
| **Funcionalidad** | Crear suscripción de webhook saliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. Endpoint externo disponible para recibir webhooks (usar RequestBin o similar). |
| **Datos de prueba** | URL: `https://webhook.site/test-uuid`, Eventos: `order.status_changed`, `order.delivered`. |
| **Pasos** | 1. Ir a Configuración → tab "Webhooks". 2. Hacer clic en "Agregar webhook". 3. Ingresar URL de destino. 4. Seleccionar eventos `order.status_changed` y `order.delivered`. 5. Guardar. |
| **Resultado esperado** | La suscripción se crea. Aparece en la lista con la URL, los eventos seleccionados y el signing secret generado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la suscripción creada con los eventos. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-206 |
| **Módulo** | Configuración — Webhooks salientes |
| **Funcionalidad** | Verificar que RUTA envía webhook cuando cambia estado de pedido |
| **Rol** | Sistema |
| **Precondiciones** | Webhook creado en PT-205. Tener el endpoint de RequestBin activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Tomar un pedido y avanzar su estado (ej. de PREPARING a AWAITING_COURIER_ASSIGNMENT). 2. Esperar unos segundos. 3. Verificar en RequestBin si llegó el webhook. |
| **Resultado esperado** | RUTA envía el webhook a la URL configurada con el payload del cambio de estado. El header `X-Ruta-Signature` está presente y puede verificarse con el signing secret. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del webhook recibido en RequestBin con el payload y la firma. |
| **Observaciones** | Algoritmo: HMAC-SHA256. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-207 |
| **Módulo** | Configuración — Webhooks salientes |
| **Funcionalidad** | Editar URL de webhook existente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe una suscripción de webhook creada. |
| **Datos de prueba** | Nueva URL: `https://webhook.site/nueva-url`. |
| **Pasos** | 1. Ir a Configuración → tab Webhooks. 2. Hacer clic en editar la suscripción existente. 3. Cambiar la URL. 4. Guardar. |
| **Resultado esperado** | La URL se actualiza. Los nuevos webhooks se envían a la nueva URL. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la suscripción con la URL actualizada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-208 |
| **Módulo** | Configuración — Webhooks salientes |
| **Funcionalidad** | Ver historial de entregas de webhook |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Se han enviado webhooks (PT-206 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → tab Webhooks. 2. Hacer clic en "Ver historial" de la suscripción. |
| **Resultado esperado** | Se muestra la lista de entregas con: fecha, resultado (exitosa/fallida), código HTTP de respuesta. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial de entregas. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-209 |
| **Módulo** | Configuración — Webhooks salientes |
| **Funcionalidad** | Reintentar webhook fallido manualmente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe una entrega de webhook fallida en el historial. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al historial de webhooks. 2. En una entrega fallida, hacer clic en "Reintentar". |
| **Resultado esperado** | RUTA reenvía el webhook a la URL configurada. El historial registra el nuevo intento con su resultado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial mostrando el reintento. |
| **Observaciones** | Los reintentos automáticos tienen intervalos: 1m, 5m, 15m, 60m, 6h, 24h. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-210 |
| **Módulo** | Configuración — Webhooks salientes |
| **Funcionalidad** | Eliminar suscripción de webhook |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existe una suscripción de webhook. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → tab Webhooks. 2. Hacer clic en eliminar una suscripción. 3. Confirmar. |
| **Resultado esperado** | La suscripción es eliminada. No se envían más webhooks a esa URL. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista sin la suscripción eliminada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-211 |
| **Módulo** | Configuración — Parámetros |
| **Funcionalidad** | Ver lista de parámetros de negocio |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Parámetros globales configurados en BD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → tab "Parámetros". |
| **Resultado esperado** | Se muestra la lista de parámetros con: clave, valor global de plataforma, valor override del cliente (si existe), descripción. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista de parámetros. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-212 |
| **Módulo** | Configuración — Parámetros |
| **Funcionalidad** | Sobrescribir un parámetro con valor propio del cliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Parámetro: `order.pending_confirm_timeout_minutes`. Nuevo valor: 90. |
| **Pasos** | 1. Ir a Configuración → tab Parámetros. 2. Hacer clic en el parámetro `order.pending_confirm_timeout_minutes`. 3. Ingresar valor 90. 4. Guardar. |
| **Resultado esperado** | El parámetro queda con override = 90. El sistema usará 90 minutos para este cliente en lugar del valor global (60). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del parámetro con el valor override guardado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-213 |
| **Módulo** | Configuración — Parámetros |
| **Funcionalidad** | Limpiar override de parámetro (volver al valor global) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Parámetro con override configurado en PT-212. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → tab Parámetros. 2. En el parámetro con override, limpiar el campo. 3. Guardar. |
| **Resultado esperado** | El override se elimina. El sistema usa nuevamente el valor global de la plataforma. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del parámetro sin override. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-214 |
| **Módulo** | Configuración — Parámetros |
| **Funcionalidad** | OPERATOR_CLIENT no puede editar parámetros |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Sesión activa como OPERATOR_CLIENT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Intentar navegar a `/admin/settings` como OPERATOR_CLIENT. |
| **Resultado esperado** | El sistema muestra "Acceso restringido". OPERATOR_CLIENT no puede editar parámetros (solo ADMIN_CLIENT y ADMIN_RUTA). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del acceso denegado. |
| **Observaciones** | Según matriz: editar parámetros del cliente = ❌ para OPERATOR_CLIENT. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

### FASE K — Auditoría y Métricas (PT-215 a PT-235)

---

| Campo | Valor |
|---|---|
| **ID** | PT-215 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Ver log de auditoría como ADMIN_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Se han realizado acciones en el sistema (pedidos, creación de productos, etc.). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al menú lateral → Auditoría. |
| **Resultado esperado** | Se muestra la tabla de eventos con: fecha, actor (usuario o API key), acción, entidad, resultado, IP. Solo aparecen eventos del propio cliente (no de otros tenants). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del log de auditoría. |
| **Observaciones** | Aislamiento: un ADMIN_CLIENT no debe ver logs de otros clientes. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-216 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Filtrar auditoría por rango de fechas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log de auditoría con eventos en distintas fechas. |
| **Datos de prueba** | Fecha desde: hoy, Fecha hasta: hoy. |
| **Pasos** | 1. Ir a Auditoría. 2. Ingresar el rango de fechas de hoy. 3. Aplicar filtro. |
| **Resultado esperado** | Solo aparecen eventos ocurridos hoy. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del filtro aplicado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-217 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Filtrar auditoría por usuario/actor |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log con eventos de distintos usuarios. |
| **Datos de prueba** | Actor: email del OPERATOR_CLIENT. |
| **Pasos** | 1. Ir a Auditoría. 2. Ingresar el email del OPERATOR_CLIENT en el filtro de usuario. 3. Aplicar. |
| **Resultado esperado** | Solo aparecen eventos generados por ese usuario. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del filtro aplicado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-218 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Filtrar auditoría por tipo de acción |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log con distintos tipos de acciones. |
| **Datos de prueba** | Acción: "cancelar" o "order.cancel". |
| **Pasos** | 1. Ir a Auditoría. 2. Filtrar por acción "cancelar". 3. Aplicar. |
| **Resultado esperado** | Solo aparecen eventos de cancelación de pedidos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-219 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Ver detalle completo de un evento de auditoría |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log con eventos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Auditoría. 2. Hacer clic en cualquier fila del log. |
| **Resultado esperado** | Se abre un modal o panel con el detalle completo del evento: fecha, actor, acción, entidad, resultado, IP, y metadata JSON completa. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del modal de detalle con el JSON de metadata. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-220 |
| **Módulo** | Dashboard ADMIN_CLIENT |
| **Funcionalidad** | Ver métricas del dashboard de hoy |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos creados y procesados el día de hoy. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Ir a `/admin/dashboard`. |
| **Resultado esperado** | Se muestran 4 métricas: "Pedidos de hoy" (total del día), "Pedidos en tránsito", "Entregados hoy", "Ingresos en tránsito". Las métricas son coherentes con los pedidos existentes. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard con las 4 métricas. |
| **Observaciones** | Verificar que las métricas solo reflejan datos del propio cliente. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-221 |
| **Módulo** | Dashboard ADMIN_CLIENT |
| **Funcionalidad** | Acciones rápidas del dashboard |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa en el dashboard. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En el dashboard, hacer clic en "Ver pedidos pendientes". 2. Volver al dashboard. 3. Hacer clic en "Abrir mapa de asignación". 4. Volver al dashboard. 5. Hacer clic en "Crear producto". |
| **Resultado esperado** | Cada acceso rápido navega correctamente a la sección correspondiente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de cada sección alcanzada. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-222 |
| **Módulo** | Dashboard ADMIN_CLIENT |
| **Funcionalidad** | Tabla de últimos 10 pedidos en el dashboard |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen más de 10 pedidos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al dashboard. 2. Revisar la sección "Últimos pedidos". |
| **Resultado esperado** | Se muestran los 10 pedidos más recientes con su estado. Hacer clic en uno navega al detalle. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la tabla de últimos pedidos. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-223 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Acción de Vista de Control registrada en auditoría del cliente |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | ADMIN_RUTA dentro de Vista de Control de cliente-piloto. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Entrar a Vista de Control. 2. Realizar una acción (ej. crear una categoría). 3. Salir de Vista de Control. 4. Ir a Auditoría del cliente. 5. Buscar la acción realizada. |
| **Resultado esperado** | La acción aparece en el log con `acting_via_control_view = TRUE` y el actor real es el email del ADMIN_RUTA. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento de auditoría con el flag de Vista de Control. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-224 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Exportar log de auditoría (si existe la funcionalidad) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log de auditoría con eventos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Auditoría. 2. Verificar si existe botón de exportar. 3. Si existe, exportar. |
| **Resultado esperado** | Si existe la función, se descarga un archivo CSV o Excel con los eventos filtrados. Si no existe, documentar la ausencia. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del botón de exportación o nota de ausencia. |
| **Observaciones** | Verificar si esta funcionalidad está en el alcance del MVP. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-225 |
| **Módulo** | Dashboard ADMIN_CLIENT |
| **Funcionalidad** | Dashboard muestra pedidos en tránsito en tiempo real |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos activos en estados de tránsito. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al dashboard. 2. Tomar nota del número de "Pedidos en tránsito". 3. Cambiar el estado de un pedido activo. 4. Refrescar el dashboard. |
| **Resultado esperado** | El contador de "Pedidos en tránsito" se actualiza reflejando el cambio. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del dashboard antes y después del cambio. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-226 |
| **Módulo** | Dashboard ADMIN_CLIENT |
| **Funcionalidad** | Métricas de ingresos en tránsito |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos con distintos montos activos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al dashboard. 2. Verificar el widget "Ingresos en tránsito". 3. Comparar con la suma manual de pedidos activos. |
| **Resultado esperado** | El valor de ingresos en tránsito coincide con la suma de totales de pedidos en estados activos (SHIPPED, IN_TRANSIT, OUT_FOR_DELIVERY, ARRIVED_AT_CUSTOMER). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del widget con el valor y verificación manual. |
| **Observaciones** | Solo aplica para montos COD pendientes de cobro. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-227 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Paginación del log de auditoría |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Más de 20 eventos en el log. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Auditoría. 2. Verificar paginación. 3. Clic en "Siguiente". |
| **Resultado esperado** | La paginación funciona. Cada página muestra 20 eventos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de páginas. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-228 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Filtrar auditoría combinando múltiples criterios |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log con múltiples tipos de eventos. |
| **Datos de prueba** | Filtro: fecha = hoy + actor = OPERATOR_CLIENT. |
| **Pasos** | 1. Ir a Auditoría. 2. Aplicar filtro de fecha (hoy) y actor (email del operador). |
| **Resultado esperado** | Solo aparecen eventos del día de hoy realizados por el OPERATOR_CLIENT especificado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los filtros combinados aplicados. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-229 |
| **Módulo** | Dashboard ADMIN_RUTA |
| **Funcionalidad** | Dashboard ADMIN_RUTA muestra clientes activos correctamente |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existen clientes activos e inactivos. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_RUTA. 2. Ir al dashboard. 3. Ver el contador de clientes activos. |
| **Resultado esperado** | El contador refleja solo los clientes activos. Si se activa/desactiva un cliente, el contador se actualiza al refrescar. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard con el contador correcto. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-230 |
| **Módulo** | Dashboard ADMIN_RUTA |
| **Funcionalidad** | Dashboard ADMIN_RUTA muestra pedidos totales de la plataforma |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Pedidos en múltiples tenants. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir al dashboard de ADMIN_RUTA. 2. Ver el contador de pedidos totales del día. |
| **Resultado esperado** | El contador muestra el total de pedidos de todos los tenants activos del día actual. No es filtrado por tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard con el total de pedidos. |
| **Observaciones** | Esta vista solo es accesible para ADMIN_RUTA. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-231 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Log de auditoría no registra datos sensibles |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log con eventos de cambio de contraseña. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Cambiar contraseña (PT-036). 2. Ir a Auditoría. 3. Abrir el evento de cambio de contraseña. 4. Revisar el JSON de metadata. |
| **Resultado esperado** | El log registra el evento pero NO incluye la contraseña (ni en texto plano ni hasheada). Solo registra el actor, la acción y la fecha. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento mostrando que no hay contraseña en el metadata. |
| **Observaciones** | Prueba de seguridad básica. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-232 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Log de Wompi webhook registrado correctamente |
| **Rol** | Sistema |
| **Precondiciones** | Webhook de Wompi recibido y procesado. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ejecutar PT-140 (pago Wompi). 2. Ir a Auditoría. 3. Buscar el evento del webhook de Wompi. |
| **Resultado esperado** | El log registra el evento del webhook con: tipo de evento, transaction_id, monto, y resultado del procesamiento. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento de webhook en el log. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-233 |
| **Módulo** | Dashboard ADMIN_CLIENT |
| **Funcionalidad** | Dashboard responsive — se ve bien en tablet 768px |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | DevTools a 768px. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir DevTools → 768px. 2. Ir al dashboard. 3. Verificar los widgets de métricas. |
| **Resultado esperado** | Los 4 widgets de métricas se muestran en layout adaptado a tablet. No hay desbordamiento. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard a 768px. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-234 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Log de importación masiva de productos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Importación masiva ejecutada (PT-054). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ejecutar PT-054 (importar Excel). 2. Ir a Auditoría. 3. Buscar el evento de importación. |
| **Resultado esperado** | El log registra el evento de importación con: actor, fecha, número de productos importados. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento de importación en auditoría. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-235 |
| **Módulo** | Auditoría |
| **Funcionalidad** | Búsqueda en auditoría por entidad (ej. ID de pedido) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Log con eventos de un pedido específico. |
| **Datos de prueba** | ID del pedido de prueba. |
| **Pasos** | 1. Ir a Auditoría. 2. Buscar por el ID del pedido. |
| **Resultado esperado** | Se muestran todos los eventos relacionados con ese pedido (aceptación, cambios de estado, cobro, etc.). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los eventos del pedido en auditoría. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE L — Vista de Control / ADMIN_RUTA (PT-236 a PT-250)

---

| Campo | Valor |
|---|---|
| **ID** | PT-236 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Acceder a Vista de Control con master password correcto |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | ADMIN_RUTA con permiso `can_use_control_view`. Master password configurado. |
| **Datos de prueba** | Cliente objetivo: cliente-piloto. Master password: `MasterTest2026!`. Razón: "Soporte ticket #TEST-001". |
| **Pasos** | 1. Iniciar sesión como ADMIN_RUTA. 2. Ir a `/ruta-admin/clients`. 3. Hacer clic en "cliente-piloto". 4. Hacer clic en "Vista de Control". 5. Ingresar master password `MasterTest2026!`. 6. Ingresar razón. 7. Hacer clic en "Entrar a Vista de Control". |
| **Resultado esperado** | El sistema redirige a `/admin/dashboard` actuando sobre cliente-piloto. Aparece un banner ámbar fijo en la parte superior: "Estás en Vista de Control del Cliente [nombre]. Todas tus acciones quedan auditadas. [Salir de Vista de Control]". |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del banner ámbar en la parte superior del dashboard. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-237 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Acceder a Vista de Control con master password incorrecto |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Sesión activa como ADMIN_RUTA. |
| **Datos de prueba** | Master password incorrecto: `ClaveIncorrecta!`. |
| **Pasos** | 1. Ir a Vista de Control de cliente-piloto. 2. Ingresar master password incorrecto. 3. Hacer clic en "Entrar a Vista de Control". |
| **Resultado esperado** | El sistema rechaza el acceso con mensaje de error. No se crea sesión de Vista de Control. El intento queda registrado en el log de auditoría. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de acceso denegado. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-238 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Acciones en Vista de Control quedan auditadas con flag |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | ADMIN_RUTA dentro de Vista de Control de cliente-piloto (PT-236 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Con el banner ámbar activo, realizar una acción como editar un producto. 2. Ir a Auditoría. 3. Buscar el evento de edición recién realizado. |
| **Resultado esperado** | El evento aparece en el log de auditoría con `acting_via_control_view = TRUE` y el actor real es el ADMIN_RUTA (no el ADMIN_CLIENT del tenant). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento en auditoría mostrando el flag `acting_via_control_view = TRUE`. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-239 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Salir de Vista de Control y volver a sesión ADMIN_RUTA |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | Dentro de Vista de Control (banner ámbar visible). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Hacer clic en "Salir de Vista de Control" en el banner ámbar. |
| **Resultado esperado** | La sesión vuelve a ser la del ADMIN_RUTA. El banner ámbar desaparece. El sistema redirige a `/ruta-admin/clients`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la vista `/ruta-admin/clients` sin el banner ámbar. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-240 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | ADMIN_CLIENT no puede acceder a Vista de Control |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa como ADMIN_CLIENT. |
| **Datos de prueba** | URL objetivo: `/ruta-admin/control-view/...` |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Intentar navegar a `/ruta-admin/control-view/1`. |
| **Resultado esperado** | El sistema muestra acceso denegado o redirige a `/admin/dashboard`. El ADMIN_CLIENT no puede entrar a Vista de Control sobre ningún cliente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del acceso denegado. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-241 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Banner ámbar se muestra en todas las páginas durante la sesión de control |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | Dentro de Vista de Control (PT-236 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Con Vista de Control activa, navegar a diferentes secciones: Dashboard, Pedidos, Catálogo, Configuración. |
| **Resultado esperado** | El banner ámbar está fijo en la parte superior en todas las páginas. El banner NO desaparece al navegar. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del banner visible en 3 páginas diferentes. |
| **Observaciones** | El banner es persistente durante toda la sesión de control. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-242 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | ADMIN_RUTA puede gestionar productos del cliente en Vista de Control |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | Dentro de Vista de Control de cliente-piloto. |
| **Datos de prueba** | Nombre: "Producto Control Test", Precio: $15.000. |
| **Pasos** | 1. Con Vista de Control activa, ir a Catálogo → Productos. 2. Crear un nuevo producto. 3. Verificar que el producto queda bajo cliente-piloto. |
| **Resultado esperado** | El producto se crea bajo el tenant cliente-piloto (no bajo un posible tenant del ADMIN_RUTA). La acción queda auditada. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del producto creado en cliente-piloto. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-243 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | ADMIN_RUTA no puede entrar a Vista de Control de un cliente inactivo |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Existe un cliente inactivo. |
| **Datos de prueba** | Cliente inactivo (PT-025 ejecutado). |
| **Pasos** | 1. Ir a `/ruta-admin/clients`. 2. Hacer clic en el cliente inactivo. 3. Intentar entrar a Vista de Control. |
| **Resultado esperado** | El sistema rechaza la Vista de Control para clientes inactivos con un mensaje apropiado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-244 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Timeout de sesión de Vista de Control |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | Dentro de Vista de Control. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Entrar a Vista de Control. 2. Verificar si hay un timeout de sesión específico para Vista de Control. 3. Si existe, esperar el timeout. |
| **Resultado esperado** | La sesión de Vista de Control expira según el parámetro configurado. Al expirar, el ADMIN_RUTA es redirigido a su vista normal. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la expiración de la sesión de control. |
| **Observaciones** | Verificar si existe parámetro `control_view.session_timeout_minutes`. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-245 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Log de acceso a Vista de Control registrado en auditoría global |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Vista de Control accedida (PT-236 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ejecutar PT-236 (acceso a Vista de Control). 2. Como ADMIN_RUTA, ir al log de auditoría global (`/ruta-admin/audit`). 3. Buscar el evento de acceso a Vista de Control. |
| **Resultado esperado** | El log global registra: quién (ADMIN_RUTA), a qué cliente accedió, la razón ingresada, la IP y el timestamp. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento de acceso a Vista de Control en el log global. |
| **Observaciones** | La trazabilidad de acceso es crítica para la seguridad. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-246 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Master password con rate limiting (3 intentos) |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Sesión activa como ADMIN_RUTA. |
| **Datos de prueba** | Master password incorrecto: "IntentosInvalidos!". |
| **Pasos** | 1. Intentar ingresar a Vista de Control con master password incorrecto 3 veces consecutivas. |
| **Resultado esperado** | Después de N intentos fallidos (verificar el límite configurado), el sistema bloquea temporalmente los intentos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del bloqueo por intentos fallidos. |
| **Observaciones** | Verificar el valor del parámetro de bloqueo para Vista de Control. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-247 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | ADMIN_RUTA con Vista de Control ve auditoría del cliente |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | Dentro de Vista de Control. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Con Vista de Control activa, ir a la sección de Auditoría. |
| **Resultado esperado** | Se muestra el log de auditoría del cliente objetivo. Los eventos incluyen los realizados por el ADMIN_RUTA en Vista de Control (marcados con el flag). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del log de auditoría visto desde Vista de Control. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-248 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Razón de Vista de Control es obligatoria |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Pantalla de acceso a Vista de Control. |
| **Datos de prueba** | Razón: vacía. |
| **Pasos** | 1. Ir a Vista de Control de un cliente. 2. Ingresar master password correcto. 3. Dejar el campo de razón vacío. 4. Clic en "Entrar". |
| **Resultado esperado** | El sistema valida que se ingrese una razón. Muestra error de validación. No se otorga acceso sin razón. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación del campo razón. |
| **Observaciones** | La razón queda registrada en el log de auditoría. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-249 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Historial de sesiones de Vista de Control |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Se han realizado múltiples accesos a Vista de Control. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Como ADMIN_RUTA, buscar el historial de sesiones de Vista de Control. 2. Puede ser en el log de auditoría global o en una sección dedicada. |
| **Resultado esperado** | Se puede ver quién accedió a Vista de Control, para qué cliente, cuándo, la razón ingresada y cuánto tiempo duró la sesión. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial de sesiones de Vista de Control. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-250 |
| **Módulo** | Vista de Control |
| **Funcionalidad** | Vista de Control no hereda permisos de operador |
| **Rol** | ADMIN_RUTA (en Vista de Control) |
| **Precondiciones** | Dentro de Vista de Control. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Con Vista de Control activa, intentar realizar acciones que solo un ADMIN_CLIENT puede hacer (ej. configurar Wompi, gestionar parámetros). |
| **Resultado esperado** | El ADMIN_RUTA en Vista de Control tiene acceso completo a las funciones del ADMIN_CLIENT del tenant. NO está limitado por permisos de operador. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de acceso exitoso a configuración en Vista de Control. |
| **Observaciones** | La Vista de Control actúa con privilegios de ADMIN_CLIENT del tenant objetivo. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE M — Pruebas de Seguridad y Permisos (PT-251 a PT-270)

---

| Campo | Valor |
|---|---|
| **ID** | PT-251 |
| **Módulo** | Seguridad — Multi-tenant |
| **Funcionalidad** | Aislamiento cross-tenant — ADMIN_CLIENT no puede ver pedidos de otro cliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen pedidos en cliente-piloto y cliente-dos. |
| **Datos de prueba** | ID de un pedido de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. En Postman, enviar GET `/admin/orders/{id_pedido_de_cliente_dos}`. |
| **Resultado esperado** | El servidor retorna HTTP 403 FORBIDDEN o 404 NOT FOUND. El ADMIN_CLIENT de cliente-piloto no puede ver datos de cliente-dos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 403/404 en Postman. |
| **Observaciones** | Regla crítica: aislamiento multi-tenant. Violación = TENANT_ISOLATION_VIOLATION (500). |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-252 |
| **Módulo** | Seguridad — Multi-tenant |
| **Funcionalidad** | Aislamiento cross-tenant — No acceder a productos de otro cliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen productos en cliente-dos con IDs conocidos. |
| **Datos de prueba** | ID de un producto de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. En Postman, enviar GET `/admin/products/{id_producto_de_cliente_dos}`. |
| **Resultado esperado** | HTTP 403 o 404. No se exponen datos de otro tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta de error. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-253 |
| **Módulo** | Seguridad — JWT |
| **Funcionalidad** | Token JWT manipulado/inválido es rechazado |
| **Rol** | N/A |
| **Precondiciones** | Postman configurado. |
| **Datos de prueba** | Token JWT con firma inválida (modificar el payload o la firma). |
| **Pasos** | 1. Obtener un JWT válido de una sesión activa. 2. Modificar el campo `user_type` en el payload (ej. cambiar OPERATOR_CLIENT a ADMIN_RUTA). 3. Recalcular o mantener la firma original (ahora inválida). 4. En Postman, usar este token manipulado en el header Authorization para llamar a `/admin/orders`. |
| **Resultado esperado** | El servidor rechaza el token manipulado con HTTP 401 `AUTHENTICATION_REQUIRED`. No se concede acceso elevado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 401 en Postman. |
| **Observaciones** | El JWT se verifica con `jose`. La firma debe ser válida. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-254 |
| **Módulo** | Seguridad — Auth |
| **Funcionalidad** | Llamada directa a API sin token de autenticación |
| **Rol** | N/A |
| **Precondiciones** | Postman sin cookies ni headers de auth. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En Postman (sin token ni cookies), enviar GET `/admin/orders`. |
| **Resultado esperado** | HTTP 401 `AUTHENTICATION_REQUIRED`. No se retornan datos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 401. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-255 |
| **Módulo** | Seguridad — Control de Acceso |
| **Funcionalidad** | COURIER no puede cancelar pedidos (403) |
| **Rol** | COURIER |
| **Precondiciones** | Sesión activa como COURIER. Existe un pedido asignado. |
| **Datos de prueba** | ID de un pedido asignado al courier. |
| **Pasos** | 1. En Postman con el token del COURIER, enviar PATCH `/admin/orders/{id}/cancel`. |
| **Resultado esperado** | HTTP 403 FORBIDDEN. El COURIER no tiene permiso para cancelar pedidos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 403. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-256 |
| **Módulo** | Seguridad — Control de Acceso |
| **Funcionalidad** | OPERATOR_CLIENT sin permiso ORDERS_CANCEL_FORCED no puede cancelar (403) |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | OPERATOR_CLIENT sin permiso `ORDERS_CANCEL_FORCED`. |
| **Datos de prueba** | ID de un pedido. |
| **Pasos** | 1. En Postman con token de OPERATOR_CLIENT sin el permiso, enviar PATCH `/admin/orders/{id}/cancel`. |
| **Resultado esperado** | HTTP 403 con código `MISSING_OPERATOR_PERMISSION`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 403 con `MISSING_OPERATOR_PERMISSION`. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-257 |
| **Módulo** | Seguridad — Idempotencia |
| **Funcionalidad** | Request duplicado con mismo X-Idempotency-Key retorna respuesta cacheada |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en VALIDATION_APPROVED. |
| **Datos de prueba** | X-Idempotency-Key: `uuid-de-prueba-001`. |
| **Pasos** | 1. En Postman, enviar PATCH `/admin/orders/{id}/accept` con header `X-Idempotency-Key: uuid-de-prueba-001`. 2. Enviar el mismo request nuevamente con el mismo idempotency key. |
| **Resultado esperado** | El primer request acepta el pedido (201/200). El segundo request retorna la misma respuesta (idempotente) sin volver a ejecutar la acción. El pedido no queda en estado duplicado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de ambos requests en Postman mostrando la misma respuesta. |
| **Observaciones** | TTL del idempotency key: 24 horas. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-258 |
| **Módulo** | Seguridad |
| **Funcionalidad** | Webhook Wompi con firma HMAC inválida es rechazado |
| **Rol** | N/A (sistema) |
| **Precondiciones** | Postman configurado para llamar al endpoint de webhook de Wompi. |
| **Datos de prueba** | Firma inválida en header `X-Wompi-Signature`. |
| **Pasos** | 1. En Postman, enviar POST al endpoint de webhook de Wompi con payload válido pero con firma HMAC incorrecta en el header. |
| **Resultado esperado** | El servidor retorna HTTP 400 o 401 con código `WEBHOOK_SIGNATURE_INVALID`. El evento NO se procesa. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta de rechazo. |
| **Observaciones** | Protege contra webhooks fraudulentos. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-259 |
| **Módulo** | Seguridad — Auditoría |
| **Funcionalidad** | OPERATOR_CLIENT no puede acceder al log de auditoría |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Sesión activa como OPERATOR_CLIENT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT. 2. Intentar navegar a `/admin/audit`. |
| **Resultado esperado** | Acceso denegado. El OPERATOR_CLIENT no puede ver el log de auditoría (solo ADMIN_CLIENT y ADMIN_RUTA). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del acceso denegado. |
| **Observaciones** | Ver log de auditoría del cliente: OPERATOR_CLIENT tiene ❌. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-260 |
| **Módulo** | Seguridad |
| **Funcionalidad** | Inyección SQL básica en campo de búsqueda |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Búsqueda: `'; DROP TABLE orders; --`. |
| **Pasos** | 1. Ir a Pedidos. 2. En el campo de búsqueda, ingresar `'; DROP TABLE orders; --`. 3. Hacer clic en "Filtrar". |
| **Resultado esperado** | El sistema maneja la entrada como texto de búsqueda normal, no como SQL. No ocurren errores de BD. No se ejecuta ningún SQL peligroso. La respuesta es una lista vacía o sin resultados. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del resultado (lista vacía o sin resultados, sin error). |
| **Observaciones** | Prisma ORM y Zod validan las entradas; el riesgo es bajo pero debe verificarse. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-261 |
| **Módulo** | Seguridad — Multi-tenant |
| **Funcionalidad** | Aislamiento de compradores — no acceder a buyer de otro tenant vía UI |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen compradores en cliente-dos con IDs conocidos. |
| **Datos de prueba** | URL directa al comprador de cliente-dos. |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT de cliente-piloto. 2. Navegar directamente a `/admin/buyers/{id_buyer_de_cliente_dos}`. |
| **Resultado esperado** | La UI muestra "no encontrado" o "acceso denegado". No se expone ningún dato del buyer de cliente-dos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error o redirección. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-262 |
| **Módulo** | Seguridad — Headers |
| **Funcionalidad** | Headers de seguridad en las respuestas HTTP |
| **Rol** | N/A |
| **Precondiciones** | Postman configurado. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En Postman, enviar GET a `/admin/orders` con autenticación. 2. Revisar los headers de la respuesta. |
| **Resultado esperado** | Los headers incluyen: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` o `SAMEORIGIN`, `Strict-Transport-Security` (en producción). No hay `X-Powered-By: Express`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los headers de respuesta en Postman. |
| **Observaciones** | Los headers de seguridad protegen contra XSS, clickjacking y sniffing. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-263 |
| **Módulo** | Seguridad — XSS |
| **Funcionalidad** | Intento de XSS en campo de nombre de producto |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: `<script>alert('XSS')</script>`. |
| **Pasos** | 1. Ir a Catálogo → Crear producto. 2. Ingresar `<script>alert('XSS')</script>` en el campo nombre. 3. Guardar. 4. Abrir el producto. |
| **Resultado esperado** | El script NO se ejecuta. El nombre se muestra como texto escapado o es rechazado por validación. No aparece ningún alert en el navegador. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del nombre mostrado como texto (no ejecutado). |
| **Observaciones** | React escapa el contenido por defecto (dangerouslySetInnerHTML nunca usado en este proyecto). |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-264 |
| **Módulo** | Seguridad — CORS |
| **Funcionalidad** | La API rechaza requests de origen no permitido (CORS) |
| **Rol** | N/A |
| **Precondiciones** | Conocer los orígenes permitidos configurados en el backend. |
| **Datos de prueba** | Origin: `https://sitio-no-autorizado.com`. |
| **Pasos** | 1. En Postman, agregar el header `Origin: https://sitio-no-autorizado.com`. 2. Enviar GET a `/admin/orders` con token válido. |
| **Resultado esperado** | El servidor retorna el header `Access-Control-Allow-Origin` solo con los orígenes permitidos. Si el origen no está en la lista, la respuesta omite el header CORS o retorna error. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los headers de respuesta CORS. |
| **Observaciones** | El CORS está configurado en el middleware de Express. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-265 |
| **Módulo** | Seguridad — Autorización |
| **Funcionalidad** | OPERATOR_CLIENT con permiso específico NO puede acceder a otros permisos |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | OPERATOR_CLIENT con solo permiso ORDERS_VIEW. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT con solo ORDERS_VIEW. 2. En Postman, intentar POST `/admin/products` (requiere PRODUCTS_CREATE). |
| **Resultado esperado** | HTTP 403 con `MISSING_OPERATOR_PERMISSION`. El operador no puede realizar acciones para las que no tiene permiso. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del 403 en Postman. |
| **Observaciones** | Sistema de permisos granular por operador. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-266 |
| **Módulo** | Seguridad — Datos |
| **Funcionalidad** | API no expone campos sensibles en respuestas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Postman configurado. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En Postman, GET `/admin/profile`. 2. Revisar la respuesta. |
| **Resultado esperado** | La respuesta NO incluye: password_hash, refresh_token, private keys de Wompi, secrets de webhook. Solo incluye campos no sensibles. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta sin campos sensibles. |
| **Observaciones** | Los campos sensibles deben ser excluidos en los DTOs de respuesta. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-267 |
| **Módulo** | Seguridad |
| **Funcionalidad** | Endpoint de webhook Wompi sin autenticación de sesión |
| **Rol** | N/A (Wompi) |
| **Precondiciones** | Postman configurado. |
| **Datos de prueba** | Firma HMAC válida (calcular con events_secret). |
| **Pasos** | 1. En Postman (sin cookies de sesión), enviar POST al endpoint de webhook de Wompi con firma HMAC válida. |
| **Resultado esperado** | El endpoint acepta el webhook (no requiere sesión de usuario). El evento se procesa correctamente. La autenticación del webhook es solo por firma HMAC. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 200 del webhook sin sesión. |
| **Observaciones** | Los endpoints de webhook no requieren sesión de usuario sino firma HMAC. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-268 |
| **Módulo** | Seguridad — Parámetros |
| **Funcionalidad** | Valores de parámetros críticos no pueden ser negativos ni extremos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Acceso a la configuración de parámetros. |
| **Datos de prueba** | `order.pending_confirm_timeout_minutes = -10`. |
| **Pasos** | 1. Ir a Configuración → Parámetros. 2. Intentar guardar un parámetro con valor negativo (-10). |
| **Resultado esperado** | El sistema rechaza el valor negativo con error de validación. Los parámetros tienen rangos mínimos y máximos definidos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | Los parámetros están validados con Zod en el backend. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-269 |
| **Módulo** | Seguridad — Enumeración |
| **Funcionalidad** | API no revela si un email existe en el sistema |
| **Rol** | N/A |
| **Precondiciones** | Postman configurado. |
| **Datos de prueba** | Email existente y email no existente. |
| **Pasos** | 1. En Postman, enviar POST `/auth/login` con email existente y contraseña incorrecta. 2. Enviar POST `/auth/login` con email que NO existe y cualquier contraseña. 3. Comparar los mensajes de error. |
| **Resultado esperado** | Los dos casos retornan el mismo mensaje de error genérico (ej. "Credenciales inválidas"). No se puede determinar si el email existe o no por el mensaje de error. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de ambas respuestas con el mismo mensaje. |
| **Observaciones** | Protección contra ataques de enumeración de usuarios. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-270 |
| **Módulo** | Seguridad — Cookies |
| **Funcionalidad** | Cookies con flag SameSite para protección CSRF |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como ADMIN_CLIENT. 2. Abrir DevTools → Application → Cookies. 3. Inspeccionar las cookies `access_token` y `refresh_token`. |
| **Resultado esperado** | Las cookies tienen `SameSite=Strict` o `SameSite=Lax` para proteger contra CSRF. También tienen `HttpOnly` y `Secure` (en producción). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de las propiedades de las cookies en DevTools. |
| **Observaciones** | Complementario a PT-016. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

### FASE N — UI/UX, Responsive y Visual (PT-271 a PT-295)

---

| Campo | Valor |
|---|---|
| **ID** | PT-271 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Layout correcto en desktop 1920x1080 |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Navegador a 1920x1080. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ajustar el navegador a 1920x1080 px. 2. Navegar por: Dashboard, Lista de Pedidos, Detalle de Pedido, Mapa, Productos, Configuración. |
| **Resultado esperado** | El layout se ve correctamente en todas las páginas. No hay elementos desbordados, texto cortado o columnas que se superpongan. El sidebar está visible y expandido. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de pantalla de las páginas principales a 1920x1080. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-272 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Layout correcto en laptop 1366x768 |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Navegador a 1366x768. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ajustar el navegador a 1366x768 px. 2. Navegar por las mismas páginas que PT-271. |
| **Resultado esperado** | El layout es funcional. El sidebar puede estar colapsado automáticamente. No hay scroll horizontal involuntario. Las tablas tienen scroll horizontal si el contenido lo requiere. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas a 1366x768. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-273 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Layout correcto en tablet 768px |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | DevTools → Responsive a 768px. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Abrir DevTools → modo responsive → 768px. 2. Navegar por: Dashboard, Pedidos, Detalle de Pedido. |
| **Resultado esperado** | El layout se adapta correctamente. El sidebar puede estar colapsado. Las tablas tienen scroll horizontal. Los formularios están apilados. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas a 768px. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-274 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Modo oscuro — verificar correctamente en todas las páginas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Activar el modo oscuro usando el `RutaThemeToggle` en el header. 2. Navegar por todas las pantallas principales. |
| **Resultado esperado** | Todas las páginas se muestran en modo oscuro con colores correctos. No hay texto negro sobre fondo negro o blanco sobre blanco. Los badges de estado mantienen sus colores semánticos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de las páginas principales en modo oscuro. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-275 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Modo claro — verificar correctamente en todas las páginas |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Activar el modo claro. 2. Navegar por todas las pantallas principales. |
| **Resultado esperado** | Todas las páginas se muestran en modo claro. Contraste de texto es adecuado (mín WCAG AA). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas en modo claro. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-276 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Estados de carga (spinners / skeletons) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Throttle la conexión en DevTools a "Slow 3G". |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Activar throttling "Slow 3G" en DevTools → Network. 2. Navegar a Lista de Pedidos. 3. Observar el estado de carga. 4. Navegar a Catálogo → Productos. 5. Observar el estado de carga. |
| **Resultado esperado** | Mientras los datos cargan, se muestra un indicador de carga (spinner o skeleton). No aparece una pantalla en blanco ni un error. El contenido se muestra cuando los datos llegan. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del spinner/skeleton visible. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-277 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Estado vacío en lista de pedidos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Aplicar filtros que devuelvan 0 resultados. |
| **Datos de prueba** | Filtro: fecha = mañana (sin pedidos). |
| **Pasos** | 1. Ir a Pedidos. 2. Filtrar por fecha = mañana. |
| **Resultado esperado** | La tabla muestra un mensaje de estado vacío ("No hay pedidos para los filtros seleccionados"). No hay error en consola. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del estado vacío. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-278 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Mensajes de éxito al guardar formularios |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Editar cualquier producto y guardar. |
| **Pasos** | 1. Ir a Catálogo → Productos. 2. Editar cualquier producto. 3. Guardar. |
| **Resultado esperado** | El sistema muestra un mensaje o toast de éxito confirmando que los cambios se guardaron. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de éxito. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-279 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Mensajes de error al fallar un request |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Simular un error de servidor (bloquear el request en DevTools). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En DevTools → Network → Block request URL (bloquear la URL del API). 2. Intentar cargar la lista de pedidos. |
| **Resultado esperado** | El sistema muestra un mensaje de error claro al usuario. No se muestra la pantalla de error genérica del navegador. El mensaje sugiere reintentar. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del mensaje de error en la UI. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-280 |
| **Módulo** | UI/UX |
| **Funcionalidad** | Navegación por breadcrumbs y botón "Volver" |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Pedidos → hacer clic en un pedido (detalle). 2. Hacer clic en el botón "Volver" o en el breadcrumb "Pedidos". |
| **Resultado esperado** | El sistema navega de vuelta a la lista de pedidos manteniendo los filtros aplicados. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista con filtros preservados. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

### FASE O — Pruebas de Regresión (PT-281 a PT-300)

*Estos casos se deben re-ejecutar después de cualquier corrección o deploy a staging.*

---

| Campo | Valor |
|---|---|
| **ID** | PT-281 |
| **Módulo** | Regresión |
| **Funcionalidad** | Login y logout de todos los roles |
| **Rol** | Todos |
| **Precondiciones** | Credenciales de todos los roles disponibles. |
| **Datos de prueba** | Ver sección 2 — Configuración. |
| **Pasos** | 1. Login como ADMIN_RUTA → verificar landing `/ruta-admin/clients` → logout. 2. Login como ADMIN_CLIENT → verificar landing `/admin/dashboard` → logout. 3. Login como OPERATOR_CLIENT → verificar landing `/admin/dashboard` → logout. 4. Login como COURIER → verificar landing `/courier` → logout. |
| **Resultado esperado** | Cada rol hace login exitoso en su landing correspondiente y logout limpio. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de cada landing. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-282 |
| **Módulo** | Regresión |
| **Funcionalidad** | Ciclo SHIP completo end-to-end |
| **Rol** | ADMIN_CLIENT + COURIER |
| **Precondiciones** | Entorno limpio con datos base. |
| **Datos de prueba** | Pedido SHIP con COD. |
| **Pasos** | Ejecutar el flujo completo del PT-143 (ciclo SHIP con COD end-to-end). |
| **Resultado esperado** | El pedido completa el ciclo exitosamente. Estado final: COMPLETED_SUCCESSFULLY → CLOSED. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Timeline completa del pedido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-283 |
| **Módulo** | Regresión |
| **Funcionalidad** | Ciclo PICKUP completo end-to-end |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pickup point activo. |
| **Datos de prueba** | Pedido PICKUP con pago online. |
| **Pasos** | Ejecutar el flujo completo del PT-163 (ciclo PICKUP con pago online). |
| **Resultado esperado** | El pedido completa el ciclo PICKUP exitosamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Timeline completa del pedido PICKUP. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-284 |
| **Módulo** | Regresión |
| **Funcionalidad** | Webhook Wompi — pago online confirmado |
| **Rol** | Sistema |
| **Precondiciones** | Wompi sandbox configurado. |
| **Datos de prueba** | Tarjeta de prueba Wompi aprobada. |
| **Pasos** | Ejecutar PT-140 (pago Wompi webhook). |
| **Resultado esperado** | payment_status = PAID tras recibir el webhook de Wompi. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del payment_status = PAID. |
| **Observaciones** | |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-285 |
| **Módulo** | Regresión |
| **Funcionalidad** | Aislamiento multi-tenant |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Dos clientes con datos distintos. |
| **Datos de prueba** | Ver PT-251 y PT-252. |
| **Pasos** | Ejecutar PT-251 y PT-252 (cross-tenant isolation). |
| **Resultado esperado** | HTTP 403/404 al intentar acceder a datos de otro tenant. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de los errores 403/404. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-286 |
| **Módulo** | Regresión |
| **Funcionalidad** | Jobs de expiración — verificar en logs |
| **Rol** | Sistema (pg-boss) |
| **Precondiciones** | pg-boss activo. |
| **Datos de prueba** | Pedido en DRAFT hace 24h (ajustar parámetro para prueba). |
| **Pasos** | 1. Crear un pedido y mantenerlo en DRAFT. 2. Ajustar `order.draft_expiration_minutes` a 1 en BD. 3. Esperar 2 minutos. 4. Verificar el estado del pedido. 5. Revisar los logs de pino para confirmar que el job se ejecutó. |
| **Resultado esperado** | El pedido DRAFT expira automáticamente (EXPIRED → CLOSED). El log registra la ejecución del job. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del log del job y del pedido en estado CLOSED. |
| **Observaciones** | Restaurar parámetro a 1440 después. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-287 |
| **Módulo** | Regresión |
| **Funcionalidad** | Vista de Control — impersonación auditada |
| **Rol** | ADMIN_RUTA |
| **Precondiciones** | Master password configurado. |
| **Datos de prueba** | Ver PT-236. |
| **Pasos** | Ejecutar PT-236 y PT-238. |
| **Resultado esperado** | Impersonación exitosa con banner ámbar. Acción auditada con `acting_via_control_view = TRUE`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del banner y del log de auditoría. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |


---

| Campo | Valor |
|---|---|
| **ID** | PT-288 |
| **Módulo** | Regresión |
| **Funcionalidad** | Gestión de usuarios — crear y desactivar operador |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Entorno limpio. |
| **Datos de prueba** | Ver PT-031 y PT-037. |
| **Pasos** | Ejecutar PT-031 (crear OPERATOR_CLIENT) y PT-037 (desactivar). |
| **Resultado esperado** | El operador se crea y desactiva correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del operador creado y desactivado. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-289 |
| **Módulo** | Regresión |
| **Funcionalidad** | Importación masiva de productos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Archivo Excel de prueba disponible. |
| **Datos de prueba** | Ver PT-054. |
| **Pasos** | Ejecutar PT-054 (importar Excel con formato válido). |
| **Resultado esperado** | 20 productos importados correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del reporte de importación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-290 |
| **Módulo** | Regresión |
| **Funcionalidad** | Configuración Wompi funcional |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Credenciales Wompi sandbox disponibles. |
| **Datos de prueba** | Ver PT-203. |
| **Pasos** | Ejecutar PT-203 (configurar Wompi). |
| **Resultado esperado** | Credenciales guardadas. URL de webhook visible. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de configuración Wompi activa. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-291 |
| **Módulo** | Regresión |
| **Funcionalidad** | Estado de carga y estados vacíos |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Throttle a Slow 3G. |
| **Datos de prueba** | Ver PT-276 y PT-277. |
| **Pasos** | Ejecutar PT-276 (estados de carga) y PT-277 (estado vacío pedidos). |
| **Resultado esperado** | Spinners visibles y estados vacíos correctos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de carga y estado vacío. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-292 |
| **Módulo** | Regresión |
| **Funcionalidad** | Mapa de asignación y asignación de courier |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedido en AWAITING_COURIER_ASSIGNMENT y courier activo. |
| **Datos de prueba** | Ver PT-127 y PT-128. |
| **Pasos** | Ejecutar PT-127 (ver mapa) y PT-128 (asignar courier). |
| **Resultado esperado** | Mapa carga correctamente. Asignación exitosa. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del mapa y asignación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-293 |
| **Módulo** | Regresión |
| **Funcionalidad** | Seguridad — tokens no en localStorage |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Ver PT-016. |
| **Pasos** | Ejecutar PT-016 (verificar que no hay tokens en localStorage). |
| **Resultado esperado** | Tokens solo en cookies HttpOnly. localStorage vacío de tokens. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de DevTools Cookies y localStorage. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-294 |
| **Módulo** | Regresión |
| **Funcionalidad** | Vista COURIER mobile completo |
| **Rol** | COURIER |
| **Precondiciones** | Dispositivo o emulador 375px con pedidos asignados. |
| **Datos de prueba** | Ver PT-182. |
| **Pasos** | Ejecutar PT-182 (responsive mobile courier). |
| **Resultado esperado** | Interfaz completamente funcional a 375px. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas mobile de lista y detalle. |
| **Observaciones** | |
| **Responsable** | QA UX-Mobile |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-295 |
| **Módulo** | Regresión |
| **Funcionalidad** | Webhooks salientes enviados correctamente |
| **Rol** | Sistema |
| **Precondiciones** | Suscripción de webhook activa. |
| **Datos de prueba** | Ver PT-206. |
| **Pasos** | Ejecutar PT-206 (verificar envío de webhook al cambiar estado de pedido). |
| **Resultado esperado** | Webhook recibido en RequestBin con firma correcta. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del webhook en RequestBin. |
| **Observaciones** | |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-296 |
| **Módulo** | Regresión |
| **Funcionalidad** | Parámetros de negocio — override y restauración |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Acceso a Configuración → Parámetros. |
| **Datos de prueba** | Ver PT-212 y PT-213. |
| **Pasos** | Ejecutar PT-212 (override parámetro) y PT-213 (limpiar override). |
| **Resultado esperado** | Override guardado y eliminado correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas del parámetro con y sin override. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-297 |
| **Módulo** | Regresión |
| **Funcionalidad** | Modo oscuro / modo claro funcionando |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Ver PT-274 y PT-275. |
| **Pasos** | Activar modo oscuro → navegar por 3 páginas. Activar modo claro → navegar por 3 páginas. |
| **Resultado esperado** | Ambos modos visuales funcionan sin elementos rotos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas en ambos modos. |
| **Observaciones** | |
| **Responsable** | QA UX |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-298 |
| **Módulo** | Regresión |
| **Funcionalidad** | Dashboard métricas ADMIN_CLIENT |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pedidos del día disponibles. |
| **Datos de prueba** | Ver PT-220. |
| **Pasos** | Ejecutar PT-220 (métricas del dashboard). |
| **Resultado esperado** | Las 4 métricas se muestran correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del dashboard con métricas. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-299 |
| **Módulo** | Regresión |
| **Funcionalidad** | Transición de estado inválida rechazada |
| **Rol** | N/A (API directa) |
| **Precondiciones** | Pedido en DRAFT. |
| **Datos de prueba** | Ver PT-142. |
| **Pasos** | Ejecutar PT-142 (intento de transición inválida). |
| **Resultado esperado** | HTTP 422 INVALID_STATE_TRANSITION. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del 422 en Postman. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-300 |
| **Módulo** | Regresión |
| **Funcionalidad** | Ciclo completo de PICKUP con validación de identidad |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Pickup point activo. |
| **Datos de prueba** | Ver PT-159. |
| **Pasos** | Ejecutar PT-159 (registrar llegada y validar identidad). |
| **Resultado esperado** | Los estados CUSTOMER_ARRIVED e IDENTITY_VALIDATED funcionan correctamente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de los estados en el detalle del pedido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

### FASE P — API Keys y Acceso Programático (PT-301 a PT-315)

*Pruebas para la gestión de API Keys de los clientes API.*

---

| Campo | Valor |
|---|---|
| **ID** | PT-301 |
| **Módulo** | API Keys |
| **Funcionalidad** | ADMIN_CLIENT genera una API Key |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Cliente de tipo API o FULL con acceso API habilitado. |
| **Datos de prueba** | Nombre de la key: "Producción ERP", Permisos: `orders:read, orders:write`. |
| **Pasos** | 1. Ir a Configuración → API Keys (o sección equivalente). 2. Hacer clic en "Generar nueva API Key". 3. Ingresar nombre y permisos. 4. Confirmar. |
| **Resultado esperado** | Se genera una API Key. El valor completo se muestra UNA SOLA VEZ en pantalla. Se guarda en la BD con su hash. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la API Key visible por única vez. |
| **Observaciones** | La API Key solo se muestra una vez. El usuario debe copiarla. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-302 |
| **Módulo** | API Keys |
| **Funcionalidad** | API Key no se puede ver después de generada |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | API Key ya generada (PT-301 ejecutado). |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → API Keys. 2. Intentar ver el valor completo de la key generada. |
| **Resultado esperado** | Solo se muestra los primeros y últimos caracteres (ej. `sk_prod_abc...xyz`). El valor completo NO está disponible. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la key truncada en la lista. |
| **Observaciones** | Seguridad: las API Keys se almacenan hasheadas y nunca se muestran completas después de la creación. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-303 |
| **Módulo** | API Keys |
| **Funcionalidad** | Usar API Key válida para autenticar request |
| **Rol** | API Key |
| **Precondiciones** | API Key generada con permisos `orders:read`. |
| **Datos de prueba** | Header: `Authorization: Bearer sk_prod_...` |
| **Pasos** | 1. En Postman, agregar header `Authorization: Bearer {api_key}`. 2. Enviar GET `/api/v1/orders`. |
| **Resultado esperado** | El servidor autentica el request con la API Key y retorna los pedidos del cliente (HTTP 200). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 200 con los pedidos. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-304 |
| **Módulo** | API Keys |
| **Funcionalidad** | API Key inválida retorna 401 |
| **Rol** | N/A |
| **Precondiciones** | Postman configurado. |
| **Datos de prueba** | API Key inválida: `sk_prod_invalida12345`. |
| **Pasos** | 1. En Postman, usar una API Key inválida. 2. GET `/api/v1/orders`. |
| **Resultado esperado** | HTTP 401 `AUTHENTICATION_REQUIRED`. No se retornan datos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del 401. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-305 |
| **Módulo** | API Keys |
| **Funcionalidad** | API Key con permiso insuficiente retorna 403 |
| **Rol** | API Key |
| **Precondiciones** | API Key con solo permiso `orders:read`. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Con API Key que solo tiene `orders:read`. 2. En Postman, POST `/api/v1/orders` (requiere `orders:write`). |
| **Resultado esperado** | HTTP 403 `FORBIDDEN`. La API Key no tiene el permiso necesario para escribir. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del 403. |
| **Observaciones** | Las API Keys tienen permisos granulares. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-306 |
| **Módulo** | API Keys |
| **Funcionalidad** | Revocar API Key |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | API Key activa generada en PT-301. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → API Keys. 2. Hacer clic en "Revocar" en la key "Producción ERP". 3. Confirmar. |
| **Resultado esperado** | La API Key queda marcada como revocada. Los requests con esa key retornan 401. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la key revocada. Captura del 401 al usarla. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-307 |
| **Módulo** | API Keys |
| **Funcionalidad** | Aislamiento multi-tenant con API Keys |
| **Rol** | API Key |
| **Precondiciones** | API Key del cliente-piloto. ID de recurso de cliente-dos. |
| **Datos de prueba** | ID de pedido de cliente-dos. |
| **Pasos** | 1. Con API Key de cliente-piloto. 2. GET `/api/v1/orders/{id_pedido_de_cliente_dos}`. |
| **Resultado esperado** | HTTP 403 o 404. La API Key de cliente-piloto no puede acceder a recursos de cliente-dos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de aislamiento. |
| **Observaciones** | El aislamiento multi-tenant aplica también a accesos via API Key. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-308 |
| **Módulo** | API Keys |
| **Funcionalidad** | Listar API Keys del cliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Existen varias API Keys. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → API Keys. |
| **Resultado esperado** | Se lista todas las API Keys con: nombre, prefijo (truncado), permisos, fecha de creación, estado (activa/revocada). |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la lista de API Keys. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-309 |
| **Módulo** | API Keys |
| **Funcionalidad** | Acciones con API Key registradas en auditoría |
| **Rol** | API Key |
| **Precondiciones** | API Key activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Realizar una acción con la API Key (ej. crear pedido). 2. Ir a Auditoría. 3. Buscar el evento. |
| **Resultado esperado** | El evento aparece en auditoría con el actor como la API Key (nombre de la key) en lugar de un usuario. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del evento en auditoría con el identificador de la API Key. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-310 |
| **Módulo** | API Keys |
| **Funcionalidad** | Rate limiting en acceso con API Key |
| **Rol** | API Key |
| **Precondiciones** | API Key activa. Postman configurado para enviar múltiples requests. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Enviar 100 requests en 1 minuto con la misma API Key. 2. Verificar si hay rate limiting. |
| **Resultado esperado** | Después del límite configurado, el servidor retorna HTTP 429 `RATE_LIMITED`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del 429. |
| **Observaciones** | Verificar el parámetro de rate limiting para API Keys. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-311 |
| **Módulo** | API Keys |
| **Funcionalidad** | Crear API Key con nombre vacío (validación) |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Sesión activa. |
| **Datos de prueba** | Nombre: vacío. |
| **Pasos** | 1. Ir a Configuración → API Keys → Generar nueva key. 2. Dejar nombre vacío. 3. Confirmar. |
| **Resultado esperado** | El formulario valida que el nombre es obligatorio. No se genera la key. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de validación. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-312 |
| **Módulo** | API Keys |
| **Funcionalidad** | API Key no puede usarse para acceder a la interfaz de admin web |
| **Rol** | API Key |
| **Precondiciones** | API Key activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. En Postman, intentar acceder a endpoints del admin (`/admin/...`) con la API Key. |
| **Resultado esperado** | HTTP 403. Las API Keys solo funcionan para los endpoints API públicos (`/api/v1/...`), no para los endpoints del panel de administración. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del 403. |
| **Observaciones** | Los endpoints del admin requieren sesión de usuario, no API Key. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-313 |
| **Módulo** | API Keys |
| **Funcionalidad** | OPERATOR_CLIENT no puede gestionar API Keys |
| **Rol** | OPERATOR_CLIENT |
| **Precondiciones** | Sesión activa como OPERATOR_CLIENT. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Iniciar sesión como OPERATOR_CLIENT. 2. Intentar navegar a la sección de API Keys. |
| **Resultado esperado** | El OPERATOR_CLIENT no tiene acceso a la gestión de API Keys. Solo ADMIN_CLIENT y ADMIN_RUTA pueden gestionarlas. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del acceso denegado. |
| **Observaciones** | |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-314 |
| **Módulo** | API Keys |
| **Funcionalidad** | Número máximo de API Keys por cliente |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Se ha alcanzado el límite máximo de API Keys (verificar el parámetro). |
| **Datos de prueba** | Crear keys hasta el límite. |
| **Pasos** | 1. Crear API Keys hasta alcanzar el límite máximo. 2. Intentar crear una más. |
| **Resultado esperado** | El sistema rechaza la creación cuando se alcanza el límite, indicando el máximo permitido. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del error de límite alcanzado. |
| **Observaciones** | Verificar si existe un parámetro de límite de API Keys por cliente. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-315 |
| **Módulo** | API Keys |
| **Funcionalidad** | API Key funciona para el flujo de creación de pedido (cliente API) |
| **Rol** | API Key |
| **Precondiciones** | Cliente tipo API con API Key activa. |
| **Datos de prueba** | Payload de pedido con buyer_id, items, delivery_type, pickup_point_id. |
| **Pasos** | 1. Con la API Key, enviar POST `/api/v1/orders` con payload completo. 2. Verificar la respuesta. |
| **Resultado esperado** | El pedido se crea exitosamente (HTTP 201). El pedido aparece en el panel admin del cliente. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura de la respuesta 201 y del pedido en el admin. |
| **Observaciones** | Este es el flujo principal de integración para clientes API. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

### FASE Q — Webhooks Salientes Avanzados (PT-316 a PT-325)

*Pruebas avanzadas del sistema de webhooks salientes.*

---

| Campo | Valor |
|---|---|
| **ID** | PT-316 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Reintentos automáticos de webhook fallido |
| **Rol** | Sistema (pg-boss) |
| **Precondiciones** | Webhook configurado. Endpoint que retorna 500 para simular fallo. |
| **Datos de prueba** | URL de endpoint que siempre retorna 500. |
| **Pasos** | 1. Configurar el webhook con una URL que retorne 500. 2. Disparar un evento (cambiar estado de pedido). 3. Esperar y verificar los reintentos en el historial. |
| **Resultado esperado** | El sistema reintenta el webhook en los intervalos configurados: 1m, 5m, 15m, 60m, 4h. Cada intento aparece en el historial de entregas. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial mostrando múltiples intentos fallidos. |
| **Observaciones** | Los reintentos son automáticos vía pg-boss. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-317 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Firma HMAC del webhook saliente es verificable |
| **Rol** | Sistema |
| **Precondiciones** | Webhook configurado. RequestBin activo. |
| **Datos de prueba** | Signing secret del webhook. |
| **Pasos** | 1. Disparar un evento para generar un webhook. 2. Recibir el webhook en RequestBin. 3. Verificar el header `X-Ruta-Signature` usando el signing secret y HMAC-SHA256. |
| **Resultado esperado** | La firma HMAC puede ser verificada correctamente usando: `HMAC-SHA256(signing_secret, payload)`. La verificación retorna verdadero. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del webhook con la firma y script de verificación exitosa. |
| **Observaciones** | Algoritmo: HMAC-SHA256. El signing secret es único por suscripción. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-318 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Webhook de `order.status_changed` contiene payload correcto |
| **Rol** | Sistema |
| **Precondiciones** | Suscripción al evento `order.status_changed`. RequestBin activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Cambiar el estado de un pedido. 2. Recibir el webhook en RequestBin. 3. Verificar el payload. |
| **Resultado esperado** | El payload contiene: `event_type: "order.status_changed"`, `order_id`, `previous_status`, `new_status`, `timestamp`, `client_id`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del payload completo del webhook. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-319 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Webhook de `order.delivered` se dispara correctamente |
| **Rol** | Sistema |
| **Precondiciones** | Suscripción al evento `order.delivered`. RequestBin activo. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Llevar un pedido al estado DELIVERED. 2. Verificar si llega el webhook `order.delivered`. |
| **Resultado esperado** | El webhook `order.delivered` llega a RequestBin con el payload del pedido entregado. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del webhook recibido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-320 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Múltiples suscripciones reciben el mismo evento |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Dos suscripciones activas al mismo evento `order.status_changed`. |
| **Datos de prueba** | URL1: RequestBin1, URL2: RequestBin2. Ambas suscritas a `order.status_changed`. |
| **Pasos** | 1. Crear dos suscripciones al mismo evento pero con URLs diferentes. 2. Cambiar el estado de un pedido. 3. Verificar ambas URLs. |
| **Resultado esperado** | Ambas URLs reciben el webhook. Se envían dos entregas independientes. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas de ambos RequestBin con el webhook recibido. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-321 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Suscripción inactiva no recibe webhooks |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Suscripción de webhook existente. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Desactivar una suscripción de webhook. 2. Cambiar el estado de un pedido. 3. Verificar si el webhook se envió. |
| **Resultado esperado** | La suscripción inactiva NO recibe el webhook. Solo las suscripciones activas reciben eventos. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial de entregas sin nuevas entregas para la suscripción inactiva. |
| **Observaciones** | |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-322 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Webhook con timeout del endpoint destino |
| **Rol** | Sistema |
| **Precondiciones** | Endpoint que tarda más de 30 segundos en responder. |
| **Datos de prueba** | URL con delay de 60 segundos. |
| **Pasos** | 1. Configurar webhook con URL que genera timeout. 2. Disparar un evento. 3. Verificar el resultado en el historial. |
| **Resultado esperado** | El sistema maneja el timeout del endpoint como un fallo. Registra la entrega como fallida y programa el reintento automático. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del historial con la entrega fallida por timeout. |
| **Observaciones** | El timeout del cliente HTTP para webhooks debe estar configurado. |
| **Responsable** | QA funcional / Dev |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-323 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | El signing secret del webhook se puede rotar |
| **Rol** | ADMIN_CLIENT |
| **Precondiciones** | Suscripción de webhook activa. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ir a Configuración → Webhooks. 2. En la suscripción, usar la opción "Rotar secret". 3. Confirmar. |
| **Resultado esperado** | El signing secret se rota. El nuevo secret se muestra una sola vez. Los webhooks futuros usan el nuevo secret. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del nuevo secret y verificación del webhook con el nuevo secret. |
| **Observaciones** | Verificar si la rotación de secrets está implementada en el MVP. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-324 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Webhook de `payment.collected` se dispara tras cobro COD |
| **Rol** | Sistema |
| **Precondiciones** | Suscripción al evento `payment.collected`. RequestBin activo. Pedido COD. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Ejecutar el cobro COD (PT-132). 2. Verificar si llega el webhook `payment.collected` a RequestBin. |
| **Resultado esperado** | El webhook `payment.collected` llega con: `order_id`, `amount`, `payment_method`, `collected_at`. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Captura del webhook recibido con el payload de cobro. |
| **Observaciones** | Verificar si el evento `payment.collected` está implementado. |
| **Responsable** | QA funcional |
| **Fecha ejecución** | |

---

| Campo | Valor |
|---|---|
| **ID** | PT-325 |
| **Módulo** | Webhooks salientes |
| **Funcionalidad** | Aislamiento multi-tenant en webhooks — un tenant no recibe eventos de otro |
| **Rol** | Sistema |
| **Precondiciones** | Dos tenants con suscripciones de webhook activas. |
| **Datos de prueba** | N/A |
| **Pasos** | 1. Configurar webhooks para cliente-piloto (URL1) y cliente-dos (URL2). 2. Cambiar estado de un pedido de cliente-piloto. 3. Verificar que solo URL1 recibe el webhook. |
| **Resultado esperado** | Solo URL1 (cliente-piloto) recibe el webhook. URL2 (cliente-dos) NO recibe ningún evento de cliente-piloto. Aislamiento estricto. |
| **Resultado obtenido** | *(dejar en blanco)* |
| **Estado** | ⬜ Pendiente |
| **Evidencia requerida** | Capturas mostrando evento en URL1 y ausencia en URL2. |
| **Observaciones** | Aislamiento multi-tenant crítico también para webhooks. |
| **Responsable** | QA seguridad |
| **Fecha ejecución** | |

---

## 4. Control de Avance

| Métrica | Valor |
|---------|-------|
| Total de pruebas | 325 |
| Pendientes | 324 |
| En progreso | 0 |
| Aprobadas | 1 |
| Fallidas | 0 |
| Bloqueadas | 0 |
| % Avance | 0.3% |
| % Éxito | 100% |
| Observaciones generales | v2.0 — 2026-06-04. 325 casos completos. PT-001 aprobada (Alexander Márquez, 2026-05-31). Producción en Render: ruta-admin / ruta-orko-api / ruta-by-orko.onrender.com. |

**Distribución por fase:**

| Fase | Rango | Cantidad |
|------|-------|----------|
| A — Autenticación y Sesiones | PT-001 a PT-020 | 20 |
| B — Gestión de Clientes/Tenants y Usuarios | PT-021 a PT-040 | 20 |
| C — Catálogo | PT-041 a PT-060 | 20 |
| D — Compradores (BUYERS) | PT-061 a PT-075 | 15 |
| E — Couriers / Repartidores | PT-076 a PT-095 | 20 |
| F — Pickup Points | PT-096 a PT-115 | 20 |
| G — Flujo SHIP | PT-116 a PT-155 | 40 |
| H — Flujo PICKUP | PT-156 a PT-175 | 20 |
| I — Vista COURIER | PT-176 a PT-200 | 25 |
| J — Configuración del Cliente | PT-201 a PT-214 | 14 |
| K — Auditoría y Métricas | PT-215 a PT-235 | 21 |
| L — Vista de Control | PT-236 a PT-250 | 15 |
| M — Seguridad y Permisos | PT-251 a PT-270 | 20 |
| N — UI/UX y Responsive | PT-271 a PT-280 | 10 |
| O — Regresión | PT-281 a PT-300 | 20 |
| P — API Keys | PT-301 a PT-315 | 15 |
| Q — Webhooks Salientes Avanzados | PT-316 a PT-325 | 10 |
| **TOTAL** | | **325** |

---

## 5. Bitácora de Ejecución

| Sesión | Fecha | Hora inicio | Hora fin | Tester | Módulos probados | Pruebas ejecutadas | Errores encontrados | Decisiones | Pendientes |
|--------|-------|------------|---------|--------|-----------------|-------------------|--------------------|-----------| -----------|
| 1 | 2026-05-31 | 22:00 | En curso | Alexander Márquez | Autenticación | PT-001 | DEF-001, DEF-002 | Corrección inmediata de bugs encontrados | PT-002 en adelante |
| 2 | | | | | | | | | |
| 3 | | | | | | | | | |
| 4 | | | | | | | | | |
| 5 | | | | | | | | | |

---

## 6. Matriz de Defectos

| ID | Caso relacionado | Módulo | Descripción | Pasos reproducción | Resultado esperado | Resultado real | Severidad | Prioridad | Estado | Evidencia | Responsable | Fecha reporte | Fecha cierre |
|----|-----------------|--------|------------|-------------------|-------------------|----------------|-----------|-----------|--------|-----------|-------------|--------------|-------------|
| DEF-001 | PT-001 | Autenticación / Navegación | `/` muestra design system showcase en lugar de redirigir al login | 1. Abrir `http://localhost:3002/` sin sesión activa. | Redirigir a `/login`. | Muestra showcase de componentes UI. | S3 — Media | P2 — Alta | ✅ Cerrado | — | Alexander Márquez | 2026-05-31 | 2026-05-31 |
| DEF-002 | PT-001 | Autenticación | ADMIN_RUTA aterriza en `/ruta-admin/clients` tras login en lugar del dashboard | 1. Login como ADMIN_RUTA. | Redirigir a `/ruta-admin/dashboard`. | Redirige a `/ruta-admin/clients`. | S4 — Baja | P3 — Media | ✅ Cerrado | — | Alexander Márquez | 2026-05-31 | 2026-05-31 |

**Escala de Severidad:**
- **S1 — Crítico:** El sistema es inutilizable o hay pérdida de datos / brecha de seguridad.
- **S2 — Alto:** Funcionalidad principal no funciona; no hay workaround.
- **S3 — Medio:** Funcionalidad afectada pero hay workaround viable.
- **S4 — Bajo:** Problema cosmético, de UI o mejora menor.

**Escala de Prioridad:**
- **P1 — Urgente:** Bloquea el go-live. Debe corregirse antes del deploy.
- **P2 — Alta:** Debe corregirse antes del go-live si el tiempo lo permite.
- **P3 — Media:** Corrección en la siguiente iteración post-lanzamiento.
- **P4 — Baja:** Mejora futura o cosmético.

---

## 7. Criterios de Aceptación Final

Para considerar el MVP listo para producción deben cumplirse **todos** los siguientes criterios:

### Cobertura de pruebas
- [ ] Mínimo 90% de las pruebas ejecutadas (al menos 293 de 325).
- [ ] Mínimo 95% de las pruebas ejecutadas en estado Aprobado.
- [ ] 100% de las pruebas de FASE O (Regresión) en estado Aprobado.
- [ ] 100% de las pruebas de FASE P (API Keys) ejecutadas si el cliente es tipo API.
- [ ] 100% de las pruebas de FASE Q (Webhooks) ejecutadas si el cliente tiene webhooks configurados.

### Defectos
- [ ] **0 defectos S1 (Críticos) abiertos.**
- [ ] **0 defectos S2 (Altos) abiertos o con workaround documentado y aceptado.**
- [ ] Defectos S3 evaluados caso a caso — no bloquean el go-live si tienen workaround.
- [ ] Defectos S4 pueden dejarse para post-lanzamiento.

### Flujos críticos validados end-to-end
- [ ] Login y logout de los 4 roles staff (PT-001 a PT-004, PT-008).
- [ ] Ciclo SHIP completo con COD (PT-143 / PT-282).
- [ ] Ciclo SHIP completo con pago Wompi online (PT-140 / PT-284).
- [ ] Ciclo PICKUP completo (PT-163 / PT-283).
- [ ] Vista de Control con auditoría (PT-236, PT-238 / PT-287).
- [ ] Aislamiento multi-tenant (PT-251, PT-252 / PT-285).

### Seguridad
- [ ] Todos los tests de FASE M ejecutados sin hallazgos S1/S2 abiertos.
- [ ] Tokens JWT no en localStorage (PT-016).
- [ ] Webhook Wompi con firma inválida rechazado (PT-258).
- [ ] Cross-tenant isolation verificado (PT-251, PT-252).
- [ ] Rate limiting activo en login (PT-015).

### Performance básica
- [ ] Las páginas principales cargan en menos de 3 segundos en condiciones normales.
- [ ] El mapa de asignación carga correctamente con al menos 10 pedidos activos.

### Compatibilidad
- [ ] UI funcional en Chrome y Firefox (últimas versiones).
- [ ] Vista COURIER funcional en móvil 375px (Chrome DevTools o dispositivo real).

### Observabilidad
- [ ] Los jobs de pg-boss se ejecutan y aparecen registrados en logs de pino.
- [ ] El flag `trace_id` está presente en los logs de requests del backend.

---

## 8. Recomendaciones Finales Antes de Producción

### Infraestructura y deploy
1. **Ejecutar `infra-ruta/scripts/migrate_prod.sh`** para aplicar el schema SQL en la BD de producción antes del primer deploy.
2. **Ejecutar `infra-ruta/scripts/backup_db.sh`** antes de cualquier migración en producción.
3. **Configurar `LOGTAIL_TOKEN`** en las variables de entorno de producción para activar el envío de logs a Better Stack.
4. **Verificar `infra-ruta/docs/deploy_produccion.md`** — checklist completo de deploy.
5. Confirmar que pg-boss tiene su base de datos de jobs inicializada y el worker está activo.

### Seguridad
6. **Rotar el master password de Vista de Control** antes del go-live (parámetro: `control_view.master_password_rotation_days = 90`).
7. **Configurar credenciales Wompi de producción** (no sandbox) en el tab Wompi de Configuración del cliente piloto.
8. **Verificar que las cookies tienen flags `Secure` y `HttpOnly`** en producción (HTTPS obligatorio).
9. **Confirmar que RLS (Row Level Security) está activo** en PostgreSQL de producción antes de onboardear el primer cliente.
10. **Revisar los rate limits** (`ratelimit.auth_attempts_per_minute_per_ip = 5`) y ajustar según el tráfico esperado.

### Datos y parámetros
11. **Insertar los parámetros globales por defecto** (`client_id = 0`) en la BD de producción usando los INSERTs de `docs-ruta/parametros_negocio.md`.
12. **Crear el primer usuario ADMIN_RUTA de producción** con contraseña segura (mínimo 10 caracteres, mayús + minús + dígito + símbolo).
13. **Configurar el primer cliente piloto** con sus datos reales antes del onboarding.

### Monitoreo
14. **Configurar alertas en Better Stack** para errores de nivel ERROR y FATAL en los logs.
15. **Verificar que los backups automáticos** de la BD de producción están programados y funcionan.
16. **Hacer una prueba de restauración** con el script `infra-ruta/scripts/backup_db.sh` antes del go-live.

### Onboarding del cliente piloto
17. **Preparar las credenciales Wompi sandbox** del cliente piloto para el período de prueba antes de activar producción.
18. **Capacitar al ADMIN_CLIENT del cliente piloto** usando las guías de usuario: `docs-ruta/guias/admin_client.md` y `docs-ruta/guias/courier.md`.
19. **Crear los couriers iniciales** del cliente piloto y enviarles las instrucciones de acceso.
20. **Configurar al menos un pickup point** si el cliente piloto va a usar el flujo PICKUP.

### Post-lanzamiento inmediato (primera semana)
21. Monitorear el log de auditoría del cliente piloto a diario durante la primera semana.
22. Revisar el historial de entregas de webhooks salientes para confirmar que el sistema del cliente piloto los está recibiendo.
23. Verificar que los jobs de pg-boss (expiración de DRAFTs, auto-confirmación de DELIVERED) se ejecutan correctamente revisando los logs cada mañana.
24. Documentar cualquier incidencia encontrada en la Matriz de Defectos de este plan como baseline para el sprint de hotfixes post-lanzamiento.

---

*Fin del documento — PLAN DE PRUEBAS RUTA MVP COMPLETO v2.0*
*Versión original v1.0: 2026-05-31 (163 casos, PT-001 a PT-287)*
*Versión v2.0: 2026-06-04 (325 casos, PT-001 a PT-325) — Producción en Render activa*
