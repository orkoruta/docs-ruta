# RUTA

**Plataforma de administración de operaciones comerciales entre Clientes y Compradores.**

RUTA permite que un negocio (Cliente) atienda los pedidos de sus Compradores
con un proceso ordenado y trazable: desde la creación del pedido hasta la
entrega, el cobro, las devoluciones y los reembolsos. Sirve tanto a negocios
que ya tienen su propia plataforma de venta y solo necesitan ayuda con la
logística, como a negocios que quieren que RUTA les provea el proceso completo.

---

## Índice

1. [En pocas palabras](#1-en-pocas-palabras)
2. [Glosario](#2-glosario)
3. [Tipos de Cliente: Cliente API y Cliente Full](#3-tipos-de-cliente-cliente-api-y-cliente-full)
4. [Roles y permisos](#4-roles-y-permisos)
5. [Registro de Clientes en la plataforma](#5-registro-de-clientes-en-la-plataforma)
6. [Catálogo de productos](#6-catálogo-de-productos)
7. [Registro de Compradores](#7-registro-de-compradores)
8. [Registro de Repartidores](#8-registro-de-repartidores)
9. [Cómo fluye un pedido (vista general)](#9-cómo-fluye-un-pedido-vista-general)
10. [Los 7 procesos detallados](#10-los-7-procesos-detallados)
11. [Pagos: cómo funcionan](#11-pagos-cómo-funcionan)
12. [Reembolsos](#12-reembolsos)
13. [Devoluciones post-entrega](#13-devoluciones-post-entrega)
14. [Pedidos recurrentes](#14-pedidos-recurrentes)
15. [Pedidos corporativos](#15-pedidos-corporativos)
16. [El principio financiero de RUTA](#16-el-principio-financiero-de-ruta)
17. [Dashboard del Cliente](#17-dashboard-del-cliente)
18. [Funciones del Administrador RUTA](#18-funciones-del-administrador-ruta)
19. [Seguridad y privacidad de la información](#19-seguridad-y-privacidad-de-la-información)
20. [Para Clientes API: integración](#20-para-clientes-api-integración)
21. [Tabla resumen: qué aplica a cada tipo de Cliente](#21-tabla-resumen-qué-aplica-a-cada-tipo-de-cliente)

---

## 1. En pocas palabras

RUTA es una plataforma multi-cliente. Cada Cliente que se registra tiene su
propio espacio, sus propios productos, sus propios Compradores, sus propios
Repartidores, sus propios pedidos y su propia información financiera. Un
Cliente nunca ve la información de otro Cliente.

Dentro de cada Cliente, RUTA orquesta el ciclo completo del pedido:

- El Comprador (o el Cliente, según el caso) **crea un pedido**.
- Se define **cómo se va a pagar** (online, contra entrega electrónica, o
  contra entrega en efectivo) y **cómo se va a entregar** (a domicilio o
  recogida en un punto físico).
- El pedido se **prepara**, se **despacha** y se **entrega** (o se recoge).
- Si hay un **problema** (cancelación, pérdida, devolución, disputa),
  RUTA tiene flujos definidos para resolverlo.
- Si corresponde un **reembolso**, RUTA lo coordina (pero no transfiere
  dinero: ese rol siempre lo cumple el Cliente; ver sección 16).

---

## 2. Glosario

A lo largo de este documento estos términos significan lo siguiente:

- **Cliente.** Empresa o negocio que contrata los servicios de RUTA y
  administra su operación dentro de la plataforma. Ejemplos: un restaurante,
  una panadería, una tienda en línea, una droguería con domicilios.
- **Comprador.** Persona natural o empresa que adquiere productos de un
  Cliente.
- **Repartidor.** Persona encargada de entregar los pedidos asignados por
  un Cliente y, cuando aplique, registrar pagos contra entrega.
- **Pedido.** Solicitud de productos hecha por un Comprador a un Cliente.
- **Pago.** Operación por la cual el Comprador paga al Cliente. Puede
  realizarse al momento del pedido (online) o al momento de la entrega
  (contra entrega).
- **Reembolso.** Devolución del dinero al Comprador cuando un pago ya
  realizado debe revertirse.
- **Devolución.** Solicitud del Comprador de regresar un producto que ya
  recibió (post-entrega) por una queja o inconformidad.
- **Disputa.** Reclamo del Comprador después de que el pedido fue entregado.
- **Recurrencia.** Mecanismo por el cual un pedido se repite automáticamente
  según una periodicidad acordada.
- **Pedido corporativo.** Pedido B2B donde el Comprador es una empresa que
  no usa directamente la página, sino que contacta al Cliente por canales
  externos y el Cliente registra el pedido.
- **Pasarela de pagos.** Servicio externo (por ejemplo Wompi) que procesa
  pagos online. Cada Cliente configura la suya en RUTA.
- **Repartidor con flota propia.** Repartidor que trabaja directamente
  para el Cliente y aparece en su lista de repartidores en RUTA.
- **Mensajero externo.** Empresa de mensajería que el Cliente usa para
  despachar (Servientrega, Coordinadora, etc.).
- **Punto físico.** Lugar donde el Comprador retira su pedido en persona
  (tienda, sucursal, locker).

---

## 3. Tipos de Cliente: Cliente API y Cliente Full

Antes que cualquier otra cosa, al contratar RUTA, cada negocio decide en
qué modalidad va a operar. **RUTA reconoce dos tipos de Cliente** según
qué tanto del proceso quieren delegar a la plataforma.

### 3.1 Cliente API

Es para negocios que **ya tienen su propia plataforma de venta** (su
propio sitio web, su propia app, su propio carrito, su propio checkout) y
solo necesitan que RUTA se encargue de la **logística**: preparar,
asignar repartidor, despachar, hacer seguimiento, cobrar contra entrega
si aplica, y cerrar el pedido.

Los Compradores en este modelo **no acceden directamente a RUTA**:
interactúan únicamente con la plataforma del Cliente. Cuando hacen un
pedido allá, la plataforma del Cliente le pasa los datos a RUTA por una
vía técnica (una integración programática) y a partir de ese momento RUTA
lo recibe en el punto del proceso donde se necesita su ayuda (por
ejemplo, listo para asignar al repartidor).

**Lo que sí gestiona RUTA para un Cliente API:**

- Preparación del pedido (opcional).
- Asignación a un Repartidor (cuando hay flota propia).
- Despacho a domicilio o coordinación de recogida en punto físico.
- Tránsito y reparto final.
- Cobro contra entrega (electrónico o en efectivo), cuando el Cliente
  decida que RUTA cobre.
- Cierre del pedido (entregado, fallido, devolución al origen).
- Trazabilidad y reportes logísticos.

**Lo que NO gestiona RUTA para un Cliente API:**

- La página de venta, el carrito, el catálogo.
- El registro y autenticación de Compradores.
- El cobro online (al momento del pedido). Cuando aplica, el pago ya
  ocurrió en la plataforma del Cliente antes de que el pedido llegue a
  RUTA.
- Los reembolsos.
- Las devoluciones post-cierre.
- Las disputas.
- La recurrencia de pedidos.
- Los pedidos corporativos.

Todo lo anterior lo maneja el Cliente en su propia plataforma. RUTA
respeta esa separación: si llega una solicitud de reembolso o de
devolución sobre un pedido de Cliente API, RUTA la rechaza con un
mensaje claro indicando que esa función no aplica en este modelo.

### 3.2 Cliente Full

Es para negocios que **no tienen plataforma propia** o que prefieren que
RUTA se encargue del proceso completo: catálogo, registro de
Compradores, carrito, pedidos, pagos, entregas, devoluciones,
reembolsos, recurrencia, pedidos corporativos y reportes.

El Cliente Full a su vez puede tener su frontend en una de dos formas:

- **Frontend nativo de RUTA.** El Comprador entra a RUTA directamente
  (por ejemplo a `ruta.com/c/restaurante-el-prado/`) y allí ve el
  catálogo, hace su pedido, paga, sigue el estado, etc.
- **Landing personalizada hecha por RUTA.** El Cliente quiere un sitio
  con su marca propia, dominio propio y diseño a la medida; el equipo
  de RUTA se lo desarrolla, pero todas las funciones están conectadas a
  la misma plataforma RUTA por debajo.

En cualquiera de las dos formas, el ciclo de vida del pedido y todas las
funciones disponibles son las mismas. La diferencia es solo cosmética y
de marca.

**Para un Cliente Full, RUTA gestiona todo:**

- Catálogo de productos.
- Registro y autenticación de Compradores.
- Carrito y checkout.
- Pago online (a través de la pasarela que el Cliente configure).
- Pago contra entrega (electrónico o en efectivo).
- Preparación del pedido.
- Asignación a Repartidor.
- Despacho, tránsito, reparto.
- Recogida en punto físico.
- Cobro en el momento de la entrega.
- Devolución al origen logístico.
- Cancelaciones, en cualquier punto del proceso.
- Disputas.
- Reembolsos.
- Devoluciones post-cierre.
- Recurrencia de pedidos.
- Pedidos corporativos.
- Reportes y métricas.

### 3.3 Tabla rápida de comparación

| Capacidad | Cliente API | Cliente Full |
|---|---|---|
| Catálogo en RUTA | Opcional (referencial) | Obligatorio |
| Compradores en RUTA | No | Sí |
| Carrito y checkout en RUTA | No | Sí |
| Pago online a través de RUTA | No (pre-pago externo) | Sí |
| Pago contra entrega | Sí (opcional) | Sí |
| Asignación de Repartidor en RUTA | Sí | Sí |
| Despacho y entrega | Sí | Sí |
| Recogida en punto físico | Sí | Sí |
| Reembolsos | No | Sí |
| Devoluciones post-cierre | No | Sí |
| Disputas | No | Sí |
| Recurrencia de pedidos | No | Sí |
| Pedidos corporativos | No | Sí |
| Dashboard logístico | Sí | Sí |
| Dashboard financiero completo | Limitado | Sí |
| Integración técnica con el sistema del Cliente | Obligatoria | Opcional |

---

## 4. Roles y permisos

RUTA reconoce cinco tipos de usuario. Cada usuario tiene permisos
diferentes según su rol.

### 4.1 Administrador RUTA

Es el rol interno del prestador del servicio (el equipo que opera RUTA).
Tiene visibilidad sobre toda la plataforma.

**Funciones principales:**

- Registrar, activar, actualizar o desactivar Clientes.
- Configurar el identificador corporativo único de cada Cliente.
- Administrar parámetros generales de la plataforma.
- Supervisar la operación global.
- Consultar información técnica, operativa y estadística de todos los
  Clientes.
- Atender soporte, auditoría y trazabilidad general.
- Entrar en **Vista de Control** sobre el espacio de un Cliente para
  asistir en soporte (ver sección 18.2).

### 4.2 Administrador del Cliente

Es el usuario principal del Cliente dentro de RUTA. Administra la
operación propia de su empresa.

**Funciones principales:**

- Actualizar la información corporativa del Cliente.
- Gestionar los usuarios internos del Cliente (Operadores, Repartidores).
- Registrar y actualizar el catálogo de productos (Cliente Full) o
  consultar el catálogo de referencia (Cliente API).
- Cargar productos de forma individual o masiva mediante archivo Excel.
- Consultar y administrar pedidos, pagos, entregas, devoluciones y
  reembolsos según el tipo de Cliente.
- Acceder al dashboard operativo de su empresa.
- Consultar reportes y métricas de su operación.
- Configurar reglas propias de operación: proveedores de pago,
  modalidad de reembolso, mecanismo de devolución, tipo de despacho.

### 4.3 Operador del Cliente

Es el usuario que ejecuta las actividades operativas del día a día.
Trabaja dentro de los permisos que le asigne el Administrador del
Cliente.

**Funciones principales:**

- Consultar pedidos realizados por los Compradores.
- Actualizar estados de pedidos.
- Gestionar entregas: marcar como preparado, listo para despacho, listo
  para recogida, asignar repartidor.
- Registrar novedades operativas.
- Apoyar procesos de devolución y reembolso (Cliente Full).
- Consultar la información necesaria para atender a Compradores.

### 4.4 Repartidor

Es el usuario que entrega los pedidos físicamente al Comprador.

**Funciones principales:**

- Consultar los pedidos que le fueron asignados.
- Actualizar el estado de cada entrega: salí, llegué, entregué, no pude
  entregar, etc.
- Registrar novedades en la entrega.
- Reportar el pago contra entrega cuando aplique (registrar que cobró,
  con qué método, por cuánto, evidencia).

Un Repartidor solo ve los pedidos del Cliente para el cual está
trabajando en ese momento. Si trabaja para varios Clientes, debe entrar
de manera diferenciada al espacio de cada uno (con sus credenciales
correspondientes para cada Cliente).

### 4.5 Comprador

Es la persona natural o empresa que compra los productos del Cliente.

**Funciones principales:**

- Registrarse a través del espacio del Cliente (solo aplica a Cliente
  Full; en Cliente API el Comprador interactúa con la plataforma propia
  del Cliente, no con RUTA).
- Consultar el catálogo de productos del Cliente.
- Realizar pedidos.
- Pagar o confirmar pagos, según el método elegido.
- Consultar el estado de sus pedidos.
- Solicitar devoluciones, cuando aplique.
- Consultar el estado de entregas y reembolsos.
- Actualizar su información básica de contacto.

---

## 5. Registro de Clientes en la plataforma

Solo el **Administrador RUTA** puede registrar un nuevo Cliente. Al
registrarlo se captura la siguiente información mínima:

- Identificador corporativo único.
- Nombre.
- Descripción.
- Responsable (nombre, identificación, teléfono).
- Logo o imagen corporativa.
- Tipo de Cliente: **API** o **Full**.
- Si es Full: modalidad del frontend (Nativo de RUTA o Landing
  personalizada).

Inmediatamente después del registro, el Administrador RUTA y el
Administrador del Cliente configuran los parámetros operativos:

- **Modalidad de reembolso.** Crédito interno del comercio o devolución
  bancaria (ver sección 12).
- **Mecanismo de devolución.** El Comprador envía vía mensajero externo,
  o el Cliente envía un repartidor a recoger (ver sección 13).
- **Tipo de despacho.** Flota propia, mensajero externo, o ambos.
- **Proveedores de pago.** Pasarela online, datáfono, transferencias,
  links de pago, QR.
- **Puntos físicos.** Direcciones, horarios, contactos (para recogida
  PICKUP).

Una vez configurado, el Cliente puede empezar a operar.

---

## 6. Catálogo de productos

### En Cliente Full

El catálogo de productos vive en RUTA y es la fuente única que ve el
Comprador en el frontend. Cada Cliente registra y administra el suyo.

El registro de productos puede hacerse de dos formas:

- **Individual**, producto por producto.
- **Masivo**, mediante archivo Excel.

Cada producto contiene como mínimo:

- Identificador del Cliente al que pertenece.
- Tipo de producto: **venta normal** o **promoción**.
- Nombre.
- Descripción y características.
- Precio unitario.
- Estado: activo o inactivo.
- Categoría (opcional, para organización).
- Imagen.
- Inventario (opcional).

### En Cliente API

El catálogo vive principalmente en la plataforma del Cliente. RUTA puede
recibir información de productos de forma referencial cuando llega un
pedido (nombre, SKU, precio, cantidad), pero no expone un catálogo al
Comprador. Si el Cliente lo desea, puede sincronizar un catálogo de
referencia en RUTA para fines de reportes.

---

## 7. Registro de Compradores

Los Compradores son **independientes por Cliente**. Una misma persona
que compre en dos negocios diferentes tiene dos registros independientes
en RUTA (uno por cada Cliente). Esto es por diseño: cada Cliente solo
ve a sus propios Compradores.

### En Cliente Full

El Comprador se registra a través del espacio del Cliente (el frontend
nativo o la landing personalizada). Define correo, contraseña, nombre y
datos básicos. Puede guardar direcciones, métodos de pago preferidos y
ver su historial de pedidos.

### En Cliente API

El Comprador se registra y se autentica en la plataforma propia del
Cliente, no en RUTA. Para que RUTA sepa "este pedido es de tal
Comprador" para fines de trazabilidad, el Cliente le indica a RUTA cuál
es su identificador interno del Comprador (un número, un correo, lo que
use). RUTA guarda ese identificador como referencia, pero no le pide
contraseña ni le crea cuenta de acceso directo.

---

## 8. Registro de Repartidores

Cada Cliente registra y administra sus propios Repartidores.

El registro mínimo incluye:

- Identificador del Cliente.
- Identificador del Repartidor (único dentro de ese Cliente).
- Nombre.
- Documento de identificación.
- Teléfono.
- Estado: activo o inactivo.
- Medio de transporte (moto, bicicleta, carro, a pie, etc.).
- Datos adicionales que el Cliente quiera capturar.

**Un mismo Repartidor puede trabajar para varios Clientes**, pero cada
vinculación es un registro independiente. Esto significa que una misma
persona física puede tener varios registros de Repartidor en RUTA, uno
por cada Cliente. Cuando entra a trabajar, lo hace al espacio del
Cliente correspondiente y solo ve los pedidos de ese Cliente.

---

## 9. Cómo fluye un pedido (vista general)

A nivel muy general, todo pedido en RUTA pasa por una secuencia de
etapas. El detalle de cada etapa está en la sección siguiente.

```
   Origen del pedido
        ↓
   Confirmación y método de pago         (solo Cliente Full)
        ↓
   Validación operativa                  (solo Cliente Full)
        ↓
   Aceptación del vendedor               (solo Cliente Full)
        ↓
   PREPARACIÓN                           ← Cliente API puede entrar aquí
        ↓
   Asignación a Repartidor (flota propia)
        ↓
   Despacho a domicilio  /  Listo en punto físico
        ↓
   En tránsito  /  Esperando al Comprador
        ↓
   Llegada al Comprador / Comprador llega al punto
        ↓
   Validación de identidad (en PICKUP) / Handoff
        ↓
   Cobro (si pago contra entrega)
        ↓
   ENTREGADO
        ↓
   Confirmación
        ↓
   CERRADO
```

En cualquier punto pueden surgir eventos que desvíen el flujo:
cancelaciones, pérdidas, devoluciones al origen, problemas de cobro,
disputas, etc. Para cada uno hay un sub-proceso definido.

---

## 10. Los 7 procesos detallados

RUTA tiene **7 procesos principales** documentados. Cada uno tiene su
propio diagrama de estados y su propia lógica.

| # | Proceso | Aplica a Cliente API | Aplica a Cliente Full |
|---|---------|----------------------|------------------------|
| 1 | Creación, confirmación y decisión de pago | ❌ | ✅ |
| 2 | Entrega a domicilio (despacho) | ✅ con matices | ✅ |
| 3 | Recogida en punto físico | ✅ con matices | ✅ |
| 4 | Reembolso | ❌ | ✅ |
| 5 | Recurrencia de pedidos | ❌ | ✅ |
| 6 | Pedidos corporativos | ❌ | ✅ |
| 7 | Devoluciones post-cierre | ❌ | ✅ |

A continuación una descripción funcional de cada uno.

### 10.1 Proceso 1 — Creación, confirmación y decisión de pago

**Aplica solo a Cliente Full.**

Es el punto de entrada del pedido para los Clientes Full. Comienza con
el Comprador haciendo un pedido en el frontend y pasa por estas etapas:

1. **Borrador.** El pedido se está armando, todavía no se confirma.
2. **Pendiente de confirmación.** El Comprador ya tiene los productos
   pero falta definir cómo va a pagar y cómo va a recibir.
3. **Selección de entrega.** Despacho a domicilio o recogida en punto
   físico.
4. **Selección de pago.** Online al momento, contra entrega electrónica,
   o contra entrega en efectivo.
5. **Confirmación del pedido.** El pedido queda enviado al sistema.
6. **Pago online (si aplica).** El Comprador es redirigido a la
   pasarela del Cliente. Si paga, el pedido continúa. Si no paga dentro
   del tiempo permitido, el pedido se cancela automáticamente.
7. **Validación operativa.** RUTA valida datos, stock, reglas anti-fraude.
8. **Aceptación del vendedor.** El Cliente acepta o rechaza el pedido.
9. **Preparación.** El pedido se prepara para despacho o para recogida.

A partir de ahí continúa en el Proceso 2 (despacho) o en el Proceso 3
(recogida).

### 10.2 Proceso 2 — Entrega a domicilio (SHIP)

**Aplica a Cliente Full y a Cliente API.**

Cubre la ruta logística cuando el Cliente despacha a la dirección del
Comprador. Incluye:

- **Asignación al Repartidor** cuando es flota propia. Los pedidos
  listos aparecen geolocalizados en un mapa (puede ser
  OpenStreetMap u otro proveedor) en la ubicación del Comprador. El
  Cliente ve todos los pedidos pendientes y los asigna manualmente a sus
  Repartidores desde una lista.
- **Cuando es mensajero externo**, no hay paso de asignación: el paquete
  se entrega al mensajero (Servientrega, Coordinadora, etc.) y se
  registra el envío.
- **Despacho, tránsito y reparto final.** El Repartidor o el mensajero
  externo lleva el paquete.
- **Llegada al Comprador.** El Repartidor llega a la dirección.
- **Cobro contra entrega.** Si el método de pago así lo definió, en
  este punto se cobra (con datáfono, link de pago, QR, transferencia o
  efectivo) y se registra la evidencia.
- **Entrega.** El paquete queda en manos del Comprador.
- **Posibles desvíos.** Comprador ausente, dirección incorrecta,
  cancelación post-despacho, problema operativo, pérdida en tránsito.
  Cada caso tiene su flujo: reintentar entrega, redirigir a punto
  físico, devolver al origen, cancelar.

Para Cliente API, las cancelaciones que originalmente vendrían del
Comprador en realidad las dispara el Cliente desde su plataforma (porque
el Comprador no está en RUTA directamente).

### 10.3 Proceso 3 — Recogida en punto físico (PICKUP)

**Aplica a Cliente Full y a Cliente API.**

Cubre la ruta logística cuando el Comprador retira el pedido en un
punto físico (tienda, sucursal, locker, etc.).

- **Listo para recogida.** El pedido queda disponible en el punto.
- **Espera al Comprador.** Si el Comprador no llega dentro del plazo
  acordado, el pedido expira y se cierra.
- **Llegada del Comprador.** El operador del Cliente lo identifica.
- **Validación de identidad.** Se verifica que es la persona correcta
  (con código de retiro, documento, etc.).
- **Cobro contra entrega** (si aplica).
- **Entrega.** El paquete queda con el Comprador.
- **Posibles desvíos.** Comprador no llega, identidad no válida,
  Comprador decide no retirar, problema operativo en el punto.

### 10.4 Proceso 4 — Reembolso

**Aplica solo a Cliente Full.**

Se dispara cuando un pedido pagado tiene que devolverle dinero al
Comprador: por cancelación, por validación rechazada, por pérdida en
tránsito, por entrega fallida con pago previo, por devolución aprobada
post-cierre, etc.

El reembolso se ejecuta según la **modalidad de reembolso** que el
Cliente eligió al contratar el servicio (ver sección 12) y, cuando es
en efectivo de vuelta, según el **método de pago original**.

Importante: RUTA **no transfiere dinero**. RUTA solo orquesta y deja
trazabilidad. El que ejecuta el reembolso es el Cliente (desde sus
cuentas internas) o el proveedor de pago del Cliente (vía API).

### 10.5 Proceso 5 — Recurrencia de pedidos

**Aplica solo a Cliente Full.**

Permite que un pedido se repita automáticamente con la periodicidad que
el Comprador defina. Hay dos modalidades:

- **Recurrencia programada.** El Comprador, al hacer un pedido por
  primera vez, lo marca como recurrente y elige la periodicidad
  (diaria, semanal, quincenal, mensual o personalizada). RUTA guarda
  esa plantilla y crea automáticamente un pedido nuevo cada vez que se
  cumple el ciclo. El Comprador puede pausarla, cancelarla o editarla
  en cualquier momento.
- **Repetir último pedido.** El Comprador entra a la página, pide
  "repetir mi último pedido", y RUTA le arma un pedido nuevo con los
  mismos productos. El Comprador puede editarlo antes de confirmarlo
  (agregar o quitar productos, cambiar dirección, cambiar método de
  pago). Esta modalidad no genera pedidos automáticos: es solo un
  acelerador para creación manual.

### 10.6 Proceso 6 — Pedidos corporativos

**Aplica solo a Cliente Full.**

Es el flujo para clientes empresariales (B2B) que no usan la página
directamente. El Comprador corporativo contacta al Cliente por canales
externos (correo, WhatsApp, llamada) y solicita el pedido. El Cliente lo
registra manualmente en RUTA con tres opciones:

- Crear un pedido corporativo nuevo.
- Crear un pedido corporativo y marcarlo como recurrente (combina con
  el Proceso 5).
- Repetir el último pedido del mismo Comprador corporativo.

A partir de ahí el pedido sigue el flujo común. Lo único diferente es
que queda marcado como "corporativo" para fines de reporte y, si el
Cliente lo prefiere, condiciones especiales de pago acordadas
externamente.

### 10.7 Proceso 7 — Devoluciones post-cierre

**Aplica solo a Cliente Full.**

Cubre las devoluciones que el Comprador solicita **después de que el
pedido ya fue entregado y cerrado exitosamente**.

Funciona así:

1. El Comprador pone una queja para devolver el pedido.
2. El Cliente tiene un tiempo definido para aprobar o rechazar la queja.
3. Si la aprueba, se dispara el reembolso en paralelo (Proceso 4) y se
   coordina la recogida física del producto según el mecanismo de
   devolución elegido por el Cliente (ver sección 13).
4. Cuando el producto llega de vuelta, se cierra la devolución.
5. El reembolso continúa según su propio flujo.

---

## 11. Pagos: cómo funcionan

### 11.1 Métodos de pago aceptados

RUTA soporta cuatro métodos de pago:

- **Online al momento del pedido.** El Comprador paga antes de que el
  Cliente prepare el pedido. Se hace a través de la pasarela que el
  Cliente haya configurado (Wompi, Mercado Pago, ePayco u otra). El
  dinero llega directo a la cuenta del Cliente; RUTA solo recibe una
  confirmación técnica de que el pago ocurrió. Aplica solo a Cliente
  Full.
- **Electrónica contra entrega.** El Comprador paga cuando recibe el
  pedido, usando un medio electrónico: datáfono, transferencia, link
  de pago o QR. El Repartidor o el operador del punto físico registra
  el cobro en RUTA como evidencia.
- **Efectivo contra entrega.** El Comprador paga en efectivo al
  momento de recibir. El Repartidor registra el cobro en RUTA con
  monto y novedades.
- **Pre-pago externo.** Aplica solo a Cliente API. Es el caso donde
  el pago ya ocurrió en la plataforma propia del Cliente antes de
  que el pedido llegue a RUTA. RUTA recibe el pedido marcado como ya
  pagado y no interviene en el cobro.

### 11.2 Cuándo se cobra

Depende del método:

- Online al momento: antes de que el Cliente prepare.
- Contra entrega: en el handoff físico, cuando el Comprador ya tiene
  el pedido al frente.
- Pre-pago externo: ya ocurrió antes de RUTA.

### 11.3 Quién recibe el dinero

**Siempre el Cliente.** RUTA no recibe ni custodia dinero. Más detalle
en la sección 16.

---

## 12. Reembolsos

### 12.1 Modalidades de reembolso

Cada Cliente Full decide al contratar el servicio cómo quiere manejar
los reembolsos. Hay dos modalidades:

- **Crédito interno (Store Credit).** El monto a devolver se acredita
  como saldo a favor en la cuenta del Comprador dentro del comercio
  del Cliente. El Comprador lo puede usar en próximos pedidos. Es la
  más simple porque no requiere mover dinero al banco del Comprador.
- **Devolución bancaria (Bank Refund).** El dinero se devuelve a la
  cuenta bancaria del Comprador. Esto depende del método de pago
  original:
  - Si pagó en efectivo contra entrega o por transferencia bancaria:
    el Cliente hace la transferencia interna desde sus cuentas, envía
    comprobante al Comprador y marca el pago como reembolsado en RUTA.
  - Si pagó con pasarela online, datáfono o QR/link de pago: el
    Cliente le pide al proveedor (Wompi, Gold, etc.) que ejecute el
    reembolso. Cuando el proveedor confirma, el Cliente envía
    comprobante al Comprador y marca el pago como reembolsado.

### 12.2 Quién ejecuta el reembolso

**Siempre el Cliente** (o el proveedor de pagos del Cliente, cuando
aplique). RUTA no transfiere dinero ni acredita créditos por sí mismo.
RUTA solo registra el estado de cada paso del reembolso para
trazabilidad.

### 12.3 Reembolsos parciales

Sí están soportados. El estado del reembolso refleja si fue total o
parcial.

---

## 13. Devoluciones post-entrega

### 13.1 Mecanismos de devolución

Cada Cliente Full elige, al contratar RUTA, cómo va a recibir el
producto físico que el Comprador devuelve. Hay dos mecanismos:

- **El Comprador envía vía mensajero externo.** El Cliente le indica
  al Comprador los proveedores autorizados (Servientrega, Coordinadora,
  etc.) y el plazo máximo. El Comprador despacha por su cuenta y el
  Cliente recibe en su bodega.
- **El Cliente envía un Repartidor a recoger.** Funciona como un
  pedido "al revés": el Cliente programa la recogida, asigna un
  Repartidor desde el mismo mapa que usa para asignar pedidos
  normales, y el Repartidor va a la dirección del Comprador a recoger
  el producto. Luego lo lleva a bodega.

### 13.2 Pasos del proceso

1. El Comprador pone una queja para devolver.
2. El Cliente revisa dentro de un plazo definido.
3. Si aprueba: se dispara el reembolso en paralelo y se coordina el
   mecanismo elegido.
4. Cuando el producto llega de vuelta al Cliente, la devolución se
   marca como recibida.
5. El reembolso se completa según su propio flujo.

### 13.3 Casos especiales

- **Producto no llega dentro del plazo** (mensajero externo): según la
  política del Cliente, se cancela el reembolso o se aprueba
  igualmente.
- **Producto se pierde en tránsito de retorno**: gestión manual, caso
  por caso.
- **Comprador no está cuando el Repartidor llega a recoger**: se
  reprograma o se cancela la devolución.

---

## 14. Pedidos recurrentes

Ya descritos en el Proceso 5 (sección 10.5). Resumen:

- **Aplica solo a Cliente Full.**
- Modalidad 1: pedido marcado como recurrente al confirmarlo, con
  periodicidad configurable. RUTA crea pedidos nuevos
  automáticamente.
- Modalidad 2: repetir último pedido bajo demanda. No es automático,
  es solo un acelerador.
- El Comprador puede pausar, cancelar o editar la plantilla en
  cualquier momento.
- Cada pedido generado tiene su propio ciclo de vida independiente.

---

## 15. Pedidos corporativos

Ya descritos en el Proceso 6 (sección 10.6). Resumen:

- **Aplica solo a Cliente Full.**
- El Comprador corporativo contacta al Cliente por fuera de la página.
- El Cliente registra el pedido manualmente.
- Tres opciones: nuevo, nuevo recurrente, repetir el último.
- El pedido sigue el flujo común; queda marcado como CORPORATE para
  reportes.

---

## 16. El principio financiero de RUTA

Este es uno de los principios fundamentales de la plataforma y conviene
entenderlo bien:

> **RUTA NO recauda dinero. RUTA NO custodia dinero. RUTA NO transfiere
> dinero. RUTA orquesta procesos y registra estados.**

¿Qué significa esto en la práctica?

- **Pagos online.** Cuando el Comprador paga en la pasarela, el dinero
  va directo de la pasarela a la cuenta del Cliente. RUTA solo recibe
  una confirmación técnica de que el pago ocurrió y actualiza el
  estado del pedido a "pagado".
- **Pagos contra entrega.** El Repartidor cobra al Comprador y
  registra el cobro en RUTA como evidencia (monto, método, fecha,
  novedades, identificación de quien cobró). El dinero recolectado
  pertenece al Cliente desde el momento del cobro. Si es efectivo, el
  Repartidor se lo entrega al Cliente. Si es electrónico (datáfono,
  transferencia, QR), llega directo a las cuentas del Cliente. RUTA
  no maneja el dinero, solo deja registro de lo ocurrido para que el
  Cliente pueda hacer su conciliación.
- **Reembolsos.** El que ejecuta el reembolso es el Cliente (desde sus
  cuentas) o el proveedor de pago del Cliente (a través de su API).
  RUTA solo marca cada paso del estado del reembolso para
  trazabilidad. Si el Cliente reporta que ya transfirió, RUTA lo
  refleja como "reembolsado". RUTA nunca ejecuta la transferencia.
- **Conciliación.** RUTA expone reportes completos para que el Cliente
  pueda conciliar lo que cobró contra entrega con lo que efectivamente
  recibió en sus cuentas, y lo que reembolsó contra lo que registró
  haber reembolsado.

Este principio simplifica enormemente la operación, evita riesgos
regulatorios y deja claro que **la relación financiera es Comprador ↔
Cliente**, no Comprador ↔ RUTA.

---

## 17. Dashboard del Cliente

Cada Cliente cuenta con un dashboard informativo y funcional que le
permite consultar y administrar su operación. El dashboard ofrece
estadísticas, métricas e indicadores **parametrizables por rango de
fechas**, relacionados con:

- **Pedidos.** Cantidad, montos, distribución por estado, distribución
  por tipo de entrega, por origen, por buyer corporativo o
  individual, etc.
- **Pagos.** Totales por método, cobros pendientes, cobros fallidos,
  conciliación.
- **Entregas.** Tasa de éxito, entregas fallidas, tiempos promedio,
  rendimiento por Repartidor.
- **Repartidores.** Pedidos asignados, entregas completadas, novedades
  reportadas.
- **Devoluciones.** Cantidad, motivos, tiempos de respuesta, tasa de
  aprobación (solo Cliente Full).
- **Reembolsos.** Cantidad, montos, métodos, tiempos hasta completar
  (solo Cliente Full).
- **Productos.** Más vendidos, menos vendidos, rotación, inventario
  (Cliente Full).
- **Compradores.** Nuevos, recurrentes, top compradores, frecuencia
  (Cliente Full).

Los Clientes API tienen un dashboard reducido enfocado en métricas
logísticas (entregas, repartidores, tiempos) porque las métricas
comerciales (compradores, productos, pagos online) viven en su
plataforma propia.

---

## 18. Funciones del Administrador RUTA

### 18.1 Gestión de Clientes

- Registrar, activar y desactivar Clientes.
- Configurar el tipo (API o Full) y el frontend (Nativo o Landing) al
  momento del registro.
- Editar información corporativa.
- Cambiar el estado del Cliente. Cuando un Cliente se desactiva, todos
  sus pedidos activos se cancelan automáticamente, sus Compradores
  pierden acceso y, si había pagos pendientes de reembolso, se
  disparan los reembolsos correspondientes.

### 18.2 Vista de Control (modo soporte)

El Administrador RUTA puede entrar al espacio de un Cliente para asistir
en soporte o auditoría, **temporalmente y bajo trazabilidad completa**.
Esto se llama Vista de Control.

- Para activarla, el Administrador RUTA debe tener un permiso explícito
  habilitado.
- Adicionalmente, debe ingresar una **contraseña maestra** distinta a
  su contraseña habitual.
- Mientras está en Vista de Control, ve y opera el espacio del Cliente
  como si fuera el Administrador del Cliente, pero todas sus acciones
  quedan registradas en auditoría con la marca "vía Vista de Control"
  y el identificador del Administrador RUTA real.
- El Administrador del Cliente no recibe notificación de la sesión de
  soporte (es una decisión operativa del prestador del servicio).
- La sesión termina cuando el Administrador RUTA cierra sesión
  manualmente; no hay duración máxima impuesta automáticamente.

### 18.3 Eliminación definitiva de un Cliente

Esta es una operación irreversible. Solo el Administrador RUTA la
puede ejecutar, y solo sobre Clientes que ya estén desactivados. Se
requiere doble confirmación (identificador del Cliente + contraseña
maestra). Toda la información operativa del Cliente se borra, excepto
los registros de auditoría, que se conservan por requerimiento legal.

---

## 19. Seguridad y privacidad de la información

### 19.1 Separación entre Clientes

Cada Cliente tiene su propio espacio. Un Cliente nunca puede ver,
consultar ni operar la información de otro Cliente. Toda la información
está obligatoriamente asociada al identificador del Cliente
correspondiente. Ningún pedido, producto, Comprador, pago, reembolso o
devolución puede existir sin pertenecer a un Cliente.

### 19.2 Sesiones de usuario

Cuando un usuario inicia sesión en RUTA recibe una credencial temporal
que va caducando y se renueva automáticamente mientras la persona esté
activa. Si cierra sesión, la credencial se invalida inmediatamente. Si
intenta usar la sesión desde otro lugar de forma sospechosa, el sistema
puede invalidarla por seguridad. Cada credencial está asociada a un
único inicio de sesión: no puede usarse simultáneamente desde dos
dispositivos compartiendo la misma credencial.

### 19.3 Diferenciación entre tipos de usuario

- Los Compradores y Repartidores están totalmente separados por
  Cliente: una misma persona física que opere para dos Clientes tiene
  dos cuentas independientes.
- Los Administradores y Operadores pertenecen a un Cliente específico.
- Los Administradores RUTA son el único rol que puede operar de forma
  transversal entre Clientes, y solo a través de las funciones
  específicas (gestión global, Vista de Control).

### 19.4 Auditoría

Toda acción operativa relevante queda registrada en un log de auditoría
inmutable: quién lo hizo (o qué llave técnica), qué hizo, sobre qué
entidad, desde dónde, cuándo, y si actuó vía Vista de Control, qué
Administrador RUTA estaba detrás de la sesión.

### 19.5 Acceso del Repartidor

Un Repartidor solo puede consultar y actualizar los pedidos que le
fueron asignados a él, dentro del Cliente correspondiente. No puede ver
pedidos de otros Repartidores ni información de otros Clientes.

---

## 20. Para Clientes API: integración

Los Clientes API integran su plataforma con RUTA por medios técnicos.
Esta sección describe en términos no técnicos cómo se ve esa
integración.

### 20.1 Llaves API

El Cliente API recibe una **llave de acceso** (compuesta por un
identificador público y un secreto privado) que su plataforma usa para
comunicarse con RUTA. Esta llave es como una credencial técnica: nadie
tiene que iniciar sesión cada vez; el sistema del Cliente se identifica
con la llave en cada llamada.

Las llaves tienen:

- Un nombre descriptivo (para saber para qué se usan).
- Un conjunto de permisos (qué acciones puede ejecutar esa llave).
- Fecha de expiración (opcional).
- Posibilidad de revocarlas en cualquier momento.

El Administrador del Cliente puede crear, listar y revocar sus
propias llaves desde su panel.

### 20.2 Notificaciones automáticas (webhooks)

RUTA puede **avisar automáticamente** al sistema del Cliente cuando
ocurren eventos importantes en sus pedidos: pedido entregado, cobro
realizado, devolución solicitada, etc. Esto se llama webhook.

El Cliente API configura:

- Una URL de su sistema donde quiere recibir los avisos.
- Qué eventos le interesan (entregas, cobros, devoluciones, etc.).

Cada vez que ocurre uno de esos eventos, RUTA hace una llamada a la URL
del Cliente con la información del evento. Si la llamada falla
(porque el sistema del Cliente está caído, por ejemplo), RUTA reintenta
varias veces durante hasta 24 horas. Si después de varios intentos
sigue sin recibirse, la notificación se marca como fallida y el Cliente
puede verla en su panel.

Cada notificación viene firmada para que el sistema del Cliente pueda
verificar que efectivamente viene de RUTA y no de un tercero.

### 20.3 Pasarelas de pago

Los Clientes API normalmente ya tienen integradas sus propias
pasarelas. Si además quieren que RUTA cobre contra entrega electrónica,
también pueden configurar pasarelas de RUTA para ese uso específico.

---

## 21. Tabla resumen: qué aplica a cada tipo de Cliente

| Función | Cliente API | Cliente Full |
|---|---|---|
| Catálogo en RUTA | Opcional (referencial) | Sí |
| Registro de Compradores en RUTA | No | Sí |
| Carrito y checkout en RUTA | No | Sí |
| Pago online procesado por RUTA | No (pre-pago externo) | Sí |
| Pago contra entrega | Opcional | Sí |
| Preparación del pedido | Opcional | Sí |
| Asignación de Repartidor (flota propia) | Sí | Sí |
| Despacho con mensajero externo | Sí | Sí |
| Despacho a domicilio | Sí | Sí |
| Recogida en punto físico | Sí | Sí |
| Registro de cobro contra entrega | Sí | Sí |
| Devolución al origen logístico | Sí | Sí |
| Cancelaciones desde el frontend | No (vía API del Cliente) | Sí |
| Disputas post-entrega | No | Sí |
| Reembolsos | No | Sí |
| Devoluciones post-cierre | No | Sí |
| Recurrencia de pedidos | No | Sí |
| Pedidos corporativos | No | Sí |
| Repartidores propios del Cliente | Sí | Sí |
| Dashboard logístico | Sí | Sí |
| Dashboard comercial y financiero completo | Limitado | Sí |
| Vista de Control para soporte | Sí | Sí |
| Llaves API | Sí (obligatorias) | Sí (opcionales) |
| Webhooks salientes | Sí (recomendados) | Sí (opcionales) |
| Auditoría completa | Sí | Sí |
| Separación total de datos entre Clientes | Sí | Sí |

---

## Anexo: documentos relacionados

Este README es la descripción funcional general de la plataforma. Para
información más detallada, ver:

- **Estrategia multi-tenant.** Decisiones técnicas y arquitectónicas:
  `docs/arquitectura/estrategia_multi_tenant_ruta.md`.
- **Flujos de proceso.** Diagramas y estados detallados de los 7
  procesos: `docs/flujos/flujo_1_*.txt` a `flujo_7_*.txt`.
- **Ciclo de vida del token de sesión.** Comportamiento detallado de la
  autenticación: `docs/seguridad/ciclo_vida_token.txt`.
- **Reglas para diagramar flujos.** Convenciones de notación para los
  diagramas: `docs/flujos/reglas_para_diagramar_flujos.txt`.
- **Modelo de datos.** Estructura de base de datos:
  `docs/bd/ruta_postgres.sql` (PostgreSQL) y `docs/bd/ruta_oracle.sql`
  (Oracle, para generar el DER).
- **Galería de estilos.** Identidad visual, tokens de diseño,
  componentes UI, modos claro/oscuro y prompt de IA para diseño de
  pantallas: `docs/diseno/galeria_estilos_ruta.md`.

docs/
├── all_ruta.md                              ← README principal
├── arquitectura/
│   └── estrategia_multi_tenant_ruta.md
├── seguridad/
│   └── ciclo_vida_token.txt
├── flujos/
│   ├── reglas_para_diagramar_flujos.txt
│   ├── flujo_1_comun_y_decision_de_pago.txt
│   ├── flujo_2_ship_completo.txt
│   ├── flujo_3_pickup_completo.txt
│   ├── flujo_4_refund_completo.txt
│   ├── flujo_5_recurrencia.txt
│   ├── flujo_6_pedidos_corporativos.txt
│   └── flujo_7_devoluciones_post_cierre.txt
├── bd/
│   ├── ruta_postgres.sql
│   └── ruta_oracle.sql
└── diseno/
    └── galeria_estilos_ruta.m