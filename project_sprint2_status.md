# Sprint 2 Status

Última actualización: 2026-05-28. **Sprint 2 COMPLETO — todas las tareas mergeadas a main.**

## Resumen

| Tarea | Estado | PR | Repo | Notas |
|---|---|---|---|---|
| `2.SHARED-1` | Mergeado | PR #3 | packages-ruta | `@orkoruta/shared@1.2.0` publicado. Tests 50/50 OK. |
| `2.BACK-1` | Mergeado | — | backend-ruta | Orders + State Machine. Merge directo a main. |
| `2.BACK-2` | Mergeado | PR #7 | backend-ruta | Validación operativa y aceptación. |
| `2.BACK-3` | Mergeado | — | backend-ruta | Wompi + payments + webhook. |
| `2.BACK-4` | Mergeado | PR #5 | backend-ruta | Jobs de mantenimiento. |
| `2.STORE-1` | Mergeado | PR #10 | frontend-ruta | Carrito persistido en BD. |
| `2.STORE-2` | Mergeado | PR #12 | frontend-ruta | Checkout multi-paso. |
| `2.STORE-3` | Mergeado | PR #11 | frontend-ruta | Mis pedidos BUYER. |
| `2.STORE-4` | Mergeado | PR #13 | frontend-ruta | Confirmación post-Wompi. |
| `2.ADMIN-1` | Mergeado | PR #9 | frontend-ruta | Lista y detalle de pedidos admin. |
| `2.QA-1` | Mergeado | PR #8 | backend-ruta | Tests state machine + Wompi. 3662 passed, 1 skipped. |

## Criterio de cierre ✓

Pedido real con Wompi sandbox completado de extremo a extremo hasta READY_TO_SHIP o READY_FOR_PICKUP. Todas las tareas del sprint mergeadas a main.

## Siguiente: Sprint 3 — Flujo SHIP completo

Tareas pendientes:
- `3.BACK-1` — Asignación de Repartidor (optimistic lock)
- `3.BACK-2` — Endpoints COURIER (transiciones propias)
- `3.BACK-3` — Cobro contra entrega (evidencia multipart)
- `3.BACK-4` — Cancelación post-despacho
- `3.BACK-5` — RETURN_TO_ORIGIN
- `3.BACK-6` — Auto-confirmación de entregados
- `3.ADMIN-1` — Mapa de asignación (Leaflet + OSM)
- `3.ADMIN-2` — Vista COURIER móvil-first
- `3.QA-1` — E2E flujo SHIP (Playwright)
