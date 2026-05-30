# Guía de usuario — OPERATOR_CLIENT

Esta guía está dirigida al **operador** del negocio en RUTA. Tu rol es gestionar la operación diaria de los pedidos. No tienes acceso a la configuración del negocio, auditoría completa ni gestión de usuarios.

> **Nota importante:** algunas de las acciones descritas en esta guía requieren que tu ADMIN_CLIENT te haya otorgado el permiso correspondiente. Si un botón no aparece o el sistema te muestra un mensaje de acceso denegado, consulta con el administrador de tu negocio.

Accedes al panel en `app.ruta.com` con tu correo y contraseña. Al iniciar sesión llegas al dashboard con el resumen del día.

---

## 1. Dashboard

Al iniciar sesión verás el dashboard con el resumen del día:

- **Pedidos de hoy**: cuántos pedidos se han creado en el día.
- **Pedidos en tránsito**: pedidos ya despachados y en camino al comprador.
- **Entregados hoy**: pedidos completados en el día.
- **Últimos pedidos**: tabla con los 10 pedidos más recientes.

Desde el dashboard puedes acceder directamente a los pedidos pendientes de acción.

---

## 2. Lista de pedidos

### Qué pedidos puedes ver

Puedes ver todos los pedidos del negocio al que perteneces. No tienes acceso a pedidos de otros negocios.

### Cómo filtrar por estado

1. En el menú lateral selecciona **Pedidos**.
2. Usa los filtros en la parte superior de la tabla:
   - **Estado**: selecciona el estado que quieres ver (por ejemplo, pendiente de aceptación, en preparación, en tránsito).
   - **Tipo de entrega**: domicilio (SHIP) o retiro en punto físico (PICKUP).
   - **Rango de fechas**.
   - **Buscar**: número de pedido o nombre del comprador.
3. La tabla se actualiza automáticamente con los resultados filtrados.

---

## 3. Operación de pedidos SHIP (entrega a domicilio)

Los pedidos de entrega a domicilio siguen estos pasos. Como operador puedes intervenir en cada uno:

### Aceptar o rechazar un pedido

Cuando un pedido pasa la validación automática del sistema, aparece en estado **Pendiente de aceptación del vendedor** (si tienes el permiso correspondiente).

1. Entra al detalle del pedido haciendo clic en su fila.
2. En la columna de acciones haz clic en **Aceptar pedido** o **Rechazar**.
3. Al aceptar el pedido pasa a estado de preparación.
4. Al rechazar el pedido se cancela.

### Marcar en preparación

1. Entra al detalle del pedido.
2. Haz clic en **Marcar preparando** para registrar que tu equipo está alistando el pedido.

### Marcar listo para despacho

1. Cuando el pedido ya está listo para ser recogido por el repartidor, entra al detalle.
2. Haz clic en **Marcar listo para despacho**.
3. Si el negocio tiene repartidores propios, el pedido aparecerá en el mapa de asignación para que le asignes un repartidor (si tienes ese permiso). Si el negocio usa mensajería externa, el pedido pasa directamente al estado de despacho.

---

## 4. Operación en punto físico (PICKUP)

Cuando un comprador elige retirar su pedido en un punto físico, tú o el personal del punto deben gestionar la entrega presencialmente.

### Cómo verificar la identidad del comprador

1. El comprador llega al punto de retiro y te indica el número de pedido.
2. Busca el pedido en RUTA (puedes filtrar por número de pedido en la lista).
3. Verifica que la persona que retira coincide con el nombre del comprador registrado.
4. En el detalle del pedido haz clic en **Validar identidad**.
   - Si la identidad es correcta: el sistema registra la validación y continúa al paso de cobro.
   - Si no coincide: haz clic en **Identidad no válida**. El pedido vuelve al estado de espera y puedes intentar de nuevo cuando el comprador presente el documento correcto.

### Cómo registrar el cobro

El proceso varía según el método de pago elegido por el comprador:

**Si el pedido ya fue pagado en línea (Wompi):**
- No hay cobro por hacer. El sistema ya tiene el pago registrado.
- Procede directamente a marcar el pedido como entregado.

**Si el pago es contra entrega (efectivo o electrónico):**
1. Después de validar la identidad, el sistema muestra la pantalla de cobro.
2. Registra el monto recibido.
3. Selecciona el método: **Efectivo** o **Electrónico** (datáfono, transferencia, etc.).
4. Si es electrónico, ingresa el ID o referencia de la transacción.
5. Sube una foto del comprobante si aplica.
6. Haz clic en **Confirmar cobro**.

### Cómo marcar el pedido como entregado

1. Una vez validada la identidad y resuelto el cobro, haz clic en **Marcar como entregado**.
2. El sistema registra la entrega y notifica al comprador.

---

## 5. Qué NO puedes hacer como OPERATOR_CLIENT

Para evitar confusión, estas son acciones que están fuera de tu alcance:

- **Configuración del negocio**: no puedes cambiar los datos del negocio, agregar pasarelas de pago ni modificar parámetros del sistema. Eso lo hace el ADMIN_CLIENT.
- **Auditoría completa**: no puedes ver el log de auditoría de todo el negocio. Solo puedes ver el registro de tus propias acciones.
- **Gestión de usuarios**: no puedes crear ni eliminar ADMIN_CLIENT ni OPERATOR_CLIENT. Dependiendo de los permisos que te haya dado el ADMIN_CLIENT, podrías crear compradores o repartidores.
- **Webhooks y API keys**: no puedes crear ni gestionar integraciones técnicas del negocio.

Si necesitas hacer algo y el sistema no te lo permite, habla con el ADMIN_CLIENT de tu negocio.
