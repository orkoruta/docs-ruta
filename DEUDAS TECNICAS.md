# DEUDAS TÉCNICAS

Atajos conscientes y problemas conocidos del código, **no** funcionalidades
pendientes (esas van en `COSAS POR HACER MAS ADELANTE.md`).

Cada entrada dice qué está mal, **cómo se comprobó**, qué se rompe si no se
arregla y por dónde entrarle. Todo lo de aquí se verificó contra el código y la
BD el **2026-08-11**; lo que no se pudo comprobar está marcado como tal.

Al resolver una, **borrar su sección** y dejar el registro en
`docs-ruta/memoria_proyecto_ruta.md`.

### Resueltas

- **2026-08-11 — Invitados huérfanos acumulándose en `users`.** Job
  `cleanup_guest_buyers`: los invitados que no llegaron a hacer ningún pedido se
  borran a los 10 minutos (`cleanup.guest_orphan_minutes`). Sin retención.
- **2026-08-11 — pg-boss moría en silencio.** Tres causas, no una: no había
  `on('error')` (y un evento `error` sin oyente es excepción no capturada en
  Node); un fallo de arranque dejaba la promesa **rechazada y memoizada**, así
  que ningún intento posterior podía prosperar; y nada exponía el estado. Ahora
  reintenta con backoff (5 s → 5 min, indefinido), registra los errores de
  ejecución y `GET /healthz/jobs` responde **503** mientras no estén corriendo,
  para poder monitorizarlo aparte sin sacar la API del balanceador. La API
  reintenta por dentro; el worker, que no expone HTTP, sale con error para que
  la plataforma lo reinicie. 6 tests nuevos.
- **2026-08-11 — `FORCE ROW LEVEL SECURITY`.** Verificado: las **21** tablas
  operativas de la BD activa tienen RLS activo y forzado (comprobado tabla por
  tabla), y el esquema es coherente (21 ENABLE = 21 FORCE). **La BD de
  producción todavía no existe** —solo hay `rutadb`—, así que se blindó el
  momento en que se cree: `verify_prod.sh` ahora falla con error si detecta RLS
  sin forzar, existe `fix_force_rls.sh` para corregirlo, y el Paso 3 de
  `deploy_produccion.md` lo exige antes de continuar.
- **2026-08-11 — Lista de códigos de error duplicada a mano.** `lib/errors.ts`
  copiaba `ERROR_CODES` de `@orkoruta/shared`; añadir un código en shared rompía
  la compilación hasta duplicarlo. Ahora se importa.
- **2026-08-11 — Filtros de fecha: 400 en tres pantallas y un día perdido en
  otra.** Los `<input type="date">` mandan `2026-08-11`, pero `pedidos`,
  `auditoría` y `reembolsos` validaban con `z.string().datetime()`, que exige
  ISO-8601 completo: **elegir una fecha devolvía 400 y rompía el listado
  entero**. En `entregas de webhook`, que sí aceptaba el día, el corte era
  `lte` a medianoche **UTC**, así que «hasta el 11» excluía el 11 completo.
  Ahora hay un único `dateFilterSchema` en shared, que acepta día calendario e
  instante ISO, y las fronteras son las del día **en Bogotá** (UTC-5, sin
  horario de verano) con límite superior exclusivo. 11 tests nuevos. De paso:
  `Date.parse('2026-02-30')` **no falla** —se desborda a 2 de marzo—, así que
  el validador comprueba que la fecha construida sea la que se escribió.
- **2026-08-11 — El buscador y el filtro de origen de pedidos no filtraban.**
  La pantalla mandaba `q` y `order_origin`; el esquema no los tenía y Zod los
  descartaba en silencio, así que salía la lista completa y parecía que no
  había coincidencias. Ahora `q` busca por nombre y correo del comprador y, si
  es solo dígitos, también por id de pedido (acotado al rango de BIGINT: un
  número mayor abortaba la consulta entera). 13 tests nuevos.
- **2026-08-11 — Marcar un reembolso como ejecutado fallaba siempre.** El panel
  mandaba `result` donde la API espera `outcome`, que además es **obligatorio**:
  no era un campo ignorado en silencio, era un 400 en cada intento. El tipo del
  frontend ahora se deriva del esquema de shared.
- **2026-08-11 — Puntos físicos: el módulo entero hablaba otro idioma.** La
  interfaz y los formularios usaban `address`, `phone` y `schedule`; la API
  emite y espera `address_line`, `contact_phone` y `opening_hours`. La dirección
  y el teléfono salían **en blanco** en la lista y en el detalle, y al guardar
  solo se persistían el nombre, la ciudad y el departamento —el único punto que
  existe en la BD lo había creado el script de seed—. Además: `opening_hours` es
  un objeto JSON, no texto (ahora se edita como `días: horas`, una franja por
  línea, y round-trip exacto); el `status` iba en el cuerpo, que no lo acepta
  (se cambia con `POST /:id/activate` y `/deactivate`); el botón «Eliminar»
  llamaba a un `DELETE` **que no existe** y devolvía 404, y borrar de verdad
  tampoco procede porque los pedidos PICKUP apuntan al punto; y `page.tsx`
  arrastraba 152 líneas de un componente de detalle duplicado y muerto.
- **2026-08-12 — Configuración muerta de JWT: eran dos, no una.**
  `JWT_ACCESS_TOKEN_LIFETIME_MINUTES` estaba documentada como muerta, pero
  `JWT_REFRESH_TOKEN_LIFETIME_DAYS` **también** lo estaba: ninguna de las dos
  se leía en ninguna parte. Las dos vidas se resuelven por rol desde
  `client_parameters` (`auth.jwt_lifetime_<rol>_minutes` y
  `auth.refresh_token_lifetime_<rol>_days`). Borradas de `env.ts`,
  `.env.example` y los dos `render.yaml`, con un comentario en `env.ts` que
  apunta a los parámetros que sí mandan.
- **2026-08-12 — «Código de empresa» decía «opcional» y no lo es.** Si se deja
  vacío el login llama a `/auth/ruta-admin/login`, así que un Cliente, un
  operador o un repartidor que lo dejara en blanco intentaba autenticarse como
  equipo RUTA y no entraba, sin pista de por qué. Ahora dice «obligatorio,
  salvo si eres del equipo RUTA». El `catch` que la deuda señalaba como vacío
  ya tenía cuerpo y comentario, y tragar ahí es lo correcto: es la restauración
  de sesión, y si falla lo que toca es caer al login.
- **2026-08-12 — `isCod` siempre falso, y la tarjeta «Pago» invisible.** Eran
  dos bugs con la misma raíz: el serializador del detalle no emitía `payment`.
  La tarjeta «Pago» del panel está envuelta en `{order.payment && …}`, así que
  **nunca se pintó**: estado, método, monto y fecha de confirmación llevaban
  ocultos desde que se escribió la pantalla.

  El arreglo propuesto en esta misma lista —usar `payment.method` para
  `isCod`— **no habría funcionado**: en PICKUP la fila de `payments` se crea al
  cobrar (`pickup_ops.service.ts:146`, exige READY_FOR_PICKUP), de modo que
  antes del cobro es `null`, justo cuando el operador necesita ver el paso.
  `isCod` ahora sale de `orders.payment_method`, que se fija al crear el pedido
  y siempre está.

  El detalle ya emite `payment` (arregla la tarjeta) **sin** la evidencia de
  cobro: es el JSONB con la foto en base64 y arrastrarla metería cientos de kB
  en cada lectura. La sirve `GET /admin/orders/:id/collection-evidence`, que es
  lo que ya usa `CollectionEvidenceCard`. 7 tests nuevos.
- **2026-08-12 — Cuatro módulos duplicados entre admin y storefront.**
  `map_theme`, `google-maps`, `phone_country_codes` y `delivery_date` estaban
  en las dos apps, byte a byte idénticos. Se movieron a un paquete nuevo,
  **`@orkoruta/web-shared`**, y se actualizaron los 13 ficheros que los
  importaban.

  No fueron a `@orkoruta/ui` porque tres de los cuatro no son design system
  (un cargador de la API de Maps, una tabla de indicativos y utilidades de
  fecha). El paquete se consume **como fuente TypeScript**, igual que `ui` y
  sin build, a propósito: un paquete con `dist/` enlazado por `link:` provoca
  carreras cuando alguien recompila con los dev servers o la suite en marcha
  (ver deuda #4). Añadido a `pnpm-workspace.yaml` y a `transpilePackages` de
  ambas apps.
- **2026-08-12 — El admin no manejaba el 401.** Al caducar el token las
  pantallas se quedaban vacías o soltaban errores sueltos, sin decir que había
  que volver a entrar; en el mapa de asignación, que recarga cada 30 s, era una
  ráfaga de 401 en consola y nada visible. Se portó el patrón del storefront
  (`lib/session-events.ts`): **20 de los 22 módulos de API** emiten
  `notifyUnauthorized()` al recibir un 401 y el layout protegido lo escucha una
  vez, limpia la sesión y manda a `/login?expired=1`, donde ahora sale «Tu
  sesión caducó».

  Dos exclusiones **a propósito**: `auth.api.ts`, porque un 401 al entrar es una
  contraseña mala y no una sesión caducada, y `control_view.api.ts`, porque su
  401 es la contraseña maestra y emitirlo echaría del panel a un ADMIN_RUTA por
  teclearla mal. También queda fuera la subida a la URL prefirmada del bucket:
  ese 401 es de la firma, no de RUTA.

  No se unificaron los ~16 `request()` en uno solo: es un refactor mayor que
  toca todos los módulos y encaja mejor con la deuda #6.
- **2026-08-12 — Mojibake en `memoria_proyecto_ruta.md` y
  `parametros_negocio.md`.** 198 y 93 líneas con UTF-8 leído como Latin-1
  (`descripciÃ³n`, `dÃ­as`). Un round-trip global no servía: los ficheros eran
  **mixtos**, con acentos correctos junto a los rotos, así que se reparó con
  `ftfy` y se comprobó línea a línea que no cambiara nada que no fuera
  mojibake. De paso se quitó el BOM de `memoria_proyecto_ruta.md`. Ya no hace
  falta escribir con la codificación rota para no empeorarlos.
- **2026-08-11 — Tests de shared que fijaban contratos inexistentes.** Los casos
  de puntos físicos y reembolsos comprobaban la copia de shared que no usaba
  nadie: `address`, `page_size` por defecto 50 (el backend siempre usó 20),
  `refund_status` en vez de `status`, `order_id` en el cuerpo cuando va en la
  ruta, y un `markRefundExecuted` con cuerpo vacío que la API rechaza. Pasaban
  en verde mientras la pantalla fallaba. Reescritos contra el contrato real.

## Índice

| # | Deuda | Impacto | Esfuerzo |
|---|---|---|---|
| 1 | Sin object storage: fotos en base64 dentro de la BD | **Mitigado** (purga a 14 días); falta el bucket | Alto |
| 2 | Tipos del frontend escritos a mano en vez de generados | **Parcial**: cero esquemas duplicados; faltan 43 sin gemelo en shared | Medio |
| 4 | Suite de tests con fallos intermitentes | Medio (erosiona confianza) | **Alto**: tres hipótesis descartadas |

---

## 1. No hay object storage: las fotos viven en base64 dentro de la BD

**Cómo se comprobó:** `POST /uploads/presigned-url` responde **501**
(`routes/uploads.ts:15`, comentario "File storage provider is TBD"). Las
evidencias de cobro se guardan en el JSONB `payments.collection_evidence`.

**Estado hoy:** 4 pagos con evidencia, **148 kB** en total (cliente 4).

**Mitigado el 2026-08-11:** el job `purge_collection_evidence` borra las fotos a
los **14 días** (`storage.evidence_retention_days`, antes 730) y deja la marca
`purged_at` para poder decir «expiró» en vez de «nunca hubo». Eso acota el
crecimiento, pero **no sustituye al bucket**: durante esas dos semanas las fotos
siguen dentro de Postgres, y una evidencia que desaparece a los 14 días no sirve
como respaldo ante una disputa tardía.

**Por qué se hizo:** decisión explícita del 2026-07-22 para no bloquear las
pruebas del MVP. El campo acepta URL http(s) **o** data URI, así que migrar no
cambia el contrato de la API.

**Qué se rompe si no se arregla:** las fotos siguen pesando dentro de Postgres
durante su vigencia, encareciendo backups y réplicas, y cada lectura del pago
arrastra cientos de kB. Y sobre todo: con object storage la retención sería una
decisión de negocio, no una obligación técnica — hoy se borran a las dos semanas
**porque no caben**, no porque convenga.

**Por dónde entrarle:**
1. Elegir proveedor (S3, R2, Supabase Storage…).
2. Implementar `POST /uploads/presigned-url` de verdad.
3. `ReceiptCapture.tsx` sube al bucket y manda solo la URL.
4. Migrar los data URI existentes a ficheros y sustituirlos por URL.
5. Restringir el validador a solo URL cuando no quede ningún data URI.

---

## 2. Los tipos del frontend se escriben a mano en vez de generarse

**Cómo se comprobó:** en la sesión del 2026-07-21/22 se contabilizaron **diez
bugs** con el mismo origen: el frontend mandaba `search`/`limit` donde el backend
esperaba `q`/`page_size`, `parameter_value` donde esperaba `value`,
`vehicle_type` donde esperaba `transport_mode`, `periodicity` donde el backend
emite `recurrence_periodicity`… El síntoma es siempre el mismo: **Zod ignora el
campo desconocido en silencio** y la pantalla se queda vacía sin error.

**Qué se rompe:** cada endpoint nuevo es una oportunidad de repetirlo, y el fallo
no aparece ni en typecheck ni en los tests, porque ambos lados se creen
correctos por separado.

### Estado 2026-08-12: no queda ningún esquema duplicado

Se revisó **cada query y cada cuerpo de mutación** que el frontend construye,
comparándolo con el esquema Zod que lo valida, y después se buscaron
sistemáticamente los **pares duplicados**: el mismo endpoint definido a la vez
en `@orkoruta/shared` y en el backend. Eran la fuente real de los bugs, porque
la copia de shared podía describir un contrato que la API nunca tuvo sin que
nada fallara —nadie la importaba—.

**Ya no queda ninguno.** Shared define, el backend reexporta. Lo que se
encontró al hacerlo:

| Esquema | Qué decía shared | Qué acepta la API |
|---|---|---|
| `updateProductSchema` | estado `ARCHIVED` | **el CHECK de la BD solo admite ACTIVE/INACTIVE** |
| `updateCategorySchema` | estado `ARCHIVED` | ídem |
| `updateCourierSchema` | `email` y `password`; sin `vehicle_plate` | al revés: no acepta credenciales y sí la placa |
| `registerBuyerSchema` | nombre y documento obligatorios, `document_type` máx. 10 | los tres opcionales, máx. 20 |
| `controlViewEnterSchema` | `reason` obligatorio | opcional |
| `updateBuyerProfileSchema` | `document_type`, `document_number`, `default_address` | solo nombre y teléfono, y con `.trim()` |

Ninguno estaba causando un fallo visible **todavía**, porque las pantallas que
los usarían aún no existen o no tocan esos campos. El de Vista de Control sí
avisó: al conectarlo, tres tests del backend pasaron a devolver 400.

**Regla que quedó fijada, con tests:** el contrato describe **lo que la API
acepta**, no lo que la interfaz exige. Una pantalla puede pedir más que la API
—el formulario de registro pide el documento y está bien—; el esquema
compartido, nunca. Endurecerlo es cambiar el endpoint, y eso afecta a los
Clientes API.

**Lo que falta:** 43 esquemas viven solo en el backend, sin gemelo en shared.
No pueden divergir porque no hay dos copias, pero tampoco protegen al frontend,
que sigue escribiendo a mano los tipos de esos endpoints (`orders`, `products`,
`returns`, `disputes`, `recurrence`, `webhooks`, `parameters`,
`corporate_orders`, `api_keys`, `clients`). Hoy coinciden —se verificaron uno a
uno—, pero nada lo garantiza.

**Por dónde entrarle:** la vía documentada (generar desde `routes/openapi.ts`)
**no es viable**: ese fichero es un esbozo con 11 rutas y sin esquemas. Las dos
opciones reales son (a) completar el OpenAPI y generar con `openapi-typescript`,
o (b) seguir moviendo esquemas a shared e importarlos desde el frontend, que es
lo que se viene haciendo y ya cubre todos los módulos donde aparecieron bugs.

---

## 4. La suite de tests falla de forma intermitente

**Sigue abierta.** El 2026-08-12 se investigó a fondo y **la hipótesis que
figuraba aquí es falsa**, igual que otras dos. Lo que sigue es sobre todo un
mapa de callejones sin salida, para que nadie los repita.

**Síntomas.** Falla ~1 de cada 3–8 corridas completas, en un fichero distinto
cada vez (`isolation`, `api_keys`, `admin_orders`, `ruta_admin_clients`,
`buyer_orders`, `corporate_orders`, `courier_collection`, `pickup_ops`…).
**Aislados pasan siempre.** Los errores no son de aserción de negocio sino de
infraestructura: un **404 en una ruta que existe**, un **`socket hang up`**, un
cuerpo de respuesta `undefined`. Eso apunta a la app o al proceso en mal
estado, no a la lógica.

**Descartado 1 — el paralelismo de vitest con `vi.mock('@orkoruta/db')`**, que
era la explicación que daba este documento. Vitest 2.1 usa `forks`: se
comprobó con dos ficheros sonda que ni `process.env` ni el registro de módulos
se comparten entre ficheros de test.

**Descartado 2 — recompilar `@orkoruta/shared` durante la suite.** Esto sí es
un problema real y **empeora la tasa** (falló 2 de 4 carreras provocadas), pero
no es la causa: sin ningún build cerca, la suite igual falla ~1 de 3. Como el
efecto existe y además es lo que rompe los dev servers de Next, se dejó el
guard `infra-ruta/scripts/build_shared.sh`, que aborta si detecta `vitest`,
`next dev` o `tsx watch` vivos.

**Descartado 3 — agotamiento de recursos por demasiados forks.** Con
`--minWorkers=1 --maxWorkers=4` la tasa **no mejora** (2/8 frente a 1/8 por
defecto; la diferencia es ruido). Serializar no arregla nada.

**Aviso de método:** `pnpm vitest run --maxWorkers=4` **no funciona** —pnpm se
come la bandera— y `--maxWorkers` a solas choca con `minWorkers` y aborta. En
los dos casos vitest imprime «no tests» y **sale con código 0**, así que un
bucle de comprobación da todo verde sin haber ejecutado nada. Usar
`pnpm exec vitest run --minWorkers=1 --maxWorkers=4` y verificar siempre el
recuento de tests.

**Por dónde entrarle ahora:** la pista buena es el `socket hang up` y el 404.
Conviene averiguar si los ficheros afectados levantan un servidor HTTP real y
si algo queda sin cerrar entre ficheros (handles abiertos, `server.listen` sin
`close`, pools de Prisma contra la BD remota). `vitest --reporter=verbose
--logHeapUsage` y `why-is-node-running` en un `afterAll` son los siguientes
pasos.

---
