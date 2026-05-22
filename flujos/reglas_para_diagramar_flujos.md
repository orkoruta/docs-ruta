===============================================================================
  FLUJO DE ESTADOS DE PEDIDO — DESCRIPCIÓN EN PSEUDOCÓDIGO
  Plataforma de Pedidos Online
  Versión: 10.0 
===============================================================================
  INSTRUCCIONES VISUALES PARA DIAGRAMAR EN FIGMA
===============================================================================

Los diagramas deben construirse en Figma como diagramas de flujo profesionales,
claros, editables y fáciles de mantener.

Cada diagrama debe crearse en un frame independiente, respetando la separación
definida en este documento:

  Diagrama 1 — Flujo Común + Decisión de Pago Online
  Diagrama 2 — Flujo SHIP completo
  Diagrama 3 — Flujo PICKUP completo
  Diagrama 4 — Flujo REFUND completo
  Diagrama 5 — Flujo RECURRENCIA
  Diagrama 6 — Flujo PEDIDOS CORPORATIVOS
  Diagrama 7 — Flujo DEVOLUCIONES post-cierre


===============================================================================
  ESTILO GENERAL EN FIGMA
===============================================================================

1. Usar tipografía en tamaño Large.
   - Texto principal dentro de cajas: Large.
   - Títulos de diagramas: más grandes y destacados.
   - Etiquetas de flechas: Large o ligeramente menor, pero siempre legible.

2. Todos los elementos deben quedar bien alineados usando Auto Layout,
   grillas o distribución uniforme.

3. Cada flecha debe ser claramente distinguible:
   - Debe verse exactamente de qué bloque sale.
   - Debe verse exactamente a qué bloque llega.
   - No deben existir flechas ambiguas.
   - Evitar cruces innecesarios entre flechas.
   - Cuando una flecha represente una condición, debe llevar etiqueta.
     Ejemplos:
       Sí
       No
       Pago aprobado
       Pago rechazado
       Si payment_status = PAID
       Si delivery_type = SHIP
       Si delivery_type = PICKUP

4. Las flechas deben usar conectores ortogonales o en ángulo recto,
   no líneas desordenadas o diagonales innecesarias.

5. Cada flecha debe terminar con punta visible.

6. Mantener separación suficiente entre bloques para que el diagrama
   sea entendible sin saturación visual.

7. Usar conectores entre diagramas cuando un flujo salte a otro diagrama.
   Ejemplo:
     → Continúa en Diagrama 4 — REFUND
     → Continúa en Diagrama 2 — SHIP
     → Continúa en Diagrama 7 — DEVOLUCIONES


===============================================================================
  CONVENCIONES DE DIAGRAMA DE FLUJO
===============================================================================

Usar las convenciones estándar de diagramas de flujo:

1. Inicio del flujo
   Forma: rectángulo con bordes redondeados.
   Color: Verde pastel.
   Uso:
     - Inicio de cada diagrama.
     - Entrada principal del flujo.

2. Fin del flujo
   Forma: rectángulo con bordes redondeados.
   Color: Azul celeste.
   Uso:
     - Estados finales.
     - CLOSED.
     - Fin de subprocesos.

3. Procesos o estados
   Forma: rectángulo.
   Uso:
     - Estados operativos.
     - Acciones del sistema.
     - Acciones del cliente, comprador, vendedor, repartidor o administrador.

4. Decisiones
   Forma: rombo.
   Color: Rojo pastel.
   Uso:
     - Bifurcaciones por condición.
     - Decisiones como:
       payment_method
       delivery_type
       delivery_carrier_type
       refund_modality
       return_mechanism
       buyer_type
       aprobación/rechazo
       éxito/fallo

5. Subprocesos
   Forma: rectángulo con doble borde o bloque visual destacado.
   Uso:
     - Referencias a otros diagramas.
     - REFUND.
     - DEVOLUCIONES.
     - RECURRENCIA.
     - ASIGNACIÓN A REPARTIDOR.

6. Documentos, comprobantes o notificaciones
   Forma: rectángulo con nota o ícono auxiliar.
   Uso:
     - Envío de comprobante.
     - Notificación al comprador.
     - Condiciones de devolución.
     - Comprobante de reembolso.


===============================================================================
  PALETA DE COLORES OBLIGATORIA
===============================================================================

1. Inicio
   Color: Verde pastel.
   Ejemplo recomendado:
     Fill: #C8E6C9
     Stroke: #2E7D32

2. Final
   Color: Azul celeste.
   Ejemplo recomendado:
     Fill: #B3E5FC
     Stroke: #0277BD

3. Decisiones
   Color: Rojo pastel.
   Ejemplo recomendado:
     Fill: #FFCDD2
     Stroke: #C62828

4. Procesos normales
   Color recomendado:
     Fill: #FFFFFF
     Stroke: #B0BEC5

5. Subprocesos o conectores a otros diagramas
   Color recomendado:
     Fill: #EDE7F6
     Stroke: #5E35B1

6. Advertencias, errores o excepciones
   Color recomendado:
     Fill: #FFF3E0
     Stroke: #EF6C00

7. Texto
   Color recomendado:
     #263238

8. Flechas
   Color recomendado:
     #455A64
   Grosor recomendado:
     2 px o 3 px


===============================================================================
  REGLAS DE LEGIBILIDAD
===============================================================================

1. No comprimir demasiado los diagramas.
   Es preferible usar frames grandes en Figma antes que hacer bloques pequeños.

2. Cada bloque debe tener un nombre corto y claro.
   Ejemplo:
     ORDER_SUBMITTED
     PAYMENT_PROCESSING
     VALIDATION_APPROVED
     READY_TO_SHIP
     CLOSED

3. Cuando sea necesario, agregar una segunda línea descriptiva debajo del estado.
   Ejemplo:
     PAYMENT_PROCESSING
     Pasarela procesando pago

4. Las decisiones deben escribirse como preguntas o condiciones.
   Ejemplo:
     ¿payment_method = ONLINE_AT_ORDER?
     ¿delivery_type = SHIP?
     ¿refund_modality = BANK_REFUND?
     ¿return_mechanism = CLIENT_PICKS_UP?

5. Las salidas de los rombos deben estar etiquetadas.
   Ejemplo:
     Sí
     No
     Aprobado
     Rechazado
     Reintentable
     Definitivo

6. No dejar caminos muertos.
   Todo camino debe terminar en:
     - CLOSED
     - un subproceso conectado
     - retorno a un estado anterior
     - gestión manual claramente indicada

7. Cuando exista un loop, debe mostrarse con flecha de retorno clara.
   Ejemplo:
     PAYMENT_FAILED_RETRYABLE → vuelve a PENDING_ONLINE_PAYMENT

8. Cuando un proceso dispare reembolso, marcarlo explícitamente:
     Si payment_status = PAID → inicia Diagrama 4 — REFUND

9. Cuando un proceso dispare devolución post-cierre, marcarlo explícitamente:
     Si closure_reason = COMPLETED_SUCCESSFULLY y hay queja
     → inicia Diagrama 7 — DEVOLUCIONES


===============================================================================
  PRINCIPIOS ARQUITECTÓNICOS
===============================================================================

  1. ESTADO FINAL ÚNICO
     Todos los caminos del pedido principal terminan en:
       → CLOSED
     El motivo del cierre se guarda en closure_reason.

  2. SEPARACIÓN DE DOMINIOS
     Para evitar ambigüedad, se separan estas dimensiones:

       order_status       — estado operativo del pedido
       payment_status     — estado financiero del pago
       refund_status      — estado financiero del reembolso
       return_status      — estado del retorno físico post-cierre
       dispute_status     — estado de disputa
       recurrence_status  — estado de la recurrencia
       delivery_type      — SHIP o PICKUP
       payment_method     — cuándo y cómo se cobra
       refund_modality    — cómo se devuelve el dinero
       return_mechanism   — cómo regresa físicamente el producto
       delivery_carrier_type — flota propia o courier externo
       buyer_type         — comprador individual o corporativo
       recurrence_mode    — programada o repetición manual
       closure_reason     — razón por la cual el pedido se cerró

  3. ESTADOS ATÓMICOS
     Cada estado representa una sola condición clara del pedido.

  4. PAGO ONLINE ANTES DE PREPARAR
     Si payment_method = ONLINE_AT_ORDER, el pago debe aprobarse antes
     de avanzar a validación operativa.

  5. PAGO EN ENTREGA / RETIRO
     Si payment_method = ELECTRONIC_ON_DELIVERY o CASH_ON_DELIVERY,
     el cobro ocurre durante el handoff (SHIP o PICKUP).

  6. NO MARCAR ENTREGADO SIN COBRO RESUELTO
     Para pagos al momento de entrega/retiro, no se pasa a DELIVERED
     si el cobro falla.

  7. REEMBOLSO COMO SUBPROCESO FINANCIERO
     El reembolso no reemplaza el estado operativo. Se maneja en
     refund_status.

  8. DEVOLUCIÓN COMO SUBPROCESO FÍSICO
     La devolución del producto no reabre el pedido principal. Se
     maneja en return_status y dispara reembolso en paralelo.

  9. RECURRENCIA COMO SUBPROCESO INDEPENDIENTE
     La recurrencia no es un estado del pedido. Es una plantilla
     activa que genera nuevos pedidos. Cada pedido generado tiene
     su propio ciclo de vida independiente.

  10. ARCHIVO POSTERIOR AL CIERRE
     ARCHIVED no reemplaza a CLOSED. Se maneja como atributo:
       archived = true
       archived_at = fecha


===============================================================================
  BLOQUE 0 — ATRIBUTOS BASE PARA DIAGRAMAR
===============================================================================

  delivery_type:
    SHIP    — entrega a domicilio / despacho
    PICKUP  — recogida en tienda, bodega o punto físico

  delivery_carrier_type:
    OWN_FLEET         — el cliente usa repartidores de su propia lista
    EXTERNAL_COURIER  — el cliente usa un proveedor externo
                        (servientrega, coordinadora, etc.)

  payment_method:
    ONLINE_AT_ORDER
      Pago online realizado durante la creación del pedido.
      Canalizado por pasarela de pagos (Wompi, etc.).
      Debe aprobarse antes de preparar.

    ELECTRONIC_ON_DELIVERY
      Pago electrónico al momento de entrega/retiro.
      Submétodos posibles: datáfono, QR, link de pago, transferencia
      bancaria inmediata. Aplica para SHIP y PICKUP.

    CASH_ON_DELIVERY
      Pago en efectivo al momento de entrega/retiro.
      Aplica para SHIP y PICKUP.

  payment_method_submethod (solo cuando payment_method = ELECTRONIC_ON_DELIVERY):
    DATAFONO            — datáfono físico de un proveedor (ej. Gold)
    BANK_TRANSFER       — transferencia bancaria directa al cliente
    PAYMENT_LINK        — link de pasarela

  buyer_type:
    INDIVIDUAL  — comprador estándar que usa la página
    CORPORATE   — comprador empresarial gestionado manualmente
                  por el cliente

  payment_status:
    PAYMENT_NOT_STARTED
    PENDING_ONLINE_PAYMENT
    PAYMENT_PROCESSING
    PAID
    PAYMENT_FAILED_RETRYABLE
    PAYMENT_REJECTED_FINAL
    PENDING_COLLECTION
    COLLECTION_PROCESSING
    PAYMENT_COLLECTED
    PAYMENT_COLLECTION_FAILED
    PAYMENT_NOT_COLLECTED

  refund_modality:
    STORE_CREDIT  — se acredita el monto como crédito interno del
                    comercio del cliente para futuros pedidos
    BANK_REFUND   — se devuelve el dinero a la cuenta bancaria
                    del comprador

  refund_status:
    REFUND_NOT_REQUIRED
    REFUND_PENDING
    REFUND_PROCESSING
    REFUND_PROVIDER_REQUESTED
    REFUNDED
    PARTIALLY_REFUNDED
    REFUND_FAILED

  return_mechanism:
    BUYER_SHIPS_VIA_COURIER
      El comprador envía el producto usando un proveedor externo
      (servientrega, coordinadora, etc.).

    CLIENT_PICKS_UP
      El cliente envía un repartidor propio a recoger el pedido
      devuelto en la dirección del comprador.

  return_status:
    RETURN_REQUESTED
    RETURN_UNDER_REVIEW
    RETURN_APPROVED
    RETURN_REJECTED
    RETURN_CANCELLED
    CUSTOMER_RETURN_IN_TRANSIT
    CUSTOMER_RETURN_EXPIRED
    CUSTOMER_RETURN_RECEIVED
    CUSTOMER_RETURN_LOST
    PICKUP_SCHEDULED
    PICKUP_OUT_FOR_COLLECTION
    PICKUP_COLLECTED
    PICKUP_FAILED

  recurrence_mode:
    SCHEDULED_RECURRING  — el sistema genera pedidos automáticamente
                           con una periodicidad configurada
    REPEAT_LAST_ORDER    — el comprador solicita repetir manualmente
                           el último pedido

  recurrence_periodicity (solo para SCHEDULED_RECURRING):
    DAILY
    WEEKLY
    BIWEEKLY
    MONTHLY
    CUSTOM_INTERVAL

  recurrence_status:
    RECURRENCE_ACTIVE
    RECURRENCE_PAUSED
    RECURRENCE_CANCELLED


===============================================================================
  BLOQUE 1 — FLUJO COMÚN: CREACIÓN DEL PEDIDO
===============================================================================

  El pedido puede nacer desde 3 orígenes:

    a) Comprador individual desde la página
       → buyer_type = INDIVIDUAL
       → entra directo en DRAFT

    b) Comprador corporativo gestionado por el cliente
       → buyer_type = CORPORATE
       → entra desde BLOQUE 18

    c) Pedido generado automáticamente por recurrencia programada
       → buyer_type heredado de la plantilla
       → entra desde BLOQUE 17-A, directamente en ORDER_SUBMITTED

  1. Cliente/Comprador inicia el pedido
     → order_status = DRAFT

  Desde DRAFT puede:

     a) Confirmar el pedido
        → order_status = PENDING_CONFIRM

     b) No hacer nada hasta vencer el tiempo máximo
        → order_status = EXPIRED
        → order_status = CLOSED
        → closure_reason = EXPIRED

     c) Cancelar
        → order_status = CANCELLED_BY_CUSTOMER
        → order_status = CLOSED
        → closure_reason = CANCELLED_BY_CUSTOMER


===============================================================================
  BLOQUE 2 — FLUJO COMÚN: SELECCIÓN DE ENTREGA Y PAGO
===============================================================================

  2. Pedido pendiente de confirmación
     → order_status = PENDING_CONFIRM

  En este punto se define:

     delivery_type:
       - SHIP
       - PICKUP

     payment_method:
       - ONLINE_AT_ORDER
       - ELECTRONIC_ON_DELIVERY
       - CASH_ON_DELIVERY

     (opcional) marcar como recurrente
       → continúa en BLOQUE 17-A para configurar periodicidad

  Desde PENDING_CONFIRM puede:

     a) Confirmar selección de entrega y pago
        → order_status = ORDER_SUBMITTED

     b) Cancelar antes de enviar el pedido
        → order_status = CANCELLED_BY_CUSTOMER
        → order_status = CLOSED
        → closure_reason = CANCELLED_BY_CUSTOMER


===============================================================================
  BLOQUE 3 — RUTA DE PAGO ONLINE AL MOMENTO DEL PEDIDO
  Aplica cuando payment_method = ONLINE_AT_ORDER
===============================================================================

  3-1. Pedido enviado con pago online inicial
       → order_status = ORDER_SUBMITTED
       → payment_status = PENDING_ONLINE_PAYMENT

  Desde aquí puede:

     a) Comprador inicia el pago
        → payment_status = PAYMENT_PROCESSING

     b) Comprador no paga dentro del tiempo permitido
        → order_status = EXPIRED → CLOSED
        → closure_reason = PAYMENT_TIMEOUT
        → refund_status = REFUND_NOT_REQUIRED

     c) Comprador cancela antes de pagar
        → order_status = CANCELLED_BY_CUSTOMER → CLOSED
        → closure_reason = CANCELLED_BY_CUSTOMER
        → refund_status = REFUND_NOT_REQUIRED

  3-2. Pasarela procesando pago
       → payment_status = PAYMENT_PROCESSING

  Desde aquí puede:

     a) Pago aprobado
        → payment_status = PAID
        → continúa en BLOQUE 5

     b) Pago fallido pero reintentable
        → payment_status = PAYMENT_FAILED_RETRYABLE
        → vuelve a PENDING_ONLINE_PAYMENT

     c) Pago fallido y se agotan los intentos
        → payment_status = PAYMENT_REJECTED_FINAL
        → order_status = CLOSED
        → closure_reason = PAYMENT_REJECTED
        → refund_status = REFUND_NOT_REQUIRED

     d) Pago rechazado definitivamente
        → payment_status = PAYMENT_REJECTED_FINAL
        → order_status = CLOSED
        → closure_reason = PAYMENT_REJECTED
        → refund_status = REFUND_NOT_REQUIRED


===============================================================================
  BLOQUE 4 — RUTA DE PAGO AL MOMENTO DE ENTREGA / RETIRO
  Aplica cuando payment_method = ELECTRONIC_ON_DELIVERY o CASH_ON_DELIVERY
===============================================================================

  4-1. Pedido enviado con pago pendiente de cobro posterior
       → order_status = ORDER_SUBMITTED
       → payment_status = PENDING_COLLECTION

  El pedido continúa directamente a validación operativa.
       → continúa en BLOQUE 5

  El cobro real se ejecuta más adelante:
       - En SHIP: cuando el repartidor llega al comprador.
       - En PICKUP: cuando el comprador llega al punto de recogida.


===============================================================================
  BLOQUE 5 — FLUJO COMÚN: VALIDACIÓN OPERATIVA
===============================================================================

  5-1. Sistema valida datos, stock, fraude operativo y reglas
       → order_status = ORDER_VALIDATING

  Desde ORDER_VALIDATING puede:

     a) Validación automática aprobada
        → order_status = VALIDATION_APPROVED

     b) Requiere revisión humana
        → order_status = MANUAL_REVIEW

     c) Rechazo automático
        → order_status = VALIDATION_REJECTED
        → order_status = CANCELLED_BY_SYSTEM → CLOSED
        → closure_reason = VALIDATION_REJECTED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING
          → continúa en BLOQUE 12

  5-2. Revisión humana
       → order_status = MANUAL_REVIEW

  Desde MANUAL_REVIEW puede:

     a) Administrador aprueba
        → order_status = VALIDATION_APPROVED

     b) Administrador rechaza
        → order_status = VALIDATION_REJECTED
        → order_status = CANCELLED_BY_ADMIN → CLOSED
        → closure_reason = VALIDATION_REJECTED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING
          → continúa en BLOQUE 12


===============================================================================
  BLOQUE 6 — FLUJO COMÚN: ACEPTACIÓN DEL VENDEDOR Y PREPARACIÓN
===============================================================================

  6-1. Pedido validado operativamente
       → order_status = VALIDATION_APPROVED

  Desde VALIDATION_APPROVED puede:

     a) Vendedor acepta el pedido
        → order_status = SELLER_CONFIRMED

     b) Vendedor rechaza
        → order_status = CANCELLED_BY_SELLER → CLOSED
        → closure_reason = CANCELLED_BY_SELLER
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING
          → continúa en BLOQUE 12

  6-2. Pedido aceptado por vendedor
       → order_status = SELLER_CONFIRMED
       → order_status = PREPARING

  6-3. Pedido en preparación
       → order_status = PREPARING

  Desde PREPARING puede:

     a) Pedido preparado para entrega a domicilio
        Condición: delivery_type = SHIP
        Si delivery_carrier_type = OWN_FLEET:
          → continúa en BLOQUE 6B (asignación a repartidor)
        Si delivery_carrier_type = EXTERNAL_COURIER:
          → order_status = READY_TO_SHIP
          → continúa en BLOQUE 7

     b) Pedido preparado para retiro en punto físico
        Condición: delivery_type = PICKUP
        → order_status = READY_FOR_PICKUP
        → continúa en BLOQUE 8

     c) Comprador cancela antes de despacho/retiro
        → order_status = CANCELLED_BY_CUSTOMER → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING
          → continúa en BLOQUE 12

     d) Administrador cancela antes de despacho/retiro
        → order_status = CANCELLED_BY_ADMIN → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING
          → continúa en BLOQUE 12


===============================================================================
  BLOQUE 6B — ASIGNACIÓN DE PEDIDO A REPARTIDOR (NUEVO EN v10.0)
  Aplica al flujo SHIP cuando delivery_carrier_type = OWN_FLEET
===============================================================================

  Cuando el cliente opera con flota propia, los pedidos listos para
  despacho aparecen geolocalizados en un mapa interactivo y el cliente
  los asigna manualmente a sus repartidores.

  6B-1. Pedido listo para asignación
        → order_status = AWAITING_COURIER_ASSIGNMENT

        Comportamiento del sistema:
          - El pedido aparece en un mapa basado en OpenStreetMap
            (u otro proveedor de mapas) en la ubicación de la
            dirección del comprador.
          - El cliente ve simultáneamente todos los pedidos
            pendientes de asignar.
          - El cliente cuenta con una lista de sus repartidores
            disponibles.

  6B-2. Cliente selecciona un pedido en el mapa y asigna un repartidor
        → order_status = COURIER_ASSIGNED
        → courier_id = (id del repartidor escogido)

  Desde COURIER_ASSIGNED puede:

     a) Repartidor acepta y recoge el paquete
        → order_status = READY_TO_SHIP (interno)
        → order_status = SHIPPED
        → continúa en BLOQUE 7

     b) Repartidor rechaza el pedido
        → vuelve a AWAITING_COURIER_ASSIGNMENT
        → cliente debe asignar otro repartidor

     c) Cliente desasigna al repartidor antes de que recoja
        → vuelve a AWAITING_COURIER_ASSIGNMENT

     d) Ningún repartidor disponible dentro del plazo
        → order_status = SHIPMENT_HOLD
        → si se resuelve: vuelve a AWAITING_COURIER_ASSIGNMENT
        → si no se resuelve:
            → order_status = CANCELLED_BY_ADMIN → CLOSED
            Si payment_status = PAID:
              → refund_status = REFUND_PENDING
              → continúa en BLOQUE 12

  NOTA PARA DIAGRAMA:
    Cuando delivery_carrier_type = EXTERNAL_COURIER, este bloque no
    aplica. El paquete se entrega directamente al courier externo y
    el pedido pasa de PREPARING → READY_TO_SHIP → SHIPPED sin paso
    de asignación manual.


===============================================================================
  BLOQUE 7 — FLUJO SHIP: ENTREGA A DOMICILIO / DESPACHO
===============================================================================

  7-1. Pedido listo para despacho
       → order_status = READY_TO_SHIP

  Desde READY_TO_SHIP puede:

     a) Transportista recoge o recibe el paquete
        → order_status = SHIPPED

     b) Comprador cancela antes de entregar al transportista
        → order_status = CANCELLED_BY_CUSTOMER → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12

     c) Problema operativo antes del despacho
        → order_status = SHIPMENT_HOLD
        → si se resuelve: vuelve a READY_TO_SHIP
        → si no: CANCELLED_BY_ADMIN → CLOSED

  7-2. Pedido entregado al transportista
       → order_status = SHIPPED

  Desde SHIPPED puede:

     a) Inicia transporte
        → order_status = IN_TRANSIT

     b) Comprador solicita cancelación post-despacho
        → order_status = CUSTOMER_CANCEL_REQUEST
        → continúa en BLOQUE 11

  7-3. Pedido en tránsito
       → order_status = IN_TRANSIT

  Desde IN_TRANSIT puede:

     a) Sale a reparto final
        → order_status = OUT_FOR_DELIVERY

     b) Queda retenido temporalmente
        → order_status = ON_HOLD
        → si se libera: vuelve a IN_TRANSIT
        → si no: CANCELLED_BY_ADMIN → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12

     c) Se declara perdido en tránsito
        → order_status = LOST_IN_TRANSIT → CLOSED
        → closure_reason = LOST_IN_TRANSIT
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12
        Si payment_status = PENDING_COLLECTION:
          → payment_status = PAYMENT_NOT_COLLECTED

     d) Comprador solicita cancelación post-despacho
        → order_status = CUSTOMER_CANCEL_REQUEST → BLOQUE 11

  7-4. Pedido en reparto final
       → order_status = OUT_FOR_DELIVERY

  Desde OUT_FOR_DELIVERY puede:

     a) Repartidor llega al comprador
        → order_status = ARRIVED_AT_CUSTOMER
        → continúa en BLOQUE 9

     b) Comprador ausente, dirección incorrecta o imposibilidad
        → order_status = DELIVERY_ATTEMPTED
        Desde DELIVERY_ATTEMPTED:
          b1) Reprogramar entrega
              → DELIVERY_RESCHEDULED → OUT_FOR_DELIVERY
          b2) Redirigir a punto físico autorizado
              → delivery_type = PICKUP
              → AT_PICKUP_POINT → BLOQUE 8
          b3) Declarar entrega fallida definitiva
              → RETURN_TO_ORIGIN → BLOQUE 10

     c) Comprador solicita cancelación post-despacho
        → CUSTOMER_CANCEL_REQUEST → BLOQUE 11


===============================================================================
  BLOQUE 8 — FLUJO PICKUP: RECOGIDA EN TIENDA O PUNTO FÍSICO
===============================================================================

  8-1. Pedido listo para recogida
       → order_status = READY_FOR_PICKUP

  Desde READY_FOR_PICKUP puede:

     a) Pedido queda disponible en punto físico
        → order_status = AT_PICKUP_POINT

     b) Comprador cancela antes de que quede disponible
        → CANCELLED_BY_CUSTOMER → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12

     c) Problema operativo antes de publicar disponibilidad
        → PICKUP_POINT_ISSUE
        → si se resuelve: vuelve a READY_FOR_PICKUP
        → si no: CANCELLED_BY_ADMIN → CLOSED

  8-2. Pedido disponible en punto físico
       → order_status = AT_PICKUP_POINT

  Desde AT_PICKUP_POINT puede:

     a) Comprador llega a recoger
        → CUSTOMER_ARRIVED_AT_PICKUP_POINT

     b) Comprador no recoge dentro del plazo
        → PICKUP_EXPIRED → CLOSED
        → closure_reason = PICKUP_EXPIRED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12

     c) Comprador cancela antes de recoger
        → CANCELLED_BY_CUSTOMER → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12

     d) Problema operativo en el punto físico
        → PICKUP_POINT_ISSUE
        → si se resuelve: vuelve a AT_PICKUP_POINT
        → si no: CANCELLED_BY_ADMIN → CLOSED

  8-3. Comprador llegó al punto de recogida
       → order_status = CUSTOMER_ARRIVED_AT_PICKUP_POINT

  Desde aquí puede:

     a) Validar identidad
        → IDENTITY_VALIDATED → BLOQUE 9

     b) Identidad no válida
        → PICKUP_AUTH_FAILED → vuelve a AT_PICKUP_POINT

     c) Comprador decide no retirar
        → PICKUP_CANCELLED_BY_CUSTOMER → CLOSED
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12


===============================================================================
  BLOQUE 9 — HANDOFF Y COBRO COMÚN
===============================================================================

  Punto común para SHIP y PICKUP, donde el comprador ya está frente al pedido.
  El comportamiento depende de payment_method.

  ─────────────────────────────────────────────────────
  CASO 9A — payment_method = ONLINE_AT_ORDER
  ─────────────────────────────────────────────────────

    Condición de entrada: payment_status = PAID

    SHIP:
      ARRIVED_AT_CUSTOMER → DELIVERED → BLOQUE 13

    PICKUP:
      IDENTITY_VALIDATED → PICKED_UP → DELIVERED → BLOQUE 13


  ─────────────────────────────────────────────────────
  CASO 9B — payment_method = ELECTRONIC_ON_DELIVERY
  ─────────────────────────────────────────────────────

    Condición de entrada: payment_status = PENDING_COLLECTION

    Se inicia cobro electrónico:
      → payment_status = COLLECTION_PROCESSING
      → order_status = PAYMENT_COLLECTION_PENDING

    Desde PAYMENT_COLLECTION_PENDING puede:

      a) Cobro exitoso
         → payment_status = PAYMENT_COLLECTED
         → order_status = PAYMENT_COLLECTED_ELECTRONIC
         SHIP: → DELIVERED → BLOQUE 13
         PICKUP: → PICKED_UP → DELIVERED → BLOQUE 13

      b) Fallo reintentable
         → payment_status = PAYMENT_COLLECTION_FAILED
         → loop: vuelve a PAYMENT_COLLECTION_PENDING

      c) Fallo definitivo
         → payment_status = PAYMENT_NOT_COLLECTED
         SHIP: → RETURN_TO_ORIGIN → BLOQUE 10
         PICKUP: → CANCELLED_NO_PAYMENT → CLOSED


  ─────────────────────────────────────────────────────
  CASO 9C — payment_method = CASH_ON_DELIVERY
  ─────────────────────────────────────────────────────

    Condición de entrada: payment_status = PENDING_COLLECTION

    → order_status = CASH_COLLECTION_PENDING

    Desde CASH_COLLECTION_PENDING puede:

      a) Comprador paga correctamente
         → payment_status = PAYMENT_COLLECTED
         → order_status = PAYMENT_COLLECTED_CASH
         SHIP: → DELIVERED → BLOQUE 13
         PICKUP: → PICKED_UP → DELIVERED → BLOQUE 13

      b) Comprador no paga
         → payment_status = PAYMENT_NOT_COLLECTED
         → order_status = CASH_PAYMENT_REJECTED
         SHIP: → RETURN_TO_ORIGIN → BLOQUE 10
         PICKUP: → CANCELLED_NO_PAYMENT → CLOSED


===============================================================================
  BLOQUE 10 — DEVOLUCIÓN AL ORIGEN (por logística, no por queja)
===============================================================================

  10-1. Paquete debe regresar al origen
        → order_status = RETURN_TO_ORIGIN

  Motivos típicos:
    - Entrega fallida definitiva
    - Comprador no pagó al momento de entrega
    - Cobro electrónico falló definitivamente
    - Cancelación post-despacho aprobada
    - Redirección logística fallida

  Desde RETURN_TO_ORIGIN puede:

     a) Paquete llega a bodega/origen
        → order_status = RETURN_TO_ORIGIN_RECEIVED

     b) Paquete se pierde en retorno
        → LOST_IN_RETURN → CLOSED
        → closure_reason = LOST_IN_RETURN
        Si payment_status = PAID:
          → refund_status = REFUND_PENDING → BLOQUE 12

  10-2. Paquete recibido en origen
        → order_status = RETURN_TO_ORIGIN_RECEIVED

  Desde RETURN_TO_ORIGIN_RECEIVED se cierra según la causa:

     a) Entrega fallida definitiva
        → CLOSED → closure_reason = DELIVERY_FAILED
        Si PAID: → REFUND_PENDING → BLOQUE 12

     b) Comprador no pagó al momento de entrega
        → CANCELLED_NO_PAYMENT → CLOSED
        → refund_status = REFUND_NOT_REQUIRED

     c) Cancelación post-despacho aprobada
        → CANCELLED_BY_CUSTOMER → CLOSED
        Si PAID: → REFUND_PENDING → BLOQUE 12

     d) Administrador decide reenviar
        → READY_TO_SHIP → BLOQUE 7


===============================================================================
  BLOQUE 11 — CANCELACIÓN POST-DESPACHO
===============================================================================

  11-1. Comprador solicita cancelar después de despachado
        → order_status = CUSTOMER_CANCEL_REQUEST

  Desde aquí puede:

     a) Solicitud aprobada
        → CANCEL_REQUEST_APPROVED → RETURN_TO_ORIGIN → BLOQUE 10

     b) Solicitud rechazada
        → CANCEL_REQUEST_REJECTED
        → vuelve al estado previo (IN_TRANSIT o OUT_FOR_DELIVERY)

     c) Administrador fuerza cancelación
        → CANCELLED_BY_ADMIN → CLOSED
        Si PAID: → REFUND_PENDING → BLOQUE 12


===============================================================================
  BLOQUE 12 — REEMBOLSO (REDISEÑADO EN v10.0)
  Subproceso financiero; no reemplaza el estado principal del pedido
===============================================================================

  El reembolso se ejecuta según dos atributos clave:

    refund_modality (definido al contratar los servicios de ruta):
      STORE_CREDIT  — se devuelve el monto como crédito interno
      BANK_REFUND   — se devuelve el dinero a la cuenta bancaria
                      del comprador

    payment_method (definido al crear el pedido):
      ONLINE_AT_ORDER         (pasarela de pagos)
      ELECTRONIC_ON_DELIVERY  (datáfono, transferencia, QR, link)
      CASH_ON_DELIVERY        (efectivo contra entrega)

  Este bloque se activa cuando:
    - El pedido estaba pagado y fue cancelado
    - El pedido estaba pagado y falló la entrega definitivamente
    - El pedido estaba pagado y se perdió en tránsito o en retorno
    - El pedido estaba pagado y venció el plazo de pickup
    - Una devolución post-cierre fue aprobada y el producto recibido

  12-1. Reembolso pendiente
        → refund_status = REFUND_PENDING

  12-2. Bifurcación por modalidad de reembolso

        Si refund_modality = STORE_CREDIT  → continúa en 12-A
        Si refund_modality = BANK_REFUND   → continúa en 12-B


  ─────────────────────────────────────────────────────
  12-A. MODALIDAD STORE_CREDIT
  ─────────────────────────────────────────────────────

  No es necesario conocer el método de pago original.
  El monto pagado se acredita como crédito interno del comprador,
  utilizable en futuros pedidos dentro del comercio del cliente.

  12-A-1. Sistema procesa el crédito
          → refund_status = REFUND_PROCESSING

  Desde REFUND_PROCESSING (STORE_CREDIT) puede:

     a) Crédito acreditado por el monto total
        → refund_status = REFUNDED
        → notificación al comprador

     b) Crédito acreditado por monto parcial
        → refund_status = PARTIALLY_REFUNDED
        → notificación al comprador

     c) Fallo técnico al acreditar
        → refund_status = REFUND_FAILED
        → requiere gestión manual


  ─────────────────────────────────────────────────────
  12-B. MODALIDAD BANK_REFUND
  ─────────────────────────────────────────────────────

  Es indispensable identificar el método de pago original porque
  define el canal de devolución y quién la ejecuta.

  12-B-1. Sistema inicia procesamiento
          → refund_status = REFUND_PROCESSING

  12-B-2. Bifurcación por payment_method:


    ─────────────────────────────────────────────────
    Caso 12-B-1 — Reembolso interno por parte del cliente
    Aplica cuando:
      payment_method = CASH_ON_DELIVERY
      o
      payment_method = ELECTRONIC_ON_DELIVERY
        con submétodo = BANK_TRANSFER
    ─────────────────────────────────────────────────

    El cliente (comercio) ejecuta el reembolso de forma interna:
      - Realiza la transferencia bancaria al comprador desde
        sus cuentas propias.
      - Envía comprobante al comprador.
      - Marca el pago del pedido como reembolsado.

    Desde aquí puede:

       a) Transferencia ejecutada y comprobante enviado
          → refund_status = REFUNDED
          o
          → refund_status = PARTIALLY_REFUNDED (si fue parcial)

       b) Fallo en la ejecución interna (datos bancarios inválidos,
          rechazo bancario, etc.)
          → refund_status = REFUND_FAILED
          → requiere gestión manual


    ─────────────────────────────────────────────────
    Caso 12-B-2 — Reembolso vía proveedor de pagos
    Aplica cuando:
      payment_method = ONLINE_AT_ORDER (pasarela)
      o
      payment_method = ELECTRONIC_ON_DELIVERY
        con submétodo = DATAFONO
      o
      payment_method = ELECTRONIC_ON_DELIVERY
        con submétodo = QR_OR_PAYMENT_LINK
    ─────────────────────────────────────────────────

    El cliente solicita el reembolso al proveedor correspondiente:
      - Pasarela de pagos → ej. Wompi
      - Datáfono          → proveedor del datáfono (ej. Gold)
      - QR/Link           → proveedor de la pasarela asociada

    Flujo:
      → refund_status = REFUND_PROVIDER_REQUESTED

    Desde REFUND_PROVIDER_REQUESTED puede:

       a) Proveedor confirma reembolso total
          → cliente envía comprobante al comprador
          → cliente marca el pago del pedido como reembolsado
          → refund_status = REFUNDED

       b) Proveedor confirma reembolso parcial
          → cliente envía comprobante al comprador
          → cliente marca el pago del pedido como reembolsado
          → refund_status = PARTIALLY_REFUNDED

       c) Proveedor rechaza la solicitud o falla técnicamente
          → refund_status = REFUND_FAILED
          → requiere gestión manual

  NOTAS PARA DIAGRAMA:
    - El pedido puede estar ya en CLOSED mientras refund_status avanza.
    - El comprobante de reembolso se envía SIEMPRE al comprador.
    - En STORE_CREDIT no importa el payment_method original.
    - En BANK_REFUND el payment_method original es decisivo para
      elegir entre reembolso interno o vía proveedor.


===============================================================================
  BLOQUE 13 — FINALIZACIÓN EXITOSA
===============================================================================

  13-1. Pedido entregado físicamente y pago resuelto
        → order_status = DELIVERED

  Desde DELIVERED puede:

     a) Comprador confirma recepción
        → CONFIRMED_BY_CUSTOMER

     b) Comprador no confirma dentro del plazo
        → sistema confirma automáticamente → CONFIRMED_BY_SYSTEM

     c) Comprador abre disputa
        → DELIVERY_DISPUTED → BLOQUE 15

  13-2. Recepción confirmada
        → CONFIRMED_BY_CUSTOMER o CONFIRMED_BY_SYSTEM
        → COMPLETED_SUCCESSFULLY → CLOSED
        → closure_reason = COMPLETED_SUCCESSFULLY


===============================================================================
  BLOQUE 14 — DEVOLUCIÓN POST-CIERRE (REDISEÑADO EN v10.0)
  Subproceso posterior a CLOSED; no reabre el pedido principal
===============================================================================

  El mecanismo de devolución se define al contratar los servicios
  de ruta:

    return_mechanism:
      BUYER_SHIPS_VIA_COURIER
        El comprador envía el producto usando un proveedor externo
        (servientrega, coordinadora, etc.). El cliente le indica los
        proveedores autorizados y el plazo máximo.

      CLIENT_PICKS_UP
        El cliente envía un repartidor propio a recoger el pedido
        devuelto en la dirección del comprador.

  Condición de entrada:
    order_status = CLOSED
    closure_reason = COMPLETED_SUCCESSFULLY

  14-1. Comprador pone queja para devolver el pedido
        → return_status = RETURN_REQUESTED

  14-2. Cliente tiene un tiempo definido de aprobación
        → return_status = RETURN_UNDER_REVIEW

  Desde RETURN_UNDER_REVIEW puede:

     a) Cliente aprueba dentro del plazo
        → return_status = RETURN_APPROVED
        → continúa en 14-3

     b) Cliente rechaza
        → return_status = RETURN_REJECTED
        → fin del subproceso (order_status permanece CLOSED)

     c) Cliente no responde dentro del plazo (según política):
        → auto-aprobar  → RETURN_APPROVED
        → auto-rechazar → RETURN_REJECTED

  14-3. Devolución aprobada
        → return_status = RETURN_APPROVED
        → refund_status = REFUND_PENDING  (dispara BLOQUE 12 en paralelo)
        → cliente envía al comprador las condiciones según
          return_mechanism

  14-4. Bifurcación por return_mechanism:


    ─────────────────────────────────────────────────
    Caso 14-A — return_mechanism = BUYER_SHIPS_VIA_COURIER
    ─────────────────────────────────────────────────

    14-A-1. Cliente envía al comprador:
              - lista de proveedores autorizados
                (servientrega, coordinadora, etc.)
              - tiempo máximo de espera para recibir el producto

    14-A-2. Comprador envía el producto por courier
            → return_status = CUSTOMER_RETURN_IN_TRANSIT

    Desde CUSTOMER_RETURN_IN_TRANSIT puede:

       a) Producto llega a bodega del cliente
          → return_status = CUSTOMER_RETURN_RECEIVED
          → el reembolso (ya en REFUND_PENDING) continúa en BLOQUE 12

       b) Producto no llega dentro del plazo máximo
          → return_status = CUSTOMER_RETURN_EXPIRED
          → según política del cliente:
              - cancelar reembolso → refund_status = REFUND_NOT_REQUIRED
              - aprobar igualmente → continúa en BLOQUE 12

       c) Producto se pierde en tránsito
          → return_status = CUSTOMER_RETURN_LOST
          → gestión manual (decisión caso a caso sobre el reembolso)


    ─────────────────────────────────────────────────
    Caso 14-B — return_mechanism = CLIENT_PICKS_UP
    ─────────────────────────────────────────────────

    14-B-1. Cliente programa la recogida y asigna repartidor propio
            (mismo mecanismo que BLOQUE 6B)
            → return_status = PICKUP_SCHEDULED
            → courier_id = (id del repartidor asignado)

    14-B-2. Repartidor sale a recoger
            → return_status = PICKUP_OUT_FOR_COLLECTION

    Desde PICKUP_OUT_FOR_COLLECTION puede:

       a) Repartidor recoge el producto exitosamente
          → return_status = PICKUP_COLLECTED

          Al llegar a bodega:
          → return_status = CUSTOMER_RETURN_RECEIVED
          → el reembolso (ya en REFUND_PENDING) continúa en BLOQUE 12

       b) Comprador no está o se niega a entregar el producto
          → return_status = PICKUP_FAILED
          → opciones:
              - reprogramar  → vuelve a PICKUP_SCHEDULED
              - cancelar devolución
                → return_status = RETURN_CANCELLED
                → refund_status = REFUND_NOT_REQUIRED

       c) Repartidor pierde el producto en el retorno
          → return_status = CUSTOMER_RETURN_LOST
          → gestión manual

  NOTAS PARA DIAGRAMA:
    - El reembolso (BLOQUE 12) se dispara al aprobarse la devolución,
      no al recibirse el producto. Esto permite paralelizar.
    - Sin embargo, el reembolso real (REFUND_PROCESSING) suele esperar
      a la recepción del producto. La política exacta depende del cliente.


===============================================================================
  BLOQUE 15 — DISPUTAS
===============================================================================

  15-1. Comprador abre disputa
        → dispute_status = DISPUTED

  Desde DISPUTED puede:

     a) Administrador resuelve sin acción adicional
        → DISPUTE_RESOLVED

     b) Administrador aprueba devolución
        → DISPUTE_RESOLVED
        → return_status = RETURN_REQUESTED → BLOQUE 14

     c) Administrador aprueba reembolso sin devolución física
        → DISPUTE_RESOLVED
        → refund_status = REFUND_PENDING → BLOQUE 12


===============================================================================
  BLOQUE 16 — CIERRE Y ARCHIVO
===============================================================================

  16-1. Estado final único del pedido principal
        → order_status = CLOSED

  16-2. Archivo posterior por antigüedad
        archived = true
        archived_at = fecha_archivo


===============================================================================
  BLOQUE 17 — RECURRENCIA DE PEDIDOS (NUEVO EN v10.0)
  Subproceso que genera pedidos automáticamente o permite repetir uno previo
===============================================================================

  Existen dos modalidades, ambas iniciadas por el comprador:

    recurrence_mode:
      SCHEDULED_RECURRING
        El comprador, al hacer un pedido nuevo, lo marca como
        recurrente. Configura la periodicidad y el sistema
        recreará el pedido automáticamente con las mismas
        condiciones de pago y entrega.

      REPEAT_LAST_ORDER
        El comprador ingresa a la página y solicita repetir el
        último pedido. Puede aceptar tal cual o editarlo antes
        de confirmarlo.

  ─────────────────────────────────────────────────────
  17-A. MODALIDAD SCHEDULED_RECURRING
  ─────────────────────────────────────────────────────

  17-A-1. Comprador hace un pedido por primera vez
          → entra al flujo común desde BLOQUE 1 (DRAFT)

  17-A-2. En PENDING_CONFIRM marca el pedido como recurrente
          → recurrence_mode = SCHEDULED_RECURRING
          → recurrence_periodicity = (DAILY | WEEKLY | BIWEEKLY |
                                       MONTHLY | CUSTOM_INTERVAL)
          → recurrence_status = RECURRENCE_ACTIVE

          Se guarda la plantilla del pedido:
            - ítems y cantidades
            - dirección de entrega
            - payment_method (y submétodo si aplica)
            - delivery_type
            - condiciones acordadas

  17-A-3. Pedido original sigue el flujo común normal
          → BLOQUE 2 en adelante

  17-A-4. Cuando se cumple la periodicidad
          → el sistema crea automáticamente un pedido nuevo
            usando la plantilla guardada
          → nuevo pedido nace en order_status = ORDER_SUBMITTED
            con los mismos atributos de la plantilla
          → continúa flujo común desde BLOQUE 5 (validación)

          NOTA: en pagos ONLINE_AT_ORDER, el sistema reutiliza el
          método de pago guardado y dispara el cobro automático
          igualmente antes de la validación operativa.

  17-A-5. Comprador puede en cualquier momento:

     a) Pausar la recurrencia
        → recurrence_status = RECURRENCE_PAUSED
        → no se generan pedidos nuevos hasta reactivar

     b) Cancelar la recurrencia
        → recurrence_status = RECURRENCE_CANCELLED
        → no se generarán más pedidos automáticos
        → los pedidos ya generados siguen su ciclo de vida

     c) Editar la plantilla
        - cambia ítems, dirección, periodicidad o método de pago
        - los próximos pedidos usan la plantilla actualizada


  ─────────────────────────────────────────────────────
  17-B. MODALIDAD REPEAT_LAST_ORDER
  ─────────────────────────────────────────────────────

  17-B-1. Comprador ingresa a la página
          → sistema le ofrece la opción de "repetir último pedido"

  17-B-2. Comprador acepta repetir
          → recurrence_mode = REPEAT_LAST_ORDER
          → sistema clona el último pedido como DRAFT
          → order_status = DRAFT

  17-B-3. Comprador puede editar:
          - añadir o quitar ítems
          - cambiar dirección
          - cambiar método de pago

  17-B-4. Comprador confirma
          → continúa flujo común desde BLOQUE 1
            (PENDING_CONFIRM → ORDER_SUBMITTED → ...)

  NOTA PARA DIAGRAMA:
    REPEAT_LAST_ORDER no genera pedidos automáticos. Es solo un
    mecanismo para acelerar la creación manual. Cada repetición
    es un pedido nuevo independiente con su propio ciclo.


===============================================================================
  BLOQUE 18 — PEDIDOS CORPORATIVOS / EMPRESARIALES (NUEVO EN v10.0)
  Flujo de creación manual por parte del cliente para compradores B2B
===============================================================================

  Un comprador corporativo normalmente no usa la página de e-commerce
  directamente. Contacta al cliente (comercio) por canales externos
  (correo, WhatsApp, llamada) y le solicita el pedido. El cliente lo
  registra en el sistema.

  buyer_type:
    INDIVIDUAL  — comprador estándar que usa la página
    CORPORATE   — comprador empresarial gestionado por el cliente

  18-1. Comprador corporativo contacta al cliente por canal externo
        → mensaje recibido con el detalle del pedido deseado

  18-2. Cliente decide cómo registrar el pedido. Tres opciones:

     a) Crear pedido corporativo nuevo
        → order_status = DRAFT
        → buyer_type = CORPORATE
        → cliente captura ítems, dirección, método de pago acordado,
          delivery_type
        → continúa flujo común desde BLOQUE 1

     b) Crear pedido corporativo y marcarlo como recurrente
        → order_status = DRAFT
        → buyer_type = CORPORATE
        → recurrence_mode = SCHEDULED_RECURRING
        → recurrence_periodicity = (valor acordado con la empresa)
        → continúa en BLOQUE 17-A y luego flujo común

     c) Repetir el último pedido corporativo del mismo comprador
        → sistema clona el último pedido de ese comprador corporativo
        → order_status = DRAFT
        → buyer_type = CORPORATE
        → cliente edita si es necesario
        → continúa flujo común desde BLOQUE 1

  18-3. A partir de aquí el pedido sigue el flujo común igual que
        un pedido individual.

        Diferencias relevantes:
          - buyer_type permanece como CORPORATE durante todo el ciclo
          - el cliente puede gestionarlo directamente sin interacción
            del comprador en la página
          - condiciones de pago y entrega pueden estar pactadas por
            fuera del flujo estándar (ej. crédito empresarial, plazos
            especiales) pero deben mapearse a uno de los payment_method
            válidos del sistema

  NOTA PARA DIAGRAMA:
    El pedido corporativo entra al flujo común en DRAFT (o directamente
    en ORDER_SUBMITTED si viene por recurrencia programada). Solo cambia
    el origen y el atributo buyer_type.


===============================================================================
  CONECTORES RECOMENDADOS PARA DIAGRAMAR
  Se deben generar 7 diagramas separados pero conectados
===============================================================================

  ─────────────────────────────────────────────────────
  DIAGRAMA 1 — FLUJO COMÚN + DECISIÓN DE PAGO
  ─────────────────────────────────────────────────────

  Contenido principal:
    DRAFT
    → PENDING_CONFIRM
    → ORDER_SUBMITTED
    → bifurca por payment_method:
        ONLINE_AT_ORDER → PENDING_ONLINE_PAYMENT → PAYMENT_PROCESSING
                          → PAID → ORDER_VALIDATING
        ON_DELIVERY     → PENDING_COLLECTION → ORDER_VALIDATING
    → VALIDATION_APPROVED → SELLER_CONFIRMED → PREPARING
    → bifurca por delivery_type y delivery_carrier_type:
        SHIP + OWN_FLEET         → BLOQUE 6B → Diagrama 2
        SHIP + EXTERNAL_COURIER  → READY_TO_SHIP → Diagrama 2
        PICKUP                   → READY_FOR_PICKUP → Diagrama 3

  Entradas externas:
    - Desde Diagrama 6 (Pedidos Corporativos) → DRAFT
    - Desde Diagrama 5 (Recurrencia programada) → ORDER_SUBMITTED

  Disparadores a Diagrama 4 (Refund):
    Cualquier cancelación con payment_status = PAID dispara reembolso.


  ─────────────────────────────────────────────────────
  DIAGRAMA 2 — FLUJO SHIP COMPLETO
  ─────────────────────────────────────────────────────

  Entrada: PREPARING (con SHIP+OWN_FLEET) o READY_TO_SHIP (con SHIP+EXTERNAL)

  Sub-bloque inicial para OWN_FLEET (BLOQUE 6B):
    AWAITING_COURIER_ASSIGNMENT (mapa geolocalizado)
    → COURIER_ASSIGNED
    → SHIPPED

  Continuación común:
    SHIPPED → IN_TRANSIT → OUT_FOR_DELIVERY → ARRIVED_AT_CUSTOMER
    → handoff por payment_method (igual que v9.0):
        ONLINE_AT_ORDER         → DELIVERED
        ELECTRONIC_ON_DELIVERY  → PAYMENT_COLLECTION_PENDING → ...
        CASH_ON_DELIVERY        → CASH_COLLECTION_PENDING → ...
    → DELIVERED → COMPLETED_SUCCESSFULLY → CLOSED

  Excepciones, devolución al origen, cancelaciones post-despacho:
    Sin cambios respecto a v9.0.

  Disparadores a Diagrama 4 — REFUND:
    Igual que en v9.0, marcados con "→ inicia reembolso si PAID".

  Disparadores a Diagrama 7 — DEVOLUCIONES:
    Solo desde COMPLETED_SUCCESSFULLY si el comprador pone queja.


  ─────────────────────────────────────────────────────
  DIAGRAMA 3 — FLUJO PICKUP COMPLETO
  ─────────────────────────────────────────────────────

  Sin cambios estructurales respecto a v9.0.

  Disparadores a Diagrama 4 y Diagrama 7: idénticos al SHIP en lógica.


  ─────────────────────────────────────────────────────
  DIAGRAMA 4 — FLUJO REFUND COMPLETO (REDISEÑADO)
  ─────────────────────────────────────────────────────

  Entrada: REFUND_PENDING (disparado desde Diagramas 1, 2, 3 o 7)

  Primera bifurcación: refund_modality
    STORE_CREDIT → REFUND_PROCESSING (interno)
                   → REFUNDED / PARTIALLY_REFUNDED / REFUND_FAILED

    BANK_REFUND  → REFUND_PROCESSING
                   → segunda bifurcación por payment_method:

      Caso 1: CASH_ON_DELIVERY o ELECTRONIC_ON_DELIVERY+BANK_TRANSFER
        Reembolso interno del cliente
        → transferencia bancaria desde cuenta del cliente
        → envía comprobante + marca como reembolsado
        → REFUNDED / PARTIALLY_REFUNDED / REFUND_FAILED

      Caso 2: ONLINE_AT_ORDER o ELECTRONIC_ON_DELIVERY+DATAFONO
              o ELECTRONIC_ON_DELIVERY+QR_OR_PAYMENT_LINK
        Reembolso vía proveedor
        → REFUND_PROVIDER_REQUESTED
        → respuesta del proveedor:
            REFUNDED / PARTIALLY_REFUNDED / REFUND_FAILED
        → cliente envía comprobante y marca como reembolsado


  ─────────────────────────────────────────────────────
  DIAGRAMA 5 — FLUJO RECURRENCIA (NUEVO)
  ─────────────────────────────────────────────────────

  Dos sub-diagramas:

    5-A: SCHEDULED_RECURRING
      DRAFT → PENDING_CONFIRM (marca recurrente)
      → guarda plantilla
      → recurrence_status = RECURRENCE_ACTIVE
      → pedido original sigue flujo común
      → ciclo automático:
          [trigger temporal] → genera nuevo pedido
          → entra a Diagrama 1 en ORDER_SUBMITTED

      Acciones del comprador:
        Pausar    → RECURRENCE_PAUSED
        Reanudar  → RECURRENCE_ACTIVE
        Cancelar  → RECURRENCE_CANCELLED
        Editar    → actualiza plantilla

    5-B: REPEAT_LAST_ORDER
      Comprador → "repetir último pedido"
      → sistema clona último pedido como DRAFT
      → comprador edita (opcional)
      → entra a Diagrama 1 en DRAFT


  ─────────────────────────────────────────────────────
  DIAGRAMA 6 — FLUJO PEDIDOS CORPORATIVOS (NUEVO)
  ─────────────────────────────────────────────────────

  Comprador corporativo contacta al cliente por canal externo
    → Cliente elige una de tres opciones:

       a) Crear pedido nuevo
          → DRAFT con buyer_type = CORPORATE
          → Diagrama 1

       b) Crear pedido recurrente
          → DRAFT con buyer_type = CORPORATE
          → Diagrama 5-A (SCHEDULED_RECURRING)

       c) Repetir último pedido del mismo comprador corporativo
          → DRAFT clonado con buyer_type = CORPORATE
          → Diagrama 1


  ─────────────────────────────────────────────────────
  DIAGRAMA 7 — FLUJO DEVOLUCIONES POST-CIERRE (NUEVO)
  ─────────────────────────────────────────────────────

  Entrada: order_status = CLOSED, closure_reason = COMPLETED_SUCCESSFULLY

  RETURN_REQUESTED
  → RETURN_UNDER_REVIEW
      → RETURN_REJECTED (fin)
      → RETURN_APPROVED
          → dispara REFUND_PENDING en paralelo (Diagrama 4)
          → bifurca por return_mechanism:

            BUYER_SHIPS_VIA_COURIER:
              Cliente envía condiciones (proveedores autorizados +
              plazo máximo)
              → CUSTOMER_RETURN_IN_TRANSIT
                  → CUSTOMER_RETURN_RECEIVED → Diagrama 4 ejecuta refund
                  → CUSTOMER_RETURN_EXPIRED → política (refund o no)
                  → CUSTOMER_RETURN_LOST → gestión manual

            CLIENT_PICKS_UP:
              Cliente asigna repartidor (usa BLOQUE 6B)
              → PICKUP_SCHEDULED → PICKUP_OUT_FOR_COLLECTION
                  → PICKUP_COLLECTED → CUSTOMER_RETURN_RECEIVED
                                     → Diagrama 4 ejecuta refund
                  → PICKUP_FAILED → reprogramar o cancelar
                                    (cancelar → REFUND_NOT_REQUIRED)
                  → CUSTOMER_RETURN_LOST → gestión manual


===============================================================================
  RESUMEN DE RUTAS PRINCIPALES (sin cambios respecto a v9.0)
===============================================================================

  RUTA A — ONLINE_AT_ORDER + SHIP
  RUTA B — ONLINE_AT_ORDER + PICKUP
  RUTA C — ELECTRONIC_ON_DELIVERY + SHIP
  RUTA D — ELECTRONIC_ON_DELIVERY + PICKUP
  RUTA E — CASH_ON_DELIVERY + SHIP
  RUTA F — CASH_ON_DELIVERY + PICKUP

  (Ver v9.0 para el detalle paso a paso de cada ruta)


===============================================================================
  RESUMEN DE closure_reason
===============================================================================

  COMPLETED_SUCCESSFULLY    — Pedido entregado/retirado y pago resuelto
  CANCELLED_BY_CUSTOMER     — Cancelado por el comprador
  CANCELLED_BY_SELLER       — Cancelado por el vendedor
  CANCELLED_BY_SYSTEM       — Cancelado automáticamente
  CANCELLED_BY_ADMIN        — Cancelado manualmente por admin
  CANCELLED_NO_PAYMENT      — Comprador no pagó al momento de entrega
  PAYMENT_TIMEOUT           — Pago online no completado a tiempo
  PAYMENT_REJECTED          — Pago online rechazado definitivamente
  VALIDATION_REJECTED       — Rechazado por validación
  DELIVERY_FAILED           — Entrega fallida definitiva
  PICKUP_EXPIRED            — Plazo de retiro vencido
  LOST_IN_TRANSIT           — Pedido perdido en tránsito
  LOST_IN_RETURN            — Pedido perdido en retorno al origen
  EXPIRED                   — Pedido vencido sin confirmación


===============================================================================
  LISTA DE ESTADOS PRINCIPALES order_status (actualizada)
===============================================================================

  --- Creación / confirmación ---
  DRAFT
  PENDING_CONFIRM
  ORDER_SUBMITTED
  EXPIRED

  --- Validación operativa ---
  ORDER_VALIDATING
  MANUAL_REVIEW
  VALIDATION_APPROVED
  VALIDATION_REJECTED

  --- Aceptación / preparación ---
  SELLER_CONFIRMED
  PREPARING

  --- Asignación a repartidor (NUEVO - BLOQUE 6B) ---
  AWAITING_COURIER_ASSIGNMENT
  COURIER_ASSIGNED

  --- Bifurcación logística ---
  READY_TO_SHIP
  READY_FOR_PICKUP

  --- SHIP ---
  SHIPMENT_HOLD
  SHIPPED
  IN_TRANSIT
  ON_HOLD
  OUT_FOR_DELIVERY
  ARRIVED_AT_CUSTOMER
  DELIVERY_ATTEMPTED
  DELIVERY_RESCHEDULED
  LOST_IN_TRANSIT

  --- PICKUP ---
  AT_PICKUP_POINT
  CUSTOMER_ARRIVED_AT_PICKUP_POINT
  IDENTITY_VALIDATED
  PICKUP_AUTH_FAILED
  PICKUP_POINT_ISSUE
  PICKUP_EXPIRED
  PICKUP_CANCELLED_BY_CUSTOMER
  PICKED_UP

  --- Handoff / cobro común ---
  PAYMENT_COLLECTION_PENDING
  PAYMENT_COLLECTED_ELECTRONIC
  PAYMENT_COLLECTED_CASH
  CASH_COLLECTION_PENDING
  CASH_PAYMENT_REJECTED

  --- Cancelación ---
  CANCELLED_BY_CUSTOMER
  CANCELLED_BY_SELLER
  CANCELLED_BY_SYSTEM
  CANCELLED_BY_ADMIN
  CANCELLED_NO_PAYMENT
  CUSTOMER_CANCEL_REQUEST
  CANCEL_REQUEST_APPROVED
  CANCEL_REQUEST_REJECTED

  --- Devolución al origen (logística) ---
  RETURN_TO_ORIGIN
  RETURN_TO_ORIGIN_RECEIVED
  LOST_IN_RETURN

  --- Entrega / cierre ---
  DELIVERED
  DELIVERY_DISPUTED
  CONFIRMED_BY_CUSTOMER
  CONFIRMED_BY_SYSTEM
  COMPLETED_SUCCESSFULLY
  CLOSED


===============================================================================
  LISTA DE ESTADOS FINANCIEROS payment_status
===============================================================================

  PAYMENT_NOT_STARTED
  PENDING_ONLINE_PAYMENT
  PAYMENT_PROCESSING
  PAID
  PAYMENT_FAILED_RETRYABLE
  PAYMENT_REJECTED_FINAL
  PENDING_COLLECTION
  COLLECTION_PROCESSING
  PAYMENT_COLLECTED
  PAYMENT_COLLECTION_FAILED
  PAYMENT_NOT_COLLECTED


===============================================================================
  LISTA DE ESTADOS DE REEMBOLSO refund_status (actualizada)
===============================================================================

  REFUND_NOT_REQUIRED
  REFUND_PENDING
  REFUND_PROCESSING
  REFUND_PROVIDER_REQUESTED     (NUEVO)
  REFUNDED
  PARTIALLY_REFUNDED
  REFUND_FAILED


===============================================================================
  LISTA DE ESTADOS DE RETORNO return_status (actualizada)
===============================================================================

  RETURN_REQUESTED
  RETURN_UNDER_REVIEW           (NUEVO)
  RETURN_APPROVED
  RETURN_REJECTED
  RETURN_CANCELLED              (NUEVO)
  CUSTOMER_RETURN_IN_TRANSIT
  CUSTOMER_RETURN_EXPIRED       (NUEVO)
  CUSTOMER_RETURN_RECEIVED
  CUSTOMER_RETURN_LOST
  PICKUP_SCHEDULED              (NUEVO - mecanismo CLIENT_PICKS_UP)
  PICKUP_OUT_FOR_COLLECTION     (NUEVO)
  PICKUP_COLLECTED              (NUEVO)
  PICKUP_FAILED                 (NUEVO)


===============================================================================
  LISTA DE ESTADOS DE RECURRENCIA recurrence_status (NUEVO)
===============================================================================

  RECURRENCE_ACTIVE
  RECURRENCE_PAUSED
  RECURRENCE_CANCELLED


===============================================================================
  LISTA DE ESTADOS DE DISPUTA dispute_status
===============================================================================

  DISPUTED
  DISPUTE_RESOLVED


===============================================================================
  VALIDACIÓN FINAL DE COHERENCIA
===============================================================================

  1. No existen caminos muertos sin salida lógica.

  2. Todo pedido principal converge en CLOSED.

  3. SHIP y PICKUP son dos rutas logísticas diferentes y diagramables
     por separado.

  4. El pago online al momento del pedido ocurre antes de PREPARING.

  5. El pago al momento de entrega/retiro ocurre en el handoff.

  6. Un pedido con pago pendiente de cobro no pasa a DELIVERED si el
     cobro falla.

  7. El reembolso bifurca primero por refund_modality y luego, si es
     BANK_REFUND, por payment_method (interno vs proveedor).

  8. La devolución post-cierre bifurca por return_mechanism y dispara
     el reembolso en paralelo al momento de aprobación.

  9. La asignación a repartidor (BLOQUE 6B) solo aplica para
     delivery_carrier_type = OWN_FLEET en SHIP.

  10. La recurrencia es un subproceso independiente: la plantilla y
      su recurrence_status no son estados del pedido. Cada pedido
      generado tiene su propio ciclo de vida.

  11. Los pedidos corporativos comparten flujo común con los
      individuales; solo cambian el origen y buyer_type.

  12. ARCHIVED no reemplaza a CLOSED; se maneja como atributo posterior.

===============================================================================