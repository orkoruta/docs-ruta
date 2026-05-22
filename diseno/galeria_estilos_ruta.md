# Ruta — Galería de Estilos

> Documento único que consolida la guía de estilos visual y el prompt de IA
> para diseño de pantallas en la plataforma **Ruta**. Reemplaza a
> `guia_estilos_ruta.md` y `ui_skill_ruta.md`.

---

## 0. Cómo usar este documento

Este archivo cumple dos funciones:

1. **Catálogo / referencia visual.** Cuando alguien (humano o IA) va a
   crear una nueva pantalla o componente para Ruta, debe revisar este
   documento primero para entender la personalidad visual, los tokens
   exactos y las restricciones.
2. **Prompt de IA.** Cuando se le pida a una IA (Claude, ChatGPT,
   Copilot, v0, etc.) que genere UI para Ruta, este documento se debe
   adjuntar como contexto o referenciar explícitamente para garantizar
   coherencia visual.

Antes de crear cualquier pantalla nueva, **revisar la galería de
estilos y reutilizar los patrones definidos aquí**. No reinventar
componentes si ya existe uno oficial (sección 8).

---

## 1. Contexto del producto

**Ruta** es una aplicación SaaS para gestión de pedidos, rutas y
entregas. Empresas (Clientes) usan Ruta para que sus Compradores
registren pedidos, consulten estados y coordinen entregas o recogidas.

> **Ruta NO es una tienda online.** Es una plataforma operativa,
> logística y administrativa.

Esta distinción es crítica: el estilo visual debe reflejarlo. Nada de
estética retail llamativa; sí estética SaaS empresarial corporativa.

---

## 2. Personalidad visual

### 2.1 Debe sentirse

- Serio
- Operativo
- Corporativo
- Limpio
- Grisáceo
- Confiable
- SaaS empresarial
- Logística, no e-commerce retail

### 2.2 No debe sentirse

- Fantasiosa
- Gaming
- Excesivamente futurista
- Saturada de neón
- Infantil
- Demasiado redondeada
- Visualmente ruidosa

---

## 3. Sistema visual base

### 3.1 Estilo principal

> **Glassmorphism sutil + cards rectangulares + tonos grisáceos +
> colores funcionales por estado.**

### 3.2 Reglas visuales generales

- Usar fondos grisáceos, `slate`, `zinc` o `neutral`.
- En modo dark, fondo tipo `#111214`, **nunca negro puro**.
- En modo light, fondo tipo `#f3f4f6`, **nunca blanco plano absoluto**.
- Cards rectangulares con bordes suaves, no píldoras.
- Usar `rounded-md` o `rounded-lg` como base.
- Evitar `rounded-3xl` salvo casos muy puntuales.
- Glassmorphism sutil: transparencia ligera, blur suave, borde fino.
- Evitar brillos fuertes, halos exagerados o degradados muy llamativos.

---

## 4. Modos de color

### 4.1 Modo claro

**Sensación:** limpio, corporativo, administrativo, fácil de leer.

**Base recomendada (CSS):**

```css
background: #f3f4f6;
surface:    rgba(255, 255, 255, 0.76);
border:     rgba(203, 213, 225, 0.9);
text:       #0f172a;
muted:      #64748b;
```

**Uso:**

- Fondo general gris claro.
- Cards blancas / translúcidas.
- Bordes sutiles slate.
- Sombras suaves.
- Buen contraste en formularios y tablas.

### 4.2 Modo oscuro

**Sensación:** serio, profesional, real, sobrio.

**Base recomendada (CSS):**

```css
background: #111214;
surface:    rgba(29, 32, 37, 0.78);
border:     rgba(255, 255, 255, 0.10);
text:       #f1f5f9;
muted:      #94a3b8;
```

**Uso:**

- Fondo gris carbón, no negro puro.
- Cards en gris oscuro con transparencia controlada.
- Bordes blancos con opacidad baja.
- Sombras negras suaves.
- Colores funcionales sin neón exagerado.

---

## 5. Colores funcionales

Los colores **no se usan por decoración**, sino por significado
operativo. Cada color tiene una intención específica.

| Color    | Uso |
|----------|-----|
| Azul     | Acción principal, en tránsito, asignar ruta, buscar pedido |
| Verde    | Entregado, completado, confirmado, éxito |
| Ámbar    | Pendiente, en espera, advertencia, pago pendiente |
| Rojo     | Cancelado, error, acción destructiva |
| Violeta  | Validado, acción secundaria especial |
| Gris     | Borrador, neutro, filtro, exportar, navegación |

---

## 6. Estados de pedido

Cada estado del pedido se representa visualmente con el color funcional
correspondiente:

| Estado     | Color    |
|------------|----------|
| Borrador   | Gris     |
| Validado   | Violeta  |
| En tránsito| Azul     |
| En espera  | Ámbar    |
| Entregado  | Verde    |
| Cancelado  | Rojo     |

Los estados deben representarse como **botones, pills o badges
rectangulares** con el color funcional correspondiente.

> Para la lista completa de estados internos del pedido (50+), ver el
> catálogo en `docs/bd/ruta_postgres.sql` (tabla `state_catalog`). La
> tabla de arriba es la simplificación visual recomendada para
> dashboards.

---

## 7. Componentes UI

### 7.1 Botones

**Reglas:**

- Rectangulares con `rounded-md`.
- Bordes visibles.
- Texto claro y directo.
- Ícono a la izquierda cuando ayude a reconocer la acción.
- Hover sutil: leve sombra o desplazamiento pequeño.
- No usar efectos exagerados.

**Acciones rápidas oficiales:**

| Acción         | Color           |
|----------------|-----------------|
| Asignar ruta   | Azul sólido     |
| Filtrar        | Gris suave      |
| Ver detalle    | Violeta suave   |
| Confirmar      | Verde suave     |
| Cancelar       | Rojo suave      |
| En espera      | Ámbar suave     |
| Reprogramar    | Violeta suave   |
| Exportar       | Gris suave      |

### 7.2 Cards

Los **cards** son los contenedores principales para:

- Métricas
- Pedido activo
- Resumen de pedido
- Timeline
- Ruta del día
- Formularios
- Tablas de operación

**Reglas:**

- Rectangulares.
- Borde fino.
- Fondo semitransparente.
- Sombra sutil.
- Padding cómodo.

### 7.3 Agrupadores

Los **agrupadores** separan bloques funcionales:

- Estados de pedido
- Acciones rápidas
- Crear / buscar pedido
- Timeline
- Resumen
- Operación actual

**Cada agrupador debe tener:**

- Título claro.
- Subtítulo pequeño en mayúscula y tracking amplio.
- Contenido organizado en grid o flex.

### 7.4 Tipografía

- Fuente sans-serif moderna: `Inter` o `system-ui`.
- Títulos fuertes con `font-black` o `font-bold`.
- Subtítulos pequeños en uppercase y tracking amplio.
- Textos secundarios con gris suave.
- Evitar textos decorativos.

**Ejemplo (TSX):**

```tsx
<p className="text-[11px] font-bold uppercase tracking-[0.24em] text-slate-500">
  agrupador rectangular
</p>
<h2 className="text-base font-bold tracking-tight">Estados de pedido</h2>
```

---

## 8. Inventario de componentes reutilizables

En la app real, la galería se traduce en componentes reutilizables.
Ninguna pantalla nueva debe reinventar estos estilos si ya existe el
componente oficial.

| Componente            | Propósito |
|-----------------------|-----------|
| `RutaCard`            | Contenedor base rectangular con glassmorphism sutil |
| `RutaButton`          | Botón rectangular estándar |
| `RutaStatusButton`    | Botón con estado funcional (color por intención) |
| `RutaPill`            | Pill rectangular para tags y estados compactos |
| `RutaSectionHeader`   | Encabezado de sección con título + subtítulo uppercase |
| `RutaMetricCard`      | Card para mostrar una métrica con número grande |
| `RutaThemeToggle`     | Toggle entre modo claro y oscuro |
| `RutaSidebar`         | Barra lateral con navegación |
| `RutaHeader`          | Cabecera superior con acciones globales |
| `RutaTimeline`        | Línea de tiempo vertical para historial de pedido |
| `RutaOrderSummary`    | Resumen lateral / inferior de un pedido |

---

## 9. Restricciones (qué NO hacer)

No crear interfaces con:

- Neón exagerado.
- Estética gaming.
- Degradados fuertes.
- Bordes demasiado redondeados (`rounded-3xl`, `rounded-full` como
  patrón principal).
- Botones tipo píldora como patrón principal.
- Colores sin significado operativo.
- Iconografía decorativa sin propósito.
- Animaciones llamativas o "wow".
- Tipografías display, manuscritas o serif decorativas.
- Fondos negros puros (`#000000`) o blancos planos absolutos
  (`#ffffff`).

---

## 10. Reglas técnicas obligatorias (Tailwind)

Para que el estilo se vea **exactamente igual** en la app real, las
opacidades que no existen en la escala estándar de Tailwind deben
escribirse con sintaxis arbitraria (corchetes).

| Forma | Correcto | Incorrecto |
|-------|----------|------------|
| Color con opacidad arbitraria | `bg-sky-500/[0.12]` | `bg-sky-500/12` |
| Blanco con opacidad arbitraria | `bg-white/[0.76]` | `bg-white/76` |
| Hex con opacidad | `bg-[#1d2025]/[0.78]` | `bg-[#1d2025]/78` |

**Por qué es crítico:** si se usa una opacidad no estándar **sin
corchetes**, Tailwind no genera el CSS y los fondos desaparecen
(especialmente notorio en modo dark, donde los botones quedan
"fantasma" sin color).

### 10.1 Fondos base exactos

```tsx
light: {
  page:  "bg-[#f3f4f6] text-slate-950",
  shell: "bg-white/[0.72] border-slate-200/80",
  card:  "bg-white/[0.76] border-slate-200/90",
  input: "bg-white/[0.85] border-slate-200"
}

dark: {
  page:    "bg-[#111214] text-slate-100",
  shell:   "bg-[#181a1e]/[0.78] border-white/10",
  sidebar: "bg-[#17191d]/[0.82] border-white/10",
  card:    "bg-[#1d2025]/[0.78] border-white/10",
  input:   "bg-white/[0.055] border-white/10"
}
```

### 10.2 Tokens exactos de botones / pills en modo dark

```tsx
blue:   "bg-sky-500/[0.12] text-sky-300 border-sky-400/25"
green:  "bg-emerald-500/[0.12] text-emerald-300 border-emerald-400/25"
amber:  "bg-amber-500/[0.12] text-amber-300 border-amber-400/25"
red:    "bg-rose-500/[0.12] text-rose-300 border-rose-400/25"
violet: "bg-violet-500/[0.12] text-violet-300 border-violet-400/25"
slate:  "bg-white/[0.06] text-slate-300 border-white/10"
```

---

## 11. Prompt para IA

Cuando se use una IA (Claude, ChatGPT, Copilot, v0, etc.) para
construir pantallas nuevas de Ruta, incluir este prompt como
instrucción de sistema o contexto:

```text
Actúa como diseñador UI/UX senior y frontend developer para la app
Ruta. Ruta es un SaaS de gestión de pedidos, rutas y entregas; no es
una tienda online.

Usa el design system de Ruta definido en galeria_estilos_ruta.md.

La interfaz debe ser:
- Seria, grisácea, corporativa y operativa.
- Con glassmorphism sutil.
- Con cards rectangulares (rounded-md / rounded-lg).
- Con botones rectangulares.
- Con colores funcionales por estado:
    - Azul: acción principal, en tránsito.
    - Verde: éxito, entregado, confirmado.
    - Ámbar: en espera, pendiente, advertencia.
    - Rojo: cancelado, error, destructivo.
    - Violeta: validado, acción secundaria especial.
    - Gris: borrador, neutro, filtros, exportar.
- En modo dark: fondo gris carbón (#111214), no negro puro.
- En modo light: fondo gris claro (#f3f4f6), no blanco plano.

No usar:
- Neón exagerado, estética gaming, degradados fuertes.
- rounded-3xl ni botones tipo píldora como patrón principal.
- Colores sin significado operativo.

Reutilizar los componentes oficiales si ya existen:
RutaCard, RutaButton, RutaStatusButton, RutaPill, RutaSectionHeader,
RutaMetricCard, RutaTimeline, RutaOrderSummary, RutaThemeToggle,
RutaSidebar, RutaHeader.

Para Tailwind, usar siempre sintaxis arbitraria de opacidad cuando el
valor no esté en la escala estándar:
- Correcto: bg-sky-500/[0.12], bg-white/[0.76], bg-[#1d2025]/[0.78]
- Incorrecto: bg-sky-500/12, bg-white/76, bg-[#1d2025]/78
```

---

## 12. Resultado esperado

Cada pantalla generada con esta guía debe parecer **parte de una
plataforma SaaS real de logística, pedidos y entregas**:

- Que un Cliente pueda usarla 8 horas seguidas sin fatiga visual.
- Que un Operador pueda identificar el estado de un pedido en menos de
  un segundo (por el color funcional).
- Que un Repartidor pueda usar la versión móvil sin distracciones.
- Que un Administrador del Cliente sienta que está usando una
  herramienta empresarial, no una app de retail.
- Que un Administrador RUTA pueda hacer soporte transversal con
  consistencia visual entre Clientes.

---

## Referencias cruzadas

- **Descripción funcional general:** `docs/all_ruta.md`
- **Estrategia multi-tenant:**
  `docs/arquitectura/estrategia_multi_tenant_ruta.md`
- **Modelo de datos:** `docs/bd/ruta_postgres.sql`
- **Estados del pedido (catálogo completo):** tabla `state_catalog` en
  `docs/bd/ruta_postgres.sql`
- **Flujos de proceso (1-7):** `docs/flujos/flujo_*.txt`
