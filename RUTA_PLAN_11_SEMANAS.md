# RUTA — Plan de Desarrollo 11 Semanas / 2 Desarrolladores Full-Stack

Este README describe el nuevo modelo de trabajo para desarrollar el MVP Fase 1 de RUTA con dos desarrolladores durante 11 semanas.

## Principio central

El trabajo no se divide por tecnología, es decir, no habrá un desarrollador solo de frontend y otro solo de backend.

La división se hace por **módulos funcionales**, y cada desarrollador es responsable de completar sus módulos de extremo a extremo.

La regla operativa es:

1. Primero se construye el **front** del módulo con mocks.
2. Luego se definen los **contratos de datos**.
3. Después se implementa el **backend** del mismo módulo.
4. Finalmente se hace la **integración front-back**, pruebas y ajustes.

## Distribución general

| Desarrollador | Responsabilidad funcional |
|---|---|
| Dev 1 | Administración, Clientes, Productos, Pedidos Admin, Vista de Control, Dashboards, Configuración, Auditoría, Webhooks, Producción |
| Dev 2 | Storefront, Comprador, Carrito, Checkout, UX de pagos, Courier, SHIP, PICKUP, Landing Template, QA funcional |

## Fases del plan

### Fase 1 — Front primero

**Semanas 1 a 4**

Objetivo: dejar navegable la aplicación completa con mock data.

En esta fase se busca validar:

- Navegación.
- Flujo de usuario.
- Estados visuales.
- Pantallas principales.
- Formularios.
- Acciones por rol.
- Contratos esperados.
- Experiencia mobile-first donde aplique.

No se bloquea el avance esperando backend real.

### Fase 2 — Backend por módulo

**Semanas 5 a 9**

Objetivo: cada desarrollador implementa el backend de los módulos que ya construyó visualmente.

El backend debe respetar los contratos definidos en la fase de front.

Cada módulo debe quedar integrado y probado.

### Fase 3 — Integración, QA y piloto

**Semanas 10 a 11**

Objetivo: estabilizar, probar, desplegar y ejecutar el primer piloto real.

Incluye:

- QA funcional.
- Pruebas E2E.
- Seguridad cross-tenant.
- Deploy producción.
- Backups y restore.
- Observabilidad.
- Documentación.
- Onboarding del cliente piloto.
- Primer pedido real.

## Cronograma resumido

| Semana | Foco |
|---|---|
| 1 | Setup general, estructura Admin y Storefront |
| 2 | Front de Clientes, Catálogo y Auth visual |
| 3 | Front operativo: productos, carrito y checkout |
| 4 | Front avanzado: pedidos, control, courier, SHIP/PICKUP visual |
| 5 | Backend auth, clientes, productos y catálogo público |
| 6 | Backend pedidos admin y pedidos comprador |
| 7 | Wompi, jobs, validación operativa y UX pago |
| 8 | Flujo SHIP y Courier |
| 9 | PICKUP, dashboards, control, auditoría y webhooks |
| 10 | QA, hardening, producción y documentación |
| 11 | Piloto real, bugs críticos y go-live |

## Reglas de ejecución

### 1. Cada módulo empieza por front

Antes de construir un endpoint real, debe existir:

- Pantalla.
- Mock data.
- Estados visuales.
- Validaciones visuales.
- Contrato del payload.
- Contrato de respuesta.
- Estados de error esperados.

### 2. Cada desarrollador es dueño full-stack de sus módulos

No debe funcionar como:

> Dev 1 hace backend y Dev 2 hace frontend.

Debe funcionar como:

> Dev 1 hace completo los módulos administrativos.  
> Dev 2 hace completo los módulos comprador/logística.

### 3. Contratos antes de backend

Los contratos permiten que el front avance sin esperar el backend real.

Ejemplo:

```ts
POST /buyer/orders

{
  client_id: number;
  delivery_type: "SHIP" | "PICKUP";
  items: OrderItem[];
  payment_method: "ONLINE" | "COD";
  address?: AddressPayload;
  pickup_point_id?: number;
}
```

### 4. Cada semana debe cerrar con demo

| Semana | Demo esperada |
|---|---|
| 1 | Apps base levantadas |
| 2 | Admin y tienda navegables |
| 3 | Catálogo, carrito y productos visuales |
| 4 | Flujo visual completo de pedidos |
| 5 | Login y catálogo reales |
| 6 | Pedido real creado |
| 7 | Pago Wompi sandbox |
| 8 | Entrega SHIP completa |
| 9 | PICKUP + dashboards |
| 10 | Producción lista y QA completo |
| 11 | Piloto real validado |

## Definición de terminado por tarea

Una tarea se considera terminada cuando cumple:

- Código implementado.
- UI funcional o endpoint funcionando, según corresponda.
- Contrato actualizado.
- Validaciones implementadas.
- Errores controlados.
- Pruebas mínimas realizadas.
- Sin ruptura de lint/typecheck.
- PR revisado y aprobado.
- Demo funcional si aplica.

## Priorización si hay retrasos

Si el plan se atrasa, se debe priorizar:

1. Auth.
2. Clientes.
3. Productos.
4. Catálogo.
5. Carrito.
6. Checkout.
7. Pedidos.
8. Wompi.
9. Admin pedidos.
10. Courier SHIP.
11. PICKUP.
12. Dashboards.
13. Vista de Control.
14. Webhooks salientes.
15. Observabilidad avanzada.

## Archivos relacionados

- `Plan_RUTA_11_Semanas_2_Desarrolladores.xlsx`: tabla operativa por semana, desarrollador, módulo, tarea, fase, estado, prioridad y estimación.
