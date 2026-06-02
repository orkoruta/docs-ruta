# Guía de landing custom (CUSTOM_LANDING_BY_RUTA)

Qué puede y qué no puede hacer una landing custom de RUTA.

---

## Qué es una landing custom

Una landing custom es un sitio web con el branding propio de un Cliente Full.
El comprador ve la marca del Cliente, no la de RUTA. Técnicamente es un repo
Next.js independiente (`landing-{slug}`) que consume la misma API del backend de RUTA.

---

## Qué SÍ puede hacer la landing

| Capacidad | Implementación |
|---|---|
| Catálogo de productos del cliente | `GET /buyer/products` |
| Registro e inicio de sesión de compradores | `POST /auth/register`, `POST /auth/login` |
| Carrito de compras | Estado local (no persiste en BD) |
| Checkout: crear pedido con SHIP o PICKUP | `POST /buyer/orders` |
| Ver estado de mis pedidos | `GET /buyer/orders`, `GET /buyer/orders/:id` |
| Solicitar devolución | `POST /buyer/orders/:id/return` |
| Abrir disputa | `POST /buyer/orders/:id/dispute` |
| Configurar pedido recurrente | `POST /buyer/orders/:id/recurrence` |
| Ver y gestionar mis plantillas recurrentes | `GET /buyer/recurrence`, pause/resume/cancel |
| Pago online (Wompi) | Redirige al widget de Wompi del cliente |
| Pago contra entrega (COD) | Flujo normal de pedido COD |
| Branding totalmente personalizado | Logo, colores, tipografías propias |

---

## Qué NO puede hacer la landing

| Restricción | Motivo |
|---|---|
| Mostrar datos de otros clientes | Aislamiento multi-tenant estricto |
| Acceder al panel admin | Admin es exclusivo de `ruta-frontend/admin` |
| Crear pedidos corporativos (Flujo 6) | Solo disponible desde el panel admin |
| Mover o acreditar dinero directamente | RUTA no custodia fondos |
| Usar componentes `@orkoruta/ui` | La landing tiene su propio design system |
| Publicar en GitHub Packages | `@orkoruta/ui` no se publica externamente |
| Exponer la marca RUTA visualmente | El comprador solo ve la marca del cliente |

---

## Proceso de creación

Ver la guía técnica completa en:
`infra-ruta/docs/crear_landing_custom.md`

Resumen del proceso:
1. `bash infra-ruta/scripts/create_landing.sh <slug>` — crea el repo.
2. Configurar `.env.local` con `NEXT_PUBLIC_API_URL` y `NEXT_PUBLIC_CLIENT_SLUG`.
3. Aplicar branding del cliente (Tailwind, logo, layout).
4. Implementar las páginas conectándolas con la API de RUTA.
5. Probar el flujo completo en local.
6. `pnpm typecheck && pnpm build` — ambos deben dar EXIT 0.
7. Deploy en el deploy final de Fase 3 (junto con todos los repos).

---

## Estructura de un repo de landing

```
landing-{slug}/
├── src/
│   ├── app/
│   │   ├── (auth)/          # login y registro
│   │   ├── cart/            # carrito
│   │   ├── checkout/        # checkout (+ toggle recurrencia)
│   │   ├── orders/
│   │   │   ├── page.tsx     # mis pedidos
│   │   │   └── [id]/        # detalle (reembolso / devolución / disputa)
│   │   ├── product/[id]/    # detalle de producto
│   │   ├── recurrence/      # mis plantillas recurrentes
│   │   └── page.tsx         # home / catálogo
│   └── lib/
│       └── api_client.ts    # helper HTTP autenticado
├── public/                  # logo, favicon, imágenes
├── tailwind.config.ts       # colores y tipografías del cliente
└── .env.local               # NEXT_PUBLIC_API_URL, NEXT_PUBLIC_CLIENT_SLUG
```

---

## Variables de entorno requeridas

| Variable | Ejemplo | Descripción |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | `https://api.ruta.com.co` | URL del backend de RUTA |
| `NEXT_PUBLIC_CLIENT_SLUG` | `restaurante-el-prado` | Slug del cliente en BD |

---

## Preguntas frecuentes

**¿Puedo reutilizar componentes entre landings?**
No directamente — cada landing es un repo independiente. Si hay componentes
reutilizables, considera publicarlos como un paquete privado del cliente.

**¿La landing comparte sesión con el storefront de RUTA?**
No. La sesión (cookies) es por dominio. Un comprador que inicia sesión en la
landing no tiene sesión en `storefront.ruta.com.co` y viceversa.

**¿Cómo manejo imágenes de productos?**
La landing obtiene `image_url` del producto vía la API. El almacenamiento de
imágenes es responsabilidad del Cliente (o se define en Fase 3 Bloque de file storage).

**¿Qué pasa si el cliente quiere un checkout personalizado?**
El checkout puede tener el diseño que quieras, pero la creación del pedido
debe llamar a `POST /buyer/orders` de la API de RUTA para garantizar el
state machine y la auditoría.
