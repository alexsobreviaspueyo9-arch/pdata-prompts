# Meal Prep — Prompt para Claude Code

Requiere: haber construido ya `prompt-01-dashboard.md`, `prompt-02-gym.md` y `prompt-03-kitchen.md` (trae el sidebar y el sistema de diseño). No necesita ninguna cuenta externa — esta sección es intencionadamente 100% local, sin sincronización, porque el plan es fijo y personal.

## Objetivo

Guiar en vivo una sesión de batch-cooking (cocinar de golpe para varios días) con un cronómetro general, temporizadores por tarea, y una vista que deja claro qué se está cocinando en paralelo (horno + varios fuegos + ollas a la vez) para no perder de vista nada mientras tienes las manos ocupadas.

## Sistema de diseño base

Mismo sistema que `prompt-01-dashboard.md`, pero con un matiz: como esto se usa mientras se cocina (manos mojadas/ocupadas), todo el texto y los botones deben ser un punto más grandes de lo habitual en el resto de la app, priorizando que se lean y se toquen de un vistazo rápido por encima de la densidad visual. Prefijo de clases sugerido: `.mp-*`.

## Añadir al sidebar

Añade este item al `.sidebar-nav` ya existente, justo debajo de "Kitchen":
```html
<button type="button" class="sidebar-item" data-view="mealPrepView">
  <span class="sidebar-item-icon" aria-hidden="true"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M6 13a4 4 0 0 1 1-7.87A4.5 4.5 0 0 1 12 3a4.5 4.5 0 0 1 5 2.13A4 4 0 0 1 18 13"/><path d="M6 13v6h12v-6"/><path d="M8 19h8"/></svg></span>
  Meal Prep
</button>
```
Y un `<div id="mealPrepView" class="hidden">` nuevo dentro de `#appRoot`, con el contenido de esta sección.

## 0. Datos base (hardcodea tu propio plan aquí — no hace falta ninguna base de datos)

Define 2 (o los días que quieras) "planes de día", cada uno como una lista plana de tareas con: minuto de inicio relativo al comienzo de la sesión, nombre, duración en minutos (o `null` si es una tarea instantánea sin temporizador — p. ej. "meter algo al horno" es instantánea, "que se haga 20 minutos" no), el "carril"/recurso que ocupa (ver punto 2), y opcionalmente una nota de "mientras tanto puedes...".

**Estructura de ejemplo** (sustitúyela por tu propio plan real de batch-cooking):

```
Día A — minuto 0:
  Min 0  — Precalentar horno (sin timer)                      [horno]
  Min 0  — Poner agua a hervir para X                          [fuego]
  Min 5  — Meter X al agua hirviendo — timer 15 min             [fuego]  · mientras tanto: cortar verduras
  Min 8  — Bandeja de verduras al horno — timer 20 min          [horno]
  Min 10 — Dorar proteína 1 en sartén — timer 12 min            [sartén]
  Min 22 — Sacar X del agua (sin timer)                         [fuego]
  Min 22 — Dorar proteína 2 en la misma sartén — timer 10 min   [sartén]
  Min 28 — Sacar verduras del horno (sin timer)                 [horno]
  Min 45 — Repartir todo en tarteras (sin timer)                [prep]
  Resultado final: [lista de platos que quedan listos]

Día B — minuto 0: (repite la misma estructura para tu segunda sesión de la semana)
```

También define una lista aparte, estática, de "no se prepara con antelación" (cosas que siempre se cocinan el mismo día, sin temporizador ni checklist — p. ej. pescado fresco, tortillas de desayuno) — se muestra igual bajo ambos días.

## Estructura visual

1. **Selector de día** arriba, tipo píldora (Día A / Día B).
2. **Botón grande "Empezar sesión"** — arranca el cronómetro general de la sesión (minuto 0). A partir de ahí, cada tarea con temporizador se activa sola en cuanto la sesión llega a su minuto programado.
3. **Vista de línea de tiempo con carriles paralelos** — la pieza central. Un Gantt horizontal con una fila ("carril") por recurso físico (p. ej. Horno / Fuego / Sartén / Olla / Preparación manual — ajusta el número de carriles según cuántas tareas de tu propio plan se solapan de verdad en el tiempo; no asumas que "Fuego" y "Sartén" son siempre lo mismo si en tu plan usas dos quemadores a la vez). Cada tarea es un bloque horizontal posicionado según su minuto de inicio, con un ancho proporcional a su duración, dentro del carril que le toca. Una línea vertical fina se mueve en tiempo real marcando el minuto actual de la sesión. Colores: gris oscuro para pendiente, degradado naranja con glow para la tarea en curso, tachado y atenuado para completada. Las tareas instantáneas (sin duración) se representan como un marcador pequeño de ancho fijo en vez de una barra proporcional.
4. **Panel de "en curso ahora"**, siempre visible arriba (`position: sticky`) mientras la sesión está activa y hay algún temporizador corriendo: tarjetas compactas, una por tarea con timer activo, con el nombre y una cuenta atrás grande en `mm:ss`. Cuando una llega a 0, cambia a un estado "¡Listo!" con un aviso visual (y opcionalmente un pitido corto vía Web Audio, sin necesidad de ningún archivo de sonido).
5. **Lista de tareas** debajo de la línea de tiempo — la vista con los controles reales: cada tarea muestra su hora prevista, duración, la nota de "mientras tanto" si la tiene, y a la derecha: un botón "Iniciar" (si tiene temporizador y aún no ha empezado) que arranca su cuenta atrás manualmente — útil si vas más rápido o más lento que el plan —, la cuenta atrás en vivo si ya está en marcha, un botón "Hecho" cuando el temporizador llega a 0 (no se autocompleta sola, hay que confirmarlo), o un simple checkbox "Hecho" si la tarea no tiene temporizador.
6. **Resumen final / celebración** — en cuanto todas las tareas del día están marcadas como completadas, la tarjeta de resultados cambia a un modo de celebración con el checklist de "platos listos" de ese día.
7. **Tarjeta de "no se prepara con antelación"** — al final de cada día, una tarjeta simple de solo lectura con esa lista estática.

## Funcionalidad e interacciones

- **Activación automática vs. manual**: una tarea con temporizador se activa sola en cuanto el cronómetro general de la sesión llega a su minuto programado — pero el usuario también puede pulsar "Iniciar" antes de ese minuto si va adelantado. Cuando se activa (de cualquiera de las dos formas), su cuenta atrás debe calcularse **a partir del minuto en que estaba planeada empezar, no del instante exacto en que se detecta** — si la pestaña estuvo cerrada varios minutos y se reabre tarde, la tarea debe reflejar el tiempo real transcurrido (posiblemente ya "sonando"), no reiniciar un temporizador completo desde cero como si nada hubiera pasado.
- **Persistencia por día**: guarda el estado de cada día (Día A / Día B) por separado en `localStorage` — así cambiar el selector para echar un vistazo al otro día nunca interfiere con una sesión en curso. Guarda: si la sesión ha empezado y cuándo, qué tareas están completadas, y qué temporizadores están en marcha (guardando la hora real de inicio de cada uno, no una cuenta atrás en segundos, para poder recalcular el tiempo restante correctamente tras recargar la página).
- **Empezar una sesión nueva** (del mismo día o del otro) resetea el estado de completado de ese día — pide confirmación si ya había una sesión en marcha o terminada, para no perderla por accidente.
- El bucle que revisa "¿ha llegado el minuto de alguna tarea?" debe seguir corriendo de fondo aunque el usuario esté mirando el otro día en el selector — si no, una sesión en marcha en el Día A dejaría de avanzar en cuanto alguien mira el Día B.

## Estructura de datos

```
localStorage:
  mealprep_session_v1  → {
    diaA: { startedAt: ISOString|null, completedTaskIds: string[], taskTimers: { [id]: { startedAt } } },
    diaB: { startedAt: ISOString|null, completedTaskIds: string[], taskTimers: { [id]: { startedAt } } },
  }
  mealprep_selected_day_v1 → "diaA" | "diaB"
```

El plan en sí (tareas, carriles, resultados, lista de "no se prepara con antelación") vive hardcodeado en el JS, no en `localStorage` — no es editable desde la interfaz, es tu plan fijo.

## Notas de implementación

- No sincronices esta sección con ninguna base de datos externa — es intencionadamente solo local.
- No hardcodees ningún plato, horario ni cantidad real de nadie en el código publicado — usa solo la estructura de ejemplo de arriba.
