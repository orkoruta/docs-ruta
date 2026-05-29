# RUTA — Memoria Operativa

Última actualización: 2026-05-29.

## Estado actual

- Sprint 0, 1, 2 y 3: completos.
- Sprint 3 cerró el 2026-05-29 con backend, frontend, integración y QA E2E en `main`.
- No quedan PRs abiertos asociados al Sprint 3.
- Siguiente foco: Sprint 4 — flujo PICKUP.

## Sprint 3

- Backend SHIP mergeado a `main`: asignación de courier, endpoints courier, cobro contra entrega, cancelación post-despacho, return-to-origin y auto-confirmación.
- Frontend mergeado a `main`: vista courier móvil-first y mapa de asignación.
- Integración backend en `api/src/app.ts` completada.
- Ruta operativa esperada del mapa: `/admin/orders/map`.
- `/admin/map` queda como redirect de compatibilidad.

## QA E2E

- Playwright agregado en `frontend-ruta`.
- Suite: `tests/e2e/ship_full_flow.spec.ts`.
- CI de frontend-ruta ejecuta `pnpm exec playwright test --project=chromium`.
- PR frontend-ruta #16 mergeado: https://github.com/orkoruta/frontend-ruta/pull/16
- Cobertura E2E Sprint 3 en CI: asignación de courier, entrega COD, intento fallido y pedido confirmado por sistema.

## Próximo paso

1. Abrir Sprint 4 — flujo PICKUP.
2. Mantener Playwright en CI para los flujos críticos nuevos.
3. Si se corre E2E localmente, instalar dependencias nativas con `pnpm exec playwright install-deps chromium` en un entorno con `sudo`.
