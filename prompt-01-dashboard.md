# Dashboard — Prompt para Claude Code

Requiere: un `index.html` base de una sola página, sin build, con una barra de navegación lateral (sidebar) ya montada con al menos un item "Dashboard" que muestra/oculta este bloque. Si no tienes eso todavía, pide primero a Claude Code que monte ese esqueleto (sidebar fijo a la izquierda en escritorio, cajón deslizante en móvil, un `<div>` por sección que se muestra/oculta según el item de sidebar activo) usando el sistema de diseño de abajo, y luego aplica este prompt.

## Objetivo

La pantalla de inicio de un dashboard personal: un vistazo rápido al día (frase motivadora rotativa a modo de "ticker", un anillo circular que marca qué parte del día ha pasado, una lista de tareas/objetivos de hoy) y al trimestre en curso (una lista de objetivos trimestrales, separada de la de hoy).

## Sistema de diseño base (aplícalo tal cual, en todo el archivo)

- **Tema oscuro.** `html { background: #0a0a0a; }`. Fuente del sistema: `-apple-system, BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, Helvetica, Arial, sans-serif`. Fuente monoespaciada para números/horas: `ui-monospace, "SF Mono", Menlo, Consolas, monospace`.
- **Colores de texto**: `--text-primary:#FAFAFA`, `--text-secondary:#B8B6B0`, `--text-tertiary:#76746E`.
- **Colores semánticos** (nunca los sustituyas por el acento naranja — llevan información real: aciertos, avisos, fallos): `--success:#6BE3A4`, `--warning:#F2C063`, `--danger:#FF6B6B`.
- **Acento cálido** (el único color vivo de la app): `--warm-accent:#FF5C1A`, `--warm-accent-2:#FF7A33` (más claro, para líneas de gráfico/texto pequeño), `--warm-accent-deep:#E8491D`, `--warm-glow: rgba(255,92,26,0.28)`, `--action-grad: linear-gradient(135deg, #FF7A33 0%, #E8491D 100%)` (degradado de todos los botones principales).
- **Fondo ambiental**: en `<body>`, un `::before` fijo con 2-3 `radial-gradient` grandes muy difuminados (`filter: blur(90px)`) en naranja tenue sobre negro, con una animación lentísima (36s, `ease-in-out infinite alternate`) que los desplaza un poco; y un `::after` con grano sutil (mosaico de 3×3px, un punto blanco al 1.4% de opacidad por celda).
- **Tarjetas "glass"**: `background: rgba(255,255,255,0.05)`, `backdrop-filter: blur(20px) saturate(1.3)`, `border: 1px solid rgba(255,255,255,0.08)`, `border-radius: 22px`, `padding: 20px`, sombra `inset 0 1px 0 rgba(255,255,255,0.06), 0 8px 32px rgba(0,0,0,0.4)`. Al hover: fondo `rgba(255,255,255,0.07)`, borde con tinte naranja `rgba(255,122,51,0.2)`.
- **Botón principal**: sin borde, `background: var(--action-grad)`, texto blanco, `border-radius:999px`, `box-shadow: inset 0 1px 0 rgba(255,255,255,0.25), 0 4px 16px rgba(0,0,0,0.4), 0 0 24px 2px var(--warm-glow)` (halo difuminado alrededor). Botón secundario: fondo `rgba(255,255,255,0.04)`, borde `1px solid rgba(255,255,255,0.06)`, sin glow.
- **Título de sección**: `<h1>` con degradado vertical de texto (`background: linear-gradient(180deg, #FFFFFF 0%, #C7C4BC 120%)` recortado al texto vía `background-clip:text` + `-webkit-text-fill-color:transparent`), 28px, peso 700, `letter-spacing:-0.025em`.
- **Movimiento**: cualquier animación debe respetar `prefers-reduced-motion: reduce` (desactivada o reducida a un cambio instantáneo).
- **Gráficos/anillos**: sin librerías externas — SVG dibujado a mano. Un anillo de progreso circular es un `<svg>` con dos `<circle>` concéntricos (uno de "pista" en `rgba(255,255,255,0.06)`, uno de "relleno" con `stroke-dasharray`/`stroke-dashoffset` controlados por JS, rotado `-90deg` para que el 0% empiece arriba).

## Estructura visual (de arriba abajo)

1. **Ticker de objetivos** — una tira horizontal tipo "bolsa de valores" que va rotando, una a una cada 5 segundos, las tareas pendientes de hoy (con animación de deslizamiento vertical entrada/salida). Fondo oscuro con una textura sutil de líneas horizontales repetidas (`repeating-linear-gradient`) para dar sensación de panel LED; un punto verde parpadeante a la izquierda (`--success`, con `box-shadow` de glow y una animación de pulso ~1.6s); la palabra "GOALS" en mayúsculas pequeñas; el texto de la tarea actual en mono; y a la derecha un contador `hechas/total` en una píldora pequeña. Si no hay tareas hoy, muestra un mensaje de placeholder ("No hay objetivos para hoy — añade uno"); si están todas hechas, un mensaje de celebración.
2. **Anillo del día** — un anillo SVG circular (unos 168px) que se rellena según qué fracción del día "despierto" ha pasado, con el número de porcentaje grande en el centro, una etiqueta de fase debajo (mañana/mediodía/tarde/noche) y la hora actual en formato 12h. A la derecha, una columna con un mensaje de estado según la fase, el tiempo restante hasta acostarse, y el rango de horas configurado.
3. **Lista de "Hoy"** — una tarjeta con: cabecera (fecha de hoy, número grande de tareas hechas/total, una barra de progreso segmentada con un segmento por tarea que se colorea en verde al completarse, y una racha (streak) de días consecutivos con todo hecho, mostrada como una píldora con un icono de rayo). Debajo, la lista de tareas en sí (checkbox personalizado, texto editable con un clic, botón de "encolar para ahora" opcional, botón de eliminar) y un formulario rápido para añadir una tarea nueva.
4. **Barra de "tiempo restante del trimestre"** — una barra de progreso simple mostrando cuántos días quedan del trimestre natural en curso (Q1/Q2/Q3/Q4), con el porcentaje restante.
5. **Lista de "Objetivos del trimestre"** — la misma estructura de lista que la de "Hoy" (checkbox, editar, encolar, eliminar, arrastrar para reordenar, añadir), pero para objetivos a más largo plazo del trimestre actual, sin fecha de caducidad diaria.

## Funcionalidad e interacciones

- **Persistencia**: todo en `localStorage`, sin necesidad de servidor. Clave por día: `goals:YYYY-MM-DD` → array de `{ text, done, doneAt? }`. Clave del trimestre: `goals:quarter-YYYY-QN` (p. ej. `goals:quarter-2026-Q3`) → mismo formato de array, pero **nunca se mezcla ni se "hace rollover"** junto a las claves diarias (ver más abajo) — sigue viva mientras dure el trimestre.
- **El día "activo" empieza a las 6:00, no a medianoche.** Si son las 2 de la madrugada, todavía cuenta como el día anterior. Calcula la fecha activa restando un día si `now.getHours() < 6`.
- **Rollover al cargar**: cualquier clave `goals:YYYY-MM-DD` (diaria, no la del trimestre) con fecha anterior a la activa: mueve sus tareas no completadas al día de hoy (sin duplicar por texto exacto) y borra la clave antigua.
- **Racha (streak)** en `goal_streak_v1` → `{ count, lastProcessedDate }`. Al cargar, recorre cada día anterior a hoy en orden: si ese día no tenía tareas, no rompe la racha (se ignora); si todas estaban hechas, +1; si alguna quedó sin hacer, la racha vuelve a 0.
- **Anillo del día**: define dos constantes ajustables arriba del todo, p. ej. `WAKE_HOUR = 6` y `SLEEP_HOUR = 22.5` (formato decimal, 22.5 = 22:30). El porcentaje es `(horaActual - WAKE_HOUR) / (SLEEP_HOUR - WAKE_HOUR) * 100`. Antes de `WAKE_HOUR`: anillo vacío, mensaje "Durmiendo". Después de `SLEEP_HOUR`: anillo lleno, aviso de "hora de dormir". Interpola el color del trazo entre 8-9 paradas de una paleta cálida que va de un dorado suave por la mañana a un naranja-rojizo hacia el final del día (sin morados/azules nocturnos si prefieres mantenerlo solo en la familia naranja del resto de la app). Actualiza cada 60 segundos.
- **Barra de progreso segmentada**: un `<div>` por tarea de hoy, en fila, cada uno se ilumina en verde con un pequeño glow al completarse.
- **Fila de tarea** (reutilizada igual en "Hoy" y en "Trimestre"): checkbox personalizado (cuadrado redondeado, se rellena de verde con una animación de "pop" al marcar), texto que se vuelve editable con un clic (Enter confirma, Escape cancela), botón opcional de "encolar" (una marca visual para "quiero hacer esto ya"), botón de eliminar, y arrastrar-para-reordenar (drag & drop nativo del navegador). Las tareas hechas bajan de opacidad y el texto se tacha. Si hay más de 5 tareas, muestra solo las primeras 5 con un botón "Ver N más".
- **Formulario de añadir**: campo de texto + botón "+ Añadir". Opcionalmente, un botón "✨ Pulir" que reescribe el texto de la tarea con una IA antes de guardarlo — si no configuras ninguna clave de API, este botón debe degradar sin más a "añadir tal cual" y mostrar un aviso breve de que hace falta una clave para pulir el texto (no lo dejes roto ni lo escondas del todo).
- **Barra del trimestre**: recalcula al cargar la página: identifica el trimestre natural actual (Q1 = ene-mar, etc.), cuenta días totales y transcurridos, y muestra `días restantes` + `% restante` en una barra simple.

## Estructura de datos

```
localStorage:
  goals:YYYY-MM-DD          → [{ text: string, done: boolean, doneAt?: number }]
  goals:quarter-YYYY-QN     → [{ text: string, done: boolean, doneAt?: number }]
  goal_streak_v1            → { count: number, lastProcessedDate: string|null }
```

No requiere ninguna tabla externa si te quedas solo con `localStorage`. Si quieres sincronizar entre dispositivos, aplica el patrón de Supabase de abajo.

## Sincronización entre dispositivos (opcional)

1. Crea un proyecto gratuito en [supabase.com](https://supabase.com) y ejecuta en el editor SQL:
   ```sql
   create table if not exists public.app_state (
     app_key    text primary key,
     state      jsonb not null default '{}'::jsonb,
     updated_at timestamptz not null default now()
   );
   alter table public.app_state enable row level security;
   create policy "app_state_select" on public.app_state for select using (true);
   create policy "app_state_insert" on public.app_state for insert with check (true);
   create policy "app_state_update" on public.app_state for update using (true);
   alter publication supabase_realtime add table public.app_state;
   ```
2. Sustituye `SUPABASE_URL` y `SUPABASE_KEY` (Settings → API de tu proyecto) por las tuyas — no hay login de usuario, así que trata esa clave como si fuera de un solo usuario, no la publiques en una app abierta al público.
3. Usa el `APP_KEY` `'dashboard'` para esta sección (cada sección de la app ocupa su propia fila).
4. Patrón: intercepta `Storage.prototype.setItem`/`removeItem` para detectar cambios en las claves `goals:*` y `goal_streak_v1`; con un debounce de ~250ms, sube todo el estado sincronizable como un único JSON a esa fila (`upsert`). Al cargar, descarga la fila remota y aplícala; suscríbete a `postgres_changes` de Supabase Realtime sobre esa fila para reflejar cambios hechos en otro dispositivo en tiempo real. Si el usuario está escribiendo en un campo, no apliques cambios remotos hasta que pierda el foco.

## Notas de implementación

- Todo en un único `index.html`, sin build ni dependencias externas salvo, opcionalmente, el SDK de Supabase por CDN.
- Los iconos del sidebar son SVG de trazo simple (`stroke="currentColor"`, grosor 2px, `viewBox="0 0 24 24"`), nunca emoji a color, para heredar el color del texto/estado activo.
- No hace falta ninguna IA para el botón "Pulir" — es un extra opcional, no bloquees el resto de la funcionalidad por su ausencia.
