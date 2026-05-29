# RUTA — Memoria Operativa

Última actualización: 2026-05-29.

## Estado actual

- Sprint 0, 1 y 2: completos.
- Sprint 3: Wave 1 backend y Wave 2 frontend completas; QA E2E en curso.
- PR abierto frontend-ruta #16: base Playwright, suite inicial SHIP y corrección de ruta canónica del mapa.
- PR abierto docs-ruta #6: actualización documental de Sprint 3.

## Sprint 3

- Backend SHIP mergeado a `main`: asignación de courier, endpoints courier, cobro contra entrega, cancelación post-despacho, return-to-origin y auto-confirmación.
- Frontend mergeado a `main`: vista courier móvil-first y mapa de asignación.
- Integración backend en `api/src/app.ts` completada.
- Ruta operativa esperada del mapa: `/admin/orders/map`.
- `/admin/map` queda como redirect de compatibilidad en PR frontend-ruta #16.

## QA E2E

- Playwright agregado en `frontend-ruta`.
- Suite inicial: `tests/e2e/ship_full_flow.spec.ts`.
- `pnpm exec playwright test --list` detecta 4 tests.
- Ejecución local de Chromium bloqueada por `libnspr4.so`.
- Para desbloquear: instalar dependencias con `pnpm exec playwright install-deps chromium` en un entorno con `sudo`.

## Próximo paso

1. Mergear frontend-ruta #16 y docs-ruta #6 cuando CI/review estén listos.
2. Instalar dependencias nativas de Playwright en el runner/local.
3. Ejecutar `pnpm exec playwright test --project=chromium`.
4. Completar `3.QA-1` con happy path real y dos flujos alternos.
