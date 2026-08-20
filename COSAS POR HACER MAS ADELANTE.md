# COSAS POR HACER MÁS ADELANTE

Funcionalidades **decididas** pero pendientes, con el motivo y el paso a paso
para retomarlas. No es una lista de deseos: todo lo que está aquí ya se discutió
y se decidió el enfoque.

Hay dos clases de pendiente y conviene no confundirlas:

- **Sin construir** (§1, §2, §5): falta el código, y además algo de fuera.
- **Construido pero apagado** (§3, §4, §6): el código está hecho y probado; lo
  que falta es un trámite, una credencial o una decisión de negocio. Encenderlos
  es cuestión de minutos una vez esté lo que falta.

| # | Pendiente | Bloqueante | Estado del código |
|---|---|---|---|
| 1 | WhatsApp al repartidor al asignarle un pedido | Presupuesto + cuenta Meta | Sin construir |
| 2 | Guía de Servientrega al despachar por externo | Credenciales + WSDL | Sin construir |
| 3 | Aviso de entrega por correo al comprador | **Dominio propio** + Resend | **Hecho**, apagado |
| 4 | Seguimiento del repartidor en vivo | App nativa (otro proyecto) | Hecho a medias (última posición) |
| 5 | Captura de leads en la landing | Decidir si compensa | Sin construir (hoy `mailto`) |
| 6 | Cerrar pedidos de Nequi abandonados | Decisión de negocio (plazo) | Hecho salvo el cierre |

Al terminar cualquiera de estos, **borrar su sección** de este archivo y dejar
el registro en `docs-ruta/memoria_proyecto_ruta.md`.

---

## 1. Notificar al repartidor por WhatsApp al asignarle un pedido

**Estado:** aplazado.
**Fecha de la decisión:** 2026-07-22.
**Proveedor elegido:** Meta WhatsApp Cloud API (la API oficial de Meta, sin
intermediario).

### Qué se quiere

Cuando un pedido pasa a `COURIER_ASSIGNED` (asignación de repartidor desde el
mapa de asignación), enviar un mensaje de WhatsApp al teléfono del repartidor
con los detalles del pedido asignado: número de pedido, dirección de entrega,
nombre del comprador, monto y si es contra entrega.

### Por qué NO se hace todavía

**Presupuesto.** La Meta WhatsApp Cloud API cobra por conversación iniciada por
el negocio (las de tipo *utility*, que es la categoría de este aviso). No hay
presupuesto asignado para ese costo recurrente en este momento. La decisión fue
dejar el enfoque documentado y retomarlo cuando haya partida.

Se eligió Meta directo (en vez de Twilio u otro intermediario) porque, aunque
tiene más trámite inicial, es más barato a escala y no agrega el margen de un
tercero — coherente con no querer costos recurrentes de más.

### Restricción de negocio que condiciona TODO el diseño

WhatsApp **no permite enviar texto libre iniciado por el negocio**. Un aviso de
"te asignaron el pedido" lo inicia RUTA (no es respuesta a un mensaje del
repartidor), así que **obligatoriamente** usa una **plantilla pre-aprobada** por
Meta (*message template*, categoría *utility*).

Consecuencias:

- La plantilla se registra una vez en el WhatsApp Manager y Meta la aprueba
  (de minutos a ~2 días).
- La plantilla solo admite **variables posicionales** (`{{1}}`, `{{2}}`…), no
  formato arbitrario ni longitud libre.
- El repartidor debe haber aceptado recibir mensajes (opt-in). Para mensajes
  *utility* a un número que el negocio ya conoce esto es más laxo, pero conviene
  registrar el consentimiento al crear el repartidor.
- Fuera de la ventana de 24 h desde el último mensaje del repartidor, **solo**
  se puede enviar con plantilla. Este caso siempre cae fuera de esa ventana.

### Cómo encaja en la arquitectura actual (no reinventar)

El proyecto ya tiene el patrón exacto para esto: **cola pg-boss con reintentos +
worker + config por cliente en BD**, usado hoy por los webhooks salientes. La
integración de WhatsApp debe copiar ese patrón, no inventar uno nuevo.

Referencias vivas a imitar:

- Productor de job encolado: `backend-ruta/api/src/services/webhooks_outgoing.service.ts`
  (usa `boss.send(JOB, data, { retryLimit: 5, retryBackoff: true })`).
- Worker: `backend-ruta/api/src/jobs/webhook_sender.job.ts`
  (`boss.createQueue` + `boss.work`, timeout de fetch, registra resultado).
- Registro de jobs al arrancar: `backend-ruta/api/src/jobs/maintenance_boss.ts`
  (cada job se añade en `initMaintenanceJobs`).
- Punto de enganche: `backend-ruta/api/src/services/orders/courier_assignment.service.ts`,
  método `assignCourier`, justo **después** del bloque que emite el webhook
  `COURIER_ASSIGNED` (buscar el comentario `// F2.BACK-6 — COURIER_ASSIGNED webhook`).
  Ahí ya se tiene a mano `courier.phone`, `courier.full_name` y `orderId`.
- Config por cliente: tabla `client_parameters` + resolución en cascada
  (`backend-ruta/api/src/lib/parameter_resolver.ts`).
- Variables de entorno / secretos: `backend-ruta/api/src/config/env.ts`
  (ahí viven `WOMPI_*`, `GOOGLE_MAPS_API_KEY`, etc. — se agregan los de Meta).

**Principio clave:** el envío NO debe bloquear ni hacer fallar la asignación. Si
WhatsApp está caído, el repartidor igual queda asignado; el mensaje se reintenta
en segundo plano. Por eso va por la cola (`setImmediate` + `boss.send`), nunca
`await` dentro de la transacción de asignación.

---

### Paso a paso detallado

#### Fase A — Cuenta y plantilla en Meta (trámite, sin código)

1. **Crear/usar una cuenta de Meta Business**
   (`business.facebook.com`). Verificar el negocio si Meta lo pide.

2. **Añadir el producto WhatsApp** en un *app* de Meta for Developers
   (`developers.facebook.com` → Crear app → tipo *Business* → agregar WhatsApp).

3. **Registrar un número de teléfono remitente** en WhatsApp Manager. Puede ser
   el número de prueba que Meta regala al inicio (sirve para desarrollo) o un
   número propio verificado (para producción). Anotar el **Phone Number ID**
   (un número largo, NO el teléfono en sí).

4. **Anotar el WhatsApp Business Account ID (WABA ID).**

5. **Generar un token de acceso permanente:** crear un *System User* en Business
   Settings con permiso sobre el WABA, y generar un token que no expire. (El
   token temporal de 24 h del panel solo sirve para la primera prueba.)

6. **Crear la plantilla del mensaje** en WhatsApp Manager → Message Templates →
   Create:
   - Categoría: **Utility**.
   - Idioma: Español (Colombia) `es_CO` o Español `es`.
   - Nombre (ejemplo): `pedido_asignado_repartidor`.
   - Cuerpo sugerido (7 variables):

     ```
     Hola {{1}}, te asignaron el pedido #{{2}}.
     Entrega en: {{3}}.
     Comprador: {{4}}.
     Total: {{5}} {{6}}.
     Cobro contra entrega: {{7}}.
     Abre tu app de RUTA para ver el detalle y empezar el despacho.
     ```

   - Enviar a revisión y **esperar la aprobación** de Meta antes de poder
     enviar en producción.

7. **Definir precios/límites.** Revisar el costo por conversación *utility* en
   Colombia y el tier de mensajería del número. Esto es lo que hoy frena el
   proyecto: confirmar que hay presupuesto para el volumen esperado.

#### Fase B — Configuración en el backend

8. **Agregar variables de entorno** en `backend-ruta/api/src/config/env.ts`
   (y en el `.env` local + en Render para producción):

   ```
   WHATSAPP_PHONE_NUMBER_ID   # el Phone Number ID de la Fase A, paso 3
   WHATSAPP_ACCESS_TOKEN      # el token permanente del paso 5
   WHATSAPP_TEMPLATE_NAME     # p.ej. "pedido_asignado_repartidor"
   WHATSAPP_TEMPLATE_LANG     # p.ej. "es_CO"
   WHATSAPP_API_VERSION       # p.ej. "v21.0" (versión del Graph API)
   ```

   Añadir estos secretos a la lista de requeridos de forma **opcional**: si no
   están configurados, el envío se salta con un log (no rompe la asignación).

9. **Parámetro por cliente** en `client_parameters` para poder activar/desactivar
   el aviso sin tocar código, siguiendo el patrón de `limits.*`:

   ```
   notifications.whatsapp_on_assignment_enabled  (BOOLEAN, default false)
   ```

   Resolverlo con el helper de `parameter_resolver.ts`. Que arranque en `false`
   evita enviar (y gastar) hasta que el cliente lo active explícitamente.

#### Fase C — Cliente de WhatsApp (código)

10. **Crear `backend-ruta/api/src/lib/whatsapp_client.ts`** — un módulo delgado
    que hace el POST al Graph API. Firma sugerida:

    ```ts
    // POST https://graph.facebook.com/{API_VERSION}/{PHONE_NUMBER_ID}/messages
    // Authorization: Bearer {ACCESS_TOKEN}
    // body: { messaging_product: 'whatsapp', to, type: 'template',
    //         template: { name, language: { code }, components: [...] } }
    export async function sendTemplateMessage(input: {
      to: string;               // teléfono E.164 SIN el '+', p.ej. "573201002001"
      templateName: string;
      languageCode: string;
      variables: string[];      // rellena {{1}}..{{n}} en orden
    }): Promise<{ ok: boolean; providerMessageId?: string; error?: string }>
    ```

    - El teléfono ya se guarda con indicativo (`+57…`) gracias al selector de
      país del formulario de repartidores; para Meta hay que **quitar el `+`**.
    - Timeout con `AbortController` (~10 s), como en `webhook_sender.job.ts`.
    - Nunca lanzar hacia arriba: devolver `{ ok: false, error }` y dejar que el
      worker decida reintentar.
    - Usar el `logger` de pino, nunca `console.log` (regla 4.12).

#### Fase D — Cola y worker (código)

11. **Crear el servicio productor**
    `backend-ruta/api/src/services/notifications/whatsapp_notifications.service.ts`
    con:
    - La constante del job: `export const SEND_WHATSAPP_JOB = 'send_whatsapp';`
    - El tipo del payload del job (`to`, `templateName`, `languageCode`,
      `variables`, `clientId`, `orderId`, `courierUserId`).
    - Una función `queueAssignmentNotification(...)` que arma las variables de la
      plantilla a partir del pedido y encola con
      `boss.send(SEND_WHATSAPP_JOB, data, { retryLimit: 5, retryBackoff: true })`.

12. **Crear el worker**
    `backend-ruta/api/src/jobs/whatsapp_sender.job.ts`, calcado de
    `webhook_sender.job.ts`:
    - `registerWhatsappSenderJob(boss)` que hace `boss.createQueue` + `boss.work`.
    - Dentro, llama a `sendTemplateMessage`. Si `ok === false`, **lanza** para
      que pg-boss reintente (respeta `retryLimit`/`retryBackoff`).
    - Registrar el resultado con el logger (y, si se quiere trazabilidad
      persistente, en `audit_events` con `action: 'whatsapp_notification_sent'`).

13. **Registrar el worker** en `maintenance_boss.ts`, dentro de
    `initMaintenanceJobs`, junto a los demás `register…Job(boss)`.

#### Fase E — Enganche en la asignación

14. En `courier_assignment.service.ts`, método `assignCourier`, **después** del
    bloque `// F2.BACK-6 — COURIER_ASSIGNED webhook`:
    - Resolver `notifications.whatsapp_on_assignment_enabled` para el cliente.
    - Si está activo y `courier.phone` existe, encolar la notificación con
      `setImmediate(() => queueAssignmentNotification(...).catch(log))`.
    - Para armar las variables de la plantilla hace falta la dirección de entrega
      y el comprador; si no vienen en el `select` actual del `assignCourier`,
      ampliar ese `select` o hacer una lectura ligera adicional dentro del job.
    - **Nunca** poner `await` del envío en la ruta crítica de la asignación.

#### Fase F — Pruebas

15. **Test del worker** (Vitest + mock de `fetch`), imitando
    `wompi_webhook.test.ts`: verifica que se arma bien el body del template, que
    un 2xx marca éxito y que un error se reintenta.

16. **Test de que la asignación NO se rompe** si WhatsApp falla: `assignCourier`
    debe devolver `COURIER_ASSIGNED` aunque el encolado/env­ío falle.

17. **Prueba manual end-to-end** con el número de prueba de Meta antes de pasar a
    un número productivo: asignar un pedido y confirmar que llega el WhatsApp con
    las variables correctas.

#### Fase G — Documentación al cerrar

18. Registrar el endpoint/flujo en `docs-ruta/contrato_api.md` si se expone algo,
    el nuevo parámetro en `docs-ruta/parametros_negocio.md`, y el avance en
    `docs-ruta/memoria_proyecto_ruta.md` (regla 0.2 del CLAUDE.md).

19. Mover esta sección de este archivo a "hecho" (borrarla de aquí) cuando quede
    en producción.

---

### Resumen de lo que falta para arrancar

| Bloqueante | Quién lo resuelve |
|---|---|
| Presupuesto para el costo por conversación *utility* | Negocio |
| Cuenta Meta Business + app + número remitente | Negocio / admin |
| Plantilla `pedido_asignado_repartidor` aprobada | Negocio (trámite en Meta) |
| Token permanente + Phone Number ID | Negocio / admin |
| Implementación Fases B–G | Desarrollo (≈ 1 día una vez estén las credenciales) |

Sin lo primero (presupuesto), lo demás no arranca. Por eso queda aquí.

---

## 2. Despacho por mensajería externa vía API de Servientrega

**Estado:** aplazado.
**Fecha de la decisión:** 2026-07-23.
**Proveedor elegido:** Servientrega (mensajería colombiana).

### Qué se quiere

Cuando el ADMIN_CLIENT marca un pedido como listo y elige **envío por externo**
(`delivery_carrier_type = 'EXTERNAL_COURIER'`), en vez de solo dejar el pedido
"listo para recoger", **generar la guía en Servientrega** vía su API: crear el
envío con los datos del pedido (remitente = negocio, destinatario = comprador,
dirección, peso/dimensiones), obtener el **número de guía / rastreo** y
guardarlo en el pedido para poder darle seguimiento.

### Por qué NO se hace todavía

Faltan dos cosas que no se pueden suplir desde el código:

1. **Cuenta y credenciales de Servientrega.** Se necesita una cuenta empresarial
   con ellos: usuario, contraseña, código de facturación / ID de cliente de su
   API. Sin eso no se puede autenticar ni generar guías.
2. **La documentación de su API.** Servientrega expone un web service
   (históricamente SOAP, operaciones tipo `GenerarGuia`, `GenerarGuiaConValores`).
   Sin su WSDL / especificación de endpoints y campos exactos, cualquier
   implementación sería adivinada y muy probablemente no cuadraría con lo que su
   servicio espera.

Además, cada guía generada tiene costo (es un envío real facturado por
Servientrega), así que conviene tener presupuesto y ambiente de pruebas antes de
activarlo.

### Estado actual del despacho (lo que ya existe)

- En el detalle del pedido, al marcar listo, el admin elige el carrier
  (`frontend-ruta/admin/src/app/(protected)/admin/orders/[id]/OrderDetailClient.tsx`,
  selector "Flota propia / Mensajería externa").
- `markReady` en `backend-ruta/api/src/routes/admin_orders.ts` (~línea 644)
  decide el estado destino:
  - `PICKUP` → `READY_FOR_PICKUP`.
  - `SHIP` + `OWN_FLEET` → `AWAITING_COURIER_ASSIGNMENT` (mapa de asignación).
  - `SHIP` + `EXTERNAL_COURIER` → `READY_TO_SHIP` **(hoy no hace nada más)**.

El enganche de Servientrega va justo en esa rama `EXTERNAL_COURIER`, **después**
de que la transición a `READY_TO_SHIP` quede confirmada.

### Cómo encaja en la arquitectura actual (no reinventar)

Mismo patrón que webhooks salientes y que la integración de WhatsApp aplazada:
**cola pg-boss con reintentos + worker + config por cliente en BD**, sin bloquear
el "marcar listo". Referencias vivas a imitar:

- Cola/productor: `backend-ruta/api/src/services/webhooks_outgoing.service.ts`.
- Worker: `backend-ruta/api/src/jobs/webhook_sender.job.ts`.
- Registro de jobs: `backend-ruta/api/src/jobs/maintenance_boss.ts`.
- Config por cliente (credenciales): la tabla `client_payment_providers` es un
  buen modelo de "config de proveedor por cliente con secretos"; puede crearse
  algo análogo `client_shipping_providers`, o reusar `client_parameters`.
- Variables de entorno / secretos: `backend-ruta/api/src/config/env.ts`.

**Principio clave:** generar la guía NO debe bloquear ni hacer fallar el "marcar
listo". Si Servientrega está caído, el pedido igual queda `READY_TO_SHIP`; la
guía se genera en segundo plano con reintentos. Por eso va por la cola
(`setImmediate` + `boss.send`), nunca `await` dentro de la transacción.

### Paso a paso detallado

#### Fase A — Cuenta y credenciales (trámite, sin código)

1. **Contratar/activar la cuenta empresarial de Servientrega** con acceso a su
   API de generación de guías.
2. **Obtener las credenciales de API:** usuario, contraseña, código de
   facturación / ID de cliente, y la URL del web service (sandbox y producción).
3. **Conseguir la documentación / WSDL** de las operaciones a usar
   (`GenerarGuia` y afines): campos obligatorios de remitente, destinatario,
   producto, forma de pago, contra entrega si aplica, etc.
4. **Confirmar presupuesto** para el costo por guía y si hay ambiente de pruebas.

#### Fase B — Modelo de datos

5. **Guardar la guía en el pedido.** Opciones:
   - Agregar columnas a `orders`: `external_tracking_number TEXT`,
     `external_carrier TEXT`, `external_label_url TEXT` (implica migración del
     esquema compartido `docs-ruta/bd/ruta_postgres.sql` + Prisma).
   - O guardarlo en un JSONB existente / campo de metadatos para evitar migración
     (más rápido, menos consultable).
6. **Config de credenciales por cliente.** Crear `client_shipping_providers`
   (análogo a `client_payment_providers`, con secretos write-only) o guardar en
   `client_parameters` bajo un grupo `shipping.servientrega.*`. Los secretos
   nunca se devuelven en texto plano (ver cómo lo hace el tab de Wompi).

#### Fase C — Cliente de Servientrega (código)

7. **Crear `backend-ruta/api/src/lib/servientrega_client.ts`** — módulo que
   habla con el web service. Como es SOAP, evaluar una librería SOAP o construir
   el sobre XML a mano con `fetch`. Firma sugerida:

   ```ts
   export async function generarGuia(input: {
     credenciales: { usuario: string; clave: string; codFacturacion: string };
     remitente: { nombre; direccion; ciudad; telefono };
     destinatario: { nombre; direccion; ciudad; telefono };
     pedido: { id: number; peso?: number; valorDeclarado?: number; contraentrega?: number };
   }): Promise<{ ok: boolean; numeroGuia?: string; urlRotulo?: string; error?: string }>
   ```
   - Timeout con `AbortController` (~15 s).
   - Nunca lanzar hacia arriba: devolver `{ ok:false, error }` y dejar que el
     worker reintente.
   - Usar el `logger` de pino, nunca `console.log`.

#### Fase D — Cola y worker (código)

8. **Servicio productor**
   `backend-ruta/api/src/services/shipping/servientrega.service.ts`: constante
   del job `SEND_SERVIENTREGA_JOB = 'generate_servientrega_guide'`, tipo del
   payload (clientId, orderId, datos del envío) y `queueGenerateGuide(...)` que
   encola con `boss.send(..., { retryLimit: 5, retryBackoff: true })`.
9. **Worker** `backend-ruta/api/src/jobs/servientrega_sender.job.ts` (calcado de
   `webhook_sender.job.ts`): llama a `generarGuia`; si `ok`, guarda el número de
   guía en el pedido (Fase B) y audita; si falla, lanza para que pg-boss
   reintente.
10. **Registrar el worker** en `maintenance_boss.ts`.

#### Fase E — Enganche en "marcar listo"

11. En `admin_orders.ts`, método `markReady`, en la rama
    `EXTERNAL_COURIER` (después de confirmar `READY_TO_SHIP`): resolver la config
    de Servientrega del cliente; si está activa, encolar
    `setImmediate(() => queueGenerateGuide(...).catch(log))`. **Nunca** `await`
    del envío en la ruta crítica.

#### Fase F — UI

12. Mostrar el número de guía y (si aplica) el enlace al rótulo en el detalle del
    pedido admin (`OrderDetailClient.tsx`) cuando exista.
13. Opcional: un tab de configuración de Servientrega en Configuración del admin,
    igual que el de Wompi, para que el cliente cargue sus credenciales.

#### Fase G — Pruebas y documentación

14. Test del worker con `fetch` mockeado (arma bien el sobre SOAP, un OK guarda
    la guía, un error reintenta).
15. Test de que `markReady` NO se rompe si Servientrega falla.
16. Prueba manual contra el ambiente de pruebas de Servientrega antes de
    producción.
17. Documentar en `contrato_api.md`, `parametros_negocio.md` y
    `memoria_proyecto_ruta.md`; mover esta sección a "hecho" al quedar en prod.

### Resumen de lo que falta para arrancar

| Bloqueante | Quién lo resuelve |
|---|---|
| Cuenta empresarial + credenciales de API Servientrega | Negocio |
| Documentación / WSDL de la API | Negocio / Servientrega |
| Presupuesto por guía + ambiente de pruebas | Negocio |
| Decisión de modelo de datos (columnas vs JSON) | Desarrollo (config de BD → consultar) |
| Implementación Fases B–G | Desarrollo (≈ 1–2 días una vez estén credenciales y spec) |

Sin las credenciales y la especificación de su API, lo demás no arranca. Por eso
queda aquí.

---

## 3. Encender el aviso de entrega por correo (falta el dominio)

**Estado:** implementado, apagado por falta de dominio propio.
**Fecha de la decisión:** 2026-08-11.
**Proveedor elegido:** Resend (API HTTP).

### Qué se quiere

Cuando un pedido se marca como entregado, enviarle un correo al comprador
avisándole. El Cliente configura el nombre con el que firma, el correo al que le
responden, el asunto y el mensaje.

### Estado actual: **ya está construido**

A diferencia de las secciones 1 y 2, aquí el código está completo y probado:

- Cliente de correo aislado en `backend-ruta/api/src/lib/email_client.ts`.
- Cola pg-boss con reintentos: `jobs/delivery_email.job.ts` +
  `services/notifications/delivery_email.service.ts`.
- Enganchado en las dos entregas: `courier_ops.service.ts` (SHIP) y
  `pickup_ops.service.ts` (PICKUP), con `setImmediate` tras el webhook.
- Configuración por Cliente en `client_parameters`
  (`notifications.delivery_email_*`), editable en la pestaña **Correos** de
  Configuración.

**Está apagado, no roto.** Sin `EMAIL_FROM` o `EMAIL_API_KEY` el worker descarta
el envío con un log y la entrega del pedido no se ve afectada.

### Por qué NO funciona todavía

**RUTA no tiene dominio propio.** El correo sale desde una dirección de RUTA
(no de cada Cliente) y esa dirección debe pertenecer a un dominio **verificado**
en el proveedor. Sin dominio no hay remitente verificable, y un correo enviado
desde un dominio sin verificar se rechaza o cae en spam — que es peor que no
enviarlo, porque nadie se entera del fallo.

### Por qué se envía desde el dominio de RUTA y no del Cliente

Decisión tomada el 2026-08-11. La alternativa —que cada Cliente enviara desde su
propio dominio— obliga a **verificar el dominio de cada uno**: un trámite de DNS
por cada negocio que entre a la plataforma, y un freno serio para el onboarding.

Con el enfoque elegido se verifica **un solo dominio, una sola vez**, y sirve
para todos los Clientes. El comprador ve el nombre del negocio como remitente y,
al responder, le escribe al negocio gracias al `Reply-To`.

### Paso a paso

1. **Comprar el dominio** de RUTA/ORKO (p. ej. `uruta.co`).
2. **Crear la cuenta en Resend** (`resend.com`). El plan gratuito cubre de sobra
   el volumen inicial.
3. **Verificar el dominio en Resend**: añadir los registros DNS que indique
   (SPF, DKIM y, si lo pide, DMARC). Tarda de minutos a unas horas en propagar.
4. **Generar la API key** en Resend.
5. **Añadir al `.env`** de `backend-ruta` (y a Render en producción):

   ```
   EMAIL_API_KEY=re_...
   EMAIL_FROM=pedidos@tudominio.co
   EMAIL_FROM_NAME=RUTA          # respaldo si el Cliente no pone nombre
   ```

6. **Reiniciar el backend.** No hay nada más que tocar en el código.
7. **Probar**: entregar un pedido de un comprador **con cuenta** (los invitados
   no reciben correo a propósito) y confirmar que llega y que al responder la
   respuesta va al correo del Cliente.

### Decisiones ya tomadas que conviene no revertir sin pensarlo

- **A los invitados no se les escribe.** Su correo es sintético
  (`guest-<uuid>@guest.ruta`) y no existe: enviarlo generaría rebotes que queman
  la reputación del dominio, que aquí es **compartido por todos los Clientes**.
- **El worker no reintenta lo que nunca va a funcionar** (sin proveedor,
  invitado, rechazo 4xx del proveedor). Solo reintenta fallos transitorios.
- **El envío nunca bloquea la entrega**: va por cola con `setImmediate`.

### Resumen de lo que falta para arrancar

| Bloqueante | Quién lo resuelve |
|---|---|
| Comprar el dominio propio | Negocio |
| Cuenta en Resend + verificación DNS del dominio | Negocio / admin |
| `EMAIL_API_KEY` y `EMAIL_FROM` en `.env` y en Render | Desarrollo (5 min) |
| Implementación | **Ya hecha** |

---

## 4. Seguimiento del repartidor en vivo (hoy es "última posición conocida")

**Estado:** aplazado. Lo que hay funciona, pero no es seguimiento continuo.
**Fecha de la decisión:** 2026-08-11.

### Qué hay hoy

El comprador ve al repartidor en un mapa desde que este pulsa "Iniciar despacho".
El repartidor reporta su posición cada 20 s (`POST /courier/location`) y el
comprador sondea cada 20 s (`GET /buyer/orders/:id/courier-location`).

### Por qué no es "en vivo"

El panel del repartidor es **una página web**. `navigator.geolocation.watchPosition`
solo corre con la pestaña en primer plano: cuando el repartidor bloquea el
teléfono o se pasa a Waze —cosa que la propia pantalla le invita a hacer, con sus
botones de Google Maps y Waze— el navegador suspende los temporizadores y deja de
reportar.

Por eso la interfaz muestra **cuándo** se tomó la posición y avisa cuando ya
está desfasada (parámetro `tracking.courier_location_ttl_seconds`, 300 s). Un
punto quieto que aparenta estar al día es peor que ninguno: deja al comprador
esperando en la puerta.

### Qué haría falta para seguimiento continuo

Una **app nativa** (o una PWA con `background geolocation`) con permiso de
ubicación en segundo plano. Es un proyecto aparte, no un ajuste:

1. Decidir plataforma: app nativa (React Native / Flutter) o PWA con plugin de
   geolocalización en segundo plano.
2. Permiso "siempre" de ubicación, que en iOS y Android exige justificarlo ante
   la tienda de aplicaciones y ante el usuario.
3. Publicación en las tiendas (cuenta de desarrollador, revisión).
4. Reusar el endpoint que ya existe: `POST /courier/location` no cambia.

**Conviene decidirlo antes de prometerle "seguimiento en vivo" a un cliente.**
Lo que hay hoy es honesto pero no es eso.

### Además, pendiente y barato

- Dejar por escrito en el contrato o la guía del repartidor que se comparte su
  ubicación mientras lleva un pedido. En pantalla ya se le avisa, pero conviene
  que quede documentado.

---

## 5. Captura de leads en la landing

**Estado:** aplazado.
**Fecha de la decisión:** 2026-08-11.

La landing de `/` en el storefront pide demos con un enlace `mailto:` a
`simon.marquez@orko.com.co`, con el asunto y el cuerpo pre-rellenos.

**No hay captura de leads**: no existe endpoint ni tabla para registrar
interesados, así que un formulario propio no guardaría nada. Se prefirió un
`mailto` honesto antes que un formulario que aparentara funcionar.

### Qué haría falta

1. Tabla `leads` (no operativa, sin `client_id`: son interesados en RUTA, no de
   un Cliente).
2. Endpoint público `POST /public/leads` con **anti-spam** (rate limit por IP y
   honeypot como mínimo; captcha si llega basura).
3. Formulario en la landing reemplazando el `mailto`.
4. Aviso al equipo comercial: correo o webhook, reusando la cola de pg-boss.

Mientras el volumen sea bajo, el `mailto` cumple. Esto se justifica cuando
lleguen suficientes solicitudes como para perder alguna.

---

## 6. Cerrar pedidos abandonados pagados por link de Nequi

**Estado:** decisión pendiente.
**Fecha de la decisión:** 2026-08-11.

Los pedidos con submétodo `PAYMENT_LINK` (Nequi) están **excluidos** del job
`payment_timeout.job.ts` a propósito: como no hay webhook que confirme el pago,
incluirlos cancelaría pedidos **ya pagados**, que es mucho peor que dejar abierto
uno abandonado.

**Efecto secundario:** un pedido de Nequi que nadie pagó se queda abierto
indefinidamente y le ensucia la lista al Cliente.

### Opciones a decidir

- **Plazo propio, más largo**, solo para pagos por link (p. ej. 24 h en vez de
  los minutos del pago online), con su propio parámetro en `client_parameters`.
- **Cierre manual cómodo**: un filtro "esperando pago por link" en la lista de
  pedidos y una acción de cancelar en lote.
- **Las dos**: plazo largo automático y la posibilidad de cerrarlos antes a mano.

La primera es la más barata y probablemente suficiente. Requiere confirmar con
el negocio cuánto tiempo es razonable esperar un pago de Nequi.
