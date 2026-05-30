# Guía de usuario — ADMIN_CLIENT

Esta guía está dirigida al **administrador del negocio** en RUTA. Como ADMIN_CLIENT tienes acceso completo a la operación de tu empresa: catálogo, pedidos, repartidores, compradores, puntos de retiro, configuración y reportes.

Accedes al panel en `app.ruta.com` con tu correo y contraseña. Al iniciar sesión el sistema te lleva directo al dashboard de tu negocio.

---

## 1. Gestión del catálogo

### Cómo crear y editar categorías

1. En el menú lateral selecciona **Catálogo** y luego **Categorías**.
2. Haz clic en **Crear categoría**.
3. Escribe el nombre de la categoría y guarda.
4. Para editar una categoría existente, haz clic en el nombre y modifica los campos que necesites.
5. Puedes activar o desactivar categorías con el interruptor que aparece en cada fila de la lista.

### Cómo crear y editar productos

1. En el menú lateral selecciona **Catálogo** y luego **Productos**.
2. Haz clic en **Crear producto**.
3. Completa el formulario:
   - **Nombre** del producto.
   - **Descripción** (opcional pero recomendada).
   - **Precio** de venta.
   - **Categoría** a la que pertenece.
   - **Tipo** de producto.
   - **Imagen**: haz clic en el área de carga y selecciona el archivo desde tu computador.
   - **Stock disponible**.
   - **Estado**: activo o inactivo.
4. Haz clic en **Guardar**.
5. Para editar un producto ya creado, búscalo en la lista y haz clic en su nombre. Modifica lo que necesites y guarda.
6. Puedes activar o inactivar un producto desde el interruptor en la lista sin necesidad de abrir el formulario.

### Importación masiva por Excel

1. En la pantalla de **Productos** haz clic en **Importar Excel**.
2. En la ventana que se abre, arrastra tu archivo Excel al área indicada o haz clic para seleccionarlo.
3. RUTA muestra una vista previa de las primeras filas para que verifiques el formato.
4. Si todo se ve bien, haz clic en **Importar**.
5. Al finalizar verás un reporte con cuántos productos se importaron correctamente y cuáles tuvieron errores.

---

## 2. Gestión de pedidos

### Cómo ver la lista de pedidos con filtros

1. En el menú lateral selecciona **Pedidos**.
2. Verás la tabla con todos los pedidos de tu negocio.
3. Usa los filtros en la parte superior para encontrar pedidos específicos:
   - **Estado**: filtra por el estado del pedido (por ejemplo, pendiente, en preparación, en tránsito, entregado).
   - **Tipo de entrega**: domicilio o punto de retiro.
   - **Método de pago**.
   - **Rango de fechas**: selecciona fecha de inicio y fin.
   - **Repartidor**: filtra pedidos asignados a un repartidor específico.
   - **Buscar**: escribe el número de pedido o el nombre del comprador.
4. Para exportar la lista filtrada haz clic en **Exportar**.

### Vista de detalle 360 de un pedido

1. Haz clic en cualquier fila de la lista de pedidos.
2. Verás la pantalla completa del pedido con:
   - Estado actual y línea de tiempo de todos los eventos.
   - Datos del comprador (nombre, contacto, dirección).
   - Detalle de entrega (tipo, repartidor asignado si aplica).
   - Resumen de los productos y totales.
   - Estado del pago y método utilizado.
   - Acciones disponibles según el estado actual (ver sección siguiente).

### Aceptar o rechazar un pedido

Los pedidos de compradores pasan primero por una validación automática del sistema. Cuando el sistema los aprueba aparecen en estado **Pendiente de aceptación del vendedor**.

1. Entra al detalle del pedido.
2. En la columna de acciones verás los botones **Aceptar pedido** y **Rechazar**.
3. Al aceptar, el pedido pasa a estado de preparación.
4. Al rechazar, el pedido se cancela. Si el comprador ya había pagado en línea, el sistema inicia el proceso de reembolso automáticamente.

### Marcar como en preparación / listo para despacho

1. Una vez aceptado el pedido, entra a su detalle.
2. Haz clic en **Marcar preparando** para indicar que tu equipo está alistando el pedido.
3. Cuando el pedido ya está listo, haz clic en **Marcar listo para despacho** (si es entrega a domicilio) o **Marcar listo para retiro** (si es retiro en punto físico).

### Aprobar o rechazar solicitud de cancelación del comprador

Si un pedido ya fue despachado y el comprador pide cancelarlo, aparecerá en estado **Solicitud de cancelación**.

1. Entra al detalle del pedido.
2. Lee el motivo indicado por el comprador.
3. Haz clic en **Aprobar cancelación** o **Rechazar cancelación**.
   - Si apruebas: el sistema inicia la devolución al origen y, si había pago, el reembolso.
   - Si rechazas: el pedido vuelve a su estado operativo anterior.

---

## 3. Mapa de asignación de repartidores

### Cómo ver el mapa

1. En el menú lateral selecciona **Mapa**.
2. Verás un mapa interactivo (OpenStreetMap) con:
   - Pines de color que representan los pedidos listos para ser asignados.
   - Íconos de repartidor que muestran la ubicación de tus repartidores disponibles.
3. El mapa se actualiza automáticamente cada 30 segundos.

### Cómo asignar un repartidor a un pedido

1. Haz clic sobre el pin de un pedido en el mapa. Se abrirá un panel con el detalle del pedido.
2. En el panel lateral derecho verás la lista de **Repartidores disponibles**.
3. Selecciona el repartidor que quieres asignar y haz clic en **Asignar**.
4. Confirma la asignación en el mensaje de verificación.
5. El pedido cambia de estado a **Repartidor asignado** y el repartidor recibe la notificación en su app.

### Ver couriers disponibles

El panel derecho del mapa muestra en todo momento la lista de repartidores activos con el número de pedidos que llevan en ese momento. También puedes verlos en el menú **Repartidores**.

---

## 4. Compradores y repartidores

### Gestión de compradores

1. En el menú lateral selecciona **Compradores**.
2. Verás la lista de todos los compradores registrados en tu negocio.
3. Usa el buscador para encontrar a un comprador por nombre o correo.
4. Haz clic en un comprador para ver su perfil, direcciones guardadas e historial de pedidos.
5. Para **crear** un comprador nuevo haz clic en **Crear comprador** y completa el formulario.
6. Para **editar** los datos de un comprador entra a su perfil y modifica lo que necesites.
7. Para **activar o desactivar** un comprador usa el interruptor en su perfil. Un comprador desactivado no puede iniciar sesión ni hacer pedidos.

### Gestión de repartidores

1. En el menú lateral selecciona **Repartidores**.
2. Verás la lista de repartidores de tu negocio con métricas básicas.
3. Para **crear** un repartidor haz clic en **Crear repartidor** y completa nombre, correo y teléfono. El repartidor recibirá instrucciones para crear su contraseña.
4. Para **editar** los datos de un repartidor entra a su perfil.
5. Para **activar o desactivar** un repartidor usa el interruptor en su perfil.

---

## 5. Pickup Points (Puntos de retiro)

1. En el menú lateral selecciona **Puntos de retiro**.
2. Verás la lista de puntos de retiro que tiene tu negocio.
3. Para **crear** un punto nuevo haz clic en **Crear punto de retiro** y completa:
   - Nombre del punto.
   - Dirección.
   - Ubicación en el mapa: arrastra el pin hasta la posición exacta.
   - Horario de atención.
   - Teléfono de contacto.
4. Para **editar** un punto existente haz clic en su nombre.
5. Puedes **activar o desactivar** un punto con el interruptor. Un punto desactivado no aparece como opción para los compradores en el checkout.

---

## 6. Configuración del negocio

Ve a **Configuración** en el menú lateral. Encontrarás cuatro pestañas:

### Información corporativa

Edita el nombre, descripción, logo y datos de contacto de tu negocio. Estos datos aparecen en el storefront que ven tus compradores.

### Configuración de pagos Wompi

1. En la pestaña **Pasarelas de pago** verás los proveedores configurados.
2. Para agregar Wompi haz clic en **Agregar Wompi** y completa:
   - **Llave pública (Public key)**: la encuentras en tu cuenta de Wompi. Esta llave es la que RUTA usa para mostrar el formulario de pago a tus compradores. No la compartas con personas fuera de tu equipo.
   - **Llave privada** y **Events secret**: úsalas para que RUTA pueda confirmar los pagos que hace Wompi.
3. La URL del webhook que Wompi debe llamar se genera automáticamente; aparece en pantalla en modo solo lectura.

### Webhooks salientes

Esta sección permite que RUTA notifique a sistemas externos cada vez que ocurre un evento importante (por ejemplo, cuando un pedido cambia de estado).

- **Historial de entregas**: muestra cada notificación enviada con su resultado (exitosa o fallida).
- **Reintentar fallidos**: si una notificación falló, haz clic en el botón de reintento para enviarla de nuevo.

### Parámetros del sistema

Son valores configurables que controlan el comportamiento de RUTA para tu negocio. Ejemplos: tiempo máximo que tiene un comprador para pagar en línea, plazo de retiro en un punto físico, etc.

1. Verás la lista de parámetros con el valor global (de la plataforma) y el valor que tu negocio tiene configurado.
2. Para sobrescribir un valor haz clic en el parámetro y escribe el nuevo valor.
3. Si dejas el campo vacío, RUTA usa el valor global por defecto.

---

## 7. Auditoría

1. En el menú lateral selecciona **Auditoría**.
2. Verás el registro completo de acciones realizadas dentro de tu negocio: quién hizo qué y cuándo.
3. Filtros disponibles:
   - **Usuario o API key**: filtra por el actor que realizó la acción.
   - **Acción**: tipo de operación (crear, editar, cancelar, etc.).
   - **Rango de fechas**.
4. Haz clic en cualquier fila para ver el detalle completo de la acción.

Para qué sirve: el log de auditoría te permite rastrear quién modificó un pedido, quién creó o eliminó un producto, y cualquier otra acción importante. Es útil para resolver disputas internas y mantener el control de tu operación.

---

## 8. Dashboard

El dashboard es la primera pantalla que ves al iniciar sesión. Muestra un resumen del estado de tu negocio:

- **Pedidos de hoy**: cuántos pedidos se han creado en el día.
- **Pedidos en tránsito**: pedidos que ya fueron despachados y están camino al comprador.
- **Entregados hoy**: pedidos que ya llegaron al comprador en el día.
- **Ingresos en tránsito**: valor total de los pedidos que están activos.
- **Acciones rápidas**: accesos directos para ver pedidos pendientes, abrir el mapa de asignación y crear un producto.
- **Últimos pedidos**: tabla con los 10 pedidos más recientes.

Los datos de métricas también están disponibles en la sección de **Reportes** con filtros por rango de fechas para pedidos, pagos, reembolsos, devoluciones y repartidores.
