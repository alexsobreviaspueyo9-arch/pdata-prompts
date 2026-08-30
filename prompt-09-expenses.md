# Expenses — Prompt para Claude Code

Requiere: haber construido ya `prompt-01-dashboard.md` a `prompt-08-books.md` (trae el sidebar y el sistema de diseño). Necesita **tu propia cuenta de Supabase** (gratuita) — a diferencia del resto de secciones, esta usa su propio proyecto de Supabase independiente, sin pasar por el patrón de sincronización genérico de las demás. Opcional: alguna forma de insertar gastos automáticamente al pagar con tarjeta (ver el punto de integración externa más abajo) — sin eso, puedes seguir usando la sección igualmente añadiendo gastos a mano directamente en la tabla de Supabase o con un pequeño formulario propio.

## Objetivo

Un panel de gastos: un total del mes con un gráfico de barras interactivo para navegar entre meses, desglose por categoría con posibilidad de "entrar" en una categoría para ver sus movimientos concretos, una tendencia de los últimos 6 meses, y una lista editable de transacciones.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`. Prefijo de clases sugerido: `.ex-*`. Los gráficos de barras de esta sección siguen un patrón consistente: barras planas en gris, y la barra seleccionada/actual con el degradado naranja de acento — usa un `id` de degradado SVG distinto por cada gráfico que tengas visible a la vez en pantalla (si dos gráficos comparten el mismo `id` de `<linearGradient>` mientras ambos son visibles, el navegador puede aplicar el color equivocado a uno de los dos).

## Añadir al sidebar

Añade este item al `.sidebar-nav` ya existente, justo debajo de "Books" — el último de la lista:
```html
<button type="button" class="sidebar-item" data-view="expensesView">
  <span class="sidebar-item-icon" aria-hidden="true"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="5" width="20" height="14" rx="2.5"/><path d="M2 10h20"/><path d="M6 15h4"/></svg></span>
  Expenses
</button>
```
Y un `<div id="expensesView" class="hidden">` nuevo dentro de `#appRoot`, con el contenido de esta sección.

## Configuración de Supabase (obligatoria para esta sección)

1. Crea un **proyecto nuevo y separado** en [supabase.com](https://supabase.com) (no reutilices el mismo proyecto que uses para sincronizar el resto de la app, aunque técnicamente podrías — mantenerlo aparte hace más fácil compartir/revocar el acceso de una automatización externa sin tocar el resto de tus datos).
2. En el editor SQL:
   ```sql
   create table if not exists public.expenses (
     id         bigint generated always as identity primary key,
     created_at timestamptz not null default now(),
     merchant   text,
     amount     numeric not null,
     category   text
   );
   alter table public.expenses enable row level security;
   create policy "expenses_select" on public.expenses for select using (true);
   create policy "expenses_insert" on public.expenses for insert with check (true);
   create policy "expenses_update" on public.expenses for update using (true);
   ```
3. Copia tu **Project URL** y tu **clave publishable** (Settings → API) en constantes propias de esta sección (`EX_SUPABASE_URL` / `EX_SUPABASE_KEY`), distintas de las que uses para el resto de la app si sincronizas otras secciones.
4. **Detalle de autenticación importante**: si tu clave es del tipo nuevo `sb_publishable_...` (no un JWT clásico), manda **solo** la cabecera `apikey` en cada petición — añadir además `Authorization: Bearer ...` rompe la petición con este tipo de clave. Compruébalo tú mismo con una petición de prueba antes de dar por hecho que el patrón antiguo de doble cabecera sigue aplicando.

## Estructura visual

1. **Tarjeta "Analytic Card"** — cabecera con una etiqueta pequeña "Total gastado" y el número grande del total del mes seleccionado, una píldora "Mensual" arriba a la derecha y el nombre del mes debajo. Debajo, un gráfico de 7 barras (una por mes, los últimos 7 meses) donde **las propias barras son el control de navegación** — no hay flechas de "anterior/siguiente" por separado, tocar una barra o su etiqueta selecciona ese mes. La barra seleccionada lleva el degradado naranja de arriba a transparente; encima, un marcador flotante (círculo + línea) con un tooltip mostrando el total y la categoría con más gasto ese mes. Por defecto, el mes actual (la barra más a la derecha) está seleccionado al cargar.
2. **Por categoría** — un gráfico de barras de categorías fijas que definas tú mismo (p. ej. Comida, Transporte, Ocio, Compras, Suscripciones, Otros — cualquier gasto sin categoría reconocida cae en "Otros"), y debajo una cuadrícula de mini-tarjetas, una por categoría, cada una con un color propio (esta es la única zona de la app donde varias categorías llevan colores distintos entre sí, en vez del acento naranja único de siempre — aquí el color codifica información, así que está justificado). Tocar una mini-tarjeta abre un desglose (en el mismo sitio, sustituyendo la cuadrícula) con las transacciones de esa categoría en el mes seleccionado, y un botón "‹ volver" para cerrar el desglose.
3. **Últimos 6 meses** — el mismo tipo de gráfico de barras, un mes por barra, solo el mes actual resaltado, para ver la tendencia de un vistazo.
4. **Transacciones** — lista cronológica del mes seleccionado. La categoría de cada fila es un `<select>` nativo estilizado como una píldora de color (más simple y más accesible que reinventar un desplegable propio); cambiarla actualiza la fila al momento contra Supabase.
5. **Estados de carga/vacío/error**, tres mensajes distintos, no uno genérico: tabla completamente vacía todavía ("Añade un gasto para verlo aquí"), mes seleccionado sin movimientos ("Sin transacciones en [mes]"), y fallo de red/petición — así una caída real nunca se confunde con "no has gastado nada".

## Funcionalidad e interacciones

- No hay suscripción en tiempo real para esta sección (a diferencia de las que sincronizan por el patrón genérico) — simplemente vuelve a pedir los datos cada vez que se entra en esta sección desde la navegación, que es suficiente para reflejar lo que se haya añadido desde fuera desde la última visita.
- Cambiar de mes en la Analytic Card cierra cualquier desglose de categoría que estuviera abierto (no intentes mantenerlo abierto contra datos de un mes distinto).

## Integración externa: añadir gastos automáticamente (opcional)

La forma más simple de que esta tabla se rellene sola es una automatización disparada por tu propio banco/tarjeta — por ejemplo, un atajo de iPhone (Atajos / Shortcuts) enganchado a una notificación de pago, que te pregunte importe/comercio/categoría y haga un `POST` directo a la API REST de Supabase:

```
POST https://TU-PROYECTO.supabase.co/rest/v1/expenses
Headers: apikey: TU_CLAVE, Content-Type: application/json, Prefer: return=minimal
Body: { "merchant": "...", "amount": 12.50, "category": "Comida" }
```

Cualquier automatización equivalente en Android o con otro banco sirve igual — lo único que importa es que acabe insertando una fila con esa forma en la tabla `expenses`. Si no quieres montar ninguna automatización, un formulario sencillo dentro de la propia sección que haga el mismo `POST` funciona igual de bien para empezar.

## Estructura de datos

```sql
expenses: id (bigint), created_at (timestamptz), merchant (text), amount (numeric), category (text)
```

## Notas de implementación

- No reutilices las credenciales de Supabase del resto de la app para esta sección — son un proyecto distinto a propósito, para poder compartir o revocar el acceso de la automatización externa sin tocar el resto de tus datos sincronizados.
- No hardcodees ninguna cifra, comercio ni categoría real de nadie en el código publicado.
- Los gráficos de barras son SVG dibujado a mano, sin librerías externas.
